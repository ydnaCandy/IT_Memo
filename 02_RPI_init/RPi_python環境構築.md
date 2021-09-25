# Python環境構築



## 動作環境

- Raspberry Pi 4B
- OS:Raspberry pi OS
  - Released:2021-05-07
- 設定
  - RaspberryPi初期設定をパッケージの更新まで終了していること



## 環境構築

1. sshでRaspberry Piに接続する or Raspberry Pi上でターミナルを起動する

2. 下記コマンドを実行して、現在のpythonのデフォルトversionを確認する

   ```bash
   $ python --version
   >>> Python 2.7.16
   ```

3. 下記コマンドを実行して、Python3系へのシンボリックリンクを作成する

   ```bash
   # ディレクトリの移動
   $ cd /usr/bin
   # python3.xのxを確認する
   $ ls -l | grep python
   >>>...3.xを確認する
   
   # デフォルトのシンボリックリンクを切る
   $ sudo unlink python
   # python3系へのシンボリックリンクを設定
   $ sudo ln -s python3 python
   # 変わったことを確認する
   $ python --version
   >>>Python 3.7.3
   ```

   

4. 下記コマンドでpythonを起動し、Python3系が立ち上がったことを確認する

   ```bash
   $ python
   # pythonの3系が表示されればOK
   Python 3.7.3 (default, Jan 22 2021, 20:04:44)
   [GCC 8.3.0] on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>>
   ```

   

