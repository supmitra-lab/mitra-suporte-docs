# Arquitetura dos Sistemas Desenvolvidos no Mitra Agent

Documentacao sobre a arquitetura tecnica dos sistemas/projetos que sao criados dentro da plataforma Mitra Agent. Cada projeto resulta em um sistema corporativo completo.

---

## Flexibilidade e customizacao

Os sistemas desenvolvidos no Mitra Agent sao **100% flexiveis e customizaveis**. O usuario pode construir praticamente qualquer coisa que um sistema corporativo tradicional faria, simplesmente descrevendo o que precisa em linguagem natural. A IA cuida do desenvolvimento do codigo React, das server functions, da estrutura do banco de dados e das integracoes.

Isso significa que features como:

- **Workflows de aprovacao multinivel** (com notificacoes, deadlines, status tracking)
- **Motores de cenarios** (Base/Best/Worst Case com premissas parametrizaveis)
- **Modelagem financeira** (P&L, EBITDA, FCF, ARPU, NRR, LTV por coorte)
- **Suporte multi-moeda** (BRL/USD/EUR com taxa configuravel)
- **Dashboards interativos** com drill-down e filtros dinamicos
- **Regras de negocio complexas** (alocacoes top-down, bridge de receita, waterfall charts)
- **Trilha de auditoria** (quem alterou, quando, valores antes/depois)
- **Notificacoes por email, Slack, WhatsApp**
- **Integracoes com qualquer sistema** (CRMs, ERPs, BIs, data warehouses)

...sao todas implementaveis via prompts para a IA, **sem precisar de feature nativa da plataforma**. A plataforma fornece os blocos fundamentais (frontend React, backend, banco, integracoes); o que sera construido em cima disso e totalmente customizavel.

### Store de templates

Alem da customizacao total via IA, o Mitra possui uma **Store de templates** que acelera projetos com casos de uso comuns (FP&A, CRM, BI financeiro, eNPS, planejamento orcamentario, etc.). O cliente pode partir de um template pronto e customizar a partir dele.

Para saber quais templates estao disponiveis e o que cada um faz, consultar o **Time da Store**.

### Mobile e responsividade

Os sistemas desenvolvidos no Mitra Agent sao **100% responsivos para mobile por padrao**. Basta solicitar a IA que o layout seja adaptado para dispositivos moveis — o sistema final pode ser acessado diretamente pelo navegador do celular, sem necessidade de app nativo.

---

## Visao geral

Um sistema desenvolvido no Mitra Agent e composto por quatro camadas:

| Camada | Tecnologia | Onde fica |
|--------|-----------|-----------|
| **Frontend** | React | AWS S3 (versionado) |
| **Backend (Server Functions)** | SQL / JavaScript | Banco MySQL do projeto |
| **Banco de dados** | MySQL | Instancia gerenciada na nuvem |
| **Integracoes** | API REST | Servidor de integracoes apartado |

---

## Frontend (React)

- O codigo do frontend e gerado pela IA dentro do sandbox E2B
- Compilado com **Vite** (build acontece dentro do E2B)
- Armazenado no **AWS S3 com versionamento ativo**
- Servido diretamente no browser do usuario final
- Versoes anteriores disponiveis por **7 dias** (politica de retencao)

O frontend se comunica com o backend via **mitra-interactions-sdk**, que fornece funcoes para:
- Chamar server functions
- Ler e escrever dados no banco
- Executar actions
- Fazer upload/download de arquivos
- Interagir com integracoes externas

---

## Backend (Server Functions)

As server functions sao o backend do sistema. Elas:
- Sao escritas em **SQL ou JavaScript**
- Ficam **salvas no banco de dados MySQL** do projeto (nao em arquivos separados)
- Sao executadas pelo **Backend Java (Mitra API)**
- Podem ser criadas/editadas pela IA (via `mitra-sdk`) ou manualmente

Casos de uso de server functions:
- Regras de negocio
- Calculos e transformacoes de dados
- Consultas complexas
- Validacoes
- Envio de emails
- Manipulacao de arquivos

---

## Banco de dados (MySQL)

Cada projeto tem seu proprio banco de dados isolado (schema MySQL independente):
- **Tenant isolado**: um projeto nao acessa dados de outro
- **Dados, metadados e server functions**: tudo fica no banco
- **Backup automatico diario** (noturno), armazenado em S3 separado
- **Conexoes JDBC**: o projeto pode se conectar a bancos externos (MySQL, PostgreSQL, SQL Server, Oracle, BigQuery, Firebird) para leitura ou importacao

---

## Integracoes de API

O Mitra permite integrar o sistema com APIs externas (ERPs, CRMs, servicos externos).

### Servidor de integracoes apartado

As credenciais de APIs externas (tokens, chaves, senhas) sao armazenadas em um **servidor apartado dedicado a integracoes** (MITRA_BASE_URL_INTEGRATIONS), separado do servidor de dados. Isso garante:
- Isolamento de credenciais sensiveis
- Credenciais nao ficam expostas no banco de dados do projeto
- Servidor pode ser dedicado por cliente em modelo on-premise

### Como funciona

1. O desenvolvedor (ou a IA) configura a integracao usando um template nativo do Mitra
2. As credenciais sao armazenadas no servidor de integracoes
3. O codigo do frontend chama a integracao via `mitra-interactions-sdk`
4. O Backend Java executa a chamada a API externa usando as credenciais armazenadas

---

## Autenticacao dos usuarios finais

O Mitra disponibiliza um **template de autenticacao nativo** para os sistemas desenvolvidos:

- Utiliza **JWT proprio** (nao depende do Mitra Space para autenticar usuarios finais)
- O sistema final gerencia seu proprio fluxo de login
- Credenciais e tokens de sessao sao gerenciados pelo Backend Java

O Mitra Space gerencia apenas os acessos dos **desenvolvedores** a plataforma (workspaces, projetos). Os **usuarios finais** do sistema desenvolvido usam a autenticacao propria do sistema.

---

## SDKs disponiveis

| SDK | Usado por | Finalidade |
|-----|----------|-----------|
| **mitra-sdk** | IA (durante o desenvolvimento) | Criar/editar tabelas, criar server functions, testar, configurar projeto |
| **mitra-interactions-sdk** | Codigo do sistema final (React) | Chamar server functions, ler/escrever dados, executar actions, upload/download |

---

## Backup e restauracao

| Componente | Metodo de backup | Retencao | RPO maximo |
|-----------|-----------------|----------|-----------|
| Frontend (React) | S3 versionado | 7 dias | Minutos (cada alteracao gera versao) |
| Banco (MySQL) | Backup diario noturno | Indefinida | 24 horas |
| Server functions | Incluidas no backup do banco | Indefinida | 24 horas |

Para detalhes completos sobre backup, congelamento e restauracao, consulte a secao "Backup, congelamento e restauracao" em [agent-arquitetura-plataforma.md](agent-arquitetura-plataforma.md).

---

## Resumo da stack de um projeto

```
[Usuario Final]
      |
      v
[Frontend React] ---- servido do S3
      |
      | mitra-interactions-sdk
      v
[Backend Java - Mitra API]
      |
      +--> [MySQL] ---- dados + server functions
      |
      +--> [Servidor de Integracoes] ---- credenciais de APIs externas
      |
      +--> [S3 Mitra Drive] ---- arquivos
      |
      +--> [Conexoes JDBC] ---- bancos externos do cliente
```
