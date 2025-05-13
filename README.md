# Nexus-K9s: Configurando Encrypting Key no Nexus Repository

## 1. Gerando uma Nova Chave de Criptografia

```bash
openssl rand -base64 32
```

## 2. Criando o Arquivo de Configuração de Chave

* Nome do arquivo: `nexus.secrets.json`
* Caminho: recomendo salvar no mesmo path do deployment.

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

* Certifique-se de que o seu nexus-deployment.yaml está montando o ConfigMap corretamente:
* Exemplo de configuração testada e funcional:

```yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nexus
  namespace: nexus-3
spec:
  replicas: 2
  selector:
    matchLabels:
      app: setup
  template:
    metadata:
      labels:
        app: setup
    spec:
      containers:
        - name: nexus
          image: sonatype/nexus3:latest
          env:
            - name: MAX_HEAP
              value: "2 GiB"
            - name: MIN_HEAP
              value: "1 GiB"
            - name: NEXUS_SECURITY_RANDOMPASSWORDS
              value: "false"
            - name: NEXUS_SECRETS_KEY_FILE
              value: /opt/sonatype/nexus/etc/nexus.secrets.json
          volumeMounts:
            - name: nexus-data
              mountPath: /nexus-data
            - name: nexus-config-volume
              mountPath: /opt/sonatype/nexus/etc/nexus.properties
              subPath: nexus.properties
            - name: nexus-secrets-volume
              mountPath: /opt/sonatype/nexus/etc/nexus.secrets.json
              subPath: nexus.secrets.json
      volumes:
        - name: nexus-data
          persistentVolumeClaim:
            claimName: nexus-data-pvc
        - name: nexus-config-volume
          configMap:
            name: nexus-config
        - name: nexus-secrets-volume
          configMap:
            name: nexus-config
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

## Re-encriptando via API

1. Acesse o site do Nexus e vá em **Admin -> System -> API**.
2. Busque o endpoint **Security Management: Secrets Encryption**.
3. Expanda "Put /v1/secrets/encryption/re-encrypt" e clique em "Try Out".
4. Substitua o valor de `secretKeyId` pelo ID da nova chave (ex: `my-new-custom-key01`) e adicione um e-mail de notificação:

```json
{
  "secretKeyId": "my-new-custom-key01",
  "notifyEmail": "seu-email@exemplo.com"
}
```

5. Clique em "Execute".
6. A resposta esperada é:

```
202 - Re-encrypt task successfully submitted
```

7. No status, a mensagem esperada é:

```
Re-encryption required: All secrets are using the same encryption key. Re-encryption is not required.
```
