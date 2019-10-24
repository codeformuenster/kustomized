# kube-prometheus

https://github.com/coreos/kube-prometheus#customizing-kube-prometheus

```bash
# get jb
curl -OL https://github.com/jsonnet-bundler/jsonnet-bundler/releases/download/v0.1.0/jb-linux-amd64
chmod u+x jb-linux-amd64
sudo mv jb-linux-amd64 /usr/local/bin/jb

# sudo docker run -v $PWD:/go/bin golang:1.13 \
#   sh -c "go get github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb"
# sudo mv ./jb /usr/local/bin/

# get gojsontoyaml
# sudo docker run -v $PWD:/go/bin golang:1.13 \
#   sh -c "go get github.com/brancz/gojsontoyaml"
# sudo mv ./gojsontoyaml /usr/local/bin/

# get jsonnet
curl -L https://github.com/google/jsonnet/releases/download/v0.14.0/jsonnet-bin-v0.14.0-linux.tar.gz \
  | tar -zx
sudo mv jsonnet jsonnetfmt /usr/local/bin/
```


```bash
cd ./jsonnet
jb init
# jb install github.com/coreos/kube-prometheus/jsonnet/kube-prometheus@release-0.1
jb install github.com/coreos/kube-prometheus/jsonnet/kube-prometheus@26750eadf59279f409de47d616527a9632f8ab23

# or better edit jsonnet/jsonnetfile.json?
jb update
```


## Apply to cluster

**FIXME**
- dont't create manifests locally but apply directly to the cluster
- show how to create locally for debugging purposes only

```bash
rm -r ../../manifests/kube-prometheus ; mkdir -p ../../manifests/kube-prometheus

jsonnet -J ./jsonnet/vendor -m ../../manifests/kube-prometheus main.jsonnet \
  | xargs -I{} sh -c 'cat {} | gojsontoyaml > {}.yaml; rm -f {}' -- {}

kubectl apply -f ../../manifests/kube-prometheus

# or
jsonnet --yaml-stream --jpath ./jsonnet/vendor ./main.jsonnet \
  > manifests-temp.json

kubectl apply -f ./manifests-temp.json

# or
jsonnet --yaml-stream --jpath ./jsonnet/vendor ./main.jsonnet \
  | kubectl apply -f -

# to pick up changes from configmaps/secrets
kubectl -n kube-prometheus rollout restart statefulset/prometheus-k8s
```

## more dashboards

https://github.com/coreos/kube-prometheus/blob/master/examples/grafana-additional-jsonnet-dashboard-example.jsonnet


## blackbox exporter

**FIXME** not yet done

https://github.com/coreos/prometheus-operator/issues/2010
https://github.com/coreos/prometheus-operator/tree/master/example/additional-scrape-configs


```bash
# FIXME
Linux: root user or CAP_NET_RAW capability is required.
    Can be set by executing setcap cap_net_raw+ep blackbox_exporter
```

## FIXME
- add loki to prometheus sources