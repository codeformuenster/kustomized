https://github.com/kiwigrid/helm-charts/tree/master/charts/fluentd-elasticsearch



helm repo add kiwigrid https://kiwigrid.github.io
helm install kiwigrid/fluentd-elasticsearch


```bash
# update
helm fetch \
  --repo https://kiwigrid.github.io \
  --untar \
  --untardir ./helm \
  --version 4.9.1 \
    fluentd-elasticsearch

# build
rm -r ./helm/output ; mkdir ./helm/output
helm template \
  --name fluentd-elasticsearch \
  --namespace fluentd-elasticsearch \
  --values ./helm/values.yaml \
  --output-dir ./helm/output \
  ./helm/fluentd-elasticsearch



```