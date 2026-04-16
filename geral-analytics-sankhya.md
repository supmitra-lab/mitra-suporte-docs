# Analytics AI (Mitra) integrado ao Sankhya ERP

Documentacao sobre a integracao do Mitra com o Sankhya ERP, cobrindo arquitetura, conectividade, licencas, gestao de acessos e troubleshooting.

---

## Visao geral

O **Analytics AI** e o produto de inteligencia analitica disponibilizado dentro do Sankhya ERP. A tecnologia por tras e o **Mitra**, porem para o cliente final o produto e apresentado como **Analytics AI**.

A integracao permite que:
- Projetos desenvolvidos no Analytics AI sejam abertos dentro do Sankhya
- A experiencia seja semelhante a telas nativas do ERP
- A IA seja utilizada diretamente no contexto do ERP

---

## Processo padrao de integracao (passo a passo)

Este e o processo padrao para iniciar um projeto novo de Analytics AI integrado ao Sankhya. Seguir na ordem.

### 1. Instalar o template "Base de Conhecimento"

Todo projeto novo comeca pela instalacao do template **Base de Conhecimento**, que existe em duas versoes:
- **MSSQL**
- **Oracle**

Instalar a versao correspondente ao banco de dados do cliente.

O template entrega uma serie de BIs prontos (Vendas, Compras, Financeiro, etc). Os dashboards utilizam **server functions de API** para apresentar os dados em tempo real. Todas as server functions do template assumem que:
- O projeto possui uma **integracao de API com o Gateway do Sankhya**
- As chamadas usam o **endpoint dbexplorer**

**Limitacoes importantes do dbexplorer:**
- Limite de **5.000 linhas por requisicao**
- O usuario configurado na API do Gateway **precisa ter acesso ao dbexplorer**, senao a integracao nao funciona

### 2. Alinhar com o cliente a abordagem de integracao

Antes de qualquer configuracao tecnica, **alinhar com o cliente se ele concorda com a abordagem padrao/recomendada (API via Gateway do Sankhya)**.

Caso o cliente nao concorde, ou o proprio consultor identifique que a API nao e viavel (por exemplo, quando e necessario **importar dados** em volume), seguir para a **abordagem alternativa via JDBC** (passo 4).

### 3. Abordagem padrao — Integracao via API (Gateway do Sankhya)

Para configurar o Gateway sao necessarias **3 credenciais**: `client_id`, `client_secret` e `x-token`.

**client_id e client_secret:**
- Recomenda-se que a **matriz** e **cada canal** tenham seu proprio usuario no **Developer Sankhya**.
- Cada usuario do Developer gera um par `client_id` / `client_secret` proprio, que sera usado na configuracao do Gateway.

**x-token:**
- O proprio **cliente** deve criar uma nova aplicacao na tela **Configuracoes Gateway** do Sankhya dele.
- A aplicacao deve ser **vinculada a aplicacao "Integracoes" do parceiro "MITRA TECNOLOGIA EM SISTEMAS LTDA"**.
- Deve-se vincular tambem um **usuario do Sankhya do cliente que tenha acesso ao dbexplorer** (obrigatorio).
- Essa tela gera o `x-token` que completa as credenciais.

### 4. Abordagem alternativa — Integracao via JDBC

Quando a abordagem via API nao for adotada, seguir por JDBC. Existem dois cenarios:

**Cenario A — Sankhya do cliente na Cloud Sankhya:**
1. O consultor deve acessar (ou pedir ao cliente para acessar) a tela **Preferencias** do Sankhya do cliente, buscar por **Analytics AI**, abrir **Configuracoes 2.0** e **copiar o JSON** que esta ali. Enviar esse JSON ao **suporte Mitra** para que seja identificado em qual servidor o Analytics AI do cliente esta hospedado.
2. Com o servidor identificado pelo suporte Mitra, pedir ao **cliente** que abra um ticket no **Service Desk da Sankhya** solicitando a liberacao do banco dele para o servidor X do Analytics AI.

   **Sugestao de titulo do ticket:**
   > Liberacao de acesso ao banco de dados para o Analytics AI (servidor [X])

   **Sugestao de descricao do ticket:**
   > Prezados,
   >
   > Solicitamos a liberacao de acesso ao nosso banco de dados Sankhya (hospedado na Cloud Sankhya) para o servidor do Analytics AI identificado como **[X]** pela equipe Mitra.
   >
   > O acesso e necessario para viabilizar a integracao via JDBC entre o Analytics AI e o nosso ERP Sankhya.
   >
   > Favor nos fornecer: **host, porta, nome do database/service name, usuario e senha** com permissao de leitura em todas as tabelas do banco.
   >
   > Obrigado.

