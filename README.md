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
<img src="https://user-images.githubusercontent.com/55233942/149553359-c802bf41-3755-4792-9618-e6e944796709.png"/>



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
## <a id="install"></a>安裝教學
依照角色可以分為"被攻擊者"和"Docker"：

### 被攻擊者

#### 建置Docker：
- `sudo apt update`
- `sudo apt upgrade`
- `sudo apt install docker.io`：安裝docker
- `sudo docker pull ubuntu`：
- `sudo docker run -itd --name fakessh -p 5000:22 ubuntu`：
- `sudo docker exec -it fakessh /bin/bash`：
#### 建置fail2ban環境：
- `sudo apt install fail2ban`：安裝fail2ban服務
- `sudo vim /etc/fail2ban/jail.conf`：
  - `maxretry=2`
  - 在 sshd 下方加入：
    - `enabled = true`
    - `filter = sshd`
    - `action = iptables[name=SSH, port=ssh, protocol=tcp]`
    - `logpath = /var/log/imwatchingu-VirtualBox/sshd.log`
   - 存檔
- `sudo vim /etc/fail2ban/action.d/iptables.conf`
![](https://i.imgur.com/DsGS6z4.png)
- `sudo vim /etc/fail2ban/action.d/iptables-common.conf`
    - `actionflush = <iptables> -t nat -F f2b-<name>`
- `ssh imwatchingu@127.0.0.1` 先連線一次讓他產生 ssh 的 log
- 重啟 fail2ban
    - `service fail2ban restart`
    - `fail2ban-client status`
- 監看 ssh 有沒有抓到：`tail -f /var/log/imwatchingu-VirtualBox/sshd.log`
- 監看 fail2ban 有沒有反映：`tail -f /var/log/fail2ban.log`
## <a id="use"></a>使用教學
## <a id="work"></a>工作分配
| 組員 | 工作內容 |
| ------ | --------------- |
| 蕭仲廷 | 發想、整合、找資料、編寫、Debug |
| 朱珮瑜 | 找資料、編寫、Debug、Demo |
| 張宥錚 | 找資料、Debug、共筆整理、PPT、口頭報告、GitHub |
| 楊叡 | 找資料、編寫、Debug、共筆整理、PPT、GitHub |
## <a id="reference"></a>參考文獻
