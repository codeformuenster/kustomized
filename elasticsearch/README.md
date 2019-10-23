# Elasticsearch


https://github.com/elastic/helm-charts/releases
https://github.com/elastic/helm-charts/blob/master/elasticsearch/examples


```bash
# update
rm -r ./helm/elasticsearch
helm fetch \
  --repo https://helm.elastic.co \
  --untar \
  --untardir ./helm \
  --version 7.4.0 \
    elasticsearch

# build
rm -r ./helm/output ; mkdir ./helm/output
helm template \
  --name elasticsearch \
  --namespace elasticsearch \
  --values ./helm/values.yaml \
  --output-dir ./helm/output \
  ./helm/elasticsearch

# edit elasticsearch/helm/output/elasticsearch/templates/statefulset.yaml
#   initContainers: []

# check
kubectl kustomize .

# apply
kubectl create namespace elasticsearch-test
kubectl apply -k .
```



## security-admin

https://opendistro.github.io/for-elasticsearch-docs/docs/security-configuration/security-admin/

```bash
cd /usr/share/elasticsearch

ls ./plugins/opendistro_security/securityconfig

./jdk/bin/java -Dorg.apache.logging.log4j.simplelog.StatusLogger.level=OFF \
  -cp "./plugins/opendistro_security/*:./lib/*" com.amazon.opendistroforelasticsearch.security.tools.OpenDistroSecurityAdmin \
  -cd ./plugins/opendistro_security/securityconfig \
  -icl \
  -nhnv \
  -cacert ./config/root-ca.pem \
  -cert ./config/admin.pem \
  -key ./config/admin-key.pem
```



---
opendistro_security.ssl.transport.keystore_filepath or opendistro_security.ssl.transport.pemkey_filepath must be set if transport ssl is reqested.


http://localhost:9200/_cat/plugins?v
> {"type": "server", "timestamp": "2019-10-16T23:31:58,740+0000", "level": "INFO", "component": "o.e.p.PluginsService", "cluster.name": "elasticsearch", "node.name": "elasticsearch-master-0",  "message": "no plugins loaded"  }


kubectl -n elasticsearch-test get pods

kubectl port-forward -n elasticsearch-test service/elasticsearch-master 9200
curl localhost:9200
curl localhost:9200/_cat
curl localhost:9200/_cat/health

```

---

https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch




opendistro certificates
  https://opendistro.github.io/for-elasticsearch-docs/docs/security-configuration/generate-certificates/


https://opendistro.github.io/for-elasticsearch-docs/docs/security-configuration/generate-certificates/#sample-script

```bash
# Root CA
openssl genrsa -out root-ca-key.pem 2048
openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem
# Admin cert
openssl genrsa -out admin-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
openssl req -new -key admin-key.pem -out admin.csr
openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem
# Node cert
openssl genrsa -out node-key-temp.pem 2048
openssl pkcs8 -inform PEM -outform PEM -in node-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node-key.pem
openssl req -new -key node-key.pem -out node.csr
openssl x509 -req -in node.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node.pem
# Cleanup
rm admin-key-temp.pem
rm admin.csr
rm node-key-temp.pem
rm node.csr
```

L=Boulder,O=ApertureAnalytics,ST=CO,C=US,OU=Engineering,CN=elasticsearch

subject=CN=elasticsearch,O=Code for Muenster,L=Muenster,ST=NRW,C=DE

Country Name (2 letter code) [AU]:DE
State or Province Name (full name) [Some-State]:NRW
Locality Name (eg, city) []:Muenster
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Code for Muenster
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:elasticsearch
Email Address []:






openssl x509 -subject -nameopt RFC2253 -noout -in node.pem

subject=C = DE, ST = NRW, L = M\C3\83\C2\BCnster, O = Code for M\C3\83\C2\BCnster, OU = Elasticsearch, CN = Elasticsearch
subject=O=Code for M\C3\83\C2\BCnster,L=M\C3\83\C2\BCnster,ST=NRW,C=DE




      - ./root-ca.pem:/usr/share/elasticsearch/config/root-ca.pem
      - ./node.pem:/usr/share/elasticsearch/config/node.pem
      - ./node-key.pem:/usr/share/elasticsearch/config/node-key.pem
      - ./admin.pem:/usr/share/elasticsearch/config/admin.pem
      - ./admin-key.pem:/usr/share/elasticsearch/config/admin-key.pem

      - ./custom-elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml

      - ./internal_users.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/internal_users.yml
      - ./roles_mapping.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/roles_mapping.yml
      - ./tenants.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/tenants.yml
      - ./roles.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/roles.yml
      - ./action_groups.yml:/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/action_groups.yml






















## FIXME

https://cert-manager.readthedocs.io/en/latest/tasks/issuers/setup-ca.html
https://blog.r3t.io/secure-logging-for-kubernetes/


```bash

