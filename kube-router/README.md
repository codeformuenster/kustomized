# kube-router

https://www.kube-router.io/

For features see https://github.com/cloudnativelabs/kube-router/blob/master/docs/user-guide.md#advertising-ips


```bash
# update from upstream
rm ./kubeadm-kuberouter-all-features-hostport.yaml

curl -sfSLO https://raw.githubusercontent.com/cloudnativelabs/kube-router/v0.3.1/daemonset/kubeadm-kuberouter-all-features-hostport.yaml


# build manifests to verify
kustomize build .

# apply to cluster
kubectl apply -k .
```
