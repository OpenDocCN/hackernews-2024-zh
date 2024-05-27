<!--yml
category: 未分类
date: 2024-05-27 12:49:58
-->

# How to stream PostgreSQL CDC to Kafka and use Propel to get an instant API

> 来源：[https://www.propeldata.com/blog/postgresql-cdc-to-kafka](https://www.propeldata.com/blog/postgresql-cdc-to-kafka)

Learn how to stream PostgreSQL Change Data Capture (CDC) to a Kafka topic and serve the data via an API with Propel. This guide provides step-by-step instructions, from setting up your PostgreSQL database and Kafka broker to deploying the Debezium PostgreSQL connector and creating a Propel Kafka Datapool.

Change Data Capture (CDC) is the process of tracking and capturing changes to your data. In PostgreSQL, we can implement CDC efficiently using the transaction log so that, rather than running batch jobs to gather data, we can capture data changes as they occur continuously. This has numerous benefits, including allowing your applications to see and react to changes in near real-time with lower impact on your source systems. In this article, we'll walk you step-by-step through how to stream CDC from PostgreSQL to a [Propel Kafka Data Pool](https://www.propeldata.com/docs/connect-your-data/kafka) to get an instant data-serving API.

## What are we trying to do?

We want to power large-scale analytics applications with data coming from PostgreSQL. As we all know, PostgreSQL, as an OLTP database, is notoriously slow in handling analytical queries that process and aggregate large amounts of data.

*   First, we want to capture all changes to our PostgreSQL database table(s) and send them as JSON messages (see below) to a [Kafka](https://kafka.apache.org/) topic.
*   Second, we’ll need to configure  [Kafka Connect](https://docs.confluent.io/platform/current/connect/userguide.html#connect-userguide) and the [Debezium PostgreSQL connector](https://debezium.io/documentation/reference/2.6/connectors/postgresql.html) to stream the CDC to a Kafka topic.
*   Lastly, create a Propel Kafka Data Pool that ingests data from the Kafka topic and exposes it via a low latency API.

We will cover how to deploy how to create and deploy all services in a Kubernetes cluster.

## How PostgreSQL CDC with Debezium works

In this section, we will provide a quick overview of how PostgreSQL change data capture works with Debezium.

Imagine you have a simple table:

```
CREATE TABLE "simple" (
	id uuid NOT NULL PRIMARY KEY,
	foo character varying,
);
```

You update a row with the following SQL statement:

```
UPDATE public.simple SET foo = 'baz' WHERE ID = 'b18df779-0cae-43bc-9f4b-8efe571abd8a';
```

The change to the column <span class="code-exp">foo</span> will result in an “update” to the transaction log, which Debezium then transforms into the JSON message below and drops into a Kafka topic.

CDC JSON with <span class="code-exp">before</span> and <span class="code-exp">after</span>:

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

The Debezium PostgreSQL connector generates a data change event for each row-level <span class="code-exp">INSERT</span>, <span class="code-exp">UPDATE</span>, and <span class="code-exp">DELETE</span> operation. The <span class="code-exp">op</span> key describes the operation that caused the connector to generate the event. In this example, u indicates that the operation updated a row. Valid values are:

*   <span class="code-exp">c</span> = create
*   <span class="code-exp">u</span> = update
*   <span class="code-exp">d</span> = delete
*   <span class="code-exp">r</span> = read (applies to only snapshots)
*   <span class="code-exp">t</span> = truncate
*   <span class="code-exp">m</span> = message

## What is the Propel Kafka Datapool?

The Kafka [Data Pool](https://www.propeldata.com/docs/connect-your-data#key-concept-2-data-pools) lets you ingest real-time streaming data into Propel. It provides an instant low-latency API on top of Kafka to power real-time dashboards, streaming analytics, and workflows.

Consider using Propel on top of Kafka when:

*   You need an API on top of a Kafka topic.
*   You need to power real-time analytics applications with streaming data from Kafka.
*   You need to ingest Kafka messages into ClickHouse.
*   You need to ingest from [self-hosted Kafka](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=self-hosted-kafka), [Confluent Cloud](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=confluent), [AWS MSK](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=aws-msk), or [Redpanda](https://www.propeldata.com/docs/connect-your-data/kafka/guides/how-to-ingest-data-from-Kafka-into-propel?kafka-platform=redpanda) into ClickHouse.
*   You need to transform or enrich your streaming data.
*   You need to power real-time personalization and recommendations for use cases.

Once we have the PostgreSQL, Debezium, and Kafka set up, we’ll ingest the data into a Propel Kafka Data Pool to expose it via an API.

## Setting up Minikube, PostgreSQL, Kafka, Kafka Connect, and Debezium

We will walk through setting this up in a Kubernetes cluster on a Mac. This can be done in other environments, but Kubernetes is a common platform for hosting and running data streaming services in the cloud.  For this example, we will use [Minikube](https://minikube.sigs.k8s.io/docs/start/) to deploy our Kubernetes cluster, [Redpanda](https://redpanda.com/) to deploy our Kafka cluster, and [Helm](https://helm.sh/) to manage Kubernetes applications like PostgreSQL.  I’ll provide the Mac-specific commands but links to the installed services if you are running a different OS.

### Setting up Minikube

**Step 1:** **Install** [**Docker Desktop**](https://www.docker.com/products/docker-desktop/)

```
brew install --cask docker
```

**Step 2:** **Install** [**Kubectl**](https://kubernetes.io/docs/tasks/tools/)

**Step 3: Install minikube**

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

**Step 4:** **Start minikube**

```
minikube start --nodes 8 #docker-desktop will need to be running
```

**Step 5: Interact with your cluster**

```
minikube kubectl -- get pods -A
```

**Step 6:** **Install helm**

### Setting up PostgreSQL

**Step 1: Install the PostgreSQL Helm Chart with extended configuration**

Extend the configuration to set the <span class="code-exp">wal_level</span>

```
cat > pg-values.yaml <<'EOF'
primary:
  extendedConfiguration: |-
    wal_level = logical
    shared_buffers = 256MB
EOF

helm install blog oci://registry-1.docker.io/bitnamicharts/postgresql -f pg-values.yaml 
```

**Step 2:** **Connect to PostgreSQL using the PostgreSQL CLI Create the database**

```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default blog-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

kubectl run blog-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.2.0-debian-12-r6 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host blog-postgresql -U postgres -d postgres -p 5432
```

**Step 3: Create the table**

```
# after connecting to the PostgreSQL instance, create the table      
CREATE TABLE simple (
id uuid NOT NULL PRIMARY KEY,
foo character varying);

ALTER TABLE public.simple REPLICA IDENTITY FULL;
```

**Step 4: Create a PostgreSQL user**

To create a PostgreSQL user with the necessary privileges for Debezium to stream changes from the PostgreSQL source tables, run this statement with superuser privileges:

```
CREATE USER blog WITH REPLICATION PASSWORD 'your_password' LOGIN;
```

### Setting up Kafka (Redpanda)

**Step 1: Add the helm repo and install dependencies**

```
helm repo add jetstack https://charts.jetstack.io
helm repo add redpanda https://charts.redpanda.com
helm repo update
helm install cert-manager jetstack/cert-manager  --set installCRDs=true --namespace cert-manager  --create-namespace
```

**Step 2: Configure and install Redpanda**

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

**Step 3: Install the Redpanda client <span class="code-exp">rpk</span>**

```
brew install redpanda-data/tap/redpanda
```

**Step 4:** **Create an alias to simplify the <span class="code-exp">rpk</span> commands**

```
alias local-rpk="kubectl --namespace redpanda exec -i -t redpanda-0 -c redpanda -- rpk -X user=superuser -X pass=changethispassword -X sasl.mechanism=SCRAM-SHA-512"
```

**Step 5: Describe the Redpanda Cluster**

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

**Step 6: Create a user**

```
local-rpk acl user create blog -p changethispassword --mechanism scram-sha-256
```

**Step 7: Create an** [**ACL**](https://docs.redpanda.com/current/manage/security/authorization/#create-acls) **for the user <span class="code-exp">blog</span>**

```
local-rpk acl create --allow-principal User:blog \
  --operation all  --topic '*' --group '*'
```

### Setting up Kafka Connect

**Step 1:  Create a namespace**

```
kubectl create namespace kafka-connect
```

**Step 2: Create the Service**

Below is the manifest you’ll need to apply to set up the Kafka Connect service with your Redpanda broker (see above).  Copy the manifest below and provide the needed values to configure the service for your environment. In the example below, we are connecting with <span class="code-exp">SASL_PLAINTEXT</span> and <span class="code-exp">SCRAM-SHA-256</span>, ensure that <span class="code-exp">CONNECT_SASL_JAAS_CONFIG</span> is using <span class="code-exp">org.apache.kafka.common.security.scram.ScramLoginModule</span> and that you have set the username and password for both the <span class="code-exp">CONNECT_</span> and <span class="code-exp">PRODUCER_</span> keys.

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

Use <span class="code-exp">kubectl</span> to install the manifest and start the Kafka Connect pod in your cluster.

```
kubectl apply -n kafka-connect -f kafka-connect-pod.yaml
```

After you apply the manifest, you should get the following output.

```
deployment.apps/kafka-connect configured
```

Check the pod log files to ensure the service can log in to the broker. You should see the following in the logs if everything has been configured correctly.

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

Check that the <span class="code-exp">connect-offsets</span> topic has been created on the Redpanda broker.

```
local-rpk topic list

NAME             PARTITIONS  REPLICAS
_schemas         1           1
connect-configs  1           1
connect-offsets  25          1
connect-status   5           1
```

### Setting up the Debezium PostgreSQL Connector

We must create a topic to store the CDC messages before deploying the Debezium PostgreSQL connector to the Kafka Connect service.   If you take a look at the config below we’ve set the property <span class="code-exp">topic.prefix</span> to “blog” <span class="code-exp">andpublication.autocreate.mode</span> to “filtered” and set the <span class="code-exp">table.include.list</span> to “public.simple”.  This configures the connector to write the CDC messages to a topic named <span class="code-exp">blog.public.simple</span>, use the command below to create the topic, then deploy the connector using the cURL command. You’ll need to provide your environment-specific details for connecting to your PostgreSQL server and the username/password for the Redpanda service you used above.

**Step 1:  **Create a topic in Redpanda

```
local-rpk topic create blog.public.simple
```

Install the Debezium PostgreSQL connector using cURL.  You’ll need to fetch the password for the PostgreSQL user and fill it in below.  We will also set up port forwarding from the Kafka Connect service to our local host.

**Step 2: Get the PostgreSQL password**

```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default blog-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
echo $POSTGRES_PASSWORD
```

**Step 3: Forward the Kafka-Connect service port (8083)**

```
# Run this in a separate terminal to get the Pod name
kubectl get pod -n kafka-connect -o wide
NAME                             READY   STATUS    RESTARTS      AGE   IP             NODE       NOMINATED NODE   READINESS GATES
kafka-connect-5498764f4f-qw5qg   1/1     Running   1 (34m ago)   35m   10.244.0.253   minikube   <none>           <none>

# Run this command to forward the traffic to local host
kubectl port-forward -n kafka-connect pod/kafka-connect-5498764f4f-qw5qg 8083:8083

Forwarding from 127.0.0.1:8083 -> 8083
```

**Step 4: Deploy the Debezium Postgres Connector with cURL**

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

### Insert data and validate the setup

Let’s validate the CDC messages are landing in the topic we created; we’ll insert some data into our PostgreSQL table and then consume a message from the topic.

**Step 1: Connect to the database**

```
kubectl run blog-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.2.0-debian-12-r6 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host blog-postgresql -U postgres -d postgres -p 5432
```

**Step 2: Insert data into the table**

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

**Step 3: Consume the CDC message from the topic**

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

We’ve validated that the Debezium CDC messages with <span class="code-exp">before</span>-and-<span class="code-exp">after</span> values are landing in the topic.  Don’t be concerned if <span class="code-exp">before</span> is not filled out in this example; if you take a look, the “op” is <span class="code-exp">c</span>, which means a new row was created in the table.

## Expose Redpanda to the internet

Deploying all of the services locally using <span class="code-exp">minikube</span> is a great way to do local development and get hands-on experience without paying a cloud provider. However, for us to use Propel and create a Kafka Data Pool, Propel has to connect to our Redpanda topic. There are a number of ways that this can be accomplished. For the sake of simplicity, we are going to install a reverse proxy and create a TCP Tunnel to expose the Redpanda port to the internet.  The reverse proxy we are going to use is called [LocalXpose](https://localxpose.io/).

💡 To use the LocalXpose TCP Tunnel with a [dedicated port](https://localxpose.io/docs/tunnels/tcp), I had to pay $6\.  Other free services also do this, but you can’t control which port the tunnel is on.  Redpanda has a limitation when creating the external NodePort that restricts it to ports <span class="code-exp">30000-32767</span>.

**Step 1: Install LocalXpose and <span class="code-exp">kcat</span>**

[https://localxpose.io/docs](https://localxpose.io/docs)

```
brew install --cask localxpose
brew install kcat
```

**Step 2: Start the TCP Tunnel**

```
loclx tunnel tcp --port 31092 -t 127.0.0.1:9093
```

You can now see the running tunnels. Copy the domain name in <span class="code-exp">From</span>, e.g., <span class="code-exp">us.localx.io</span>. This will be used in step 3 and later when creating the Kafka Data Pool.

**Step 3: Upgrade the Redpanda install and expose an external NodePort**

Save the yaml below to <span class="code-exp">external-dns.yaml</span>, and replace the <span class="code-exp-bracket">domain</span> with the value from step 2.

```
external:
  enabled: true
  type: NodePort
  addresses:
    - <domain>
```

Upgrade Redpanda.

```
helm upgrade --install redpanda redpanda/redpanda --namespace redpanda --create-namespace --values external-dns.yaml --reuse-values --wait
```

**Step 4: Forward the traffic to <span class="code-exp">minikube</span>**

```
kubectl port-forward -n redpanda svc/redpanda-external 9093:9094
```

**Step 5: Validate you can connect externally using <span class="code-exp">kcat</span>**

```
kcat -L -b us.loclx.io:31092 -X sasl.username=blog -X sasl.password=changethispassword -X sasl.mechanism=SCRAM-SHA-256 -X security.protocol=SASL_PLAINTEXT
```

You can now tunnel traffic to Redpanda running in <span class="code-exp">minikube</span>, allowing Propel’s backend to connect and consume the CDC messages from the topic.

## Create a Propel Kafka Data Pool

Login to the [Propel console](https://console.propeldata.com) or create a new account.  Then choose your environment and create a new Kafka Datapool, for this example I’m using the <span class="code-exp">development</span> environment.

Next, choose Add new credentials to configure the connection to our Redpanda cluster. (use the domain and port from the TCP Tunnel that you created above for the Bootstrap Servers)

Click Create and test Credentials

Ensure the topics are visible

Next, choose the topic <span class="code-exp">blog.public.simple</span> to start pulling in the CDC data.

Click Next to choose the primary timestamp.

Give the Data Pool a unique name.

Click Next to start syncing the data from the topic into Propel.

Once you see two records in the Datapool, you can click Preview Data.

## Query the data

To query the data we have in Propel, you’ll need to create an Application. Follow the guide in the [docs](https://www.propeldata.com/docs/applications) to create an application. Once you’ve created an Application, you can get a token, as seen in the image below. We’ll use that token in the subsequent API calls below.

Use the [Data Grid API](https://www.propeldata.com/docs/query-your-data/data-grid) to query the data in the newly created Kafka Data Pool.  Make sure to copy the token you created above and replace <span class="code-exp-bracket">TOKEN</span> with the value.  You will also need to replace <span class="code-exp-bracket">DATAPOOL_ID</span> with the ID of the newly created Kafka Data Pool. Copy the <span class="code-exp">curl</span> command below and execute it from the command line.

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

This will return a Data Grid with headers and rows.

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

### Update the source PostgreSQL database

If we update the PostgreSQL database with a new value for one of the rows, we’ll see that change reflected in the Propel Data Pool. Let’s execute the SQL statement below and re-run the GraphQL query we ran above.

```
# Connec to the database
kubectl run blog-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.2.0-debian-12-r6 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host blog-postgresql -U postgres -d postgres -p 5432

UPDATE public.simple SET foo = 'baz' WHERE id = '9cb52b2a-8ef2-4987-8856-c79a1b2c2f73';
```

When we re-run the GraphQL query, we get the following response.

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

This shows that we have three rows in our Data Pool; it reflects the raw CDC messages that have been sent to the Kafka Data Pool. In order to use the data for analytics, we need to transform it to correctly represent the current state in the source database.

## Create a real-time transformation to unpack the CDC JSON message

Now that we’re getting the stream of CDC messages to Propel and landing them in a Kafka Data Pool, we need to write some SQL to transform the messages based on the “op” type.  The columns we are interested in using are <span class="code-exp">id</span>, <span class="code-exp">_timestamp</span>, and <span class="code-exp">foo</span>.  Our <span class="code-exp">SELECT</span> statement below will query the source table <span class="code-exp">blog-post</span> and extract <span class="code-exp">id</span> and <span class="code-exp">foo</span> from <span class="code-exp">_propel_payload.after</span>.  This will handle both the create and update operations.  In the case of an update operation a duplicate row will be inserted and then de-duped by Propel.  This will ensure that when we set <span class="code-exp">id</span> as the <span class="code-exp">uniqueId</span> column in the MaterializedView only one version of the <span class="code-exp">id</span> exists with the latest value from the source table.

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

You should get a response like the one below.

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

### Query the Materialized View Data Pool

Now that we’ve created the MaterialzedView let’s query the Data Pool that was created along with it.  Replace <span class="code-exp-bracket">token</span> with the token you created earlier, and use the response from the above <span class="code-exp">curl</span> command to get the destination data pool id.  This is available at <span class="code-exp">data.createMaterializedView.materializedView.destination.id</span> in the JSON.  Replace <span class="code-exp">datapool_id</span>with that value and run the <span class="code-exp">curl</span> command from the command line.

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

The response you get should only have two rows and the updated value of <span class="code-exp-bracket">baz</span> for <span class="code-exp">9cb52b2a-8ef2-4987-8856-c79a1b2c2f73</span>.

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

## Visualize the results

Let’s write a small <span class="code-exp">node.js</span> application that combines all of the Propel concepts and renders a table with the results from the MaterializedView.  Instead of providing a token, we’ll use the Propel API to mint a token using the OAuth2 client credentials from the Propel Application we created.  These are found in the Propel console within the Applications section, as seen below.

**Step 1: OAuth2 client credentials**

**Step 2: Set environment variables**

Replace the tokens with the values from the console, then paste and execute the commands.

```
export CLIENT_ID=<CLIENT_ID> \
export CLIENT_SECRET=<CLIENT_SECRET> \
export TOKEN_HOST=https://auth.us-east-2.propeldata.com \
export TOKEN_PATH=/oauth2/token
```

**Step 3: Install the dependencies**

```
npm install simple-oauth2 cli-table
```

**Step 4: Copy the code**‍

Copy the code below and replace <span class="code-exp-bracket">DATAPOOL_ID</span> with the data pool ID for the MaterializedView from above.

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

**Step 5: Run the script**

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

## Wrap up

In wrapping up this post, it's important to recap the key skills and knowledge you've acquired. You have learned how to effectively stream PostgreSQL Change Data Capture (CDC) to a Propel Kafka DataPool, a significant step in managing real-time data.

Further, you've gained the ability to query this data, which opens up a wealth of opportunities for analysis and insights. This guide also walked you through the process of updating the source PostgreSQL database, ensuring that you can maintain the accuracy and relevance of your data.

One of the highlights of this guide was teaching you how to create a real-time transformation to unpack the CDC JSON message. This is a fundamental skill that elevates your data management capabilities, enabling you to manipulate and interpret data for various applications.

Finally, you've learned how to visualize the results. Visualization plays a crucial role in data analysis, making complex data more understandable, revealing trends and outliers, and contributing to more effective communication.

We hope you found the content both informative and practical and that you enjoyed the learning process. Propel is dedicated to empowering you with the tools and knowledge to transform your data into valuable insights. We look forward to supporting you in your future endeavors with Propel. Keep exploring and transforming your data!