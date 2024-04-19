1. opensshをインストール

```bash
sudo apt install openssh-server

# 確認
systemctl status ssh
>>> Active

# 自動起動
systemctl enable ssh
```


2. /etc/ssh/sshd_configを編集

- とりあえずパスワード認証を想定した手順 

```bash
nano /etc/ssh/sshd_config
```

```conf
# RootユーザのログインをNG
PermitRootLogin no

# パスワード設定がないユーザーを拒否
PermitEmptyPassword no

# リトライ数
MaxAuthTries 6
```

3. サービスをリスタートする

```bash
sudo systemctl restart ssh
```

4. 接続できることを確認する