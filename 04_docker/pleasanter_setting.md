# ラズパイにPleasanterをインストールする手順

## 手順

1. .net6をインストール

```bash
# インストール
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version 6.0.418

# パスを追加
echo 'export DOTNET_ROOT=$HOME/.dotnet' >> ~/.profile
echo 'export PATH=$PATH:$HOME/.dotnet' >> ~/.profilw
source ~/.bashrc

# インストール確認
dotnet --version
>>> 6.0.418
```

2. pleasanterのDocker Hubを確認
    - https://hub.docker.com/r/implem/pleasanter

3. .envとyamlファイルを作成

```env
POSTGRES_USER={{Sa User}}
POSTGRES_PASSWORD={{Sa Password}}
POSTGRES_DB={{System DB name}}
POSTGRES_HOST_AUTH_METHOD=scram-sha-256
POSTGRES_INITDB_ARGS="--auth-host=scram-sha-256"
Implem_Pleasanter_Rds_PostgreSQL_SaConnectionString='Server=db;Database={{System DB}};UID={{Sa User}};PWD={{Sa password}}'
Implem_Pleasanter_Rds_PostgreSQL_OwnerConnectionString='Server=db;Database=#ServiceName#;UID=#ServiceName#_Owner;PWD={{Owner password}}'
Implem_Pleasanter_Rds_PostgreSQL_UserConnectionString='Server=db;Database=#ServiceName#;UID=#ServiceName#_User;PWD={{User password}}'
```

```yml
services:
  db:
    container_name: postgres
    image: postgres:15
    ports:
      - "15432:5432"
    volumes:
      - /var/dockervol/psql/data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER
      - POSTGRES_PASSWORD
      - POSTGRES_DB
      - POSTGRES_HOST_AUTH_METHOD
      - POSTGRES_INITDB_ARGS
    restart: always

  pleasanter:
    container_name: pleasanter
    image: implem/pleasanter
    depends_on:
      - db
    environment:
      Implem.Pleasanter_Rds_PostgreSQL_SaConnectionString: ${Implem_Pleasanter_Rds_PostgreSQL_SaConnectionString}
      Implem.Pleasanter_Rds_PostgreSQL_OwnerConnectionString: ${Implem_Pleasanter_Rds_PostgreSQL_OwnerConnectionString}
      Implem.Pleasanter_Rds_PostgreSQL_UserConnectionString: ${Implem_Pleasanter_Rds_PostgreSQL_UserConnectionString}
    ports:
      - "13500:8080"
    restart: always

  codedefiner:
    container_name: codedefiner
    image: implem/pleasanter:codedefiner
    depends_on:
      - db
    environment:
      Implem.Pleasanter_Rds_PostgreSQL_SaConnectionString: ${Implem_Pleasanter_Rds_PostgreSQL_SaConnectionString}
      Implem.Pleasanter_Rds_PostgreSQL_OwnerConnectionString: ${Implem_Pleasanter_Rds_PostgreSQL_OwnerConnectionString}
      Implem.Pleasanter_Rds_PostgreSQL_UserConnectionString: ${Implem_Pleasanter_Rds_PostgreSQL_UserConnectionString}

volumes:
  pg_vol:
```

4. 起動する

```bash

docker compose run --rm codedefiner _rds
docker compose up pleasanter -d

```