# Arquitetura da Plataforma Mitra Agent

Documentacao sobre a arquitetura tecnica da plataforma Mitra Agent — os servicos que compoe a plataforma em si, nao os sistemas desenvolvidos dentro dela.

---

## Visao geral

O Mitra Agent e uma plataforma onde usuarios criam sistemas corporativos completos via linguagem natural (prompts). A IA gera codigo React, server functions e configuracoes de banco de dados em tempo real.

O fluxo completo e:

```
Usuario envia prompt
  -> Frontend (mitra-nuxt) via WebSocket
  -> Backend WebSocket (Node.js) orquestra
  -> Sandbox E2B (IA gera codigo React + Build Vite)
  -> Streaming em tempo real pro usuario
  -> Build armazenado no S3
  -> Sistema final servido no browser
  -> Sistema se comunica com Backend Java (Mitra API)
```

---

## Componentes da plataforma

### Servicos gerenciados pela Mitra (nao parametrizaveis)

Estes servicos sao operados exclusivamente pela Mitra e nao podem ser implantados na infraestrutura do cliente:

| Servico | Tecnologia | Funcao |
|---------|-----------|--------|
| **mitra-nuxt** | Nuxt.js | Frontend/IDE onde o desenvolvedor faz prompts e ve o codigo sendo gerado em tempo real |
| **mitra-agent-websocket** | Node.js | Orquestra a comunicacao entre o frontend e o sandbox E2B |
| **Mitra Space** | Backend proprio | Gerencia usuarios, workspaces e controle de acessos |

### Servicos parametrizaveis por cliente (on-premise possivel)

Estes servicos podem ser dedicados por cliente, rodando na nuvem da Mitra ou na nuvem do proprio cliente:

| Servico | Funcao | Parametrizacao |
|---------|--------|---------------|
| **E2B Sandbox** | Ambiente isolado onde a IA gera codigo React e faz build (Vite) | Cliente pode usar sua propria API key do E2B |
| **S3 Storage** | Armazena o build do frontend (codigo compilado, versionado) | Bucket dedicado por cliente (API Key + Secret Key) |
| **Backend Java (Mitra API)** | Backend de dados: MySQL, conexoes JDBC, server functions, integracoes | Instancia exclusiva por cliente (MITRA_BASE_URL_API) |
| **Servidor de Integracoes** | Armazena credenciais de APIs externas de forma apartada | Instancia exclusiva por cliente (MITRA_BASE_URL_INTEGRATIONS) |
| **S3 Backup** | Backup diario do banco de dados | Bucket dedicado por cliente |
| **S3 Mitra Drive** | Upload/download de arquivos dos projetos | Bucket dedicado por cliente |

### Firebase

O Firebase armazena:
- Conversas do agente de IA (historico de chats)
- Configuracoes de whitelabel (logos, cores, nomes)

---

## Fluxo detalhado

### 1. Prompt do usuario

O usuario (desenvolvedor) digita um prompt descrevendo o que quer construir ou alterar no sistema.

### 2. Frontend (mitra-nuxt)

A interface do desenvolvedor. Aqui ele:
- Envia prompts para a IA
- Ve o codigo sendo gerado em tempo real (streaming)
- Faz upload/download de arquivos (via S3 Mitra Drive)
- Gerencia projetos e configuracoes

### 3. Backend WebSocket (Node.js)

Recebe o prompt do frontend e orquestra a execucao:
- Envia o prompt para o sandbox E2B
- Recebe o output da IA em streaming
- Repassa em tempo real para o frontend

### 4. Sandbox E2B

Ambiente isolado (e2b.dev) onde:
- Uma CLI de IA gera codigo React
- O codigo e compilado com Vite (build acontece dentro do E2B)
- O output e streamado em tempo real de volta pro frontend
- Cada cliente pode usar sua propria API key do E2B

### 5. Storage (S3 + Firebase)

Apos o build:
- O codigo compilado e armazenado no **S3** com versionamento ativo
- Versoes anteriores ficam disponiveis por **7 dias** (politica de retencao)
- Metadata e conversas ficam no **Firebase**

### 6. Preview / Sistema final no browser

O sistema compilado e servido pro usuario. Usuarios Business acessam diretamente o sistema final (nao passam pelo mitra-nuxt).

### 7. Backend Java (Mitra API)

O sistema renderizado no browser se comunica com o Backend Java via SDKs:
- **mitra-sdk**: usada pela IA durante o desenvolvimento (criar/editar tabelas, actions, testar)
- **mitra-interactions-sdk**: usada pelo codigo do sistema final para chamar server functions

O Backend Java gerencia: banco MySQL, conexoes JDBC, server functions, API REST, integracoes externas.

### 8. Mitra Space

Gerencia apenas: usuarios, workspaces e controle de acessos. Nao lida com dados de projetos.

---

## Diagrama simplificado

```
[Usuario] --> [mitra-nuxt] --> [Backend WebSocket] --> [E2B Sandbox]
                                                          |
                                                    (streaming)
                                                          |
                                                    [Build Vite]
                                                          |
                                                    [S3 Storage]
                                                          |
                                                   [Sistema Final]
                                                          |
                                              [Backend Java - Mitra API]
                                                    /           \
                                              [MySQL]    [Servidor Integracoes]
                                                |
                                           [S3 Backup]
```

---

## Modelo on-premise / dedicado

Para clientes que exigem isolamento total, os seguintes componentes podem ser dedicados:

- **E2B**: API key propria do cliente
- **S3 Storage**: bucket dedicado
- **S3 Backup**: bucket dedicado
- **S3 Mitra Drive**: bucket dedicado
- **Backend Java**: instancia exclusiva
- **Servidor de Integracoes**: instancia exclusiva

Os componentes gerenciados (mitra-nuxt, backend WebSocket, Mitra Space) permanecem na infraestrutura da Mitra.

Nuvens homologadas para componentes dedicados: Azure, AWS, GCP. Specs e valores alinhados pelo time comercial.
