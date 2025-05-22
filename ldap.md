# 🎯 Integração do Nexus Repository com LDAP

Este guia tem como objetivo documentar passo a passo como realizar a configuração de autenticação via LDAP no **Nexus Repository**.

---

## 📚 Pré-requisitos

- Servidor LDAP ativo e funcional.
- Acesso administrativo ao Nexus Repository.
- Dados do LDAP:
  - IP/Hostname
  - Base DN
  - Usuário de bind (admin)
  - Senha de bind
- Porta liberada (389 para LDAP ou 636 para LDAPS).

---

## 🔧 Configurando a Conexão LDAP

### 1️⃣ Acessando as configurações

- No menu lateral do Nexus, acesse:
  
> **Administration → Security → LDAP**

Clique em **"Create connection"**.

---

### 2️⃣ Configurações Gerais da Conexão

| Campo                     | Valor                                      |
|---------------------------|---------------------------------------------|
| **Name**                  | ldap connection                             |
| **LDAP server address**   | ldap://192.168.17.169:389                   |
| **Search base DN**        | dc=in,dc=iti,dc=br                          |
| **Authentication method** | Simple Authentication                       |
| **Username or DN**        | cn=admin,dc=in,dc=iti,dc=br                 |
| **Password**              | 🔒 (Senha do usuário `cn=admin`)            |

#### 🔥 Configurações de Timeout

| Campo    | Valor         |
|----------|----------------|
| **Wait** | 30 segundos    |
| **Retry**| 300 segundos   |
| **Max failed attempts** | 3 |

Após preencher, clique em **"Verify connection"** para testar a conexão com o LDAP.

---

## 👥 Configurando o Mapeamento de Usuários

### 1️⃣ Template de Configuração

| Campo                        | Valor           |
|------------------------------|-----------------|
| **Configuration template**    | Generic LDAP Server |

---

### 2️⃣ Dados de Usuário

| Campo                                   | Valor                                     |
|-----------------------------------------|--------------------------------------------|
| **User relative DN**                    | ou=People                                  |
| **User subtree**                        | (✔️) habilitado se usuários estão em subestruturas (opcional) |
| **Object class**                        | inetOrgPerson                              |
| **User filter**                         | (uid=*)                                    |

---

### 3️⃣ Atributos do LDAP

| Campo                    | Valor         | Descrição                                        |
|--------------------------|---------------|--------------------------------------------------|
| **User ID attribute**    | uid           | Atributo usado como login                        |
| **Real name attribute**  | cn            | Nome completo do usuário                         |
| **Email attribute**      | mail          | E-mail do usuário                                |
| **Password attribute**   | userPassword  | (Opcional) – Pode deixar em branco para bind     |

---

### 4️⃣ Mapeamento de Grupos

| Campo                           | Valor             |
|---------------------------------|-------------------|
| **Map LDAP groups as roles**    | ✔️ Ativado        |
| **Group type**                  | Dynamic Groups    |
| **Group member of attribute**   | memberOf          |

---

## ✅ Verificações

Após preencher todas as informações:

- Clique em **"Verify user mapping"** para validar se os usuários estão sendo listados corretamente.
- Utilize **"Verify login"** para testar se um usuário específico consegue se autenticar.

---

## 🐞 Possíveis Erros e Soluções

| Erro                                          | Solução                                                    |
|------------------------------------------------|------------------------------------------------------------|
| **Failed to connect to LDAP Server**          | Verifique IP, porta, usuário de bind e senha.              |
| **User cannot be authenticated**              | Confirme DN, senha correta e se o usuário está no OU correto. |
| **LDAP search does not return users**         | Verifique `User relative DN`, `User filter` e atributos.   |

---

## 🏁 Conclusão

Após concluir essa configuração, os usuários do seu servidor LDAP poderão realizar login diretamente no Nexus Repository, além de poder mapear grupos para controle de permissões.

---

## 🖥️ Exemplo Visual

### 🔗 Conexão LDAP

![Conexão LDAP](./5115b49b-7c2a-45d6-86d7-a9637c1b6bea.png)

### 👥 Mapeamento de Usuários

![Mapeamento de Usuários](./c5ae181a-51b3-4b5c-ab49-47e70889b4c5.png)

---

## 👩‍💻 Autor

Desenvolvido por [Seu Nome].

---

## 📝 Licença

Este projeto está licenciado sob a licença MIT. Consulte o arquivo [LICENSE](./LICENSE) para mais detalhes.
