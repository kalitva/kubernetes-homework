- Установить consul в кластер
```shell
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install consul hashicorp/consul --create-namespace -n consul -f consul-values.yaml
```
- Установить vault кластер
```shell
helm install vault hashicorp/vault --create-namespace -n vault -f vault-values.yaml
```
- Инициализировать vault
    - Инициализировать (сохранить ключи, показанные при выводе)
    ```shell
    kubectl exec -it vault-0 -n vault -- vault operator init -key-shares=1 -key-threshold=1
    ```
    - "Распечатать" vault, в поле ввода ввести ключ
    ```shell
    kubectl exec -it vault-0 -n vault -- vault operator unseal
    ```
    - Таким же образом инициализировать другие два пода vault
- Создать хранилище секретов и секрет cred
    - Установить vault cli для удобства
    ```shell
    wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install vault
    ```
    - Пробросить порт одного из подов vault и сохранить адрес и root token в переменные окружения
    ```shell
    kubectl port-forward vault-0 -n vault 8200:8200
    export VAULT_ADDR=http://127.0.0.1:8200
    export VAULT_TOKEN=<initial_root_token>
    ```
    - Создать хранилище и секрет
    ```shell
    vault secrets enable -version=2 -path=otus/ kv
    vault kv put otus/cred username="otus" password="asajkjkahs"
    ```
- Создать service-account, secret с jwt для этого service-account, и cluster-role-binding
[vaulth-auth.yaml](./vault-auth.yaml), применить
- Сконфигурировать авторизацию "kubernetes"
```shell
TOKEN_REVIEWER_JWT=$(kubectl get secret -n vault vault-auth-secret -o jsonpath='{.data.token}' | base64 --decode)
KUBE_CA_CERT=$(kubectl get secret -n vault vault-auth-secret -o jsonpath='{.data.ca\.crt}' | base64 --decode)
KUBE_HOST=https://$(kubectl -n vault exec vault-0 -- sh -c 'echo $KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT')

vault auth enable kubernetes
vault write auth/kubernetes/config \
    token_reviewer_jwt="$TOKEN_REVIEWER_JWT" \
    kubernetes_host="$KUBE_HOST" \
    kubernetes_ca_cert="$KUBE_CA_CERT"
```
- Создать и применить [policy](./otus-policy.hcl)
```shell
vault policy write otus-policy otus-policy.hcl
```
- Создать роль
```shell
vault write auth/kubernetes/role/otus \
    bound_service_account_names=vault-auth \
    bound_service_account_namespaces=vault \
    policies=otus-policy \
    ttl=1h
```
- Установить External Secrets
```shell
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets -n vault --set installCRDs=true
```
- Создать и применить [SecretStore](./secret-store.yaml) и [ExternalSecret](./external-secret.yaml)
- Проверить, что создался secret otus-cred
```shell
kubectl get secret -n vault otus-cred -o json | jq -r .data.username | base64 -d
kubectl get secret -n vault otus-cred -o json | jq -r .data.password | base64 -d
```
