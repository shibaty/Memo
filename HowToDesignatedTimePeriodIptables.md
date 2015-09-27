期間指定でiptablesを有効にする設定方法
===
iptablesのコマンドライン
---
```sh
sudo iptables -I INPUT -p tcp --dport 80 -m time --timestart 00:00 --timestop 08:00 -j REJECT --reject-with icmp-host-prohibited
```
httpのアクセスを00:00-08:00の間遮断する。  
-Iで指定するのはルールの先頭に追加するため（-Aは末尾に追加）。他のルール踏まえ、必要であれば挿入箇所を指定する。
本指定の場合はestablishedの通信についてもREJECTされる。  
時間帯で通信中のもの含め完全遮断したいときに。

動作の確認方法
---
Host:Win10、Guest:CentOS6.7で実施。  
Guest側でtcpdumpにてパケットを確認する。

```sh
sudo tcpdump -i eth1 -s 0 -w test.pcap port 80 or icmp
```
iptablesにて遮断後、Guest側からICMP(Destination Unreachable)となることが確認できる。  
通信中のホスト・クライアントは、HTTPのタイムアウトで切断検知されることに期待する。
