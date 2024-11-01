# I-am-Watching-You
- [發展理念](#Concept_Develop)
- [硬體、軟體設備](#device)
- [LSA課堂知識運用](#LSA_class)
- [安裝教學](#install)
- [使用教學](#use)
- [工作分配](#work)
- [參考文獻](#reference)

## <a id="Concept_Develop"></a>發展理念
### <a>背景故事</a>
廷廷常常出去約會回來後，
發現他的電腦有被使用過的痕跡。
他很生氣想將此人繩之以法！
為了得知這個壞蛋是誰，
以及蒐集他的犯罪資料，
他決定在自己的電腦動手腳！
讓我們一起來找出 . . .
兇手究竟是誰 . . .

### <a>功能介紹</a>
模擬當有駭客想要透過SSH連線來連入被攻擊者的裝置進行操作時的情況，在嘗試多次輸入密碼未成功，以致於讓"被攻擊者"偵測到入侵後，"被攻擊者"會將駭客導向事先架設在自己本機的Docker中，並記錄下駭客在Docker中所執行的操作，再透過log檔的形式將蒐集到的資料傳送回"被攻擊者"主機，以達到實時監控的效果
- 示意圖
![示意圖](https://user-images.githubusercontent.com/55233942/150155524-facaea86-9db3-4bca-a90e-874265f579ea.jpg)

#### 說明
- Hacker欲透過ssh連線至IMWathcingU的22port，而嘗試多次輸入密碼未成功後，便無法進到IMWatchingU的本機中，而是會被導向事先建立好的環境"Docker"中，並且Docker的使用的22port就是IMWatchingU上的5000port。而在Hacker在Docker中所執行的指令或操作，都會被記錄下來，並透過rsyslog服務傳回IMWatchingU的514port


## <a id="device"></a>硬體、軟體設備、使用服務
| 名稱 | 圖片 | 用途 |
| ---- | ---- |---|
|虛擬機（Ubuntu 系統）x2|<img src="https://i.imgur.com/cU5FEmA.png" width="150px"/>|作為"入侵的主機"以及"被入侵的主機"|
|Docker|<img src="https://i.imgur.com/rPBTqdW.png" width="150px"/>|偵測入侵後被導向的環境|
|SSH|<img src="https://i.imgur.com/1b8GtuI.png" width="150px"/>|建立遠端連線的服務|
|Fail2ban|<img src="https://i.imgur.com/PELqrvA.png" width="150px"/>|偵測到密碼錯誤多次，對入侵 ip 進行操作|
|rsyslog|<img src="https://i.imgur.com/14jszoJ.png" width="150px"/>|將記錄檔導出至被入侵之主機，即時更新目標行為|
|iptables|<img src=https://i.imgur.com/8sp0L5A.png width="150px"/>|允許 TCP/UDP到本機的 514 port|

## <a id="LSA_class"></a>LSA課堂知識運用
### SSH連線
- [Week 06(2021/10/28) 基本指令/SSH](/CVfc0ILPTRycXBR_pG2NDA)
- [Week 07(2021/11/04) SSH](/wl1BJaOIRQqavZ5rn0jMNQ)
### iptables設定
- [Week 11(2021/12/02) TCP/IP/iptables](/RXmaXIaiRjaPsn-fj9yECA)
- [Week 12(2021/12/09) iptables](/KDeYZ5PcSDKmkrC2wiG5Bw)

## <a id="install"></a>安裝教學
依照角色可以分為"被攻擊者"和"Docker"：

### 被攻擊者
#### 更新：
- `sudo apt update`
- `sudo apt upgrade`
- `sudo sysctl -w net.ipv4.ip_forward=1`
#### 建置Docker：
- `sudo apt install docker.io`：安裝docker
- `sudo docker pull ubuntu`：
- `sudo docker pull jrei/systemd-ubuntu:22.04`:拉取特製image
- `sudo docker run -itd  --privileged=true --name fakessh -p 5000:22 -v /sys/fs/cgroup:/sys/fs/cgroup:ro jrei/systemd-ubuntu:22.04`:掛接讓服務能執行
- `sudo docker run -itd --name fakessh -p 5000:22 ubuntu`：
- `sudo docker exec -it fakessh /bin/bash`：進入Docker

#### 建置rsyslog
- `sudo apt install rsyslog`
- `service rsyslog start`
- `service rsyslog status`
- `sudo vim /etc/rsyslog.conf`
    - 把TCP、UDP下面兩行取消註解<br>
    ![image](https://user-images.githubusercontent.com/55233942/149632669-39c2dc6c-beb4-4ea4-a6f6-bdbb7055d3f0.png)
    - 打在global上面：`$template RemoteLogs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log" *.* ?RemoteLogs & ~`<br>，用來產生存放log檔的資料夾<br>
    ![image](https://user-images.githubusercontent.com/55233942/149632781-1109ffdc-49fd-48cd-ab74-7e6647e7ea05.png)
- `service rsyslog restart`
- 新增防火牆規則
  - `sudo ss -tulnp | grep "rsyslog"`
  - `iptables -A INPUT -p tcp --dport 514 -j ACCEPT`
  - `iptables -A INPUT -p udp --dport 514 -j ACCEPT`

#### 建置fail2ban環境：
- `sudo apt install fail2ban`：安裝fail2ban服務
- `sudo vim /etc/fail2ban/jail.conf`
  - `maxretry=2`
  - 在 sshd 下方加入：
    - `enabled = true`
    - `filter = sshd`
    - `action = iptables[name=SSH, port=ssh, protocol=tcp]`
    - `logpath = /var/log/imwatchingu-VirtualBox/sshd.log`
   - 存檔
- `sudo vim /etc/fail2ban/action.d/iptables.conf`
![image](https://user-images.githubusercontent.com/55233942/150308153-e93a538a-2088-4ca2-83c1-7793a8847f57.png)
- `sudo vim /etc/fail2ban/action.d/iptables-common.conf`
    - `actionflush = <iptables> -t nat -F f2b-<name>`
- `ssh imwatchingu@127.0.0.1` 先連線一次讓他產生 ssh 的 log
- 重啟 fail2ban
    - `service fail2ban restart`
    - `fail2ban-client status`
- 監看 ssh 有沒有抓到：`tail -f /var/log/imwatchingu-VirtualBox/sshd.log`
- 監看 fail2ban 有沒有反映：`tail -f /var/log/fail2ban.log`


### Docker
- `passwd`：新增密碼
- `adduser imwatchingu`：新增使用者
- `apt update`
- `apt upgrade`
- `apt install vim`
- `apt install rsyslog`：安裝rsyslog服務
- `apt install openssh-server`
- `apt install sudo`
- `vim /etc/sudoers`
    - `imwatchingu ALL=(ALL:ALL) ALL`：給予權限
- `su imwatchingu`
- `sudo vim /etc/rsyslog.conf`
    - 最後一行加：`*.* @@172.17.0.1:514`：允許Docker用tcp將蒐集到的資料傳回被攻擊者的514port
        ![image](https://user-images.githubusercontent.com/55233942/149630824-b10cc7fe-0d60-420a-9834-958e82b33022.png)
- `service rsyslog start`
- `service rsyslog status`
- `sudo vim /etc/ssh/sshd_config`
    - `PermitRootLogin yes`：允許Root使用ssh登入
    ![image](https://user-images.githubusercontent.com/55233942/149631027-f1d9f25b-9ede-4d65-95f3-c95b1de6af5f.png)
- 將本機`/etc/ssh/`路徑下的`ssh_host_ecdsa_key.pub`及`ssh_host_ecdsa_key`複製到Docker的同路徑下，並取代原有檔案
- `sudo service ssh reload`：重啟ssh服務


## <a id="use"></a>使用教學
- [實際操作影片](https://www.youtube.com/watch?v=1YplvXJDJvc&ab_channel=ChungTingXiao)

## <a id="work"></a>工作分配
| 組員 | 工作內容 |
| ------ | --------------- |
| 蕭仲廷 | 發想、整合、找資料、編寫、Debug |
| 朱珮瑜 | 找資料、編寫、Debug、Demo |
| 張宥錚 | 找資料、Debug、共筆整理、PPT、口頭報告、GitHub |
| 楊叡 | 找資料、編寫、Debug、共筆整理、PPT、GitHub |

## <a id="reference"></a>參考文獻

### fail2ban

#### 環境建置
- [fail2ban 教學](https://xenby.com/b/107-%E6%95%99%E5%AD%B8-%E4%BD%BF%E7%94%A8fail2ban%E9%98%B2%E6%AD%A2%E6%9A%B4%E5%8A%9B%E7%99%BB%E5%85%A5%E6%94%BB%E6%93%8A)
- [SSH 5 分鐘內登入失敗 3 次](https://iter01.com/563209.html)
- [fail2ban-基本設定及安裝筆記](https://crowsnest1217.com/%E5%AD%B8%E7%BF%92%E8%88%87%E5%88%86%E4%BA%AB/fail2ban-setting.html)

#### 操作
- [action](https://blog.xuite.net/bearsir/blog/61701441)
- [手動解除 fail2ban 封鎖的 IP](https://www.ichiayi.com/tech/fail2ban_unban)

### rsyslog
- [How to Configure Remote Logging with Rsyslog on Ubuntu 18.04](https://kifarunix.com/how-to-configure-remote-logging-with-rsyslog-on-ubuntu-18-04/)
- [How to Send Linux Logs to a Remote Server](https://linuxhint.com/send_linux_logs_remote_server/)
- [詳細解說的很棒的網址](https://www.tecmint.com/install-rsyslog-centralized-logging-in-centos-ubuntu/)
- [Docker Syslog 日誌記錄和故障排除](https://www.loggly.com/use-cases/docker-syslog-logging-and-troubleshooting/)
- [使用 Syslog 集中您的 Docker 日誌記錄](https://betterprogramming.pub/docker-centralized-logging-with-syslog-97b9c147bd30)

### Docker

#### 環境建置
- [Ubuntu Linux 安裝 Docker 步驟與使用教學](https://blog.gtwang.org/virtualization/ubuntu-linux-install-docker-tutorial/)
- [Day 5 關於 Container 的那些大小事](https://ithelp.ithome.com.tw/articles/10193534)
- [Docker Container 基礎入門篇 1](https://azole.medium.com/docker-container-%E5%9F%BA%E7%A4%8E%E5%85%A5%E9%96%80%E7%AF%87-1-3cb8876f2b14)

#### 操作
- [進入docker跟映射ssh docker內設定](https://blog.csdn.net/qq_43914736/article/details/90608587)
- [想用 SSH 連線到 Docker container](https://bingdoal.github.io/deploy/2021/02/use-ssh-connect-on-docker-container/)
- [How to Fix SSH Failed Permission Denied](https://phoenixnap.com/kb/ssh-permission-denied-publickey)
- [docker指令大全](https://joshhu.gitbooks.io/dockercommands/content/Containers/DockerRun.html)
- [在Ubuntu容器中創建sudo用戶](https://blog.csdn.net/bryanwang_3099/article/details/109787215)

### 共筆討論
https://hackmd.io/@tjvxaJBrR96kvrKo_YYl-g/B1WQtpnct
