# dockerでNGINXを構築

## 環境
ラズパイOS(bullseye)で検証

## 手順
1. 任意のディレクトリに`docker-compose.yml`を作成

    ```yml
    version: "3"

    services:
    nginx:
        container_name: nginx
        image: nginx:latest
        build:
        ./nginx
        ports:
        - "14080:80"
        volumes:
        - ./nginx/html:/usr/share/nginx/html 
        - ./nginx/conf.d:/etc/nginx/conf.d # nginxのconf.dをマウント
    ```

2. マウントしたconfディレクトリにnginxへの設定ファイルを作成

    ```bash
    # nginx/con.dで操作
    $ nano default.conf
    ```

    ```conf
    server {
        listen       80;
        server_name  localhost;
        location / {
            # docker内のディレクトリを指定
            root /usr/share/nginx/html;
            index  index.html index.htm;
        }
    }
    ```

3. `nginx/html`内に`index.html`を作成してなにか書いておく

4. 設定ファイルをNGINXに反映する

    ```bash
    # コンテナにログイン
    $ sudo docker exec -it nginx bash
    # 設定ファイルのチェック
    $ nginx -t
    # 設定ファイルの反映
    $ nginx -s reloa
    ```


5. `docker compose up -d`でコンテナを起動

6. 80番ポートにアクセスして、`index.html`の内容が表示されたことを確認