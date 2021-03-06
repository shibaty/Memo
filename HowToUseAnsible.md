# Ansibleの使い方

AnsibleはPythonベースだが、WindowsのPyhtonではうまく動かないらしい。  
よって、vagrantでターゲットホストに加えて管理用ホストを同時に起動し、管理用ホストからansibleを実施する。
それぞれのホストは以降以下で記載する。
* 管理用ホスト:manage
* ターゲットホスト:node1

## vagrantの環境設定

### ノードの設定
vagrant init後に*Vagrantfile*に以下の設定を追加する(最後のendの手前付近)。

```rb
  # Manage Node for ansible
  config.vm.define :manage do |node|
    node.vm.box = "[box name]"
    node.vm.host_name = "manage"
    node.vm.network :forwarded_port, guest: 22, host: 2223, id: "ssh"
    node.vm.network :private_network, ip: "192.168.33.11"
  end

  # target node 1
  config.vm.define :node1 do |node|
    node.vm.box = "[box name]"
    node.vm.host_name = "node1"
    node.vm.network :forwarded_port, guest: 22, host: 2224, id: "ssh"
    node.vm.network :private_network, ip: "192.168.33.12"
  end
```

### sshの設定

vagrant up後にsshの設定を行う。

```console
$ vagrant ssh-config manage >> ~/.ssh/config
$ vagrant ssh-config node1 >> ~/.ssh/config
$ scp -F ~/.ssh/config /c/Users/[User Name]/.vagrant.d/insecure_private_key manage:.ssh/id_rsa
<- manageからnode1にsshできるよう、vagrantの秘密鍵をコピー
$ ssh manage
```

scpしたid_rsaのパーミッションを変更。

```console
$ chmod 600 ~/.ssh/id_rsa
```

### ターゲットホストのホスト名を管理用ホストに登録

管理用ホストの*/etc/hosts*を書き換える

```console
$ sudo vi /etc/hosts
192.168.33.12 node1
```

### ansibleのインストール

管理用ホストにインストールする。

```console
$ sudo yum -y install ansible
```

### ansible環境作成

**gmpをアップデートしろというWarningは無視してよい。  
  初回接続時はknown_hostsの追加が行われる。**

```console
$ echo '192.168.33.12' > hosts
<- インベントリファイルの作成
$ ansible -i hosts 192.168.33.12 -m ping
192.168.33.12 | success >> {
    "changed": false,
    "ping": "pong"
}
<- 動作確認OK
```


