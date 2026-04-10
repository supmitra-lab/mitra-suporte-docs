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
| [agent-guia-integracao-dados.md](agent-guia-integracao-dados.md) | Agent | Guia de integracao de dados para consultores |

---

## Repositorio

- **GitHub:** [supmitra-lab/mitra-suporte-docs](https://github.com/supmitra-lab/mitra-suporte-docs)
