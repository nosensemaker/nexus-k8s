# Criando um Repositório Docker Hosted no Nexus com Acesso via Kubernetes (NodePort)
---

## Pré-requisitos

- Um cluster Kubernetes funcional
- Helm (opcional, mas recomendado para instalação do Nexus)
- Nexus Repository Manager já implantado no cluster Kubernetes
- Acesso ao painel de administração do Nexus (usuário admin)
- `kubectl` configurado
- Permissões suficientes para criar serviços e editar o Nexus

---

## Passo 1 – Criar o Repositório Docker Hosted no Nexus

1. Acesse o painel do Nexus: `http://<IP-EXTERNO>:<PORTA-NODEPORT>`
2. Vá até **Settings** (ícone de engrenagem) → **Repositories**
3. Clique em **Create repository**
4. Escolha o tipo: **docker (hosted)**
5. Preencha as informações abaixo:
   - **Name**: `docker-hosted`
   - **Online**: ✅
   - **Deployment policy**: Allow redeploy
   - **HTTP Port**: por exemplo, `5000`
6. Clique em **Create repository**

> A porta especificada aqui (ex: 5000) será usada internamente pelo Nexus.

---

##  Passo 2 – Expor o Nexus via NodePort no Kubernetes

Se o Nexus foi instalado como um Pod no cluster, você precisa de um `Service` para expor a porta configurada para o repositório Docker.

### Exemplo de Serviço (NodePort):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nexus-docker-service
  namespace: default
spec:
  type: NodePort
  selector:
    app: nexus
  ports:
    - name: docker
      port: 5000           # Porta configurada no repositório Nexus
      targetPort: 5000     # Porta no container do Nexus
      nodePort: 30500      # Porta exposta no Node (acesso externo)
```

> Lembre-se: `nodePort` deve estar entre 30000-32767, e deve ser única no cluster.

### Aplicar o serviço:

```bash
kubectl apply -f nexus-docker-service.yaml
```

---

##  Passo 3 – Acessar o Repositório Externamente

Agora você pode acessar o repositório Docker pelo IP do seu nó Kubernetes:

```
http://<IP-DO-NODE>:30500
```

Exemplo de login no Docker:

```bash
docker login <IP-DO-NODE>:30500
```

---

## Testar com Imagem Docker

Após criar o repositório e expô-lo:

1. Faça o build da imagem:
   ```bash
   docker build -t minha-imagem:1.0 .
   ```

2. Tag com o caminho do Nexus:
   ```bash
   docker tag minha-imagem:1.0 <IP-DO-NODE>:30500/docker-hosted/minha-imagem:1.0
   ```

3. Push:
   ```bash
   docker push <IP-DO-NODE>:30500/docker-hosted/minha-imagem:1.0
   ```

---

##  Dicas de Segurança

- Use HTTPS para produção (configurar certificado no Nexus ou usar Ingress TLS)
- Crie usuários com permissões limitadas para automações
- Configure política de retenção e limpeza de imagens antigas

---

