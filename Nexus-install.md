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

Nesta etapa, configuraremos o Deployment do Nexus para garantir que o ConfigMap com as configurações de encriptação seja corretamente montado e utilizado pelo Nexus.

### Verificando o ConfigMap

* Certifique-se de que o ConfigMap `nexus-config` foi criado corretamente e contém os dados de configuração esperados (como a chave de encriptação no arquivo `nexus.properties` e o JSON com as chaves no `nexus.secrets.json`).
* Use o comando abaixo para verificar o conteúdo do ConfigMap criado:

  ```bash
  kubectl get configmap nexus-config -n nexus-3 -o yaml
  ```

### Configurando o Deployment do Nexus

* Verifique se o seu arquivo de deployment (`nexus-deployment.yaml`) está configurado corretamente, garantindo que:

  * O ConfigMap está sendo montado corretamente.
  * O volume para o arquivo `nexus.secrets.json` está apontando para o caminho correto.
  * A variável de ambiente `NEXUS_SECRETS_KEY_FILE` está configurada para o caminho do arquivo de chave.

### Exemplo de Deployment Testado e Funcional:

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
          resources:
            requests:
              cpu: "4"
              memory: "4Gi"
            limits:
              memory: "4Gi"
              cpu: "4"
          ports:
            - containerPort: 8081
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

### Explicação dos Componentes:

* **`NEXUS_SECRETS_KEY_FILE`**: Indica o caminho onde o Nexus deve buscar o arquivo de chave de encriptação.
* **`nexus-config-volume`** e **`nexus-secrets-volume`**: São os volumes que montam os arquivos de configuração e chave de encriptação diretamente no container.
* **`subPath`**: Garante que apenas o arquivo específico do ConfigMap será montado no caminho especificado.

## 5. Criando Service

```
apiVersion: v1
kind: Service
metadata:
  name: nexus-service
  namespace: nexus-3
spec:
  type: NodePort
  selector:
    app: setup
  ports:
    - port: 8081
      targetPort: 8081
      nodePort: 32000
```

### Explicação 

- `type: NodePort:` expõe o serviço para fora do cluster através da porta `32000` no IP do nó.

- `port:` porta acessível dentro do cluster.

- `targetPort:` porta exposta pelo container do Nexus.

- `nodePort:` porta do nó Kubernetes pela qual o serviço pode ser acessado externamente (ex: `http://<IP_DO_NO>:32000`).

- `selector`: garante que o tráfego será roteado corretamente para os pods com o label `app: setup`.

## 6. Criando Volume Persistente (PVC)

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nexus-data-pvc
  namespace: nexus-3
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: local-path
```
### Explicação

- `accessModes`: ReadWriteOnce: permite que apenas um pod monte o volume para leitura e escrita.

- `resources.requests.storage: 10Gi`: reserva 10 GB de espaço em disco.

- `storageClassName`: local-path: define que o volume será criado com a `StorageClass` local-path, comum em clusters locais.

Esse PVC será usado no deployment, dentro do volume nexus-data:

```
volumes:
  - name: nexus-data
    persistentVolumeClaim:
      claimName: nexus-data-pvc
```

E montado no container:

```
volumeMounts:
  - name: nexus-data
    mountPath: /nexus-data
```
## 7. Recriando Recursos (Opcional)

Para garantir que tudo funcione corretamente, você pode recriar os recursos:

Deletar

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

## 8. Verificação

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

9. No status, a mensagem esperada é:

```
Re-encryption required: All secrets are using the same encryption key. Re-encryption is not required.
```
