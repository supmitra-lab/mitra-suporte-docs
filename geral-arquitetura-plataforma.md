# Arquitetura da Plataforma Mitra

Documentacao sobre a arquitetura tecnica, modelos de hospedagem, isolamento de dados e portabilidade. Aplica-se a todas as versoes.

---

## Visao geral

O Mitra e uma plataforma SaaS para criacao de projetos e dashboards. A arquitetura e composta por:

- **Frontend**: aplicacao web (Vue.js) executada no browser do usuario
- **Backend**: API REST em Java/Spring Boot
- **Banco de dados**: MySQL (um schema isolado por projeto/tenant)
- **Agente de IA**: integracao com LLMs via chave de API do proprio cliente

---

## Modelos de hospedagem

O cliente pode escolher entre tres modelos:

### 1. Ambiente compartilhado (multi-tenant) — padrao

- Infraestrutura compartilhada entre clientes
- Cada projeto possui seu proprio banco de dados isolado (schema MySQL independente)
- Nao ha compartilhamento de tabelas entre clientes
- Analogia: predio comercial — infraestrutura compartilhada (elevador, energia), mas cada sala tem sua propria chave

### 2. VM exclusiva na nuvem da Mitra

- Banco de dados, aplicacao e dados inteiramente dedicados ao cliente
- Sem compartilhamento de infraestrutura com outros clientes
- Nuvens homologadas: Azure, AWS, GCP (datacenter no Brasil)
- Specs e valores alinhados pelo time comercial

### 3. VM exclusiva na nuvem do proprio cliente

- Mesmo modelo da VM exclusiva, porem na infraestrutura do cliente
- Nuvens homologadas: Azure, AWS ou GCP
- A Mitra fornece as specs necessarias e o time comercial alinha os valores
- O cliente tem controle total sobre a infraestrutura

Em todos os modelos, o provedor de nuvem pode ser alterado no futuro. A Mitra nao fica presa a um provedor especifico.

---

## Isolamento de dados (multi-tenant)

No modelo multi-tenant:

- Cada projeto tem seu proprio banco de dados (schema MySQL independente)
- Um tenant nao consegue acessar dados de outro
- Nao ha compartilhamento de tabelas, views ou procedures entre tenants
- As conexoes JDBC de cada projeto sao isoladas e criptografadas

Na camada de IA:

- Os prompts sao enviados por sessao individual do usuario
- A chave de API do LLM e do proprio cliente
- Nao ha compartilhamento de contexto entre tenants
- Nao existe risco de data leakage via LLM entre clientes

---

## Fluxo de dados (data flow)

O caminho completo de um dado do sistema do cliente ate o dashboard:

```
Fonte do cliente (SAP, banco de dados, API)
  -> Conexao criptografada (JDBC ou API REST via HTTPS/TLS)
  -> Backend Mitra (Java/Spring Boot)
  -> API REST via HTTPS/TLS
  -> Browser do usuario
  -> Renderizacao do dashboard
```

O provedor de LLM (Anthropic ou OpenAI) so e acionado quando o usuario interage com o agente de IA. A renderizacao de dashboards em si nao passa pelo LLM.

---

## Integracoes e conectores

### Conexoes JDBC nativas

| Banco de dados | Suporte |
|---------------|---------|
| MySQL | Sim |
| PostgreSQL | Sim |
| SQL Server | Sim |
| Oracle (SID e Service) | Sim |
| Google BigQuery | Sim |
| Firebird | Sim |

### Conectores de API

| Servico | Suporte |
|---------|---------|
| Sankhya (API e DB direto) | Sim |

### Integracao generica

A plataforma e **agnostica quanto a sistemas externos**. E possivel integrar com qualquer ferramenta que disponibilize:
- API REST
- Banco de dados acessivel via JDBC

Nao existe uma lista fixa de conectores "suportados" — **qualquer sistema que exponha API ou banco de dados acessivel pode ser integrado**, seja ERP, CRM, BI, ferramenta de mensageria, marketing, ou qualquer outro tipo de sistema corporativo.

### Integracoes 100% customizaveis via IA (Agent)

Na versao **Agent**, integracoes com sistemas externos sao 100% customizaveis atraves da IA. Basta descrever em linguagem natural qual integracao se deseja e a IA constroi a integracao utilizando as APIs disponibilizadas pelos sistemas.

As integracoes desenvolvidas ficam armazenadas como parte do projeto. Credenciais e tokens de APIs externas sao armazenados no servidor de integracoes apartado (MITRA_BASE_URL_INTEGRATIONS), com isolamento entre projetos.

Credenciais e tokens de APIs externas sao armazenados no servidor de integracoes apartado (MITRA_BASE_URL_INTEGRATIONS), com isolamento entre projetos.

### Modos de leitura de dados

O cliente escolhe entre:

1. **Leitura em tempo real**: query sob demanda no banco/API do cliente, sem replicacao de dados para o Mitra
2. **Importacao**: dados sao importados para o banco do projeto Mitra (hospedado em nuvem no Brasil)

Ambas as opcoes estao disponiveis tanto via JDBC quanto via API.

Para conexoes JDBC, o cliente pode:
- Liberar o IP do Mitra no firewall
- Instalar o Cloudflare Tunnel para conexao segura sem exposicao de IP

---

## API REST e extensibilidade

A plataforma disponibiliza API REST. E possivel criar chaves de API para realizar operacoes nos dados do projeto:

- **GET**: leitura de dados das tabelas
- **POST**: insercao de dados
- **PUT**: atualizacao de dados
- **DELETE**: remocao de dados

Isso permite que sistemas externos leiam e escrevam dados diretamente no Mitra, viabilizando:
- Integracoes bidirecionais com ERPs, CRMs e outros sistemas
- Automacoes externas (scripts, pipelines de dados)
- Conexao com ferramentas de terceiros (BI, ETL, etc.)

---

## Ambientes (Dev/QA/Prod)

No desenvolvimento da plataforma Mitra em si, existe separacao formal de ambientes (Dev, QA, Producao).

Para os projetos do cliente, e possivel ter ambientes separados (ex: projeto de desenvolvimento e projeto de producao), a criterio do cliente.

---

## Portabilidade e exit strategy

Em caso de encerramento do contrato:

- **Codigo frontend** dos projetos: exportavel
- **Banco de dados** (dados e metadados): exportavel
- **Backend**: proprietario da plataforma Mitra (padrao SaaS)

Nao ha vendor lock-in nos dados. O cliente pode solicitar exportacao completa a qualquer momento. Em caso de rescisao, a exportacao e realizada em ate 5 (cinco) dias uteis.

---

## Infraestrutura de backup

- Backups automaticos diarios do banco de dados
- Backups armazenados em provedor independente do ambiente de producao (principio de isolamento)
- Producao em um provedor, backups em outro
- Retencao e acesso aos backups independentes da disponibilidade do ambiente de producao
