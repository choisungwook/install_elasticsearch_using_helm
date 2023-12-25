# 개요
* helm으로 EFK 설치

<br>

# 사전 지식
* Elasticsearch, kibana는 7.10.2까지 무료. 그 이후 버전은 라이센스 구매해야 함
  * 라이센스를 구매 못하는 상황이면, Elasticsearch말고 Loki로 대체
* 2023년 5월? 이후 부터 ECK operator로 Elasticsearch 설치하는 것으로 표준으로 됨
* helm chart 7.10.2버전은 비밀번호 설정이 없어서 별도로 해야 함
  * 참고자료: https://github.com/elastic/helm-charts/tree/main/elasticsearch/examples/security

<br>

# 준비
* helm
* Elasticsearch helm chart 저장소 추가

```bash
helm repo add elastic https://helm.elastic.co
helm repo add fluent https://fluent.github.io/helm-charts
```

<br>

# ElasticSearch 설치
* logging namespace에 설치

```bash
helm upgrade --install elasticsearch \
  --version 7.10.2 \
  -n logging --create-namespace \
  -f elasticsearch_values.yaml \
  elastic/elasticsearch
```

* 조회

```bash
$ kubectl get po,svc -n logging
NAME                         READY   STATUS    RESTARTS   AGE
pod/elasticsearch-master-0   1/1     Running   0          2m46s

NAME                                    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                         AGE
service/elasticsearch-master            NodePort    10.96.78.7   <none>        9200:30920/TCP,9300:30175/TCP   2m46s
service/elasticsearch-master-headless   ClusterIP   None         <none>        9200/TCP,9300/TCP               2m46s
```

<br>

# kibana 설치 방법
* logging namespace에 설치

```bash
helm upgrade --install kibana \
  --version 7.10.2 \
  -n logging --create-namespace \
  -f kibana_values.yaml \
  elastic/kibana
```

* 조회

```bash
$ kubectl get po,svc -n logging
NAME                                 READY   STATUS              RESTARTS   AGE
pod/elasticsearch-master-0           1/1     Running             0          4m
pod/kibana-kibana-7dd48d7448-vcc2b   1/1     Running             0          2m

NAME                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
service/elasticsearch-master            NodePort    10.96.172.97    <none>        9200:30920/TCP,9300:30466/TCP   4m
service/elasticsearch-master-headless   ClusterIP   None            <none>        9200/TCP,9300/TCP               4m
service/kibana-kibana                   NodePort    10.96.215.232   <none>        5601:30561/TCP                  2m
```

# 삭제 방법

```bash
# elasticsearch 삭제
helm uninstall elasticsearch -n logging
# kibana 삭제
helm uninstall kibana -n logging
```

# flunetd 설치 방법
* logging namespace에 설치

```bash
helm upgrade --install fluentd \
  --version 0.3.2 \
  -n logging --create-namespace \
  --set podSecurityPolicy.enabled=false \
  fluent/fluentd
```

* 조회

```bash
$ kubectl get pod -n logging
NAME                             READY   STATUS              RESTARTS   AGE
elasticsearch-master-0           1/1     Running             0          15m
fluentd-jzdxj                    0/1     ContainerCreating   0          23s
fluentd-pqxtk                    0/1     ContainerCreating   0          23s
fluentd-x5twf                    0/1     ContainerCreating   0          23s
kibana-kibana-7dd48d7448-vcc2b   1/1     Running             0          11m
```


# 참고자료
* https://github.com/elastic/helm-charts?tab=readme-ov-file
