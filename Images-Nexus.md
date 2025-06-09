# Deploy de Imagem Docker no Nexus

## Pré-requisitos

- Docker instalado e em execução
- Acesso ao Nexus com IP e porta do repositório (ex: `192.168.17.175:30500`)
- Permissões de login no repositório Nexus

---

## Passo a Passo

### 1. Build da Imagem

Crie a imagem Docker usando `--build-arg` para passar o modo de build:

```bash
docker build --build-arg BUILD_MODE=full -t  assinador:2.17.3 .
```
> o build_mode é o parametro passado dentro do Dockerfile.
> Se certifique de executar o comando no diretório que contenha o Dockerfile. 

---

### 2. Login no Servidor Nexus

Faça login no Nexus usando o IP e porta do repositório Docker:

```bash
docker login 192.168.19.203:30500
docker login nexus-repo.iti.br
```
> É o service do nodePort, acessivel apenas por IP.
> O domain configurado pelo ingress.
> Você será solicitado a inserir seu nome de usuário e senha do Nexus.

---

### 3. Tag da Imagem

A imagem precisa ser tagueada com o caminho completo do repositório Nexus:

1. Verifique o nome e a tag da imagem.

```bash
docker images

````
2. No exemplo abaixo:
- `ityhy:3.8.1` é a imagem que você acabou de criar no passo 1.
- `docker-hosted` é o nome do repositório no Nexus.
- O caminho completo define onde a imagem será armazenada no servidor.

```bash

docker tag assinador:2.17.3 nexus-repo.iti.br/docker-images/assinador:2.17.3

  ````

---

### 4. Push da Imagem para o Nexus

Depois de taguear, envie a imagem para o repositório:

```bash
docker push nexus-repo.iti.br/docker-images/assinador:2.17.3
```

---

## Como Fazer o Pull da Imagem

Para baixar a imagem em outra máquina (ou na mesma), use:

```bash
docker pull nexus_domain_or_ip:porta/path_to/ityhy:3.8.1
```

> Lembre-se de fazer login com `docker login` se o repositório exigir autenticação.
> docker login nexus-repo.iti.br/imagem:tag - para fazer o login
> docker logout nexus-repo.iti.br/imagem:tag - para fazer o logout

---

## Executar a Imagem

Você pode testar a imagem com:

```bash
docker run --rm -it 192.168.17.175:30500/docker-hosted/ityhy:3.8.1
```


