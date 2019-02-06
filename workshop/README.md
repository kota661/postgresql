# PostgreSQL

PostgreSQLをIKS上にデプロイしてみよう。

* 学べること
    Persistent Volumeを活用しデータの永続化を行う方法

* 事前準備
  * IBM Cloudの有料アカウント
  * IKSの標準クラスタ（有料）


## Persistent Volume とは？

クラスターにボリュームとして追加される仮想ストレージ・インスタンスです。
PVを利用することで、DBなど永続化すべきデータの保管が行えます。
また、ワーカーノードのストレージを利用しないため、どのノードにPodが配置されたとしても利用することが可能です。
IKSでは仮想ストレージとしてIBM CloudのEndurance Storageを利用しています。

詳しくは：Kubernetes ストレージの基本について
https://console.bluemix.net/docs/containers/cs_storage_basics.html#kube_concepts


## PostgreSQLのコンテナイメージを試す
1. ローカルでPostgresを動かす
    ```
    docker run --name postgre -p 5432:5432 -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password -d -v pgdata:/var/lib/postgresql/data postgres:11
    ```

2. clientツールでDBにアクセスしてみる

   * 代表的なツール
     * CLI: [psql](https://www.postgresql.jp/document/9.1/html/app-psql.html)
     * GUI: [DBeaver](https://dbeaver.io/)


## IKSへのデプロイ方法
1. IKSにアクセス

   * 作成したクラスタの**アクセスタブ**を参照ください。
   * ```kubectl get nodes``` が実行できればOKです。


2. Persistent Volume Clameを作成

    ```
    kubectl apply -f pg-pv-claim.yml

    # 結果確認
    kubectl get pvc
    ```


3. DeploymentとServiceを作成

    ```
    kubectl apply -f pg-deployment.yml
    kubectl apply -f pg-service.yml

    # 結果確認
    kubectl get deploy,svc
    ```


## Tips
* IOS

* オーダーしたEndurance StorageはPVCを削除した場合、自動で削除されますか？
  * 名前に retainがついているStorageClassの場合は削除されません。それ以外の場合は削除されます
    ```
    NAME                         PROVISIONER        AGE
    default                      ibm.io/ibmc-file   1h
    ibmc-file-bronze (default)   ibm.io/ibmc-file   1h
    ibmc-file-custom             ibm.io/ibmc-file   1h
    ibmc-file-gold               ibm.io/ibmc-file   1h
    ibmc-file-retain-bronze      ibm.io/ibmc-file   1h
    ibmc-file-retain-custom      ibm.io/ibmc-file   1h
    ibmc-file-retain-gold        ibm.io/ibmc-file   1h
    ibmc-file-retain-silver      ibm.io/ibmc-file   1h
    ibmc-file-silver             ibm.io/ibmc-file   1h
    ```

## APPENDIX
* IBM Cloud の IBM ファイル・ストレージへのデータの保管
    - https://console.bluemix.net/docs/containers/cs_storage_file.html#file_storage