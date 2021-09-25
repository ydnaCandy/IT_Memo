# Raspberry Pi設定

## 使用デバイス

- Raspberry Pi 3B+

## OSのインストール

Raspberry Pi OSのインストールを想定した手順を示す。

### Raspberry Pi Imagerを使ってインストール

https://www.raspberrypi.org/software/からRaspberry Pi Managerをインストールした状態からの手順を示す。

1. Raspberry Pi Imagerを立ち上げる
1. `CHOOSE OS`から`Erase`を選択
1. `CHOOSE STORAGE`からOSを書き込みたいSDカードを選択
1. `Write`をクリックする
1. `CHOOSE OS`から`Raspberry Pi OS`を選択
1. `CHOOSE STORAGE`からOSを書き込みたいSDカードを選択
1. `Write`をクリックする
1. 書き込みが終わったら、PCからSDカードを取り外す

### Raspberry Piを起動する

OSを書き込んだSDカードをRaspberry Piに差し込み、Raspberry Piをモニターに接続してから電源を入れる。

Raspberry Piが起動してデスクトップ画面が表示されると初期設定ウィンドウが表示されるので、下記の通り設定する。

1. Welcome to Raspberry Piというウィンドウが表示されるのでNextをクリック
2. Set Country画面でCountryをJapanにして、Nextをクリック
3. Change Password画面では何も入力せず、Nextをクリック
4. Set Up Screen画面では何も入力せず、Nextをクリック
5. Select Wireless Network画面では何も入力せず、Skipをクリック
6. Update Software画面では何も入力せず、Skipをクリック
7. 最後の画面でDoneをクリックする。





## ネットワークの設定

### 無線LANの場合

1. ターミナルを開く。

1. SSIDをステルスモードにしているため、下記コマンドで接続設定を行う。

   ```bash
   # パスフレーズを暗号化。SSID、パスワード名を入力
   # SSID名がTest、パスワードがtest_pswdの場合は、
   # wpa_passphrase "Test" "test_pswd"というコマンドで実行
   $ wpa_passphrase "SSID" "PASSPHRASE"
   >>> network={
   >>>         ssid="SSID"
   >>>         #psk="PASSPHRASE"
   >>>         psk=XXXXXXXXXXXX
   >>> 		}
   
   # 出力された情報を右クリックでコピーしておく。
   
   # コンフィグファイルをnanoエディターで開く。
   $ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
   ```

   nanoで開いたファイル内の一番下に、コピーした内容を貼り付ける。

   ```text
   ~中略~
   network={
           ssid="SSID"
           psk=XXXXXXX
          }
   ```

1. Ctrl+Oで上書き状態にし、Enterで上書き、Ctrl+Xでnanoエディターを閉じる。

1. IPアドレスを固定するため、nanoエディタで設定ファイルを開く。

   ```bash
   # nanoエディターでコンフィグファイルを開く
   $ sudo nano /etc/dhcpcd.conf
   ```

   

1. nanoエディタで開いたファイルの一番下に下記の内容を記載する。

   ※自宅の環境に合わせて設定。

   ```bash
   # staric wifi profile
   interface wlan0
   static ip_address=[設定したい固定IPアドレス]/24
   static routers=[デフォルトゲートウェイのIPアドレス]
   static domain_name_servers=[DNSサーバーのIPアドレス]
   ```

   

1. Ctrl+Oで上書き状態にし、Enterで上書き、Ctrl+Xでnanoエディターを閉じる。

1. 再起動する。

   ```bash
   $ sudo reboot now
   ```

1. ターミナルを開く or sshで接続し、IPアドレスが固定されたか確認する。

   ```bash
   $ ifconfig
   >>>wlan0
   >>>設定した内容
   ```

   



## SSHの設定

### PC側での操作

1. Raspberry PiのOSが入ったSDカードをPCに差し込む。
1. SDカードのboot直下に、`ssh`というファイルを作成する。
1. SDカードを取り出す。

