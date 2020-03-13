# ansible-usage

* Ansibleのお試し

## やったこと

* 複数サーバの構成管理
    * ファイル転送
    * DBのクラスタリング(MariaDB Galera Cluster)
* Windowsの構成管理

## 実行

* （手軽&どこでも実行できるように）DockerコンテナでAnsibleの実行ホストと、構築対象サーバを用意しました
* docker及びdocker-composeを実行できる環境を用意してください
    * windows10 WSL2(Ubuntu16.04)上で構築しました
* 今回はホストがUbuntu16.04、対象がCentOS7

### Dockerインストール

* 基本以下の公式を見ればできます
    * http://docs.docker.jp/engine/installation/linux/ubuntulinux.html
    * https://docs.docker.com/compose/install/
* WSL自動起動（必要であれば）
    ```
    echo 'sudo service docker start' >> ~/.bashrc
    ```

### 手順

1. [ansible-usage](git@github.com:shou6216/ansible-usage.git) をclone
2. docker-composeファイルの場所に移動
    ```
    $ cd /path/to/ansible-usage
    ```
3. コンテナ構築
    ```
    $ docker-compose up -d
    ```
4. ansible実行ホストコンテナに入る
    ```
    $  docker exec -it ansible（コンテナ名） bash
    ```
5. Ansible実行
    1. ファイル転送
        ```
        $ cd helloworld
        $ ansible-playbook -i inventory.ini test_playbook.yml
        ```
    2. DBのクラスタリング(MariaDB Galera Cluster)
        ```
        $ cd wordpress
        $ ansible-playbook -i inventory.ini wordpress_deploy.yml
        ```

## Ansibleについて

### Ansibleとは

* Red Hatが開発するオープンソースの構成管理ツール

### 構成管理ツールとは

* サーバ構成をファイルで管理するツール
* 複数台の構築を一度にできる
* 冪等性（何回実行しても同じ状態になる）

### Ansibleの特徴

* エージェントレス(構築先にインストール不要)
* YAMLで書くので、（プログラムで書くより）敷居が低い
* ホストと対象にpythonとsshがあれば利用できる（大体ある）

### インストール

* 普通にaptやyumでやる

```
$ apt install ansible
```

### 実行ファイル

* 大きく分けて以下の2つ
    * inventory：実行対象のサーバを列挙
        * helloworld/inventory.ini
        * wordpress/inventory.ini
    * playbook：実行する処理を書く
        * helloworld/test_playbook.yml
        * wordpress_deploy.yml

## ファイル転送

* シンプルで単純
* 対象サーバは以下
    ```ini
    [test_servers]
    target1 //IP or ホストなど
    target2
    ```
* 処理
    ```yml
    ---
    - hosts: test_servers // inventoryで指定した対象サーバのグループ名
    tasks:
            - name: create directory // 1つの処理
                file:                // モジュールと呼ばれる機能。いろいろある。これはファイル作成用
                    path: /home/ansible/tmp
                    state: directory
                    owner: ansible
                    mode: 0755

            - name: copy file
                copy:                // ansibleホストから対象サーバへファイルコピーするタスク
                    src: /etc/hosts
                    dest: /home/ansible/tmp/hosts
                    owner: ansible
                    mode: 0644
    ```

## DBのクラスタリング(MariaDB Galera Cluster)

* 構築処理が多くなると、1つのplaybookにまとめることが難しくなってくる
* 可読性や保守性が下がるので、処理を分割する仕組みがある

### ロール

* 構築処理を処理単位で分けて管理する
    * yumのupdate用ロール
    * DBのインストール用ロール
    * DBの設定更新用ロール
* playbookと同列にrolesディレクトリを作成する。そこにロール名ごとディレクトリを用意
* tasks
    * 分割した構築処理
* vars
    * 変数定義
* templates
    * 設定ファイルなどのテンプレートファイル

### Task

#### yum
    ```
    - name: check_install / Remove necessary packages
    yum:
        name: mariadb-libs
        state: absent
    ```
* `yum remove  mariadb-libs`と同義


#### サービス

```
- name: configure / Stop all MariaDB for initialization
  systemd:
    name: mysql
    state: stopped
```

* サービスの起動/停止