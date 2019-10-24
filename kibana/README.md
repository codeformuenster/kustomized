# Kibana

https://github.com/elastic/helm-charts/releases


```bash
# update
rm -r ./helm/kibana
helm fetch \
  --repo https://helm.elastic.co \
  --untar \
  --untardir ./helm \
  --version 7.4.0 \
    kibana

# build
rm -r ./helm/output ; mkdir ./helm/output
helm template \
  --name kibana \
  --namespace kibana \
  --values ./helm/values.yaml \
  --output-dir ./helm/output \
  ./helm/kibana

# check
kubectl kustomize .

# apply
kubectl create namespace elasticsearch-test
kubectl apply -k .


kubectl -n elasticsearch-test get pods

kubectl port-forward -n elasticsearch-test service/kibana-kibana 5601
xdg-open http://localhost:5601


# curl localhost:9200
# curl localhost:9200/_cat
# curl localhost:9200/_cat/health
```

## FIXME

- add https://opendistro.github.io/for-elasticsearch-docs/docs/kibana/plugins/

```bash
sudo docker run -ti docker.elastic.co/kibana/kibana-oss:7.2.0 bash

kibana-plugin list

kibana-plugin install \
  https://d3g5vo6xdbdb9a.cloudfront.net/downloads/kibana-plugins/opendistro-security/opendistro_security_kibana_plugin-1.2.0.0.zip

kibana-plugin install \
  https://d3g5vo6xdbdb9a.cloudfront.net/downloads/kibana-plugins/opendistro-alerting/opendistro-alerting-1.2.0.0.zip

kibana-plugin list
```

