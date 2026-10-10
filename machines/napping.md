`sudo netdiscover -i eth0` (Detect which devices are online on the same subnet)
![image](https://hackmd.io/_uploads/H1yXPlm9ke.png)
`nmap` scan
![image](https://hackmd.io/_uploads/r1LWFlQ9kg.png)
`enum4linux -a 192.168.121.131` (List shared resources)
![image](https://hackmd.io/_uploads/BJ4J0g79ye.png)
![image](https://hackmd.io/_uploads/HkE2Aem51e.png)
Access `tmp` via `rc`
![image](https://hackmd.io/_uploads/S1lViZQ51l.png)
TCP ports 512, 513, and 514 are the “r” services and were misconfigured to allow remote access from any host, so switch to root.
![image](https://hackmd.io/_uploads/B1nysUYc1l.png)
`http://192.168.121.131` (port 80 is open)
![image](https://hackmd.io/_uploads/H1eD1wK9Je.png)
