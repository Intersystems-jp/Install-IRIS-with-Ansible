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

ここでは「Red Hat = Ansible サーバ」「Ubuntu = IRIS インストール先」です。  
ターゲット先 Ubuntu アドレス = 192.168.10.2 としています。  
以下の手順はすべて、**コントロールを行う Ansible サーバ = Red Hat** で実施します。本手順では root でログインして作業しています。  
  
(1) Ansible をインストール
```
# dnf install ansible-core
# ansible --version
ansible [core 2.14.14]
```

(2) ターゲット先 Ubuntu に パスワードなしで ssh ログインできるようにしておく   
```
以下コマンドで公開鍵作成。全部リターンで /root/.ssh/id_xxx.pnb が作成される
# ssh-keygen

作成した公開鍵を、以下コマンドでターゲット先 Ubuntu にコピー
# ssh-copy-id root@192.168.10.2

パスワードなしでログインできることを確認
# ssh root@192.168.10.2
```

(3) 本リポジトリで公開している ansible 以下を、/etc/ansible としてコピー
```
# ls /etc/ansible
ansible.cfg  hosts  install_iris.yml  roles
```
  
(4) Ansible 経由で疎通チェック
```
接続先情報は /etc/ansible/hosts に記述
# ansible bunvmub24 -m ping
192.168.10.2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
  
(5) IRISインストールキットとライセンスキーを用意し、適切な場所に置く
```
# ls /etc/ansible/roles/install_iris/files/
IRIS-2024.1.1.347.0-lnxubuntu2404x64.tar.gz  iris.key  iris.script  merge.cpf  naka_IRIS.DAT
```

(6) Ansible 実行ログに時刻を表示する目的で、[community.general](https://docs.ansible.com/ansible/latest/collections/community/general/index.html) をインストール
```
# ansible-galaxy collection install community.general
```
  
(7) 実行
```
# ansible-playbook /etc/ansible/install_iris.yml

PLAY [Install IRIS demo] ************************************************************************************** 14:06:48

TASK [Gathering Facts] **************************************************************************************** 14:06:48
ok: [192.168.10.2]

TASK [install_iris : Install Apache on Ubuntu] **************************************************************** 14:06:50
changed: [192.168.10.2]

TASK [install_iris : Start and enable Apache on Ubuntu] ******************************************************* 14:07:04
ok: [192.168.10.2]

TASK [install_iris : Create the kit directory] **************************************************************** 14:07:05
changed: [192.168.10.2]

TASK [install_iris : Transfer the IRIS install file] ********************************************************** 14:07:05
changed: [192.168.10.2]

TASK [install_iris : Transfer the license key] **************************************************************** 14:07:20
changed: [192.168.10.2]

TASK [install_iris : Transfer the IRIS configuration merge file] ********************************************** 14:07:21
changed: [192.168.10.2]

TASK [install_iris : Transfer IRIS.DAT for NAKA] ************************************************************** 14:07:21
changed: [192.168.10.2]

TASK [install_iris : Uncompress the IRIS install file] ******************************************************** 14:07:22
changed: [192.168.10.2]

TASK [install_iris : Create the IRIS install directory] ******************************************************* 14:07:37
changed: [192.168.10.2]

TASK [install_iris : Create the database directory for NAKA] ************************************************** 14:07:37
changed: [192.168.10.2]

TASK [install_iris : Put IRIS.DAT for NAKA] ******************************************************************* 14:07:37
changed: [192.168.10.2]

TASK [install_iris : Run the IRIS silent installer] *********************************************************** 14:07:38
changed: [192.168.10.2]

TASK [install_iris : Display installation output] ************************************************************* 14:08:23
ok: [192.168.10.2] => {
    "install_output.stdout_lines": [
        "Installation completed successfully"
    ]
}

TASK [install_iris : Put the IRIS license key file] *********************************************************** 14:08:23
changed: [192.168.10.2]

TASK [install_iris : Restart IRIS to apply the license] ******************************************************* 14:08:23
changed: [192.168.10.2]

TASK [install_iris : Display the portal URL] ****************************************************************** 14:08:33
ok: [192.168.10.2] => {
    "msg": "Portal URL : http://192.168.10.2/csp/sys/UtilHome.csp"
}

TASK [install_iris : Transfer iris.script] ******************************************************************** 14:08:33
changed: [192.168.10.2]

TASK [install_iris : Run iris.script to check global and class method] **************************************** 14:08:34
changed: [192.168.10.2]

TASK [install_iris : Get result.txt] ************************************************************************** 14:08:34
ok: [192.168.10.2]

TASK [install_iris : Display the result] ********************************************************************** 14:08:34
ok: [192.168.10.2] => {
    "msg": "^naka=123 at 10/23/2024 14:08:34"
}

PLAY RECAP **************************************************************************************************** 14:08:34
192.168.10.2              : ok=21   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

(8) 実行ログどおり http://192.168.10.2/csp/sys/UtilHome.csp から管理ポータルにアクセス可能

## 参考資料

(1) Ansible 基本 (Red Hat, 日本語)  
https://www.redhat.com/ja/topics/automation/learning-ansible-tutorial  

(2) Ansible Script ドキュメント (Ansible Community, 英語)  
https://docs.ansible.com/ansible/latest/

(3) Ansible Script ドキュメント (Ansible Community, 2.9 日本語)  
https://docs.ansible.com/ansible/2.9_ja/
 
(4) IRIS: Windows での自動インストール ドキュメント (InterSystems, 日本語)  
https://docs.intersystems.com/iris20241/csp/docbookj/DocBook.UI.Page.cls?KEY=GIWIN_unattended
 
(5) IRIS: Linux での自動インストール ドキュメント (InterSystems, 日本語)  
https://docs.intersystems.com/iris20241/csp/docbookj/DocBook.UI.Page.cls?KEY=GIUNIX_unattended

(6) "Leveraging Automaiton Tools for Developing InterSystems IRIS Applications" (InterSystems Global Summit 2024 Video, 英語)  
https://www.intersystems.com/leveraging-automation-tools-for-deploying-intersystems-iris-applications-intersystems/
