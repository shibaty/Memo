# Ansibleの使い方

AnsibleはPythonベースだが、WindowsのPyhtonではうまく動かないらしい。  
よって、vagrantでターゲットホストに加えて管理用ホストを同時に起動し、管理用ホストからansibleを実施する。
それぞれのホストは以降以下で記載する。
* 管理用ホスト:manage
* ターゲットホスト:node1

## vagrantの環境設定

vagrant init後に*Vagrantfile*に以下の設定を追加する(最後のendの手前付近)。

```rb

```
