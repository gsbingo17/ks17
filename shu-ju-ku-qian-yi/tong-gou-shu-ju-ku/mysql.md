# MySQL

### **Failure connecting to the source database**

若是在 Test migration Job 過程, 發現Error: **Failure connecting to the source database** \
![](<../../.gitbook/assets/image (1).png>)

這樣代表無法從 CloudSQL端 連線到 來源資料庫主機.\
**PrivateIP環境:** 進入跟 CloudSQL 做Peering的VPC後, 點選 Private Service Connection --> Allocated Ranges For Services\
找到 CloudSQL 的 IP Range之後 將其加入 來源資料庫主機 網路環境中的 Firewall allow ingress 即可

![](<../../.gitbook/assets/image (36).png>)\
\
**PubliceIP環境:** 則須在CloudSQL instance的地方, 找到真正用來連線到 來源資料庫主機 的 PublicIP如下, 將其加入 來源資料庫主機 網路環境中的 Firewall allow ingress 即可\
![](<../../.gitbook/assets/image (11).png>)\
