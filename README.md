# crontab_pacemaker

```
환경 : centos 7.9
목적 : crontab 데몬 이중화
테스트 장비 : client1(10.73.20.31), client2(10.73.20.32)
```


1. 패키지설치
```
대상 장비 모두 패키지 설치
# yum install -y pcs pacemaker corosync
```

2. cluster 설정
```
client1 # passwd hacluster
Changing password for user hacluster.
New password: secret
Retype new password: secret
passwd: all authentication tokens updated successfully.

client2 # passwd hacluster
Changing password for user hacluster.
New password: secret
Retype new password: secret
passwd: all authentication tokens updated successfully.

client1 # systemctl enable --now pcsd
client2 # systemctl enable --now pcsd

client1 # pcs auth cluster client1 client2
Username: hacluster
Password: secret
client1: Authorized
client2: Authorized

[root@client1 vagrant]# pcs cluster setup --start --name crontab-cluster client1 client2 --enable --token 17000
Destroying cluster on nodes: client1, client2...
client2: Stopping Cluster (pacemaker)...
client1: Stopping Cluster (pacemaker)...
client2: Successfully destroyed cluster
client1: Successfully destroyed cluster

Sending 'pacemaker_remote authkey' to 'client1', 'client2'
client2: successful distribution of the file 'pacemaker_remote authkey'
client1: successful distribution of the file 'pacemaker_remote authkey'
Sending cluster config files to the nodes...
client1: Succeeded
client2: Succeeded

Starting cluster on nodes: client1, client2...
client1: Starting Cluster (corosync)...
client2: Starting Cluster (corosync)...
client2: Starting Cluster (pacemaker)...
client1: Starting Cluster (pacemaker)...
client1: Cluster Enabled
client2: Cluster Enabled

Synchronizing pcsd certificates on nodes client1, client2...
client1: Success
client2: Success
Restarting pcsd on the nodes in order to reload the certificates...
client1: Success
client2: Success
```