**Cenario B — Sankhya do cliente em nuvem terceira ou on premise:**
1. Solicitar ao cliente o **IP interno** e a **porta** do banco de dados do Sankhya dele.
2. Criar um **Cloudflare Tunnel** nas configuracoes do Workspace do Analytics AI, informando IP interno e porta na criacao da rota.
3. Entregar ao cliente o **token do tunnel** junto com as instrucoes de como instalar o Cloudflare Tunnel.
4. Solicitar tambem ao cliente as credenciais de acesso ao banco de dados do Sankhya: **host, porta, nome do database/service name, usuario e senha** com permissao de leitura em todas as tabelas do banco.

---

## Arquitetura

O Analytics AI possui **ambiente proprio e separado** do Sankhya:

- O ambiente roda em servidores na **cloud da Sankhya**
- O ERP do cliente pode estar na cloud Sankhya, Sky.One, Wevy ou outra
- O banco de dados do Analytics AI e **separado** do banco do ERP
- Toda integracao ocorre por conectores (nao ha compartilhamento direto de banco)

---

## Conectividade com o banco do Sankhya

### Opcoes de conexao

Existem duas formas de conectar o Analytics AI ao banco do ERP:

**1. Via API (Gateway do Sankhya)**
- Utiliza a API nativa do Sankhya como intermediaria
- Nao requer exposicao direta do banco de dados

**2. Via JDBC**
- Conexao direta ao banco de dados do ERP
- Quando o ERP esta na **Cloud Sankhya**: nao precisa de Cloudflare Tunnel. O cliente deve:
  1. Solicitar ao suporte Mitra qual servidor hospeda seu ambiente
  2. Abrir ticket no Service Desk da Sankhya solicitando a conexao do Analytics AI com o DB, informando o servidor
  3. A Sankhya fornecera: host, porta, database/service name, usuario e senha
  4. O usuario deve ter permissao de leitura e acesso as tabelas necessarias
- Quando o ERP esta **fora da Cloud Sankhya**: utilizar **Cloudflare Tunnel** para conexao segura

### Conector Sankhya DB

Em todos os projetos do Analytics AI e criado automaticamente um conector chamado **Sankhya DB**, que permite executar queries no banco do ERP.

---

## Licencas

O Analytics AI verifica licencas diretamente no Sankhya:

1. Acessar **Administracao do Servidor** → aba **Licencas**
2. Verificar:
   - **30965** — Desenvolvedor
   - **30967** — Viewer

O usuario **SUP** (admin padrao do Sankhya) nao consome licenca.

---

## Perfis de acesso

### Desenvolvedor (DEV)
- Acesso total ao projeto
- Criar/editar telas, Actions, integracoes, variaveis
- Acesso completo ao banco do Sankhya via conector (equivalente ao DBExplorer)

**Se precisar dar perfil DEV sem acesso ao banco do Sankhya:** solicitar ao suporte Mitra a criacao de um projeto separado, sem conexao com o Sankhya.

### Administrador do Workspace
- Acesso a todos os projetos
- Pode gerenciar permissoes

### Viewer
- Acessa projetos liberados
- Enxerga todas as telas do projeto
- Nao possui permissoes tecnicas
- Acesso a IA pode ser habilitado/desabilitado via toggle

---

## Telas automaticas no Sankhya

Ao instalar o Analytics AI, duas telas sao criadas no ERP:

1. **Analytics Studio** — Home do Workspace (para devs e admins)
2. **Converse com seus dados** — Playground para interacao com a IA

### Publicacao de telas
Telas desenvolvidas em projetos podem ser publicadas (tres pontos na interface) e passam a aparecer como telas dentro do Sankhya.

---

## Gestao de acessos — ponto critico

A gestao de acessos **NAO deve ser feita pelo controle de telas do Sankhya**.

As liberacoes devem ser feitas **pelo botao de acessos no Workspace dentro do Analytics Studio**.

