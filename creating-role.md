# Criar Role de Permissões de Push e Pull para um Repositório Específico no Nexus Repository

Este guia explica como criar uma **role personalizada** no Nexus Repository Manager com permissões de **push** (deploy) e **pull** (read) para um repositório específico, e como atribuí-la a um usuário.

---

##  Pré-requisitos

- Acesso de administrador ao Nexus Repository Manager.
- Um repositório já criado (por exemplo, `meu-repo`).
- Um usuário existente ao qual a role será atribuída.

---

## 1.0: Criar uma Role com Permissões Específicas

1. Acesse o Nexus Repository Manager com uma conta administrativa.
2. No menu lateral, vá até **Security > Roles**.
3. Clique em **Create role**.
4. Preencha os campos da seguinte forma:

| Campo | Valor |
|-------|-------|
| **Role ID** | `repo-meu-repo-push-pull` |
| **Role name** | `Permissões push/pull - meu-repo` |
| **Description** | Permite deploy e download no repositório `meu-repo`. |

5. Em **Privileges**, adicione os seguintes privilégios (substitua `meu-repo` pelo nome do seu repositório):

   - `nx-repository-view-maven2-meu-repo-read`
   - `nx-repository-view-maven2-meu-repo-browse`
   - `nx-repository-view-maven2-meu-repo-add`
   - `nx-repository-view-maven2-meu-repo-edit`

> **Nota**: Troque `maven2` pelo tipo do seu repositório se for diferente (ex: `raw`, `npm`, etc.).

> read permite baixar ( pull )
> add permite enviar ( push )
> edit permite sobrescrever artefatos
> browser permite navegar.

6. Clique em **Create role** para salvar.

---

## 2.0 : Atribuir a Role ao Usuário

1. Vá em **Security > Users**.
2. Clique no usuário ao qual deseja conceder acesso.
3. Na seção **Roles**, clique em **+** e selecione `repo-meu-repo-push-pull`.
4. Clique em **Save**.
