# vagrantのboxファイルを作る方法(for CentOS6.6)

CentOS6.6のISOイメージを事前に入手(minimal)

## VirtualBoxのVMを作成

+ タイプ:Linux
+ バージョン:RedHat(64bit)

## CentOSをインストール

ISOをマウントして通常インストール

## CentOSの初期設定

### SELinux無効化

    # setenforce 0
    
    # vi /etc/selinux/config
    -SELINUX=enforcing
    +SELINUX=disabled
    
    shutdown -r now
