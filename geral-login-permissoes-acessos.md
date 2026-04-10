# Login, Permissoes e Acessos no Mitra

Documentacao sobre as formas de autenticacao, modelo de permissoes e controle de acesso da plataforma Mitra. Aplica-se a todas as versoes (Agent, MitraLab, MitraSheet).

---

## Login e autenticacao

O Mitra oferece diferentes formas de autenticacao, permitindo flexibilidade conforme a politica de TI de cada empresa.

As opcoes de login disponiveis sao:

- **Cadastro direto na plataforma Mitra** (usuario e senha)
- **SSO com Google** (OAuth 2.0)
- **SSO com Microsoft** (Azure AD / Microsoft Entra ID)

Todas as opcoes podem coexistir no mesmo ambiente. Um workspace pode ter usuarios que fazem login por cadastro direto, outros por Google e outros por Microsoft simultaneamente.

---

## Cadastro direto na plataforma

No cadastro direto:

- O usuario cria suas credenciais (email e senha) na propria plataforma Mitra
- O acesso e associado ao email do usuario
- As senhas sao armazenadas com criptografia (BCrypt)

### Acesso via convite (obrigatorio)

Para que o cadastro seja realizado corretamente, o usuario **deve acessar o Mitra por meio do link de convite** enviado por email.

Esse convite:

- Vincula o usuario ao Workspace correto
- Define o contexto inicial de acesso (perfil, projetos)
- Evita criacao de contas fora do ambiente esperado

**Sem o uso do link de convite, o cadastro pode nao associar o usuario corretamente ao Workspace.**

> **Atencao:** Sempre valide se o email utilizado no cadastro ou no login via Google/Microsoft e **exatamente o mesmo email que recebeu o convite**, pois o invite e vinculado a um endereco especifico.

---

## Login via SSO Google

O login via SSO Google utiliza OAuth 2.0:

- O usuario clica em "Login com Google" na tela de login
- E redirecionado para a tela de autenticacao do Google
- Apos autenticar, retorna ao Mitra com a sessao ativa
- O email da conta Google deve ser o mesmo do convite

Nao e necessaria nenhuma configuracao especial por parte do cliente para o SSO Google.

---

## Login via SSO Microsoft (Azure AD / Microsoft Entra ID)

O login via SSO Microsoft utiliza integracao com **Microsoft Entra ID** (antigo Azure Active Directory).

### Como funciona tecnicamente

1. O frontend utiliza a biblioteca **MSAL** (Microsoft Authentication Library) para abrir o popup de login da Microsoft
2. O usuario se autentica no Azure AD do tenant dele usando suas credenciais corporativas
3. Apos autenticacao, o frontend recebe um **token JWT** (idToken) do Azure AD
4. Esse token e enviado ao backend do Mitra, que **valida a assinatura** do JWT contra as chaves publicas da Microsoft (`https://login.microsoftonline.com/common/discovery/v2.0/keys`) usando algoritmo RSA-256
5. Apos validacao, o Mitra gera um token JWT interno para a sessao do usuario

### Integracao com o AD do cliente

A configuracao e feita por tenant. Cada cliente pode ter seu proprio `clientId` e `tenantId` do Azure AD configurado, o que permite que **o Active Directory do proprio cliente controle quem tem acesso**. Isso significa que:

- A gestao de identidades fica centralizada no AD do cliente
- Usuarios desativados no AD perdem acesso automaticamente ao Mitra
- Politicas de senha, MFA e acesso condicional do AD sao respeitadas

### Usuarios SSO Microsoft na plataforma

Usuarios que fazem login via Microsoft SSO:

- Sao identificados pelo grupo **"Azure SSo User"** na plataforma
- **Nao podem editar seu perfil** (nome, email) dentro do Mitra, pois os dados vem do AD
- O logout do Mitra tambem faz logout da sessao Microsoft (via `logoutPopup`)

### Bloqueios corporativos

Em alguns ambientes corporativos, o login via Microsoft pode ser bloqueado por politicas internas de seguranca ou restricoes de dominio. Quando isso ocorre:

- O bloqueio e realizado pela infraestrutura de TI da propria empresa
- O Mitra nao tem controle sobre essa restricao
- E necessario que a **TI da empresa libere os dominios do Mitra** para autenticacao Microsoft

