Para configurar a primeira api no Kong é necessario fazer algumas requisições.

### Autorização
A autorização das rotas pode ser feita via o plugin https://docs.konghq.com/hub/kong-inc/acl/.

### Configurando a rota

**Cria o service que vai ser associado com a rota**

curl -i -s -X POST http://localhost:8001/services \
  --data name=background_check \
  --data url='http://host.docker.internal:8003'

**Cria o path que vai ser acionado na rota**

curl -i -X POST http://localhost:8001/services/background_check/routes \
  --data 'paths[]=/background_check' \
  --data name=search_route

### Configurando a autenticação

**Cria um usuario**

curl -i -X POST http://localhost:8001/consumers/ \
  --data username=lohanna

**Configura o plugin de autenticação via api_key no query params**

curl -X POST http://localhost:8001/plugins/ \
    --data "name=key-auth"  \
    --data "config.key_names=api_key"

**Configura o plugin de autenticação via Authorization usando Basic auth**

curl -X POST http://localhost:8001/plugins \
    --data "name=basic-auth"  \
    --data "config.hide_credentials=true"

Para que um usuario seja possibilitado a usar varios métodos de autenticação, ele deve ser configurado em todos os plugins usados para a autenticação.

**Configura uma chave**

curl -i -X POST http://localhost:8001/consumers/lohanna/key-auth \
  --data key=top-secret-key

**Configura basic auth**

curl -sX POST http://localhost:8001/consumers/lohanna/basic-auth \
  -H "Content-Type: application/json" \
  --data '{"username": "lohanna", "password": "hunter3"}'

**Configura um usuario anonimo como fallback**

curl -sX POST http://localhost:8001/consumers \
  -H "Content-Type: application/json" \
  --data '{"username": "anonymous"}'

**Configura o usuario fallback nos plugins**

curl -sX POST http://localhost:8001/services/background_check/plugins/ \
  -H "Content-Type: application/json" \
  --data '{"name": "key-auth", "config": { "hide_credentials": true, "anonymous": "d955c0cb-1a6e-4152-9440-414ebb8fee8a"} }'
 
```json
{"created_at":1517528304000,"config":{"key_in_body":false,"hide_credentials":true,"anonymous":"d955c0cb-1a6e-4152-9440-414ebb8fee8a","run_on_preflight":true,"key_names":["apikey"]},"id":"bb884f7b-4e48-4166-8c80-c858b5a4c357","name":"key-auth","service_id":"a2a168a8-4491-4fe1-9426-cde3b5fcd45b","enabled":true}
```

curl -sX POST http://localhost:8001/services/background_check/plugins/ \
  -H "Content-Type: application/json" \
  --data '{"name": "basic-auth", "config": { "hide_credentials": true, "anonymous": "d955c0cb-1a6e-4152-9440-414ebb8fee8a"} }'

```json
{"created_at":1517528499000,"config":{"hide_credentials":true,"anonymous":"d955c0cb-1a6e-4152-9440-414ebb8fee8a"},"id":"e5a40543-debe-4225-a879-a54901368e6d","name":"basic-auth","service_id":"a2a168a8-4491-4fe1-9426-cde3b5fcd45b","enabled":true}
```

**Configura um request terminator para o usuario de fallback**


curl -sX POST http://localhost:8001/consumers/d955c0cb-1a6e-4152-9440-414ebb8fee8a/plugins/ \
  -H "Content-Type: application/json" \
  --data '{"name": "request-termination", "config": { "status_code": 401, "content_type": "application/json; charset=utf-8", "body": "{\"error\": \"Authentication required\"}"} }'


### Chamando api via gateway

```bash
curl -X POST "http://localhost:8000/background_check/search?api_key=top-secret-key"

curl -X POST "http://localhost:8000/background_check/search" -H 'Authorization: Basic bG9oYW5uYTpodW50ZXIz'
```

Referências

[Setup kong](https://hub.docker.com/_/kong)

[Setup auth](https://docs.konghq.com/gateway/latest/kong-plugins/authentication/allowing-multiple-authentication-methods/)

[getting started](https://docs.konghq.com/gateway/3.0.x/get-started/services-and-routes/)
