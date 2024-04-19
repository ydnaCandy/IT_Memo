# Ubuntu OSでIPアドレスを固定する手順

## 手順

1. netplanのデフォルトファイルをコピーする

    ```bash
    $ cd /etc/netplan
    $ ls
    >>> 01-network-manager-all.yaml  99-custom.yaml

    $ cp 99-custom.yaml 99-custom.yaml_bk
    ```

2. netplanを編集する

    ```yml
    network:
    version: 2
    wifis:
        wlp3s0:
        dhcp4: false
        dhcp6: false
        optional: false
        addresses: [192.168.3.101/24]
        gateway4: 192.168.3.1
        nameservers:
            addresses: [192.168.3.1]
        access-points:
            "chat_wifi":
            hidden: true
            password: "aguer010"
    ```

3. netplanを適用する

    ```bash
    $ sudo netplan apply
    ```