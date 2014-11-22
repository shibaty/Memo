# vagrantのboxファイルを作る方法(for CentOS6.6)

CentOS6.6のISOイメージを事前に入手(minimal)

## VirtualBoxのVMを作成

* タイプ:Linux
* バージョン:RedHat(64bit)
* 以降はデフォルト

## CentOSをインストール

ISOをマウントして通常インストール

## CentOSの初期設定

### SELinux無効化
ローカル環境の場合のみ

```console
# setenforce 0
# vi /etc/selinux/config
```
```diff
-SELINUX=enforcing
+SELINUX=disabled
```
```console
# getenforce
Disabled <- 無効化を確認
# shutdown -r now
```

### iptablesの無効化
ローカル環境オンリーの場合のみ

```console
# service iptables stop
# service ip6tables stop
# chkconfig iptables off
# chkconfig ip6tables off
# chkconfig
<- iptables/ip6tablesの無効化を確認
```

### eth0の有効化

```console
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
```diff
-ONBOOT=no
+ONBOOT=yes
```
```console
# service network restart
# ifconfig
<- eth0の有効化を確認
```

### VirtualboxのDNSの設定を修正
ホストOSのDNS解決が動かない不具合があるので設定を修正する。  
*C:\\Users\\[User Name]\\VirtualBox VMs\\[VM Name]\\[VM Name].vbox*の*Adapter slot="0"*配下を編集。  
 **VMを落として編集すること。  
   Virtualboxのマネージャーを起動している場合は、再起動しないと有効にならないので注意。**

```diff
-<DNS pass-domain="true" use-proxy="false" use-host-resolver="false"/>
+<DNS pass-domain="false" use-proxy="false" use-host-resolver="true"/>
```

### 一時的に外部からsshできるようにする
VirtualBox上だと作業しにくいので、一時的に外部からsshできるようにする。  
**VMを落として編集すること。**

* 設定->ネットワーク->アダプター1
  * 高度->ポートフォワーディング
    * ホストポート:2222
    * ゲストポート:22
  
```console
$ ssh root@localhost -p 2222
<- ログインできればOK
```

### vagrantユーザの作成

```console
# groupadd vagrant
# useradd vagrant -g vagrant -G wheel
# echo "vagrant" | passwd --stdin vagrant
# echo "vagrant ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/vagrant
# chmod 0440 /etc/sudoers.d/vagrant
```

### vagrantユーザのsshを設定

```console
# su - vagrant
$ mkdir -pm 700 .ssh
$ cd .ssh
$ curl -k -L -o authorized_keys 'https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub'
$ chmod 600 authorized_keys
$ exit
```

### パッケージ最新化

```console
# yum -y install epel-release
<- EPELレポジトリを追加
# yum -y update
# shutdown -r now
<- カーネルバージョンアップがあったら再起動
```

### 最低限必要なパッケージをインストール

```console
# yum -y install man man-pages-ja gcc-g++ perl kernel-devel dkms
```

### Guest Additionsのインストール

* VMのメニューからデバイス->Guest AdditionsのCDイメージを挿入
  * コンソールから以下を実施

```console
# mount -t iso9660 /dev/cdrom1 /mnt
# /mnt/VBoxLinuxAdditions.run
# shutdown -r now
<- 一応再起動
```

### box作成用のクリーンアップ

```console
# yum clean all
# dd if=/dev/zero of=/EMPTY bs=1M
# rm -rf /EMPTY
```

### 環境構築日を記録して終了

```console
# date > /etc/vagrant_box_build_time
# shutdown -h now
```

### 一時的なssh設定の削除
**VMを落として編集すること。**

* 設定->ネットワーク->アダプター1
  * 高度->ポートフォワーディング の設定を削除。

### boxの作成
ホストOSのコンソールで実施。

```console
$ vagrant package --base [VM Name]
<- package.boxが出来上がる
```
