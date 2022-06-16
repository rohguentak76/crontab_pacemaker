# crontab_pacemaker

```
환경 : centos 7.9
목적 : crontab 이중화
테스트 장비 : client1(10.73.20.31), client2(10.73.20.32)
테스트 목적 : 
	- 크론탭 스크립트를 공유 파일시스템 내부에 위치 시켜 테스트장비 모두에서 크론탭 스크립트 수정가능 여부 확인
	- 심볼릭 링크 리소스 및 크론탭 모니터링 리소스를 활용하여 실제 사용자들의 크론탭 이중화 여부확인		 
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

```
[root@client1 vagrant]# pcs status
Cluster name: crontab-cluster
Stack: corosync
Current DC: client2 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Thu Jun 16 06:42:58 2022
Last change: Thu Jun 16 06:42:54 2022 by root via cibadmin on client1

2 nodes configured
0 resource instances configured

Online: [ client1 client2 ]

No resources


Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled

```