Esse processo:

- **Nao envolve atuacao do time Mitra** (nao podemos desbloquear)
- Deve ser tratado diretamente pelo time de TI do cliente
- Geralmente envolve liberar o dominio da aplicacao Mitra nas politicas de acesso condicional do Azure AD

---

## Modelo de permissoes - Visao geral

O modelo de permissoes do Mitra e organizado em **dois niveis**:

```
Workspace (nivel mais alto)
  |
  |-- Projeto A
  |     |-- Usuarios Desenvolvedores
  |     |-- Usuarios Business
  |
  |-- Projeto B
  |     |-- Usuarios Desenvolvedores
  |     |-- Usuarios Business
  |
  ...
```

Cada nivel possui papeis especificos com responsabilidades e permissoes bem definidas. **Todos os acessos sao segregados por projeto**, garantindo que cada usuario visualize e interaja apenas com os recursos aos quais foi explicitamente autorizado.

---

## Workspace

O **Workspace** e o nivel mais alto de acesso dentro do Mitra. Ele funciona como um conteiner que:

- Agrupa projetos
- Define quem pode criar ou excluir projetos
- Controla o acesso global dos usuarios

### Owner do Workspace

O **Owner** e o perfil de maior privilegio dentro do Mitra.

Permissoes:

- Possui todas as permissoes de um Administrador
- E o **unico perfil que pode excluir o Workspace**
- Acessa todos os projetos como Desenvolvedor
- Pode criar e excluir projetos
- Pode gerenciar todos os usuarios do Workspace

**Existe apenas um Owner por Workspace.**

### Administrador do Workspace

O **Administrador do Workspace** possui amplos poderes de gestao.

Permissoes:

- Acessa todos os projetos como Desenvolvedor
- Pode criar novos projetos
- Pode excluir projetos
- Pode convidar usuarios para o Workspace
- Pode remover usuarios do Workspace

> Ao convidar um usuario para o Workspace, ele entra **automaticamente como Administrador**, salvo politicas especificas do ambiente.

### Gestao de acessos no Workspace

A gestao de usuarios do Workspace e feita pelos botoes **"Compartilhar"** e **"Gerenciar Membros"**, disponiveis na home do Workspace.

Nessa tela e possivel:

- Visualizar todos os usuarios do Workspace
- Ver quem possui acesso a pelo menos um projeto
- Adicionar ou remover usuarios do Workspace

### Separacao entre Workspace e Projetos

A tela de gestao do Workspace:

- **Nao altera** permissoes dentro dos projetos
- Serve apenas para controle de acesso ao Workspace

As permissoes especificas de cada projeto (Business ou Desenvolvedor):

- Sao configuradas **dentro de cada projeto**
- Devem ser ajustadas individualmente

---

## Projetos

Dentro de cada projeto, os usuarios podem possuir **dois tipos de permissao**:

- **Desenvolvedor**
- **Business**

Essas permissoes controlam o que o usuario pode ou nao fazer **dentro daquele projeto especifico**.

### Usuario Desenvolvedor (Projeto)

O usuario com perfil **Desenvolvedor** possui **acesso total ao projeto**.

Permissoes principais:

- Acessar todas as telas e funcionalidades
- Criar, editar e excluir telas
- Criar e editar componentes (tabelas, graficos, labels, HTML, etc.)
- Criar e gerenciar Actions, DML Actions e automacoes
- Configurar variaveis
- Gerenciar integracoes, conexoes JDBC, importacoes e tabelas online
- Excluir o projeto (quando permitido)
- Definir permissoes de outros usuarios no projeto
- Acessar o banco de dados do projeto (Database Explorer)

### Usuario Business (Projeto)

O usuario com perfil **Business** possui acesso **restrito**, focado apenas no uso funcional do sistema.

Permissoes principais:

- Acessar somente as telas explicitamente liberadas
- Utilizar dashboards e funcionalidades disponiveis
- Interagir com componentes permitidos (filtros, seletores, botoes)
- O acesso ao chat com IA pode ser habilitado ou nao, conforme definicao do projeto

Restricoes:

- **Nao** possui acesso as configuracoes do projeto
- **Nao** pode criar ou editar telas
- **Nao** pode acessar banco de dados, Actions ou integracoes
- **Nao** possui permissoes administrativas

