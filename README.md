# Mitra Suporte Docs

Base de conhecimento interna da Mitra. Este repositorio centraliza documentacoes que auxiliam o time de suporte, consultores, canais, parceiros e demais areas da empresa a entenderem os processos, features e funcionamento da plataforma Mitra.

---

## Objetivo

Fornecer uma fonte unica e confiavel de documentacoes sobre:

- **Features e funcionalidades** da plataforma
- **Processos padrao** de suporte e operacao
- **Duvidas frequentes** de clientes, parceiros e equipes internas
- **Processos de DevOps** (deploy, infraestrutura, ambientes)
- **Seguranca e LGPD** (controles de acesso, SSO, criptografia)
- **Guias de troubleshooting** para problemas recorrentes

---

## Publico-alvo

- Time de Suporte
- Consultores internos
- Canais e parceiros
- Equipes de vendas e pre-vendas (para duvidas tecnicas de prospects)
- Bot de atendimento (IA)

---

## Estrutura de documentos

### Convencao de nomes

Cada documento segue o padrao:

```
[versao]-[tema-do-documento].md
```

Onde `[versao]` indica a qual versao/produto da plataforma o documento se aplica:

| Prefixo | Produto/Versao | Descricao |
|---------|----------------|-----------|
| `geral` | Todas as versoes | Documentacao que se aplica a qualquer versao da plataforma |
| `agent` | Mitra Agent | Versao Agent da plataforma |
| `mitralab` | MitraLab | Versao MitraLab |
| `mitrasheet` | MitraSheet | Versao MitraSheet |

**Exemplos:**

```
geral-login-permissoes-acessos.md      # Aplica-se a todas as versoes
agent-configuracao-ia.md               # Especifico do Mitra Agent
mitralab-deploy-s3.md                  # Especifico do MitraLab
mitrasheet-formulas-avancadas.md       # Especifico do MitraSheet
```

### Como identificar a versao pelo link de acesso

| Versao | Padrao de URL | Observacao |
|--------|--------------|------------|
| **MitraECP** | Links variados | Poucos clientes antigos |
| **MitraSheet** | `app.mitrasheet.*` | |
| **MitraLab** | `app.mitralab.*` | |
| **Preview** | `ai.mitralab.*` | Versao identica ao MitraLab |
| **Agent** | `agent.mitralab.io` | Versao mais recente |

**Importante:** Quando o link nao seguir nenhum desses padroes, o time de suporte deve confirmar a versao antes de prosseguir com o atendimento.

---

## Assuntos que devem ser direcionados para outros times

Alguns assuntos fogem do escopo do bot de suporte (que responde com base apenas nas docs deste repo) e devem ser direcionados para times especializados:

| Assunto | Time responsavel | O que direcionar |
|---------|-----------------|------------------|
| **Templates da Store** | Time da Store | Quais templates existem, o que cada um faz, cases de uso especificos (ex: FP&A, CRM, BI financeiro, eNPS). O bot nao tem acesso aos templates — apenas o time da Store conhece o catalogo completo. |
| **Precos, propostas, contratos** | Time Comercial | Modelos de licenciamento, TCO, SLA de disponibilidade, clausulas, valores de VM exclusiva, planos e descontos. |
| **Parcerias e whitelabel** | Time Comercial | Onboarding de novos parceiros, specs e valores de infraestrutura dedicada. |
| **Tickets de investigacao tecnica** | Time de Suporte | Quando o bot nao conseguir resolver um ticket de duvida ou investigacao sozinho com base nas docs, redirecionar para o time de suporte humano. |
| **Infraestrutura e DevOps** | Time de Suporte/Infra | Questoes que envolvam alteracoes em infraestrutura, provisionamento de VMs, configuracao de ambientes dedicados, BYOC. |
| **Consultoria em projetos** | Time de Consultoria | Modelagem de negocio, estruturacao de dashboards, definicao de KPIs, migracao de sistemas legados. |

**Regra geral:** se o bot nao encontrar resposta clara nas docs ou o assunto envolver negociacao, cobranca, templates especificos ou infraestrutura, direcionar para o time apropriado ao inves de tentar responder.

---

### Categorias sugeridas

Os documentos podem cobrir qualquer tema, mas as categorias mais comuns sao:

| Categoria | Exemplos de temas |
|-----------|-------------------|
| **Acesso e Seguranca** | Login, SSO, permissoes, LGPD, controle de acesso |
| **Features** | Componentes, actions, dataloaders, IA, integrações |
| **DevOps** | Deploy, ambientes, infraestrutura, monitoramento |
| **Troubleshooting** | Erros comuns, diagnostico, solucoes conhecidas |
| **Processos** | Como abrir chamados, fluxo de atendimento, SLA |
| **Integrações** | APIs, conectores, JDBC, SSO, webhooks |
| **FAQ** | Perguntas frequentes por tema |

---

## Documentos disponiveis

| Arquivo | Versao | Tema |
|---------|--------|------|
| [geral-login-permissoes-acessos.md](geral-login-permissoes-acessos.md) | Todas | Login, SSO, permissoes e controle de acesso |
| [geral-arquitetura-plataforma.md](geral-arquitetura-plataforma.md) | Todas | Arquitetura, hospedagem, isolamento, integracoes, portabilidade |
| [geral-seguranca-lgpd-ia.md](geral-seguranca-lgpd-ia.md) | Todas | Criptografia, LGPD, privacidade, DRP, logs, certificacoes |
| [geral-governanca-ia.md](geral-governanca-ia.md) | Todas | Modelo agnostico de LLM, prompt engineering, custos, controles de IA |
| [geral-sla-suporte.md](geral-sla-suporte.md) | Todas | SLA, canais de suporte, pen test, gestao de vulnerabilidades |
| [geral-analytics-sankhya.md](geral-analytics-sankhya.md) | Todas | Integracao Analytics AI com Sankhya ERP, troubleshooting |
| [geral-api-rest.md](geral-api-rest.md) | Todas | API REST (GET/POST/PUT/DELETE), autenticacao, Actions via API |
| [geral-resposta-incidentes.md](geral-resposta-incidentes.md) | Todas | Plano de resposta a incidentes de seguranca |
| [geral-politica-privacidade.md](geral-politica-privacidade.md) | Todas | Politica de privacidade e tratamento de dados |
| [agent-arquitetura-plataforma.md](agent-arquitetura-plataforma.md) | Agent | Arquitetura da plataforma Agent, servicos, on-premise, backup e congelamento |
| [agent-arquitetura-projetos.md](agent-arquitetura-projetos.md) | Agent | Stack dos sistemas desenvolvidos (React + server functions + MySQL) |
| [agent-guia-integracao-dados.md](agent-guia-integracao-dados.md) | Agent | Guia de integracao de dados para consultores |
| [geral-onboarding-whitelabel.md](geral-onboarding-whitelabel.md) | Todas | Onboarding de parceiros whitelabel |
| [mitralab-congelamento-exclusao-backup.md](mitralab-congelamento-exclusao-backup.md) | MitraLab/Sheet | Backup, congelamento, exclusao, restauracao, duplicacao |

---

## Repositorio

- **GitHub:** [supmitra-lab/mitra-suporte-docs](https://github.com/supmitra-lab/mitra-suporte-docs)
