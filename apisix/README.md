O apisix segue uma abordagem parecida com o kong.

Não suporta dois ou mais metodos de autenticação ao mesmo tempo. https://github.com/apache/apisix/issues/6217.

É possivel usar um servidor externo para autenticar as requisições com o plugin forward-auth.
https://apisix.apache.org/docs/apisix/plugins/forward-auth/.

É possivel adicionar autorização usando o plugin https://apisix.apache.org/docs/apisix/plugins/authz-casbin/


### Criando uma rota

```bash
curl "http://127.0.0.1:9180/apisix/admin/routes/2" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X PUT -d '
{
  "methods": ["POST"],
  "uri": "/search",
  "plugins": {
        "forward-auth": {
            "uri": "http://host.docker.internal:8003/auth",
            "request_headers": ["Authorization"],
            "upstream_headers": ["X-User-ID"],
            "client_headers": ["Location"],
            "request_methods": ["POST"]
        }
   },
  "upstream": {
    "type": "roundrobin",
    "nodes": {
      "host.docker.internal:8003": 1
    }
  }
}'
```

### Criando consumer 

```bash
curl http://127.0.0.1:9180/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "username": "lohanna",
    "plugins": {
        "key-auth": {
            "key": "testekey"
        },
        "basic-auth": {
            "username": "lohanna",
            "password": "foo"
        }
    }
}'
```

### Consumindo a api via gateway

```bash

curl -i -XPOST "http://127.0.0.1:9080/search" -H "Authorization: testekey"

curl -i -XPOST "http://127.0.0.1:9080/search?api_key=testekey"
```