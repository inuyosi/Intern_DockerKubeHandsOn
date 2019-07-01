# CKAD 模擬試験

# 1

kubernetes dashboard を cluster 上に deploy して表示してください。

- https://github.com/kubernetes/dashboard を参考に install してください
- kubectl proxy することでアクセスできることを確認してください

# 2

CronJob を作成してください。

- namespace は `ex2` とする
- Job の名前は `ex2-job` とする
- Job は毎分実行すること 
- Job で実行するコンテナーの spec は下記とする
```yaml
containers:
- name: hello
  image: busybox
  command:
  - sh
  - -c
  - exit $((RANDOM % 2))
 ```           
- 同時に job が実行されないようにすること
- 成功した job の履歴は 5 世代残すこと

# 3

Pod が起動しない原因を探して下さい。

- namespace `ex3` を作成し、下記の manifest を適用すると Pod の Status が Pending になることを確認すること
- Pending の原因を調査し、Running な状態にするためにはどうしたらいいか考えること
- パラメーターを変更するのみでパラメーターを削除するとかはしないこと

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ex3-pod
  namespace: ex3
spec:
  containers:
  - name: db
    image: mysql
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "password"
    resources:
      requests:
        memory: "32Gi"
        cpu: "1000m"
      limits:
        memory: "64Gi"
        cpu: "2000m"
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "32Gi"
        cpu: "1000m"
      limits:
        memory: "64Gi"
        cpu: "2000m"
    volumeMounts:
    - mountPath: /wordpress
      name: data
  volumes:
  - name: data
    hostPath:
      path: /tmp/data
      type: Directory
```

# 4

Deployment の挙動について答えてください。

- strategy の Type が `Recreate` の場合と `RollingUpdate` の場合で Deployment の更新時にどのような挙動の違いがあるのか答えて下さい
- `RollingUpdate` の Deployment を `Rrecreate` と同じ挙動になるように設定するにはどのようにしたらいいか答えて下さい (replicas は 3 とします)
- 答えは @長谷川誠/makocchi に DM で送って下さい

# 5

NetworkPolicy を設定してください。

- namespace は `ex5` を使用すること
- nginx の image を使用した Pod を展開し、`type=frontend` の label を付与してください
- もう一つ nginx の image を使用した Pod を展開し、`type=backend` の label を付与してください
- `type=backend` の label が付いた Pod は `type=frontend` の label が付いた Pod からのみ接続可能になるように NetworkPolicy を設定してください

# 6

Pod を 4 つ作成し、全て同じ node に配置してください。

- namespace は `ex6` を使用すること
- 使用する image はそれぞれ `nginx`, `memcached`, `redis`, `httpd` を使用すること

# 7

initContainers を使って html ファイルを配置し、nginx で表示してください。

- namespace は `ex7` を使用すること
- 作成する Pod の名前は `ex7-pod` とする
- pod の image は`nginx`を使用すること
- 取得する html は `https://adtech.cyberagent.io/` とする
- initContainers を使い `/usr/share/nginx/html` 以下に取得した html を配置すること
- 永続化しない volume を使用すること

# 8

DaemonSet を作成してください。

- namespace は `ex8` を使用すること
- DaemonSet の名前は `ex8-ds` とする
- pod の image は`redis`を使用すること
- node の 1 台に `ex8-node=true` の label を付けること
  - (例kubectl label node "node名" ex8-node=true)
- `ex8-node=true` の label が付与されている node のみ pod が配置されること

# 9

StatefulSet を作成してください。

- namespace は `ex9` を使用すること
- StatefulSet の名前は `ex9-sts` とする
- replica の数は 3 とする
- `ssd` の volume をそれぞれの pod の `/usr/share/nginx/html` に mount すること
- volume の容量は 10Gi とする
- pod の image は `nginx` を使用すること

# 10

cluster 上に Nginx ingress controller を deploy し、ingress を作成してください。
- namespace は `ex10` を使用すること
- ingress controller には `nginx-ingress-controller` を使用すること
- LoadBalancer service を作成し、外部から ingress 経由で HTTP アクセスできるようにすること
- ingress のルールは下記で作成すること
  - ルールの名前は `ex10-ingress` とする
  - `/srv1` に来た traffic は `srv1` の service に route される
  - `/srv2` に来た traffic は `srv2` の service に route される
  - それ以外は `default-http-backend` の service に route される
  - virtual host には `ex10.cyberagent.local` を使用する
  - service に使用する port は 80

`srv1` および `srv2` は http で通信できるものであれば何でも構いません。
例として 1 つ表示しておきます。

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: srv1
  namespace: ex10
  labels:
    run: srv1
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: deploy-srv1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-srv1
  namespace: ex10
spec:
  selector:
    matchLabels:
      run: deploy-srv1
  replicas: 2
  template:
    metadata:
      labels:
        run: deploy-srv1
    spec:
      containers:
      - name: nginx
        image: makocchi/docker-nginx-hostname:latest
        ports:
        - containerPort: 80
```
