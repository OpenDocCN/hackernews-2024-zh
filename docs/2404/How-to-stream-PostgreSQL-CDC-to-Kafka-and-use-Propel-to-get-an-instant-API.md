<!--yml

category: 未分类

date: 2024-05-27 12:49:58

-->

# 如何将 PostgreSQL CDC 流式传输到 Kafka 并使用 Propel 实现即时 API

> 来源：[https://www.propeldata.com/blog/postgresql-cdc-to-kafka](https://www.propeldata.com/blog/postgresql-cdc-to-kafka)

学习如何将 PostgreSQL 变更数据捕获（CDC）流式传输到 Kafka 主题，并使用 Propel 提供 API 服务数据。本指南提供逐步说明，从设置 PostgreSQL 数据库和 Kafka Broker 到部署 Debezium PostgreSQL 连接器和创建 Propel Kafka 数据池。

变更数据捕获（CDC）是跟踪和捕获数据变更的过程。在 PostgreSQL 中，我们可以利用事务日志有效地实现 CDC，这样我们就不必运行批处理作业来收集数据，而是可以连续地捕获数据变更。这有很多好处，包括允许应用程序几乎实时地查看和响应变更，同时对源系统的影响较小。在本文中，我们将逐步介绍如何将 PostgreSQL 的 CDC 流式传输到 [Propel Kafka 数据池](https://www.propeldata.com/docs/connect-your-data/kafka) 以获取即时数据服务 API。

## 我们试图做什么？

我们希望使用来自 PostgreSQL 的数据来支持大规模分析应用程序。众所周知，作为 OLTP 数据库，PostgreSQL 在处理处理和聚合大量数据的分析查询时速度很慢。

+   首先，我们希望捕获我们的 PostgreSQL 数据库表的所有更改，并将它们作为 JSON 消息发送到 [Kafka](https://kafka.apache.org/) 主题。

+   其次，我们需要配置 [Kafka Connect](https://docs.confluent.io/platform/current/connect/userguide.html#connect-userguide) 和 [Debezium PostgreSQL connector](https://debezium.io/documentation/reference/2.6/connectors/postgresql.html) 来将 CDC 流式传输到 Kafka 主题。

+   最后，创建一个 Propel Kafka 数据池，通过低延迟 API 从 Kafka 主题摄取数据。

我们将介绍如何在 Kubernetes 集群中创建和部署所有服务。

## PostgreSQL CDC 与 Debezium 的工作原理

在本节中，我们将简要介绍 PostgreSQL 变更数据捕获与 Debezium 的工作原理。

想象你有一个简单的表格：

```
CREATE TABLE "simple" (
	id uuid NOT NULL PRIMARY KEY,
	foo character varying,
);
```

你可以使用以下 SQL 语句更新行：

```
UPDATE public.simple SET foo = 'baz' WHERE ID = 'b18df779-0cae-43bc-9f4b-8efe571abd8a';
```

对列 <span class="code-exp">foo</span> 的更改将导致“update”到事务日志，然后 Debezium 将其转换为下面的 JSON 消息，并将其放入 Kafka 主题。

CDC JSON，包含 <span class="code-exp">before</span> 和 <span class="code-exp">after</span>：

```
{
  "before": {
    "id": "b18df779-0cae-43bc-9f4b-8efe571abd8a",
    "foo": "bar",
  },
  "after": {
    "id": "b18df779-0cae-43bc-9f4b-8efe571abd8a",
    "foo": "baz"
  },
  "source": {
    "version": "2.5.0.Final",
    "connector": "postgresql",
    "name": "pg",
    "ts_ms": 1707250399964,
    "snapshot": "false",
    "db": "production",
    "sequence": "[\"573123170952\",\"573123068760\"]",
    "schema": "public",
    "table": "simple",
    "txId": 55986537,
    "lsn": 573123068760,
    "xmin": null
  },
  "op": "u",
  "ts_ms": 1707250400533,
  "transaction": null
}
```

Debezium PostgreSQL 连接器为每个行级别的 <span class="code-exp">INSERT</span>、<span class="code-exp">UPDATE</span> 和 <span class="code-exp">DELETE</span> 操作生成数据变更事件。 <span class="code-exp">op</span> 键描述了导致连接器生成事件的操作。在此示例中，u 表示操作已更新行。有效值包括:

+   <span class="code-exp">c</span> = create

+   <span class="code-exp">u</span> = update

+   <span class="code-exp">d</span> = delete

+   <span class="code-exp">r</span> = read（仅适用于快照）

+   <span class="code-exp">t</span> = truncate

+   <span class="code-exp">m</span> = message

## 什么是 Propel Kafka 数据池？

Kafka [数据池](https://www.propeldata.com/docs/connect-your-data#key-concept-2-data-pools) 让您将实时流数据导入 Propel。它在 Kafka 之上提供即时低延迟的 API，用于支持实时仪表板、流式分析和工作流。

在以下情况考虑使用 Propel 在 Kafka 之上：

+   您需要在 Kafka 主题之上的 API。

+   您需要通过 Kafka 的流数据来支持实时分析应用。

+   您需要将 Kafka 消息导入 ClickHouse。

+   您需要从 [自托管 Kafka](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=self-hosted-kafka)、[Confluent Cloud](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=confluent)、[AWS MSK](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=aws-msk) 或 [Redpanda](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=redpanda) 导入 ClickHouse。

+   您需要转换或丰富您的流数据。

+   您需要为实时个性化和推荐用例提供支持。

一旦我们设置好 PostgreSQL、Debezium 和 Kafka，我们将把数据导入 Propel Kafka 数据池，以便通过 API 公开。

## 设置 Minikube、PostgreSQL、Kafka、Kafka Connect 和 Debezium

我们将在 Mac 上的 Kubernetes 集群中进行设置。这也可以在其他环境中完成，但 Kubernetes 是云中托管和运行数据流服务的常见平台。为了本例，我们将使用 [Minikube](https://minikube.sigs.k8s.io/docs/start/) 部署我们的 Kubernetes 集群，[Redpanda](https://redpanda.com/) 部署我们的 Kafka 集群，并使用 [Helm](https://helm.sh/) 管理像 PostgreSQL 这样的 Kubernetes 应用程序。如果您使用不同的操作系统，我将提供 Mac 特定的命令以及安装服务的链接。

### 设置 Minikube

**步骤 1:** **安装** [**Docker Desktop**](https://www.docker.com/products/docker-desktop/)

```
brew install --cask docker
```

**步骤 2:** **安装** [**Kubectl**](https://kubernetes.io/docs/tasks/tools/)

**步骤 3: 安装 minikube**

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

**步骤 4：** **启动 minikube**

```
minikube start --nodes 8 #docker-desktop will need to be running
```

**步骤 5：与您的集群交互**

```
minikube kubectl -- get pods -A
```

**步骤 6：** **安装 helm**

### 设置 PostgreSQL

**步骤 1：使用扩展配置安装 PostgreSQL Helm Chart**

扩展配置以设置 <span class="code-exp">wal_level</span>

```
cat > pg-values.yaml <<'EOF'
primary:
  extendedConfiguration: |-
    wal_level = logical
    shared_buffers = 256MB
EOF

helm install blog oci://registry-1.docker.io/bitnamicharts/postgresql -f pg-values.yaml 
```

**步骤 2：** **使用 PostgreSQL CLI 连接到 PostgreSQL 创建数据库**

```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default blog-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

kubectl run blog-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.2.0-debian-12-r6 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host blog-postgresql -U postgres -d postgres -p 5432
```

**步骤 3：创建表**

```
# after connecting to the PostgreSQL instance, create the table      
CREATE TABLE simple (
id uuid NOT NULL PRIMARY KEY,
foo character varying);

ALTER TABLE public.simple REPLICA IDENTITY FULL;
```

**步骤 4：创建一个 PostgreSQL 用户**

要为 Debezium 从 PostgreSQL 源表中流式传输更改设置具有必要特权的 PostgreSQL 用户，请使用超级用户权限运行此语句：

```
CREATE USER blog WITH REPLICATION PASSWORD 'your_password' LOGIN;
```

### 设置 Kafka（Redpanda）

**步骤 1：添加 helm 仓库并安装依赖项**

```
helm repo add jetstack https://charts.jetstack.io
helm repo add redpanda https://charts.redpanda.com
helm repo update
helm install cert-manager jetstack/cert-manager  --set installCRDs=true --namespace cert-manager  --create-namespace
```

**步骤 2：配置和安装 Redpanda**

```
 helm install redpanda redpanda/redpanda \
  --namespace redpanda \
  --create-namespace \
  --set statefulset.initContainers.setDataDirOwnership.enabled=true \
  --set image.tag=v23.3.4-arm64 \
  --set tls.enabled=false \
  --set statefulset.replicas=1 \
  --set auth.sasl.enabled=true \
  --set "auth.sasl.users[0].name=superuser" \
  --set "auth.sasl.users[0].password=changethispassword" \
  --set external.domain=customredpandadomain.local 
```

**步骤 3：安装 Redpanda 客户端 <span class="code-exp">rpk</span>**

```
brew install redpanda-data/tap/redpanda
```

**步骤 4：** **创建一个别名以简化 <span class="code-exp">rpk</span> 命令**

```
alias local-rpk="kubectl --namespace redpanda exec -i -t redpanda-0 -c redpanda -- rpk -X user=superuser -X pass=changethispassword -X sasl.mechanism=SCRAM-SHA-512"
```

**步骤 5：描述 Redpanda 集群**

```
local-rpk cluster info

CLUSTER
=======
redpanda.69087c28-e59b-4b25-bee2-bd199e6a9ab2

BROKERS
=======
ID    HOST                                             PORT
0*    redpanda-0.redpanda.redpanda.svc.cluster.local.  9093

TOPICS
======
NAME      PARTITIONS  REPLICAS
_schemas  1           1
```

**步骤 6：创建一个用户**

```
local-rpk acl user create blog -p changethispassword --mechanism scram-sha-256
```

**步骤 7：为用户** [**ACL**](https://docs.redpanda.com/current/manage/security/authorization/#create-acls) **创建一个 <span class="code-exp">blog</span>**

```
local-rpk acl create --allow-principal User:blog \
  --operation all  --topic '*' --group '*'
```

### 设置 Kafka Connect

**步骤 1：创建一个命名空间**

```
kubectl create namespace kafka-connect
```

**步骤 2：创建服务**

下面是您需要应用的清单，以设置与您的 Redpanda broker 连接的 Kafka Connect 服务（见上文）。复制下面的清单并提供配置环境所需的值。在下面的示例中，我们使用 <span class="code-exp">SASL_PLAINTEXT</span> 和 <span class="code-exp">SCRAM-SHA-256</span> 进行连接，请确保 <span class="code-exp">CONNECT_SASL_JAAS_CONFIG</span> 使用 <span class="code-exp">org.apache.kafka.common.security.scram.ScramLoginModule</span>，并且已为 <span class="code-exp">CONNECT_</span> 和 <span class="code-exp">PRODUCER_</span> 键设置了用户名和密码。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kafka-connect
  labels:
    app: kafka-connect
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kafka-connect
  template:
    metadata:
      labels:
        app: kafka-connect
    spec:
      containers:
      - name: kafka-connect
        image: debezium/connect:2.5.2.Final # Use the appropriate Debezium version
        env:
        - name: LOG_LEVEL
          value: INFO
        - name: BOOTSTRAP_SERVERS
          value: "redpanda-0.redpanda.redpanda.svc.cluster.local.:9093" # Replace with your Kafka broker address
        - name: GROUP_ID
          value: "blog-connect-cluster" # Replace with your group ID
        - name: CONFIG_STORAGE_TOPIC
          value: "connect-configs"
        - name: OFFSET_STORAGE_TOPIC
          value: "connect-offsets"
        - name: STATUS_STORAGE_TOPIC
          value: "connect-status"
        - name: CONNECT_SECURITY_PROTOCOL # additional values https://kafka.apache.org/24/javadoc/index.html?org/apache/kafka/common/security/auth/SecurityProtocol.html
          value: "SASL_PLAINTEXT"
        - name: CONNECT_SASL_MECHANISM # additional values https://docs.confluent.io/platform/current/kafka/authentication_sasl/auth-sasl-overview.html
          value: "SCRAM-SHA-256"
        - name: CONNECT_SASL_JAAS_CONFIG # additional values https://docs.confluent.io/platform/current/kafka/authentication_sasl/index.html
          value: "org.apache.kafka.common.security.scram.ScramLoginModule required username='blog' password='changethispassword';"
        - name: CONNECT_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
          value: "" # Set to an empty string to disable hostname verification
        - name: PRODUCER_SECURITY_PROTOCOL
          value: "SASL_PLAINTEXT"
        - name: PRODUCER_SASL_MECHANISM
          value: "SCRAM-SHA-256"
        - name: PRODUCER_SASL_JAAS_CONFIG
          value: "org.apache.kafka.common.security.scram.ScramLoginModule required username='blog' password='changethispassword';"
        - name: PRODUCER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM
          value: "" # Set to an empty string to disable hostname verification
        ports:
        - containerPort: 8083 
```

使用 <span class="code-exp">kubectl</span> 安装清单并在集群中启动 Kafka Connect pod。

```
kubectl apply -n kafka-connect -f kafka-connect-pod.yaml
```

应用清单后，您应该得到以下输出。

```
deployment.apps/kafka-connect configured
```

检查 pod 日志文件，确保服务可以登录到 broker。如果一切配置正确，您应该在日志中看到以下内容。

```
2024-03-03 23:40:40,484 INFO   ||  Successfully logged in.   [org.apache.kafka.common.security.authenticator.AbstractLogin]
2024-03-03 23:40:40,504 INFO   ||  Kafka version: 3.6.1   [org.apache.kafka.common.utils.AppInfoParser]
2024-03-03 23:40:40,504 INFO   ||  Kafka commitId: 5e3c2b738d253ff5   [org.apache.kafka.common.utils.AppInfoParser]
2024-03-03 23:40:40,504 INFO   ||  Kafka startTimeMs: 1709509240504   [org.apache.kafka.common.utils.AppInfoParser]
2024-03-03 23:40:40,508 INFO   ||  Kafka Connect worker initialization took 5851ms   [org.apache.kafka.connect.cli.AbstractConnectCli]
2024-03-03 23:40:40,508 INFO   ||  Kafka Connect starting   [org.apache.kafka.connect.runtime.Connect]
2024-03-03 23:40:40,510 INFO   ||  Initializing REST resources   [org.apache.kafka.connect.runtime.rest.RestServer]
2024-03-03 23:40:40,510 INFO   ||  [Worker clientId=connect-1, groupId=propel-connect-cluster] Herder starting   [org.apache.kafka.connect.runtime.distributed.DistributedHerder]
2024-03-03 23:40:40,510 INFO   ||  Worker starting   [org.apache.kafka.connect.runtime.Worker]
```

检查 <span class="code-exp">connect-offsets</span> 主题是否已在 Redpanda broker 上创建。

```
local-rpk topic list

NAME             PARTITIONS  REPLICAS
_schemas         1           1
connect-configs  1           1
connect-offsets  25          1
connect-status   5           1
```

### 设置 Debezium PostgreSQL 连接器

在部署Debezium PostgreSQL连接器到Kafka Connect服务之前，我们必须创建一个主题来存储CDC消息。如果查看下面的配置，我们已将属性<span class="code-exp">topic.prefix</span>设置为“blog”，<span class="code-exp">publication.autocreate.mode</span>设置为“filtered”，并将<span class="code-exp">table.include.list</span>设置为“public.simple”。这将配置连接器将CDC消息写入名为<span class="code-exp">blog.public.simple</span>的主题。使用以下命令创建主题，然后使用cURL命令部署连接器。您需要提供连接到您的PostgreSQL服务器的特定环境详细信息以及您上面使用的Redpanda服务的用户名/密码。

-   **Step 1: 创建Redpanda中的主题**

```
local-rpk topic create blog.public.simple
```

使用cURL安装Debezium PostgreSQL连接器。您需要获取PostgreSQL用户的密码并在下面填写它。我们还将设置从Kafka Connect服务到我们的本地主机的端口转发。

-   **Step 2: 获取PostgreSQL密码**

```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default blog-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
echo $POSTGRES_PASSWORD
```

-   **Step 3: 转发Kafka-Connect服务端口（8083）**

```
# Run this in a separate terminal to get the Pod name
kubectl get pod -n kafka-connect -o wide
NAME                             READY   STATUS    RESTARTS      AGE   IP             NODE       NOMINATED NODE   READINESS GATES
kafka-connect-5498764f4f-qw5qg   1/1     Running   1 (34m ago)   35m   10.244.0.253   minikube   <none>           <none>

# Run this command to forward the traffic to local host
kubectl port-forward -n kafka-connect pod/kafka-connect-5498764f4f-qw5qg 8083:8083

Forwarding from 127.0.0.1:8083 -> 8083
```

-   **Step 4: 使用cURL部署Debezium Postgres连接器**

```
curl -X POST http://127.0.0.1:8083/connectors -H "Content-Type: application/json" --data '{
                "name": "debezium-postgres-connector",
                "config": {
                  "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
                  "after.state.only": "false",
                  "tasks.max": "1",
                  "plugin.name": "pgoutput",
                  "database.hostname": "blog-postgresql.default.svc.cluster.local",
                  "database.port": "5432",
                  "database.user": "postgres",
                  "database.password": "<replace>",
                  "database.dbname": "postgres",
                  "topic.prefix": "blog",
                  "publication.autocreate.mode": "filtered",
                  "table.include.list": "public.simple",
                  "key.converter": "org.apache.kafka.connect.json.JsonConverter",
                  "value.converter": "org.apache.kafka.connect.json.JsonConverter",
                  "key.converter.schemas.enable": "false",
                  "value.converter.schemas.enable": "false",
                  "producer.override.sasl.mechanism": "SCRAM-SHA-256",
                  "producer.override.security.protocol": "SASL_PLAINTEXT",
                  "producer.override.sasl.jaas.config": "org.apache.kafka.common.security.scram.ScramLoginModule required username='blog' password='changethispassword';",
                  "producer.override.ssl.endpoint.identification.algorithm": "",
                  "producer.override.tls.insecure.skip.verify": "true"
                }
}'
```

### 插入数据并验证设置

让我们验证CDC消息是否落在我们创建的主题中；我们将向PostgreSQL表中插入一些数据，然后从主题中消费一条消息。

-   **Step 1: 连接到数据库**

```
kubectl run blog-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.2.0-debian-12-r6 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host blog-postgresql -U postgres -d postgres -p 5432
```

-   **Step 2: 插入数据到表中**

```
INSERT INTO public.simple VALUES (
'9cb52b2a-8ef2-4987-8856-c79a1b2c2f71',
'foo'
);

INSERT INTO public.simple values(
'9cb52b2a-8ef2-4987-8856-c79a1b2c2f73',
'bar'
);
```

-   **Step 3: 消费来自主题的CDC消息**

```
local-rpk topic consume blog.public.simple --num 1

{
  "topic": "blog.public.simple",
  "key": "{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f71\"}",
  "value": "{\"before\":null,\"after\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f71\",\"foo\":\"foo\"},\"source\":{\"version\":\"2.5.2.Final\",\"connector\":\"postgresql\",\"name\":\"blog\",\"ts_ms\":1710755794795,\"snapshot\":\"false\",\"db\":\"postgres\",\"sequence\":\"[null,\\\"22407328\\\"]\",\"schema\":\"public\",\"table\":\"simple\",\"txId\":747,\"lsn\":22407328,\"xmin\":null},\"op\":\"c\",\"ts_ms\":1710755795117,\"transaction\":null}",
  "timestamp": 1710755795494,
  "partition": 0,
  "offset": 0
}
```

我们验证了Debezium CDC消息的<span class="code-exp">before</span>和<span class="code-exp">after</span>值是否落在主题中。如果在此示例中<span class="code-exp">before</span>未填写，不必担心；如果查看，“op”为<span class="code-exp">c</span>，这意味着在表中创建了新行。

## 将Redpanda暴露到互联网

在本地使用<span class="code-exp">minikube</span>部署所有服务是进行本地开发和获得实际操作经验的好方法，而无需支付云服务提供商。但是，为了使用Propel并创建Kafka数据池，Propel必须连接到我们的Redpanda主题。有多种方法可以实现这一点。为了简单起见，我们将安装一个反向代理，并创建一个TCP隧道将Redpanda端口暴露到互联网。我们将使用的反向代理称为[LocalXpose](https://localxpose.io/)。

💡 要使用LocalXpose TCP隧道及其[专用端口](https://localxpose.io/docs/tunnels/tcp)，我不得不支付$6。其他免费服务也这样做，但您无法控制隧道所在的端口。创建外部NodePort时，Redpanda存在一个限制，仅限于端口<span class="code-exp">30000-32767</span>。

-   **Step 1: 安装LocalXpose和<span class="code-exp">kcat</span>**

[https://localxpose.io/docs](https://localxpose.io/docs)

```
brew install --cask localxpose
brew install kcat
```

-   **Step 2: 启动TCP隧道**

```
loclx tunnel tcp --port 31092 -t 127.0.0.1:9093
```

您现在可以看到正在运行的隧道。复制 <span class="code-exp">From</span> 中的域名，例如 <span class="code-exp">us.localx.io</span>。这将在步骤 3 和创建 Kafka 数据池时使用。

**步骤 3：升级 Redpanda 安装并公开一个外部 NodePort**

将下面的 yaml 保存到 <span class="code-exp">external-dns.yaml</span>，并用步骤 2 的值替换 <span class="code-exp-bracket">domain</span>。

```
external:
  enabled: true
  type: NodePort
  addresses:
    - <domain>
```

升级 Redpanda。

```
helm upgrade --install redpanda redpanda/redpanda --namespace redpanda --create-namespace --values external-dns.yaml --reuse-values --wait
```

**步骤 4：将流量转发到 <span class="code-exp">minikube</span>**

```
kubectl port-forward -n redpanda svc/redpanda-external 9093:9094
```

**步骤 5：验证您可以使用 <span class="code-exp">kcat</span> 进行外部连接**

```
kcat -L -b us.loclx.io:31092 -X sasl.username=blog -X sasl.password=changethispassword -X sasl.mechanism=SCRAM-SHA-256 -X security.protocol=SASL_PLAINTEXT
```

您现在可以将流量隧道到运行在 <span class="code-exp">minikube</span> 上的 Redpanda，允许 Propel 的后端连接并消费来自主题的 CDC 消息。

## 创建 Propel Kafka 数据池。

登录到 [Propel 控制台](https://console.propeldata.com) 或创建一个新帐户。然后选择您的环境并创建一个新的 Kafka 数据池，例如我在 <span class="code-exp">development</span> 环境中使用。

接下来，选择添加新凭据以配置到我们的 Redpanda 集群的连接。（使用上面为 Bootstrap 服务器创建的 TCP 隧道中的域和端口）

点击创建并测试凭据。

确保主题可见。

接下来，选择主题 <span class="code-exp">blog.public.simple</span>，开始拉取 CDC 数据。

点击下一步选择主要时间戳。

给数据池一个唯一的名称。

点击下一步开始将主题数据同步到 Propel 中。

一旦您在数据池中看到两条记录，您可以点击预览数据。

## 查询数据。

要查询在 Propel 中拥有的数据，您需要创建一个应用程序。请按照 [文档](https://www.propeldata.com/docs/applications) 中的指南创建一个应用程序。创建应用程序后，您可以获取一个令牌，如下图所示。我们将在下面的后续 API 调用中使用该令牌。

使用 [Data Grid API](https://www.propeldata.com/docs/query-your-data/data-grid) 查询新创建的 Kafka 数据池中的数据。确保复制您创建的令牌，并用值替换 <span class="code-exp-bracket">TOKEN</span>。您还需要用新创建的 Kafka 数据池的 ID 替换 <span class="code-exp-bracket">DATAPOOL_ID</span>。复制下面的 <span class="code-exp">curl</span> 命令，并在命令行中执行它。

```
curl 'https://api.us-east-2.propeldata.com/graphql' \
-H 'Authorization: Bearer <TOKEN>' \
-H 'Content-Type: application/json; charset=utf-8' \
-d '{
  "query": "query DataGridQuery($input: DataGridInput!) {
    dataGrid(input: $input) {
      headers
      rows
      pageInfo {
        hasNextPage
        hasPreviousPage
        endCursor
        startCursor
      }
    }
  }",
  "variables": {
    "input": {
      "dataPool": {
        "id": "<DATAPOOL_ID>"
      },
      "columns": [
        "_key",
        "_timestamp",
        "_propel_payload"
      ],
      "orderByColumn": 1,
      "sort": "DESC",
      "filters": [],
      "first": 5
    }
  }
}'
```

这将返回一个带有标题和行的数据网格。

```
{
  "dataGrid": {
    "headers": [
      "_key",
      "_timestamp",
      "_propel_payload"
    ],
    "rows": [
      [
        "{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\"}",
        "2024-03-18T09:56:37Z",
        "{\"before\":null,\"after\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\",\"foo\":\"bar\"},\"source\":{\"version\":\"2.5.2.Final\",\"connector\":\"postgresql\",\"name\":\"blog\",\"ts_ms\":1710755796308,\"snapshot\":\"false\",\"db\":\"postgres\",\"sequence\":\"[\\\"22407624\\\",\\\"22407680\\\"]\",\"schema\":\"public\",\"table\":\"simple\",\"txId\":748,\"lsn\":22407680,\"xmin\":null},\"op\":\"c\",\"ts_ms\":1710755796782,\"transaction\":null}"
      ],
      [
        "{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f71\"}",
        "2024-03-18T09:56:35Z",
        "{\"before\":null,\"after\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f71\",\"foo\":\"foo\"},\"source\":{\"version\":\"2.5.2.Final\",\"connector\":\"postgresql\",\"name\":\"blog\",\"ts_ms\":1710755794795,\"snapshot\":\"false\",\"db\":\"postgres\",\"sequence\":\"[null,\\\"22407328\\\"]\",\"schema\":\"public\",\"table\":\"simple\",\"txId\":747,\"lsn\":22407328,\"xmin\":null},\"op\":\"c\",\"ts_ms\":1710755795117,\"transaction\":null}"
      ]
    ],
    "pageInfo": {
      "hasNextPage": false,
      "hasPreviousPage": false,
      "endCursor": "eyJvZmZzZXQiOjJ9",
      "startCursor": "eyJvZmZzZXQiOjB9"
    }
  }
}
```

### 更新源 PostgreSQL 数据库。

如果我们更新 PostgreSQL 数据库中某行的新值，我们将看到在 Propel 数据池中反映出这种变化。让我们执行下面的 SQL 语句，并重新运行上面运行的 GraphQL 查询。

```
# Connec to the database
kubectl run blog-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.2.0-debian-12-r6 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host blog-postgresql -U postgres -d postgres -p 5432

UPDATE public.simple SET foo = 'baz' WHERE id = '9cb52b2a-8ef2-4987-8856-c79a1b2c2f73';
```

当我们重新运行 GraphQL 查询时，我们得到以下响应。

```
{
  "dataGrid": {
    "headers": [
      "_key",
      "_timestamp",
      "_propel_payload"
    ],
    "rows": [
      [
        "{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\"}",
        "2024-03-18T09:56:37Z",
        "{\"before\":null,\"after\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\",\"foo\":\"bar\"},\"source\":{\"version\":\"2.5.2.Final\",\"connector\":\"postgresql\",\"name\":\"blog\",\"ts_ms\":1710755796308,\"snapshot\":\"false\",\"db\":\"postgres\",\"sequence\":\"[\\\"22407624\\\",\\\"22407680\\\"]\",\"schema\":\"public\",\"table\":\"simple\",\"txId\":748,\"lsn\":22407680,\"xmin\":null},\"op\":\"c\",\"ts_ms\":1710755796782,\"transaction\":null}"
      ],
      [
        "{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\"}",
        "2024-03-18T17:38:53Z",
        "{\"before\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\",\"foo\":\"bar\"},\"after\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f73\",\"foo\":\"baz\"},\"source\":{\"version\":\"2.5.2.Final\",\"connector\":\"postgresql\",\"name\":\"blog\",\"ts_ms\":1710783532893,\"snapshot\":\"false\",\"db\":\"postgres\",\"sequence\":\"[\\\"22407880\\\",\\\"22408168\\\"]\",\"schema\":\"public\",\"table\":\"simple\",\"txId\":749,\"lsn\":22408168,\"xmin\":null},\"op\":\"u\",\"ts_ms\":1710783533138,\"transaction\":null}"
      ],
      [
        "{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f71\"}",
        "2024-03-18T09:56:35Z",
        "{\"before\":null,\"after\":{\"id\":\"9cb52b2a-8ef2-4987-8856-c79a1b2c2f71\",\"foo\":\"foo\"},\"source\":{\"version\":\"2.5.2.Final\",\"connector\":\"postgresql\",\"name\":\"blog\",\"ts_ms\":1710755794795,\"snapshot\":\"false\",\"db\":\"postgres\",\"sequence\":\"[null,\\\"22407328\\\"]\",\"schema\":\"public\",\"table\":\"simple\",\"txId\":747,\"lsn\":22407328,\"xmin\":null},\"op\":\"c\",\"ts_ms\":1710755795117,\"transaction\":null}"
      ]
    ],
    "pageInfo": {
      "hasNextPage": false,
      "hasPreviousPage": false,
      "endCursor": "eyJvZmZzZXQiOjJ9",
      "startCursor": "eyJvZmZzZXQiOjB9"
    }
  }
}
```

这显示我们的数据池中有三行数据；它反映了已发送到 Kafka 数据池的原始 CDC 消息。为了将数据用于分析，我们需要对其进行转换，以正确表示源数据库中的当前状态。

## 创建一个实时转换来解包 CDC JSON 消息

现在我们已经将 CDC 消息流发送到 Propel 并将其落入 Kafka 数据池，我们需要编写一些 SQL 来根据“op”类型转换消息。我们感兴趣的列是 <span class="code-exp">id</span>、<span class="code-exp">_timestamp</span> 和 <span class="code-exp">foo</span>。下面的 <span class="code-exp">SELECT</span> 语句将查询源表 <span class="code-exp">blog-post</span> 并从 <span class="code-exp">_propel_payload.after</span> 中提取 <span class="code-exp">id</span> 和 <span class="code-exp">foo</span>。这将处理创建和更新操作。在更新操作的情况下，将插入一个重复的行，然后由 Propel 进行去重。这将确保在我们将 <span class="code-exp">id</span> 设置为 MaterializedView 中的 <span class="code-exp">uniqueId</span> 列时，只存在一个版本的 <span class="code-exp">id</span>，其值来自源表的最新值。

```
SELECT 
    "blog-post"."_propel_payload.after.id" AS id,
    "_timestamp" AS timestamp,
    "blog-post"."_propel_payload.after.foo" AS foo
FROM 
    "blog-post" # the name of the Kafka Data Pool from above
WHERE 
    "blog-post"."_propel_payload.op" = 'c' OR 
    "blog-post"."_propel_payload.op" = 'u';
```

你应该得到如下的响应。

```
{
  "data": {
    "createMaterializedView": {
      "materializedView": {
        "id": "MAT01HSY2DHN9HWKD1QAG0B9C115J",
        "sql": "SELECT \"blog-post\".\"_propel_payload.after.id\" as id, \"_timestamp\" as timestamp, \"blog-post\".\"_propel_payload.after.foo\" as foo FROM \"blog-post\" WHERE \"blog-post\".\"_propel_payload.op\"='c' OR \"blog-post\".\"_propel_payload.op\" ='u'",
        "destination": {
          "id": "DPO01HSY2DHTZZDRZP2BDZAMX6PKP",
          "uniqueName": "Data Pool for Materialized View MAT01HSY2DHN9HWKD1QAG0B9C115J",
          "tableSettings": {
            "partitionBy": null,
            "primaryKey": null,
            "orderBy": [
              "\"id\""
            ]
          }
        },
        "source": {
          "id": "DPO01HS98EGS0W40ARYHBR2SDVR5X",
          "uniqueName": "blog-post",
          "tableSettings": {
            "partitionBy": null,
            "primaryKey": null,
            "orderBy": null
          }
        }
      }
    }
  }
}
```

### 查询 Materialized View 数据池

现在我们已经创建了 MaterializedView，让我们查询与其一同创建的 Data Pool。用之前创建的令牌替换 <span class="code-exp-bracket">token</span>，并使用上述 <span class="code-exp">curl</span> 命令的响应来获取目标数据池 ID。这在 JSON 中可以通过 <span class="code-exp">data.createMaterializedView.materializedView.destination.id</span> 找到。将 <span class="code-exp">datapool_id</span> 替换为该值，然后从命令行运行 <span class="code-exp">curl</span> 命令。

```
 curl 'https://api.us-east-2.propeldata.com/graphql' \
-H 'Authorization: Bearer <TOKEN>' \
-H 'Content-Type: application/json; charset=utf-8' \
-d '{
  "query": "query DataGridQuery($input: DataGridInput!) { dataGrid(input: $input) { headers rows pageInfo { hasNextPage hasPreviousPage endCursor startCursor } } }",
  "variables": {
    "input": {
      "dataPool": {
        "id": "<DATAPOOL_ID>"
      },
      "columns": [
        "id",
        "foo"
      ],
      "orderByColumn": 1,
      "sort": "DESC",
      "filters": [],
      "first": 5
    }
  }
}'
```

你所得到的响应应该只有两行，并且更新了 <span class="code-exp-bracket">baz</span> 的值为 <span class="code-exp">9cb52b2a-8ef2-4987-8856-c79a1b2c2f73</span>。

```
{
  "dataGrid": {
    "headers": [
      "id",
      "foo"
    ],
    "rows": [
      [
        "9cb52b2a-8ef2-4987-8856-c79a1b2c2f73",
        "baz"
      ],
      [
        "9cb52b2a-8ef2-4987-8856-c79a1b2c2f71",
        "foo"
      ]
    ],
    "pageInfo": {
      "hasNextPage": false,
      "hasPreviousPage": false,
      "endCursor": "eyJvZmZzZXQiOjF9",
      "startCursor": "eyJvZmZzZXQiOjB9"
    }
  }
}
```

## 可视化结果

让我们编写一个小的 <span class="code-exp">node.js</span> 应用程序，结合 Propel 的所有概念，并使用我们创建的 Propel 应用程序的 OAuth2 客户端凭据来呈现包含 MaterializedView 结果的表格。这些凭据可以在 Propel 控制台的应用程序部分中找到，如下所示。

**步骤 1：OAuth2 客户端凭据**

**步骤 2：设置环境变量**

用控制台中的值替换令牌，然后粘贴并执行命令。

```
export CLIENT_ID=<CLIENT_ID> \
export CLIENT_SECRET=<CLIENT_SECRET> \
export TOKEN_HOST=https://auth.us-east-2.propeldata.com \
export TOKEN_PATH=/oauth2/token
```

**步骤 3：安装依赖项**

```
npm install simple-oauth2 cli-table
```

**步骤 4：复制代码**‍

复制下面的代码，并将 <span class="code-exp-bracket">DATAPOOL_ID</span> 替换为上面 MaterializedView 的数据池 ID。

```
const { ClientCredentials } = require("simple-oauth2");
const Table = require("cli-table");

async function getToken() {
  const config = {
    client: {
      id: process.env.CLIENT_ID,
      secret: process.env.CLIENT_SECRET,
    },
    auth: {
      tokenHost: process.env.TOKEN_HOST,
      tokenPath: process.env.TOKEN_PATH,
    },
  };

  const client = new ClientCredentials(config);

  try {
    const accessToken = await client.getToken();
    return accessToken.token.access_token;
  } catch (error) {
    console.error("Access Token Error", error.message);
    return null;
  }
}

async function fetchDataGrid(accessToken) {
  const url = "https://api.us-east-2.propeldata.com/graphql";
  const query = `
    query DataGridQuery($input: DataGridInput!) {
      dataGrid(input: $input) {
        headers
        rows
      }
    }
  `;
  const variables = {
    input: {
      dataPool: { id: "<DATAPOOL_ID>" },
      columns: ["id", "foo"],
      sort: "ASC",
      first: 5,
      last: 5,
    },
  };

  try {
    const response = await fetch(url, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${accessToken}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        query,
        variables,
      }),
    });

    const data = await response.json();
    return data.data.dataGrid;
  } catch (error) {
    console.error("Error fetching data grid:", error.message);
    return null;
  }
}

async function main() {
  const accessToken = await getToken();

  if (!accessToken) {
    console.error("Failed to retrieve access token");
    return;
  }

  const dataGrid = await fetchDataGrid(accessToken);

  if (!dataGrid) {
    console.error("Failed to fetch data grid");
    return;
  }

  const { headers, rows } = dataGrid;

  const table = new Table({
    head: headers,
  });

  rows.forEach((row) => {
    table.push(row);
  });

  console.log(table.toString());
}

main(); 
```

**第五步：运行脚本**

```
node script.js                                                                                                                 <aws:prod-propel>
┌──────────────────────────────────────┬─────┐
│ id                                   │ foo │
├──────────────────────────────────────┼─────┤
│ 9cb52b2a-8ef2-4987-8856-c79a1b2c2f71 │ foo │
├──────────────────────────────────────┼─────┤
│ 9cb52b2a-8ef2-4987-8856-c79a1b2c2f73 │ baz │
└──────────────────────────────────────┴─────┘
```

## 总结

在结束本篇文章时，重点是回顾您已经掌握的关键技能和知识。您已经学会了如何有效地流式传输PostgreSQL变更数据捕获（CDC）到Propel Kafka DataPool，这是管理实时数据的重要一步。

此外，您已经掌握了查询这些数据的能力，这为分析和洞察力开辟了大量机会。本指南还指导您如何更新源PostgreSQL数据库，确保您可以保持数据的准确性和相关性。

本指南的亮点之一是教您如何创建实时转换以解析CDC JSON消息。这是提升您数据管理能力的基本技能，使您能够为各种应用程序操作和解释数据。

最后，您学会了如何可视化结果。可视化在数据分析中起着关键作用，使复杂数据更易理解，揭示趋势和异常值，并有助于更有效的沟通。

希望您觉得这些内容既具信息性又实用，并且享受学习过程。Propel致力于为您提供工具和知识，将您的数据转化为有价值的洞察力。我们期待着在您使用Propel的未来旅程中支持您。继续探索和转化您的数据！