### Raspberry Pi側での操作

1. 取り出したSDカードをRaspberry Piに差し込み、電源を入れる



### PC側での操作

- 今回はMac＋VSCodeからの接続

1. ターミナルを開き、sshコマンドを実行する

   ```bash
   $ ssh pi@[固定したIPアドレス]
   >>> パスワード # 初期パスワードはraspberry
   ```

   

2. sshでの接続を確認

   ```bash
   $ pi@raspberrypi:~ $ 
   ```







## ユーザー名の変更とユーザーの登録

sshで接続した状態 or Raspberry Pi上のターミナルで操作

1. 下記コマンドで、新規ユーザーを追加

   ```bash
   $ sudo useradd -M tmp
   ```

   

2. 下記コマンドで、`tmp`ユーザーを`sudo`グループに追加

   ```bash
   $ sudo gpasswd --add tmp sudo
   >>> ユーザ tmp をグループ sudo に追加
   ```

   

3. 下記コマンドで、`tmp`ユーザーのパスワードを変更

   ```bash
   $ sudo passwd tmp
   >>> 新しいパスワード: # hoge
   >>> 新しいパスワードを再入力してください:
   >>> passwd: パスワードは正しく更新されました
   ```

   

4. 下記コマンドで、raspp-configを開く

   ```bash
   $ sudo raspi-config
   ```

   

5. `1 System Option > S5 Boot/Auto Login > B3 Desktop`を選択し、ログイン時にユーザー名とパスワードを求める設定にする

6. finishを選択し、再起動する

7. 再度sshにて、piユーザーでログイン

8. 下記コマンドを実行して、ファイルを編集する

   ```bash
   $	sudo nano /etc/systemd/system/autologin@.service
   ```

   ファイル内

   ```text
   # 変更前
   ExecStart=-/sbin/agetty --autologin pi --noclear %I $TERM
   
   # 変更後
   ExecStart=-/sbin/agetty --autologin tmp --noclear %I $TERM
   ```

   

9. Ctrl+Oで上書きして、Ctrl+Xで閉じる

10. 再起動する

   ```bash
   $ sudo reboot now
   ```

   

11. 再度SSHでログイン(ユーザー：tmp、パスワード：hoge) or ラズパイのターミナルを開く

12. 下記コマンドを実行して、tmpユーザーしかいないことを確認

    ```bash
    $ who
    tmp      pts/0        2021-07-11 17:37 (XXX.XXX.X.X)
    ```

    

13. piユーザーを任意のユーザーに変更する

    ```bash
    $ sudo usermod -l [user] pi
    ```

    

14. piディレクトリを変更したユーザーでそのまま使えるようにする

    ```bash
    $ sudo usermod -d /home/admin -m admin
    
    $ sudo groupmod -n admin pi
    ```

    

15. 再起動する

    ```bash
    $ sudo reboot now
    ```

    

16. 再度SSHでログイン(ユーザー：[user]、パスワード：raspberry) or ラズパイのターミナルを開く

17. 新規ユーザーのパスワードを変変更する

    ```bash
    $ sudo passwd admin
    >>> [sudo] [user] のパスワード: # raspberyy
    >>> 新しいパスワード: # 新規のパスワード
    >>> 新しいパスワードを再入力してください:
    >>> passwd: パスワードは正しく更新されました
    ```

    

18. 下記コマンドを実行して、コンフィグファイルを編集する

    ```bash
    $ sudo nano /etc/systemd/system/autologin@.service 
    ```

    ファイル内

    ```text
    # 変更前
    ExecStart=-/sbin/agetty --autologin tmp --noclear %I $TERM
    
    # 変更後
    ExecStart=-/sbin/agetty --autologin [user] --noclear %I $TERM
    ```

    

19. 再起動する

    ```bash
    $ sudo reboot now
    ```

    

20. 再度SSHでログイン(ユーザー：[user]、パスワード：[password]) or ラズパイのターミナルを開く

