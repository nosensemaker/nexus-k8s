# Nexus-K9s: Configurando Encrypting Key no Nexus Repository

## 1. Gerando uma Nova Chave de Criptografia

```bash
openssl rand -base64 32
```

## 2. Criando o Arquivo de Configuração de Chave

* Nome do arquivo: `nexus.secrets.json`
* Caminho: o mesmo do deployment

Conteúdo do arquivo:

```json
{
  "active": "my-new-custom-key01",
  "keys": [
    {
      "id": "my-new-custom-key01",
      "key": "add-here-the-new-key"
    }
  ]
}
```

* `id`: É o nome da chave personalizada.
* `key`: A chave gerada no passo 1.

## 3. Criando o ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
  namespace: nexus-3
data:
  nexus.properties: |
    nexus.security.secretKey=<sua-chave-gerada>
  nexus.secrets.json: |
    {
      "active": "my-new-custom-key01",
      "keys": [
        {
          "id": "my-new-custom-key01",
          "key": "<sua-chave-gerada>"
        }
      ]
    }
```

## 4. Configurando o Deployment

* Certifique-se de que o deployment inclua as configurações da chave.
* Exemplo de configuração:

```yaml
[conteúdo do deployment conforme fornecido]
```

## 5. Recriando Recursos (Opcional)

Para garantir que tudo funcione corretamente, você pode recriar os recursos:

```bash
kubectl delete deployment nexus -n nexus-3
kubectl delete service nexus -n nexus-3
kubectl delete configmap nexus-config -n nexus-3
kubectl delete pvc nexus-data-pvc -n nexus-3
```

E recriar:

```bash
kubectl apply -f nexus-configmap.yaml -n nexus-3
kubectl apply -f nexus-pvc.yaml -n nexus-3
kubectl apply -f nexus-deployment.yaml -n nexus-3
kubectl apply -f nexus-service.yaml -n nexus-3
```

## 6. Verificação

Para verificar se tudo está funcionando corretamente:

```bash
kubectl get all -n nexus-3
```
