# ArgoCD Clusters

## Structure of this repo

```
apps/
    cluster_1/
    cluster_2/
apps-of-apps/
    aoa-cluster-1.yaml
    aoa-cluster-2.yaml
secrets/
    common/
        secret-cred.yaml
    cluster_2/
        secret-aws.yaml
static/
    cluster_1/
        some-config.yaml
the-apps-of-apps.yaml
```
