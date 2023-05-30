# Inigo Apollo Router Kubernetes Demo

1. Inigo Rust Apollo Router (https://docs.inigo.io/deployment/rust_apollo_router) (`docker pull inigohub/inigo_apollo_router`)
1. Apollo Router Helm Chart (https://github.com/apollographql/router/tree/v1.19.0/helm/chart/router

## Create k3d Cluster


```
k3d cluster create inigo -p "8080:80@loadbalancer" -p "8443:443@loadbalancer"
```

## Install Apollo Router (Inigo Build)


```
helm install --create-namespace --namespace apollo-router apollo-router oci://ghcr.io/apollographql/helm-charts/router  --set-file supergraphFile="starstuff.graphql" --values values.yaml
```

### Check Apollo Router Pods

Make sure the pod is running and ready

```
kubectl get pods -n apollo-router
```

## Execute Sample GraphQL Query

```
query='query myTopProducts {
  me {
    name
  }
  topProducts {
    name,
    price,
    reviews {
        author {
            name
        }
        body
    }
  }
}'
variables='{}'
curl -i -X POST http://localhost:8080 \
  -H 'Host: apollo-router.local' \
  -H 'Content-Type: application/json' \
  -d @- <<EOF
      {"query": "$(echo $query)", "variables": $variables}
EOF

```
