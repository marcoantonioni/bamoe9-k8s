# Test commands

```
# minikube name/ip must be set in your /etc/hosts for OIDC url validation
KC_HOST=minikube
```

## Login token
```
USER_NAME=alice
USER_PWD=alice

KC_PORT=45201
KC_REALM=my-realm-1
KC_CLIENT_USER=my-client-bpm
KC_CLIENT_SECRET=my-secret-bpm
KC_TOKEN_EXPIRATION=""
KC_TOKEN_SCOPE=""
KC_FULL_TOKEN=$(curl -sk -X POST http://${KC_HOST}:${KC_PORT}/realms/${KC_REALM}/protocol/openid-connect/token \
  --user ${KC_CLIENT_USER}:${KC_CLIENT_SECRET} -H 'content-type: application/x-www-form-urlencoded' \
  -d 'username='${USER_NAME}'&password='${USER_PWD}'&grant_type=password&scope=openid')
if ([[ ! -z "${KC_FULL_TOKEN}" ]] && [[ "${KC_FULL_TOKEN}" != "null" ]]) then echo "Logged in"; else echo "Not logged in"; fi
if ([[ ! -z "${KC_FULL_TOKEN}" ]] && [[ "${KC_FULL_TOKEN}" != "null" ]]) then KC_TOKEN=$(echo "${KC_FULL_TOKEN}" | jq '.access_token' | sed 's/"//g'); else KC_TOKEN=""; fi
if ([[ ! -z "${KC_FULL_TOKEN}" ]] && [[ "${KC_FULL_TOKEN}" != "null" ]]) then KC_TOKEN_EXPIRATION=$(echo $KC_FULL_TOKEN | jq .expires_in | sed 's/"//g'); fi
if ([[ ! -z "${KC_FULL_TOKEN}" ]] && [[ "${KC_FULL_TOKEN}" != "null" ]]) then KC_TOKEN_SCOPE=$(echo $KC_FULL_TOKEN | jq .scope | sed 's/"//g'); fi
echo "Token expires in: ${KC_TOKEN_EXPIRATION}"
echo "Token scopes: ${KC_TOKEN_SCOPE}"
```

## Start process instance
```
_BAMOE_FRONTEND_HOST=${KC_HOST}
_BAMOE_FRONTEND_PORT=45202
_PROCESS_NAME=hiring
curl -v -H "Content-Type: application/json" -H "Accept: application/json" -H "Authorization: Bearer "${KC_TOKEN} \
  -X POST http://${_BAMOE_FRONTEND_HOST}:${_BAMOE_FRONTEND_PORT}/bamoe/process-instances/${_PROCESS_NAME} \
    -d '{"candidateData": { "name": "Jon", "lastName": "Snow", "email": "jon@snow.org", "experience": 5, "skills": ["Java", "Kogito", "Fencing"]}}' | jq .    
```
