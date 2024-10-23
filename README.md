# Install-IRIS-with-Ansible
Ansible を使って別サーバに IRIS を自動インストール＋初期設定を実施するサンプル・スクリプトです。

## サンプル実行環境について
| サーバ | OS |
---- | ----
| Ansible サーバ (スクリプトを実行する側) | Red Hat 9 |
| ターゲットサーバ (インストール先) | Ubuntu 24 |

- 上記環境でスクリプトの動作を確認していますが、基本的に Linux / Mac であれば他 OS でも動作します。
- 一般に、Ansible サーバは Linux / Mac に限られますが、ターゲットサーバは Windows も可能です。

## サンプル概要

Ansible スクリプトで、以下を順番に実施します。

- ターゲットサーバに、Apache をインストールします。
- ターゲットサーバに、IRIS インストールに必要な関連ファイルを転送します。
- ターゲットサーバ先で、IRIS インストールスクリプトを実行し、IRIS をインストール。インストールと同時に以下の設定を行います。
   - グローバルキャッシュ・ルーチンキャッシュを設定
   - ネームスペース NAKA を定義
- ターゲットサーバ先で、IRIS にログインし、ネームスペース NAKA のグローバル ^naka を確認してファイルに出力します。
- Ansible サーバから、ターゲット先の (4) のファイルの中身を取得し、画面に表示します。

## サンプル実行手順

以下、Red Hat = Ansible サーバ、Ubuntu = IRIS インストール先、とします。  
ここでは Ubuntu アドレス = 192.168.10.2 としています。また Red Hat に root でログインして作業しています。  
すべて **コントロールを行う Ansible サーバ、Red Hat ** で実施します。

1. Ansible をインストール
```
# dnf install ansible-core
# ansible --version
```

2. ターゲット先 Ubuntu に パスワードなしで ssh ログインできるようにしておく   
```
以下コマンドで公開鍵作成。全部リターンで /root/.ssh/id_xxx.pnb が作成される
# ssh-keygen

上記を以下コマンドで、ターゲット先 Ubuntu にコピーする
# ssh-copy-id root@192.168.10.2

パスワードなしでログインできることを確認
# ssh root@192.168.10.2
```
3. 本リポジトリで公開している ansible 以下を、/etc/ansible としてコピー
```
# ls /etc/ansible
ansible.cfg  hosts  install_iris.yml  roles
```

4. Ansible 経由で疎通チェック
接続先情報は /etc/ansible/hosts に記述
```
# ansible bunvmub24 -m ping
```


3. Ansible 経由で疎通チェック
```
# ansible
```
5. 
6.  
