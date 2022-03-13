# ansible
AWSのEC2インスタンスにRailsアプリを実行する環境構築を行うサンプルコード
# 前提
ローカル環境からEC2インスタンスに対してSSH接続し、Ansibleを実行する。
## 実行環境
macOS Monterey Version 12.2.1
# 環境構築
```bash
$ brew install ansible
# インストールされたか確認
$ ansible --version
```
# EC2インスタンスへの接続設定
ローカル環境下に`~/.ssh/config`ファイルを作成し、EC2インスタンスへの接続情報を入力する。
```
Host {your_host_name}
      Hostname {ec2_instance_ipv4_dns}
      IdentityFile ~/.ssh/{your_instance_private_key}.pem
      User ec2-user
```
### ansibleの設定ファイル
インベントリ（接続対象）を`./hosts`ファイルに記載する。
```
[{host_group}]
{your_host_name} # ~/.ssh/configファイルで設定したホスト名と同じ
```
### 接続確認
```bash
$ ansible -m ping {host_group}
```
# ディレクトリ構造
```
.
├── ansible.cfg
├── group_vars
│   └── all.yml
├── hosts
├── roles
│   ├── mysql
│   │   └── tasks
│   │       └── main.yml
│   ├── nginx
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       ├── nginx.conf.j2
│   │       └── rails.conf.j2
│   ├── nodejs
│   │   └── tasks
│   │       └── main.yml
│   └── rails
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   └── rbenv.sh.j2
│       └── vars
│           └── main.yml
└── site.yml
```
# ファイル構成
## 主な実行内容
- Railsの環境構築（rbenv, ruby-build）
- nodejs / yarnのインストール
- mysqlのインストール
- nginxのインストール、設定ファイルの作成、再起動
## roles
Ansibleで実行するタスクの内容をrole単位に分割して配置。<br>
各role内のtasks/main.ymlファイルで環境の構築を行う。<br>
configファイルやbashスクリプトなどを生成する必要がある場合は、templates内のjinja2テンプレートを参照する。<br>
role内で変数を参照する必要がある場合は、各role内のvarsディレクトリを参照する。
## ansible.cfg
インベントリファイルの指定およびSSH接続時の設定を記述。
## hosts
実行対象となるサーバーを記述。
## site.yml
マスターとなるPlaybook。各roleの実行順序などを規定する。
# コマンドの実行
```bash
# 構文確認
$ ansible-playbook site.yml --syntax-check
# Playbookの実行
$ ansible-playbook site.yml
```