O perfil Business e ideal para: usuarios finais, gestores, times operacionais e consumidores de informacao.

### Observacao importante

Usuarios que sao **Owner** ou **Administrador do Workspace**:

- Sempre entram nos projetos como **Desenvolvedores**
- Nao podem receber o perfil **Business** dentro de projetos

Isso garante que usuarios com poder administrativo no Workspace nunca fiquem restritos dentro de um projeto.

---

## Perfis de seguranca (Seguranca Corporativa)

Alem dos papeis de Desenvolvedor e Business, o Mitra possui um sistema de **Perfis de Seguranca** que permite controle mais granular dos dados visiveis por cada grupo de usuarios.

### O que sao perfis

Perfis permitem:

- Definir **quais telas** cada grupo de usuarios pode acessar
- Aplicar **filtros de dados automaticos** (ex: "Regiao = Sul" para gestores regionais)
- Configurar **dimensoes visiveis** por grupo
- Definir uma **tela inicial** diferente por perfil

### Como funciona na pratica

Exemplo: uma empresa com filiais em varias regioes pode criar:

- **Perfil "Diretoria"**: acesso a todas as telas, sem filtro de dados
- **Perfil "Gestor Regional Sul"**: acesso apenas a telas operacionais, com filtro automatico `Regiao = Sul`
- **Perfil "Gestor Regional Sudeste"**: mesmas telas, mas com filtro `Regiao = Sudeste`

Os filtros de perfil sao aplicados **automaticamente** e o usuario nao consegue remove-los. Isso garante que cada grupo veja apenas os dados pertinentes.

### Hierarquia de prioridade

As selecoes de perfil tem alta prioridade no sistema. Se o usuario define selecoes proprias na mesma dimensao, a selecao do usuario prevalece. Se nao, o filtro do perfil e aplicado.

---

## Resumo dos papeis e permissoes

| Papel | Escopo | Pode criar projetos | Acesso a projetos | Pode gerenciar usuarios |
|-------|--------|--------------------|--------------------|------------------------|
| **Owner** | Workspace | Sim | Todos (como Dev) | Sim + pode excluir workspace |
| **Administrador** | Workspace | Sim | Todos (como Dev) | Sim |
| **Desenvolvedor** | Projeto | Depende do workspace | Total no projeto | Sim (dentro do projeto) |
| **Business** | Projeto | Nao | Restrito a telas liberadas | Nao |

---

## Conformidade com LGPD

O modelo de acesso do Mitra atende aos requisitos de controle de acesso e confidencialidade previstos pela LGPD:

- **Segregacao por projeto**: cada usuario acessa apenas os projetos e telas aos quais foi autorizado
- **Perfis de seguranca**: filtros automaticos de dados garantem que usuarios vejam apenas informacoes pertinentes ao seu escopo
- **SSO via Azure AD**: a gestao de identidades pode ser centralizada no AD do cliente
- **Auditoria**: acoes na plataforma sao rastreadas
- **Propriedade dos dados**: os dados na plataforma sao de propriedade exclusiva do cliente (contratante)

---

## Duvidas frequentes

### "O usuario nao consegue fazer login via Microsoft"
Provavelmente a TI da empresa bloqueou o dominio do Mitra nas politicas do Azure AD. O time de TI do cliente precisa liberar. Nao ha nada que o suporte Mitra possa fazer nesse caso.

### "O usuario se cadastrou mas nao aparece no workspace"
Verificar se o usuario acessou pelo **link de convite**. Sem o convite, o cadastro pode criar a conta mas nao associar ao workspace correto.

### "O usuario Business nao consegue ver uma tela"
A tela precisa ser **explicitamente liberada** para o usuario Business dentro do projeto. Verificar as configuracoes de permissao do projeto.

### "O usuario Business ve dados diferentes do Desenvolvedor"
Pode haver um **perfil de seguranca** aplicado ao usuario Business que filtra os dados automaticamente. Verificar os perfis configurados no projeto.

### "Um Administrador do Workspace esta vendo dados de todos os projetos"
Isso e esperado. Administradores e Owners sempre acessam projetos como Desenvolvedores, com acesso total.
