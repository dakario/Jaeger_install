# Jaeger Install

 We will at first install the operators and the CRD before customing and deploying every single component.

## Install Elasticsearch

For Elasticsearch, we will install the operator, the CRD and the RABC by running the following commands:

```
kubectl apply -f https://download.elastic.co/downloads/eck/1.0.0/all-in-one.yaml
```

Now we will create an Elasticsearch with 6 nodes (3) Master nodes and 3 Data nodes:

```
kubectl create namespace observability
Kubectl apply -f elasticsearch/elastic.yaml
```

## Install Kafka

Now we can start installing kafka by creating a dedicated namespace:

```
kubectl create namespace kafka
```

Next we apply the installation file, including ClusterRoles, ClusterRoleBindings and Custom Resource Definition (CRD).

```
curl -L https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.16.2/strimzi-cluster-operator-0.16.2.yaml \
  | sed 's/namespace: .*/namespace: kafka/' \
  | kubectl apply -f - -n kafka 
```

Once the operator and CRD created, we will create the kafka custom ressource :

```
Kubectl apply -f kafka/kafka.yaml
```
***Notice: Be sure, by checking the logs, that kafka and the elasticsearch cluster are UP & Running before starting the Jaeger installation with the next steps*** 

## Install Jaeger

First we will install the operator and the CRD by running :

```
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
```

Before deploying Jaeger we have to get the password from Elasticsearch secret and create a new secret for Jaeger with username and password.

```
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode)

kubectl create secret generic jaeger-secret --from-literal=ES_PASSWORD=${PASSWORD} --from-literal=ES_USERNAME=elastic
```

Now we can deploy Jaeger :

```
Kubectl apply -f jaeger/jaeger.yaml
```

We can notice on the ***jaeger.yaml*** that we identified the Kafka configuration used by the collector, to produce the messages, and the ingester to consume the messages.
