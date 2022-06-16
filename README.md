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
```
