# OLS4 backend deployment

## Install the backend (Windows PowerShell):

### Secrets
```
kubectl -n zbmed-ts-health create secret generic ols4-backend-auth-token --from-literal=MATOMO_AUTH_TOKEN={have a look into the keypass-database}
```

### QA
```
helm install zbmed-ts-health-ols4 `
  --namespace="zbmed-ts-health"
  --set-string ingress.dns="ols4-health.qa.km.k8s.zbmed.de" `
  --set-string ingress.path="/ols4" `
  --set-string backend.backendImage="ghcr.io/zbmed/ols4-backend:969e2e6ad13c19a12f09a2fd10b398cc4dcd977d" `
  --set-string solrImage="ghcr.io/ebispot/ols4-solr:9.8.1" `
  --set-string neo4jImage="ghcr.io/ebispot/ols4-neo4j:2025.03.0-community" `
  --set-string backend.context="/ols4" `
  --set-string neo4jTarballUrl="http://ols4-dataserver/mesh-test_neo4j.tgz" `
  --set-string solrTarballUrl="http://ols4-dataserver/mesh-test_solr.tgz" `
  --set-string partOfLabel="zbmed-ts-health"
ols4-backend-deployment/ols4-backend
```

Test instance
```
helm install zbmed-ts-health-ols4-test `
    --namespace="zbmed-ts-health" `
    --set-string ingress.dns="ols4-health-test.qa.km.k8s.zbmed.de"  `
    --set-string ingress.path="/olstest"  `
    --set-string backend.backendImage="ghcr.io/zbmed/ols4-backend:3f3def560da06322bcd0c01a88ef397c244f6219"  `
    --set-string solrImage="ghcr.io/ebispot/ols4-solr:9.8.1"  `
    --set-string neo4jImage="ghcr.io/ebispot/ols4-neo4j:2025.03.0-community"  `
    --set-string backend.context="/olstest"  `
    --set-string neo4jTarballUrl="http://zbmed-ts-health-ols4-dataserver/health-may26_neo4j.tgz"  `
    --set-string solrTarballUrl="http://zbmed-ts-health-ols4-dataserver/health-may26_solr.tgz"  `
IdeaProjects/ols4-backend-deployment/k8s/ols4-backend
```

### PROD
Add this for enabling certification on prod cluster:
```
--set-string ingress.enableSSL="true"  `
--set-string ingress.certIssuer="letsencrypt-prod"  `
```

### Install the dataserver and download data
```
helm install zbmed-ts-health-ols4-dataserver-blue `
    --namespace="zbmed-ts-health" `
    --set-string partOfLabel="zbmed-ts-health" `
IdeaProjects/ols4-backend-deployment/k8s/dataserver
```

#### Install s3cmd
```
kubectl exec -it -n <namespace> ols4-dataserver-XXXXX -- /bin/bash
apt update
apt install -y s3cmd
s3cmd --version
```

#### Download data into the dataserver

```
S3_KEYID=abc
S3_KEY=def
S3_HOST=bla
S3_BUCKET=semlookp-data

s3cmd ls s3://${S3_BUCKET}/ --host=${S3_HOST} --access_key=${S3_KEYID} --secret_key=${S3_KEY} --host-bucket=${S3_BUCKET}.${S3_HOST}
s3cmd get s3://${S3_BUCKET}/health-test_neo4j.tgz /usr/share/nginx/html/health-may26_neo4j.tgz --host=${S3_HOST} --access_key=${S3_KEYID} --secret_key=${S3_KEY} --host-bucket=${S3_BUCKET}.${S3_HOST}
```

### Production
#### HEALTH

```
helm upgrade zbmed-ts-health-ols4-green `
    --namespace="zbmed-ts-health" `
    --set-string ingress.dns="ols4-nfdi4health-green.prod.km.k8s.zbmed.de"  `
    --set-string ingress.path="/olsgreen"  `
    --set-string backend.backendImage="ghcr.io/zbmed/ols4-backend:d3ff2c1fbbb5d23136c4a4db98c54c82f6b86d6b"  `
    --set-string solrImage="ghcr.io/ebispot/ols4-solr:9.8.1"  `
    --set-string neo4jImage="ghcr.io/ebispot/ols4-neo4j:2025.03.0-community"  `
    --set-string backend.context="/olsgreen"  `
    --set-string neo4jTarballUrl="http://zbmed-ts-health-ols4-dataserver/health-may26_neo4j.tgz"  `
    --set-string solrTarballUrl="http://zbmed-ts-health-ols4-dataserver/health-may26_solr.tgz"  `
    --set-string ingress.enableSSL="true"  `
    --set-string ingress.certIssuer="letsencrypt-prod"  `
IdeaProjects/ols4-backend-deployment/k8s/ols4-backend
```


-------------------

Deployment Chart for old ols4 version:
```
ols4-backend-deployment/ols4-backend --version 0.1.8
```
```
helm install zbmed-ts-health-ols4-blue `
    --namespace="zbmed-ts-health" `
    --set-string ingress.dns="ols4-nfdi4health-blue.prod.km.k8s.zbmed.de"  `
    --set-string ingress.path="/olsblue"  `
    --set-string backend.backendImage="ghcr.io/zbmed/ols4-backend:ee8c112fab85da653e28db0779bb9f6ea8ab9360"  `
    --set-string solrImage="ghcr.io/ebispot/ols4-solr:9.0.0"  `
    --set-string neo4jImage="ghcr.io/ebispot/ols4-neo4j:4.4.9-community"  `
    --set-string backend.context="/olsblue"  `
    --set-string neo4jTarballUrl="http://zbmed-ts-health-ols4-dataserver-blue/health-sep25_neo4j.tgz"  `
    --set-string solrTarballUrl="http://zbmed-ts-health-ols4-dataserver-blue/health-sep25_solr.tgz"  `
    --set-string ingress.enableSSL="true"  `
    --set-string ingress.certIssuer="letsencrypt-prod"  `
IdeaProjects/ols4-backend-deployment/k8s/ols4-backend-deprecated 
```
