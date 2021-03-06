# ingress-nginx

*FIXME*
- Use json for logs https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/configmap.md#log-format-upstream
  - Adjust Loki
  - create metrics for TLS versions that connect, see next
- Add TLSv3 https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/configmap.md#ssl-protocols
- maybe use-proxy-protocol? https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/configmap.md#use-proxy-protocol
- use-geoip2?

https://github.com/kubernetes/ingress-nginx/pull/4055
and https://kubectl.docs.kubernetes.io/pages/app_management/secrets_and_configmaps.html
check:
```yaml
configMapGenerator:
- name: nginx-configuration
  behavior: merge
  literals:
  - use-http2="false"
  - ssl-protocols="TLSv1 TLSv1.1 TLSv1.2"
  - worker-shutdown-timeout="13s"
  # etc
```
also in general:
```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
configMapGenerator:
- name: my-application-properties
  files:
  - application.properties
```


```bash
# refresh from upstream
rm -r ./cloud-generic ./cluster-wide ./grafana

curl -L https://github.com/kubernetes/ingress-nginx/archive/master.tar.gz \
  | tar -xz --strip-components=2 \
    ingress-nginx-master/deploy/cloud-generic \
    ingress-nginx-master/deploy/cluster-wide \
    ingress-nginx-master/deploy/grafana/dashboards

curl -L https://github.com/kubernetes/ingress-nginx/archive/nginx-0.25.1.tar.gz \
  | tar -xz --strip-components=2 \
    ingress-nginx-nginx-0.25.1/deploy/cloud-generic \
    ingress-nginx-nginx-0.25.1/deploy/cluster-wide \
    ingress-nginx-nginx-0.25.1/deploy/grafana/dashboards

# place manifests in local directory
kubectl kustomize .

kubectl create namespace ingress-nginx
kubectl apply -k .
```

In Grafana import dashboard https://grafana.com/dashboards/9614

## FIXME
- set [Pod Disruption Budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
- use `hostPort` instead of hostNetwork https://kubernetes.io/docs/concepts/configuration/overview/#services