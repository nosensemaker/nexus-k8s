# nexus-k9s

Adicionando uma Encrypting Key ao Nexus Repository

1 # Gere uma chave nova usando o seguinte comando:
  $ openssl rand -base64 32

2 # Use o template para criar o json file.
nome do arquivo: nexus.secrets.json
caminho do arquivo: o mesmo do deployment

conteudo do json:

{
  "active": "my-new-custom-key01",
  "keys": [
    {
      "id": "my-new-custom-key01",
      "key": "add-here-the-new-key"
    }
  ]
}

no id: É o nome da chave personalizada
key: a chave que você gerou

3 # Crie o ConfigMap

conteudo:

apiVersion: v1
kind: ConfigMap
metadata:
  name: nexus-config
  namespace: nexus-3
data:
  nexus.properties: |
    nexus.security.secretKey=chave.que.voce.criouu
  nexus.secrets.json: |
    {
      "active": "my-new-custom-key01",
      "keys": [
        {
          "id": "my-new-custom-key01",
          "key": "chave.que.voce.criou"
        }
      ]
    }

4 # acrescente ao deployment as configurações da chave e do arquivo da chave.

exemplo do deployment:

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


Opcional: ( Para mim, funcionou melhor deletar o deployment, service, configmap, pvc e subir novamante. )

$ kubectl delete deployment nexus -n nexus-3

$ kubectl delete service nexus -n nexus-3

$ kubectl delete configmap nexus-config -n nexus-3

$ kubectl delete pvc nexus-data-pvc -n nexus-3

5 # Criando os arquivos novamente:

# ConfigMap

$ kubectl apply -f nexus-configmap.yaml -n nexus-3

# PVC

$ kubectl apply -f nexus-pvc.yaml -n nexus-3

# Deployment

$ kubectl apply -f nexus-deployment.yaml -n nexus-3

# Service

$ kubectl apply -f nexus-service.yaml -n nexus-3

# Verificar se tudo está funcionando:

$ kubectl get all -n nexus-3





