# Inigo Apollo Router Kubernetes Demo

This project demonstrates how to use Inigo with Apollo Router which is deployed onto Kubernetes using Helm. Apollo's sample `starstuff.graphql` supergraph schema is used for the demo.

This project uses the following:

1. [Inigo Rust Apollo Router](https://docs.inigo.io/deployment/rust_apollo_router)
2. [Apollo Router Helm Chart](https://github.com/apollographql/router/tree/v1.19.0/helm/chart/router)

## 1. Create Apollo Router in Inigo

In the Inigo dashboard, you must setup an Apollo Router and copy the Inigo token.

![Create Apollo Router in Inigo](images/create-apollo-router.png)

![Copy Token](images/copy-token.png)

## 2. Add your Inigo Token to `values.yaml`

```yaml
    plugins:
      inigo.middleware:
        token: "ey...
```

## 3. Create k3d Kubernetes Cluster

[k3d](https://k3d.io) allows  you to run Kubernetes on your local machine.


```shell
k3d cluster create inigo -p "8080:80@loadbalancer" -p "8443:443@loadbalancer"
```

## 4. Install Apollo Router (Includes Inigo)


```shell
helm install --create-namespace --namespace apollo-router apollo-router oci://ghcr.io/apollographql/helm-charts/router  --set-file supergraphFile="starstuff.graphql" --values values.yaml
```

### Check Apollo Router Pods

Make sure the pod is running and ready

```shell
kubectl get pods -n apollo-router
```

## 5. Execute Sample GraphQL Query

Use `curl` to execute a GraphQL query against the `starstuff.graphql` supergraph schema. 

```shell
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

### Expected Result

```
HTTP/1.1 200 OK
content-type: application/json
vary: origin
content-length: 416
date: Tue, 30 May 2023 02:39:56 GMT

{"data":{"me":{"name":"Ada Lovelace"},"topProducts":[{"name":"Table","price":899,"reviews":[{"author":{"name":"Ada Lovelace"},"body":"Love it!"},{"author":{"name":"Alan Turing"},"body":"Prefer something else."}]},{"name":"Couch","price":1299,"reviews":[{"author":{"name":"Ada Lovelace"},"body":"Too expensive."}]},{"name":"Chair","price":54,"reviews":[{"author":{"name":"Alan Turing"},"body":"Could be better."}]}]}}
```

## 6. Check Inigo for GraphQL Query Data

Find the queries for `myTopProducts` in the Inigo Dashboard.

![Request Data](images/query-analytics.png)

## 7. Apply Max Depth

The sample query (in the `curl` command above) has a max depth of 4. What if you want to restrict to a max depth of 3? This is possible by applying the following config.


```shell
inigo apply inigo/security.yaml
```


## 8. Apply Rate Limit

```shell
inigo apply inigo/rate-limit.yaml
```



## 9. Fine Grain Access Control

inigo delete rate_limit apollo-router:starstuff
inigo delete security apollo-router:starstuff


```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNzE2MjM5MDIzLCJ1c2VyX3Byb2ZpbGUiOiJzdGFyc3R1ZmYiLCJ1c2VyX3JvbGVzIjpbInN0YXJzdHVmZiJdfQ.xk7jqibv9qlw8F26kAu9CezvtxXeEf_oGGjpNyhIROE
```

```shell
query='query myTopProducts {
  topProducts {
    reviews {
        author {
            name
        }
        body
    }
    name
    price
  }
  
}'
variables='{}'
curl -i -X POST http://localhost:8080 \
  -H 'Host: apollo-router.local' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNzE2MjM5MDIzLCJ1c2VyX3Byb2ZpbGUiOiJzdGFyc3R1ZmYiLCJ1c2VyX3JvbGVzIjpbInN0YXJzdHVmZiJdfQ.xk7jqibv9qlw8F26kAu9CezvtxXeEf_oGGjpNyhIROE' \
  -d @- <<EOF
      {"query": "$(echo $query)", "variables": $variables}
EOF
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
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNzE2MjM5MDIzLCJ1c2VyX3Byb2ZpbGUiOiJzdGFyc3R1ZmYiLCJ1c2VyX3JvbGVzIjpbInN0YXJzdHVmZiJdfQ.xk7jqibv9qlw8F26kAu9CezvtxXeEf_oGGjpNyhIROE' \
  -d @- <<EOF













helm upgrade --namespace apollo-router apollo-router oci://ghcr.io/apollographql/helm-charts/router  --set-file supergraphFile="starstuff.graphql" --values values.yaml