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
docker build --build-arg BUILD_MODE=full -t ityhy:3.8.1 .
```
> Se certifique de executar o comando no diretório que contenha o Dockerfile. 

---

### 2. Login no Servidor Nexus

Faça login no Nexus usando o IP e porta do repositório Docker:

```bash
docker login 192.168.17.175:30500
```

> Você será solicitado a inserir seu nome de usuário e senha do Nexus.

---

### 3. Tag da Imagem

A imagem precisa ser tagueada com o caminho completo do repositório Nexus:

1. Verifique o nome e a tag da imagem.

```bash
docker images

````
2. Faça a tag: primeiro defina a imagem usada e em seguida o caminho que ela vai ficar taggeada
para ser enviada ao nexus: o caminho precisa conter o nome do repositorio que ela vai ficar.

```bash

docker tag ityhy:3.8.1 192.168.17.175:30500/docker-hosted/ityhy:3.8.1

  ````

---

### 4. Push da Imagem para o Nexus

Depois de taguear, envie a imagem para o repositório:

```bash
docker push 192.168.17.175:30500/docker-hosted/ityhy:3.8.1
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


