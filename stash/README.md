# Stash


https://github.com/stashed/installer

https://appscode.com/products/stash/v0.9.0-rc.0/concepts/crds/backupconfiguration/

## Installation
  https://github.com/stashed/installer/blob/master/deploy/stash.sh

- waiting until stash operator deployment is ready
- waiting until stash apiservice is available
- waiting until stash crds are ready
  (who installs those?)


## FIXME

- https://github.com/stashed/stash/tree/master/contrib/monitoring

https://github.com/appscode/charts/tree/master/stable/stash


helm fetch \
  --repo https://charts.appscode.com/stable \
  --untar \
  --untardir ./helm \
  --version v0.9.0-rc.0 \
    stash

helm template \
  --name stash \
  --namespace stash \
  --values ./helm/values.yaml \
  --output-dir ./helm/output \
  ./helm/stash



---
https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/#download-and-install-cfssl

kubectl get csr
kubectl certificate approve csr-6khkt



---
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: stash-cleaner
  labels:
    app: "stash"
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-delete-policy": hook-succeeded,hook-failed
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 120
  template:
    spec:
      serviceAccountName: stash
      containers:
      - name: busybox
        image: appscode/kubectl:v1.12
        command:
          - sh
          - -c
          - "sleep 2; \
           kubectl delete validatingwebhookconfigurations admission.stash.appscode.com || true; \
           kubectl delete mutatingwebhookconfiguration admission.stash.appscode.com || true; \
           kubectl delete functions.stash.appscode.com update-status pvc-backup pvc-restore || true; \
           kubectl delete tasks.stash.appscode.com pvc-backup pvc-restore"
        imagePullPolicy: IfNotPresent
      restartPolicy: Never
```
