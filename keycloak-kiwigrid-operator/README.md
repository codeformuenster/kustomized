# keycloak0operator

[WIP] This is meant as a base to kustomize.


```bash
# build manifests to verify
kubectl kustomize

# apply to cluster
kubectl create namespace keycloak-test1
kubectl apply -k .


kubectl api-resources --api-group k8s.kiwigrid.com
kubectl get -A keycloaks.k8s.kiwigrid.com

```



---

## errors


{
  "timestampSeconds": 1565119462,
  "timestampNanos": 497000000,
  "severity": "WARN",
  "thread": "pool-1-thread-1",
  "logger": "com.kiwigrid.keycloak.controller.keycloak.KeycloakController",
  "message": "Connecting to keycloak-test1 failed: Secret keycloak-test1/keycloak-http not found.",
  "context": "default",
  "serviceContext": {
    "version": "1.0",
    "service": "keycloak-controller"
  }
}


kubectl -n keycloak-test1 create secret generic keycloak-http \
  --from-literal=password=(openssl rand -base64 32 | tr -d "\n" | base64) --dry-run -o yaml


{
  "timestampSeconds": 1565120842,
  "timestampNanos": 753000000,
  "severity": "WARN",
  "thread": "pool-1-thread-1",
  "logger": "com.kiwigrid.keycloak.controller.keycloak.KeycloakController",
  "message": "Connecting to keycloak-test1 failed: Unable to connect: RESTEASY004655: Unable to invoke request",
  "context": "default",
  "serviceContext": {
    "version": "1.0",
    "service": "keycloak-controller"
  }
}

---

apiVersion: k8s.kiwigrid.com/v1beta1
kind: Keycloak
metadata:
  name: keycloak-test1
spec:
  clientId: admin-cli
  passwordSecretKey: password
  passwordSecretName: keycloak-http
  passwordSecretNamespace: keycloak-test1
  realm: master
  url: https://keycloak-test1.mydomain.org/auth
  username: admin