21. tmpユーザーを削除

    ```bash
    $ sudo userdel tmp
    ```

    

22. 再起動する

    ```bash
    $ sudo reboot now
    ```



## sudoコマンド時にパスワードを要求させる

1. sshでログイン

2. 下記コマンドを実行して、パスワードを要求させる

   ```bash
   $ sudo nano /etc/sudoers.d/010_pi-nopasswd
   ```

   ```text
   # 変更前
   pi ALL=(ALL) NOPASSWD: ALL
   
   # 変更後
   [user] ALL=(ALL) NOPASSWD: ALL
   ```

   



## パッケージリストの更新

1. sshでログイン

2. 下記コマンドを実行して、パッケージリストを更新する

   ```bash
   $ sudo apt-get update
   ```

   

3. 下記コマンドを実行して、opensssh-serverをアップデートする

   ```bash
   $ sudo apt install openssh-server
   ```

   

4. 下記コマンドを実行して、パッケージ更新して新しいバージョンにアップグレードする

   ```bash
   $ sudo apt-get upgrade
   ```

   

## ファイアウォールの設定

1. ラズパイ上でターミナルを開く

2. 下記コマンドでファイアウォールをインストールする

   ```bash
   $ sudo apt install ufw
   ```

   

3. ファイアウォールの状況を確認する

   ```bash
   $ sudo ufw status
   # まだアクティブになってない
   >>> Status: inactive
   ```

   

4. まずはすべて拒否する設定をする

   ```bash
   $ sudo ufw default deny
   ```

   

5. ファイアウォールを起動する

   ```bash
   # ファイアウォールを起動する
   $ sudo ufw enable
   
   # 起動を確認
   $ sudo ufw status
   >>> Status: active
   ```

6. PC側からsshでログインしてみて、拒否されているかどうかを確認

7. ラズパイのターミナル上から、ファイアウォールを停止する

   ```bash
   $ sudo ufw disable
   >>> Firewall stopped
   ```

8. 接続を許可したいIPもしくはポートを設定する

   ```bash
   # 今回は家のネットワーク内からの接続はすべて許可する設定
   $ sudo ufw allow from XXX.XXX.X.0/24
   
   # shhを許可する場合
   $ sudo ufw allow from XXX.XXX.X.0/24 any port 22 proto tcp
   ```

   

9. ファイアウォールを起動する

   ```bash
   $ sudo ufw enable
   ```

   

10. PCからsshでログインできるか確認する

11. sshでログインした状態で、ステータスを確認し、設定が反映されていることを確認する。

    ```bash
    sudo ufw status
    Status: active
    
    >>> To                         Action      From
    >>> --                         ------      ----
    >>> Anywhere                   ALLOW       XXX.XXX.X.0/24 
    ```







## fail2banを設定

1. sshでログイン

2. 下記コマンドで、fail2banをインストールする

   ```bash 
   $ sudo apt install fail2ban
   ```

   

3. 設定ファイルの拡張子を変更してコピーする

   ```bash
   $ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   ```

   

4. コピーしたファイルを編集する

   ```bash
   $ sudo nano /etc/fail2ban/jail.conf
   ```

   変更前

   ```text
   <...略...>
   [sshd]
   # To use more aggressive sshd modes set filter parameter "mode" in jail.local:
   # normal (default), ddos, extra or aggressive (combines all).
   # See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and det$
   #mode   = normal
   port    = ssh
   logpath = %(sshd_log)s
   backend = %(sshd_backend)s
   ```

   変更後

   ```text
   [sshd]
   
   # To use more aggressive sshd modes set filter parameter "mode" in jail.local:
   # normal (default), ddos, extra or aggressive (combines all).
   # See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and det$
   #mode   = normal
   enable = ture  ←追記
   filter = sshd  ←追記
   port    = ssh 
   bantime = 21600  ←追記
   maxretry = 5  ←追記
   ```

   

5. 再起動する

   
