- export lonode cluster config to local **KUBECONFIG**
  `export KUBECONFIG=kubeconfig/kubedb-dashboard-kubeconfig.yaml`

install **kubedb**
```
helm repo update
helm install kubedb appscode/kubedb \  
              --version v2022.02.22 \
              --namespace kubedb --create-namespace \
              --set kubedb-provisioner.enabled=true \
              --set kubedb-dashboard.enabled=true \
              --set-file global.license=/path/to/licence/file

```

create namespace demo
`kubectl create ns demo`


deploy elasticsearch 
`kubectl apply -f elasticsearch_topology.yaml `

deploy dashboard
`kubectl apply -f dashboard_sample.yaml`

port forward dashboard governing service
`kubectl port-forward -n demo service/ed-sample 5601`

get elasticsearch credentials and login to kibana from chrome
```
kubectl get secret -n demo es-topology-elastic-cred -o jsonpath='{.data.username}' | base64 -d
kubectl get secret -n demo es-topology-elastic-cred -o jsonpath='{.data.password}' | base64 -d
```

set ilm policy  (name it k8s-log)
set Hot Phase 
- Maximum primary shard size to 1 MB
- Maximum age to 1 minutes
- Maximum documents to 1000
set Warm Phase
- Move data into phase when: 1 minutes old
set Cold Phase
- Move data into phase when: 2/3 minutes old

set elastic search credential in logstash yaml

deploy logstash
`kubectl apply -f logstash.yaml `

deploy filebeat
`kubectl apply -f filebeat.yaml`

go to kibana > stack management > index patterns
set k8s-log* as index pattern
go to discover and show logs, show the index
go to home > dev tools

test the ILM , hot-warm-cold node transition in dev tools
```
GET _ilm/policy/k8s-log

GET _cat/indices/k8s-log

GET k8s-log/_ilm/explain

DELETE /k8s-log-000001

PUT _cluster/settings
{
  "transient": {
    "indices": {
      "lifecycle": {
        "poll_interval": "10s"
      }
    }
  }
}

```
