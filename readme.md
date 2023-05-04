## 環境構築

### 開発環境の構築

WSL2を利用して環境構築を実施する。

#### WSL2の有効化＆Ubuntuのインストール

- [参考文献１（構築手順）](https://www.kkaneko.jp/tools/wsl/wsl2.html)
- [参考文献２（トラブルシューティング）](https://qiita.com/hali/items/bf04a1e4012025a38d6b)

#### Dockerのインストール

- [参考文献１（構築手A順）](https://qiita.com/na-777/items/617fc64d512f20b8e457)
- [参考文献２（トラブルシューティング）](https://zenn.dev/tkzwhr/articles/trouble-shooting-wsl2-docker)

### LocalStackのインストールから起動

[公式サイト](https://docs.localstack.cloud/getting-started/installation/#docker-compose)のDcoker-Composeの起動方法の手順を参考に起動する

1. 任意のディレクトリにdocker-compose.ymlを作成する

    ```
    touch docker-compose.yml
    ```

2. docker-compose.ymlの下記内容を記載する

    ```
    version: "3.8"
    
    services:
      localstack:
        container_name: "${LOCALSTACK_DOCKER_NAME-localstack_main}"
        image: localstack/localstack
        ports:
          - "127.0.0.1:4566:4566"            # LocalStack Gateway
          - "127.0.0.1:4510-4559:4510-4559"  # external services port range
        environment:
          - DEBUG=${DEBUG-}
          - DOCKER_HOST=unix:///var/run/docker.sock
        volumes:
          - "${LOCALSTACK_VOLUME_DIR:-./volume}:/var/lib/localstack"
          - "/var/run/docker.sock:/var/run/docker.sock"
    ```

3. 次のコマンドを実行してコンテナを起動します

    ```
    docker-compose up
    ```

以上で、LocalStackのサーバが起動された。

### AWS CLIのインストール
`awscli`をUbuntuにインストールするには、以下の手順に従ってください。

1. 最新のパッケージリストを取得します。

    ```
    sudo apt-get update
    ```

2. `awscli`をインストールするために必要なパッケージをインストールします。

    ```
    sudo apt-get install awscli
    ```

3. インストールが完了したら、`awscli`が正しくインストールされたかどうかを確認するために、以下のコマンドを実行してバージョン情報を取得します。

    ```
    aws --version
    ```

4. LocalStackにアクセスするため、credentialsを設定する。その際、credentialsの設定値は任意で問題ない。

    ```
    aws configure --profile localstack
    
    AWS Access Key ID [None]: test
    AWS Secret Access Key [None]: test
    Default region name [None]:ap-northeast-1 
    Default output format [None]: json
    ```

5. 動作確認として、`awscli`を利用して、LocalStackにアクセスする。

    ```
    # バケットを作成する
    aws s3 mb s3://localstack-bucket --endpoint-url=http://localhost:4566 --profile localstack
    >>> make_bucket: localstack-bucket
    
    # 作成したバケットを確認する
    aws s3 ls --endpoint-url=http://localhost:4566 --profile localstack
    >>> 2023-05-04 14:50:30 localstack-bucket
    ```
    
    また、上記コマンドの実行時、LocalStackのコンテナからも下記のログを見ることができたため、正常にアクセスできたことを確認できる。
    
    ```
    localstack_main  | 2023-05-04T05:50:30.080  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.CreateBucket => 200
    localstack_main  | 2023-05-04T05:54:05.592  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.ListBuckets => 200
    ```

以上で、Ubuntuに`awscli`をインストールして、LocalStackにアクセスまでを確認できた。

### awscli-localのインストール

上記までで疎通確認までは完了しているが、`awscli`を利用して、プロファイルの指定とエンドポイントの指定では、操作ミスに伴う、事故が起こる可能性が高くなると考えれる。
LocalStackが提供しているAWS CLIのラッパーである`awscli-local`を利用することで操作ミスを起こしにくいと思われるため、個人的にはこちらの利用を推奨する。

1. pipをインストールする。

    ```
    sudo apt-get update
    sudo apt-get install python3-pip
    ```

2. `awscli-local`をインストールする。

    ```
    pip3 install awscli-local
    ```

3. インストールが完了したら、以下のコマンドを実行してバージョン情報を確認します。

    ```
    awslocal --version
    ```

4. 動作確認として、`awscli-local`を利用して、LocalStackにアクセスする。

    ```
    # バケットを作成する
    awslocal s3 mb s3://localstack-bucket-awscli-local
    >>> make_bucket: localstack-bucket-awscli-local
    
    # 作成したバケットを確認する
    awslocal s3 ls
    >>> 2023-05-04 14:50:30 localstack-bucket
    >>> 2023-05-04 14:50:30 localstack-bucket-awscli-local
    ```

    また、上記コマンドの実行時、LocalStackのコンテナからも下記のログを見ることができたため、正常にアクセスできたことを確認できる。

    ```
    localstack_main  | 2023-05-04T05:59:49.676  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.CreateBucket => 200
    localstack_main  | 2023-05-04T05:59:57.678  INFO --- [   asgi_gw_0] localstack.request.aws     : AWS s3.ListBuckets => 200
    ```

以上で、Ubuntuに `awscli-local` をインストールして設定する手順が完了しました。

###　おわりに

まずは環境構築のみで、「1.はじめに」で問題提起しました「運用」までは触れてません。
これらから、`LocalStack`の知見を深めて、さらに記事を投稿していこうと思います。
