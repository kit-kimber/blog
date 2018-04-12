---
title: CentOS6.8でBondingの設定を行う
date: 2018-03-31 22:35:13
tags:
- Linux
- CentOS6.8
---
自分がはてなブログに書いていた記事の移植です。

CentOS6.8でBonding(チーミング）する手順です。

# NetworkManagerの無効化
まずNetworkManagerを無効化します。  

```
sudo service NetworkManager stop
sudo chkconfig NetworkManager off
```

# Bondingの準備
bondingモジュールのエイリアスをbond0に張ります。

```
sudo echo "alias bonding bond0" >> /etc/modprobe.d/bonding.conf
```

Bondingモジュールを有効化します。

```
sudo modprobe bonding
```

モジュール有効化を確認します。  

```
lsmod | grep bonding
```

# BondingとNICの設定  
今回はBondingを行うだけなので固定IPではなく、DHCPでIPアドレスを取得します。

## bond0の設定(/etc/sysconfig/network-script/ifcfg-bond0)
今回はmodeを１にしているので負荷分散はされません。  
普段はプライマリに設定したNICが使用され、プライマリ側がダメになったら他のNICでの通信に切り替わります。
miimonで設定している値はNICの状態を監視する頻度です。  
単位はミリ秒です。

```
DEVICE=bond0
BOOTPROTO=dhcp
ONBOOT=yes
BONDING_OPTS="mode=1 primary=eth0 miimon=500"
```

## eth0の設定(/etc/sysconfig/network-script/ifcfg-eth0,ifcfg-eth1)
普段の設定とは別にMASTERとSLAVEの設定を行う必要があります。  
MASTERはBondingに使用するデバイスを指定し、
SLAVEでは自らをSLAVEであることを示します。

```
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

```
DEVICE=eth1
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```
  
## networkの再起動
ここで失敗した場合は設定を見直してください。  
それかNetworkManagerの無効化を忘れている場合があります。

```
sudo service network restart
```

# 動作の確認
ifconfigを叩くとbond0にIPアドレスが割り当てられていると思います。  
また 

```
cat /proc/net/bonding/bond0
```

でbondingの状態が確認できます。