Se um usuario for liberado apenas pelo controle de telas do Sankhya:
- A tela aparecera no menu
- Mas ao acessar, surgira mensagem de que nao possui acesso no Analytics

O controle de permissao e feito pelo ambiente do Analytics (Mitra), nao pelo ERP.

---

## Troubleshooting

### Erro 404 ao acessar tela do Analytics
**Causa:** Addon do Analytics AI desinstalado ou desatualizado.
**Solucao:** Reinstalar/atualizar o addon via Sankhya Place ou Minhas Solucoes.

### Analytics indisponivel ("Ops, Analytics AI temporariamente indisponivel")
**Solucao:** Fechar e reabrir a tela. Se persistir, abrir ticket para o suporte Mitra.

### Lista de membros nao carrega
**Causa provavel:** Duplicidade de emails em usuarios diferentes no Sankhya.
**Investigacao:**
```sql
SELECT U.CODUSU, U.NOMEUSU, U.EMAIL
FROM TSIUSU U
WHERE U.EMAIL IN (
    SELECT EMAIL FROM TSIUSU
    WHERE CODUSU IN (SELECT CODUSU FROM TGMTOKEN)
    GROUP BY EMAIL HAVING COUNT(*) > 1
)
AND U.CODUSU IN (SELECT CODUSU FROM TGMTOKEN)
ORDER BY U.EMAIL;
```
**Correcao:** Ajustar emails duplicados no Sankhya, depois acionar suporte Mitra para remover membros duplicados.

### Problema de threads abertas (lentidao no ERP)
**Causa:** Configuracao `allowWebSocket: true` em ambientes legados.
**Verificacao:** Preferencias → Analytics AI → copiar JSON → verificar `allowWebSocket`.
**Correcao:** Alterar para `"allowWebSocket": false` e **reiniciar o Sankhya**.
Se persistir, abrir chamado enviando o JSON completo.

### Sankhya DB offline

**Passo 1 — Atualizar o addon**
Administracao do Servidor → Addons Instalados. Atualizar via Sankhya Place ou Minhas Solucoes.

**Passo 2 — Verificar parametros em Preferencias**
- `allowWebSocket`: clientes novos = False, clientes antigos = True
- `allowQueryThroughAPI`: sempre True

**Passo 3 — Verificar no banco do projeto**
```sql
SELECT w.id id_workspace, i.id id_projeto, w.NAME workspacename,
       w.ENABLEQUERYTHROUGHAPI, SKWAPIJDBCCONNECTIONCONFIGID, CUSTOMURLSANKHYA
FROM tenant_configuration.INT_TENANT i
INNER JOIN tenant_configuration.INT_WORKSPACE w ON w.id = i.WORKSPACEID
WHERE dbname = database()
```
- `ENABLEQUERYTHROUGHAPI` deve ser TRUE
- `CUSTOMURLSANKHYA` deve ter o link publico base do Sankhya (sem caminhos adicionais, sem barra final)

**Correcoes se necessario:**
```sql
UPDATE tenant_configuration.INT_WORKSPACE
SET ENABLEQUERYTHROUGHAPI = TRUE WHERE ID = <id_workspace>;
```
```sql
UPDATE tenant_configuration.INT_WORKSPACE
SET CUSTOMURLSANKHYA = 'https://cliente.sankhya.com.br' WHERE ID = <id_workspace>;
```

**Atencao com URLs que comecam com "n":** O caractere `\n` e interpretado como quebra de linha pelo MySQL. Usar CONCAT:
```sql
UPDATE tenant_configuration.INT_WORKSPACE
SET CUSTOMURLSANKHYA = CONCAT('http://', 'nordpharma.nuvemdatacom.com.br:10069')
WHERE ID = <id_workspace>;
```

### Sankhya DB offline por alteracao de endereco da API
Se houve mudanca de ambiente/dominio/infraestrutura, atualizar `CUSTOMURLSANKHYA` com o novo endereco.

---

## Identificacao do ambiente para suporte

Para diagnostico, sempre solicitar ao cliente:
1. Acessar **Preferencias** no Sankhya
2. Buscar **Analytics AI**
3. Copiar o **JSON completo**

Esse JSON contem workspaceId, URLs, endpoints e configuracoes do ambiente. Enviar junto com qualquer ticket de suporte.
