`sudo netdiscover -i eth0` (偵測在同一個網段內有哪些裝置在線)
![image](https://hackmd.io/_uploads/H1yXPlm9ke.png)
`nmap`掃描
![image](https://hackmd.io/_uploads/r1LWFlQ9kg.png)
`enum4linux -a 192.168.121.131`(查看共享清單)
![image](https://hackmd.io/_uploads/BJ4J0g79ye.png)
![image](https://hackmd.io/_uploads/HkE2Aem51e.png)
RC進去tmp
![image](https://hackmd.io/_uploads/S1lViZQ51l.png)
TCP 連接埠 512、513 和 514 稱為「r」服務，並且被錯誤地配置為允許從任何主機進行遠端存取，故切換為root
![image](https://hackmd.io/_uploads/B1nysUYc1l.png)
`http://192.168.121.131`(有80 port)
![image](https://hackmd.io/_uploads/H1eD1wK9Je.png)
