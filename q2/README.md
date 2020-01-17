# 문제 2번

## controller compute vm 설치
os = centos7
### ip 할당 
- controller = 10.0.0.200
- compute = 10.0.0.201

## 방화벽, 네트워크매니저 해제
```
[root@controller ~]# systemctl disable firewalld
[root@controller ~]# systemctl stop firewalld
[root@controller ~]# systemctl disable NetworkManager
[root@controller ~]# systemctl stop NetworkManager
```

## openstack packstack 설치
```
[root@controller ~]# yum install -y openstack-packstack
```

## openstack 설치 확인
```
[root@controller ~]# yum install centos-release-openstack-rocky
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.kakao.com
 * centos-qemu-ev: mirror.kakao.com
 * extras: mirror.kakao.com
 * updates: mirror.kakao.com
Package centos-release-openstack-rocky-1-1.el7.centos.noarch already installed and latest version
Nothing to do
```

## 펙스택 설치 설정파일 answer.txt 생성
[root@controller ~]# packstack --gen-answer-file=answer.txt
Packstack changed given value  to required value /root/.ssh/id_rsa.pub
```
CONFIG_DEFAULT_PASSWORD=abc123
CONFIG_CEILOMETER_INSTALL=n
CONFIG_AODH_INSTALL=n
CONFIG_KEYSTONE_ADMIN_PW=abc123
CONFIG_HEAT_INSTALL=y
CONFIG_MAGNUM_INSTALL=y
CONFIG_TROVE_INSTALL=y
CONFIG_NEUTRON_L2_AGENT=openvswitch
CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:ens33
CONFIG_COMPUTE_HOSTS=10.0.0.200,10.0.0.201
CONFIG_SWIFT_INSTALL=y
CONFIG_SWIFT_KS_PW=947bc8d05e0b4e27
```

## 팩스택 확인
```
[root@controller ~]# cat answer.txt| grep -E "CONFIG_DEFAULT_PASSWORD|CONFIG_CEILOMETER_INSTALL|CONFIG_AODH_INSTALL|CONFIG_KEYSTONE_ADMIN_PW|CONFIG_HEAT_INSTALL|CONFIG_MAGNUM_INSTALL|CONFIG_TROVE_INSTALL|CONFIG_NEUTRON_L2_AGENT|CONFIG_NEUTRON_OVS_BRIDGE_IFACES|CONFIG_COMPUTE_HOSTS"
```
```
[root@controller ~]# packstack --answer-file=/root/answer.txt
...
Applying 10.0.0.200_controller.pp
10.0.0.200_controller.pp:                            [ DONE ]   
Applying 10.0.0.200_network.pp
10.0.0.200_network.pp:                               [ DONE ]   
Applying 10.0.0.201_compute.pp
Applying 10.0.0.200_compute.pp
10.0.0.200_compute.pp:                               [ DONE ]   
...
...
 **** Installation completed successfully ******
...
```

## compute(10.0.0.201) 에 nova 설치 확인
```
[root@controller ~(keystone_admin)]# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  4 | nova-conductor   | controller | internal | enabled | up    | 2020-01-17T07:28:12.000000 |
|  5 | nova-scheduler   | controller | internal | enabled | up    | 2020-01-17T07:28:09.000000 |
|  7 | nova-consoleauth | controller | internal | enabled | up    | 2020-01-17T07:28:12.000000 |
|  8 | nova-compute     | controller | nova     | enabled | up    | 2020-01-17T07:28:10.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
[root@controller ~(keystone_admin)]# openstack hypervisor list
+----+---------------------+-----------------+------------+-------+
| ID | Hypervisor Hostname | Hypervisor Type | Host IP    | State |
+----+---------------------+-----------------+------------+-------+
|  1 | controller          | QEMU            | 10.0.0.201 | up    |
+----+---------------------+-----------------+------------+-------+

```

```
[root@controller ~]# systemctl start libvirtd openstack-nova-compute
[root@controller ~]# systemctl enable libvirtd openstack-nova-compute
```

```
vi /etc/glance/glance-api.conf
systemctl restart openstack-glance-api opensatck-glance-registry
```