# generate via kustomize?
kubectl create secret tls ca-key-pair \
   --cert=ca.crt \
   --key=ca.key \
   --namespace=elasticsearch-test


create
  kind: Issuer

create
  kind: Certificate
    https://github.com/jetstack/cert-manager/blob/master/deploy/manifests/00-crds.yaml#L1834

    secretName: elasticsearch-internal-tls
    issuerRef:

    # commonName: elasticsearch
    organization:
    - "Code for Münster"
    dnsNames:
    - elasticsearch-master-0.elasticsearch-test.svc
    - elasticsearch-master-1.elasticsearch-test.svc
    - elasticsearch-master-2.elasticsearch-test.svc
    # - elasticsearch-master-2.elasticsearch-test.svc.cluster.local ?
    # - *.elasticsearch-test.svc.cluster.local ?
    usages:
    - server auth
    - client auth



elasticsearch statefulset
  volumes
        - name: elasticsearch-tls-ca
          secret:
            secretName: elasticsearch-tls-ca
            items:
              - key: ca.pem
                path: ca.pem
        - name: elasticsearch-tls-cert
          secret:
            secretName: elasticsearch-tls
            items:
              - key: tls.crt
                path: tls.crt
        - name: elasticsearch-tls-key
          secret:
            secretName: elasticsearch-tls
            items:
              - key: tls.key
                path: tls.key

  volumeMounts
    - name: elasticsearch-tls-ca
      mountPath: /usr/share/elasticsearch/config/ca.pem
      subPath: ca.pem
      readOnly: true
    - name: elasticsearch-tls-cert
      mountPath: /usr/share/elasticsearch/config/cert.pem
      subPath: tls.crt
      readOnly: true
    - name: elasticsearch-tls-key
      mountPath: /usr/share/elasticsearch/config/key.pem
      subPath: tls.key
      readOnly: true

  port?
            - containerPort: 9600
              name: performance
              protocol: TCP



configmap

  opendistro_security.allow_default_init_securityindex: true
  opendistro_security.audit.type: internal_elasticsearch
  opendistro_security.check_snapshot_restore_write_privileges: true
  opendistro_security.enable_snapshot_restore_privilege: true
  opendistro_security.nodes_dn:
    - "L=Boulder,O=ApertureAnalytics,ST=CO,C=US,OU=Engineering,CN=elasticsearch"
  opendistro_security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
  opendistro_security.ssl.http.enabled: true
  opendistro_security.ssl.http.pemcert_filepath: cert.pem
  opendistro_security.ssl.http.pemkey_filepath: key.pem
  opendistro_security.ssl.http.pemtrustedcas_filepath: ca.pem
  opendistro_security.ssl.transport.enabled: true
  opendistro_security.ssl.transport.enforce_hostname_verification: false
  opendistro_security.ssl.transport.pemcert_filepath: cert.pem
  opendistro_security.ssl.transport.pemkey_filepath: key.pem
  opendistro_security.ssl.transport.pemtrustedcas_filepath: ca.pem



```