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
模擬當有駭客想要透過SSH連線來連入被攻擊者的裝置進行操作時的情況，在嘗試多次輸入密碼未成功，而在"被攻擊者"偵測到入侵後，"被攻擊者"會將駭客導向事先架設在自己本機的Docker中，並記錄下駭客在Docker中所執行的操作，再透過log檔的形式將蒐集到的資料傳送回"被攻擊者"主機，以達到實時監控的效果



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
## <a id="use"></a>使用教學
## <a id="work"></a>工作分配
## <a id="reference"></a>參考文獻
