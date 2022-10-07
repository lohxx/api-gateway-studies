Com tyk é possivel usar os 3 tipos de autenticação que já usamos hoje para autenticar os usuarios da API.

A autenticação é configurada nas chaves use_keyless e auth.

use_keyless = define se qualquer cliente pode acessar os endpoints ou se vai ter verificação de identidade.

auth = configura como a chave da API vai ser passada para o Tyk.

### Criando API

```bash 
curl --request POST \
  --url http://localhost:8080/tyk/apis \
  --header 'Content-Type: application/json' \
  --header 'x-tyk-authorization: foo' \
  --data '{
	"name": "background-check",
	"slug": "background-check",
	"api_id": "background-check",
	"org_id": "1",
	"use_keyless": false,
	"auth": {
		"auth_header_name": "authorization",
		"use_param": true,
		"param_name": "api_key",
		"use_cookie": false,
		"cookie_name": ""
	},
	"definition": {
		"location": "header",
		"key": "x-api-version"
	},
	"version_data": {
		"not_versioned": true,
		"versions": {
			"Default": {
				"name": "Default",
				"use_extended_paths": true
			}
		}
	},
	"proxy": {
		"listen_path": "/background_check/",
		"target_url": "http://host.docker.internal:8000/",
		"strip_listen_path": true
	},
	"active": true
}'
```

### Gerando chave de acesso

```bash
curl --request POST \
  --url http://localhost:8080/tyk/keys/create \
  --header 'Content-Type: application/json' \
  --header 'x-tyk-authorization: foo' \
  --data '{
    "allowance": 1000,
    "rate": 1000,
    "per": 1,
    "expires": -1,
    "quota_max": -1,
    "org_id": "1",
    "quota_renews": 1449051461,
    "quota_remaining": -1,
    "quota_renewal_rate": 60,
    "access_rights": {
      "background-check": {
        "api_id": "background-check",
        "api_name": "background-check",
        "versions": ["Default"]
      }
    },
    "meta_data": {}
  }'

{
	"key": "1d30fd48968154402b05e73866a7fa42f",
	"status": "ok",
	"action": "added",
	"key_hash": "2a4a779a"
}
```

### Acessando a api via gateway

```bash
curl --request POST \
  --url 'http://localhost:8080/background_check/search?api_key=1d30fd48968154402b05e73866a7fa42f' \
  --header 'Authorization: Basic IDoxZDMwZmQ0ODk2ODE1NDQwMmIwNWU3Mzg2NmE3ZmE0MmY=' \
  --header 'Content-Type: application/json' \
  --data '{
	"document": "38927128915"
}'
```