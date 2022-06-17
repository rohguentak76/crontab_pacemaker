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
리소스 생성 전 
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
symlink 리소스 생성

```
/var/spool/cron 디렉토리 symlink 리소스로 생성, /test/CRON_SCRIPT 경로에 사용자 크론탭 공유
link : symbolic link 경로
target : symbolic link 를 걸 경로
backup_suffix : 심볼릭 링크 생성 시 동일 경로에 동일 디렉토리 혹은 파일이 있을 경우 충돌로 인하여 심볼링크 생성에 실패할 수 있어
	        본래 이름에 backup_suffix로 설정된 string을 덧붙여 mv 해놓는다. 미설정시 심볼링크 생성 거부

# pcs resource create CRON_SYMLINK ocf:heartbeat:symlink link=/var/spool/cron target=/test/CRON_SCRIPT backup_suffix=_backup_by_pcs

[root@client2 cron]# pcs status
Cluster name: crontab-cluster
Stack: corosync
Current DC: client2 (version 1.1.23-1.el7_9.1-9acf116022) - partition with quorum
Last updated: Wed Jun 15 21:29:17 2022
Last change: Wed Jun 15 21:21:46 2022 by root via crm_resource on client2

2 nodes configured
2 resource instances configured

Online: [ client1 client2 ]

Full list of resources:

 AFD    (lsb:afd):      Started client2
 CRON_SYMLINK   (ocf::heartbeat:symlink):       Started client2

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
[root@client2 cron]# ls -al /var/spool/
total 0
drwxr-xr-x.  7 root root  97 Jun 15 21:21 .
drwxr-xr-x. 19 root root 265 Jun 15 08:59 ..
drwxr-xr-x.  2 root root  63 Jan 20  2020 anacron
lrwxrwxrwx   1 root root  17 Jun 15 21:21 cron -> /test/CRON_SCRIPT
drwx------.  2 root root   6 Aug  8  2019 cron_backup_by_pcs
drwxr-xr-x.  2 root root   6 Apr 11  2018 lpd
drwxrwxr-x.  2 root mail  32 Jan 20  2020 mail
drwxr-xr-x. 16 root root 201 Jan 20  2020 postfix

```
