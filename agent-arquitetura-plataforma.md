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
  -> Codigo-fonte commitado no GitHub
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
| **Firebase** | Firebase | Armazena conversas do agente de IA (historico de chats) e configuracoes de whitelabel (logos, cores, nomes) |

### Servicos parametrizaveis por cliente (on-premise possivel)

Estes servicos podem ser dedicados por cliente, rodando na nuvem da Mitra ou na nuvem do proprio cliente:

| Servico | Funcao | Parametrizacao |
|---------|--------|---------------|
| **E2B Sandbox** | Ambiente isolado onde a IA gera codigo React e faz build (Vite) | Cliente pode usar sua propria API key do E2B |
| **GitHub** | Hospeda o codigo-fonte dos projetos (React + server functions) com versionamento via commits | Organizacao dedicada do cliente em modelo on-premise |
| **S3 Storage** | Armazena o build do frontend (codigo compilado) | Bucket dedicado por cliente (API Key + Secret Key) |
| **Backend Java (Mitra API)** | Backend de dados: MySQL, conexoes JDBC, integracoes | Instancia exclusiva por cliente (MITRA_BASE_URL_API) |
| **Servidor de Integracoes** | Armazena credenciais de APIs externas de forma apartada | Instancia exclusiva por cliente (MITRA_BASE_URL_INTEGRATIONS) |
| **S3 Backup** | Backup diario do banco de dados | Bucket dedicado por cliente |
| **S3 Mitra Drive** | Upload/download de arquivos dos projetos | Bucket dedicado por cliente |

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
- Uma CLI de IA gera codigo React (componentes, telas, dashboards)
- O codigo e compilado com Vite (build acontece dentro do E2B)
- **As server functions do projeto tambem sao executadas dentro do E2B**
- O output e streamado em tempo real de volta pro frontend
- Cada cliente pode usar sua propria API key do E2B

**Lifecycle do sandbox:**
- Um sandbox e criado por prompt
- Timeout automatico estendido em 20 minutos a cada nova interacao do usuario
- Sandbox permanece ativo enquanto houver interacoes
- Ao cessar a atividade e vencer o timeout, o sandbox e encerrado automaticamente
- Sandbox encerrado e destruido (efemero) — nenhum dado persiste
- O codigo compilado (build) e armazenado no S3 antes do encerramento

**Isolamento:**
- O E2B usa microVMs Firecracker (mesma tecnologia da AWS Lambda)
- Cada sandbox tem seu proprio kernel Linux isolado, impedindo escapes entre sandboxes

**Acesso a dados (via SDK com JWT):**
- A IA no sandbox acessa dados do projeto via **SDK nativa do Mitra com autenticacao JWT**
- A SDK se comunica com o Backend Java da Mitra, que faz a leitura/escrita no banco
- Isso permite validacao e teste de queries durante o desenvolvimento
- O acesso respeita o modelo de permissoes do Mitra (RBAC + perfis de seguranca)
- A IA opera exclusivamente dentro do escopo autorizado para o usuario

**Acesso a internet:**
- O sandbox possui acesso a internet habilitado (necessario para comunicacao com o Backend Java da Mitra via SDK, API do provedor de LLM e pesquisas durante o desenvolvimento)
- Nao e possivel restringir esse acesso

**Token E2B:**
- O token da API do E2B fica exclusivamente no backend WebSocket (servidor Node.js)
- Nunca e exposto ao frontend/browser do usuario

**Logs:**
- O Mitra armazena apenas o historico de conversas (prompts e respostas) no Firebase
- Logs detalhados de execucao dentro do sandbox (comandos, chamadas de API) nao sao armazenados pelo Mitra
- O proprio E2B fornece metricas e logs via dashboard da plataforma

### 5. Storage (GitHub + S3 + Firebase)

**GitHub (codigo-fonte):**
- O codigo-fonte dos projetos (React + server functions) e commitado no GitHub
- Cada alteracao feita pela IA gera um commit, proporcionando historico completo de versionamento
- Em modelo on-premise, pode ser uma organizacao GitHub do proprio cliente
- O historico de commits permite restauracao granular de qualquer ponto no tempo (dentro da retencao)

**S3 (build):**
- O codigo compilado (build Vite) e armazenado no S3
- Versoes anteriores ficam disponiveis por **15 dias** (politica de retencao)
- O S3 armazena apenas o build, nao o codigo-fonte

**Firebase:**
- Conversas do agente de IA (historico de chats)
- Configuracoes de whitelabel

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
                                                +---------+---------+
                                                |                   |
                                         [Codigo-fonte]       [Build Vite]
                                            [GitHub]           [S3 Storage]
                                          (com commits)        (15 dias)
                                                                    |
                                                             [Sistema Final]
                                                                    |
                                                       [Backend Java - Mitra API]
                                                             /           \
                                                       [MySQL]    [Servidor Integracoes]
                                                          |
                                                    [S3 Backup]
                                                    (diario noturno)
```

---

## Modelo on-premise / dedicado

Para clientes que exigem isolamento total, os seguintes componentes podem ser dedicados:

- **E2B**: API key propria do cliente
- **GitHub**: organizacao dedicada do cliente (codigo-fonte dos projetos)
- **S3 Storage**: bucket dedicado
- **S3 Backup**: bucket dedicado
- **S3 Mitra Drive**: bucket dedicado
- **Backend Java**: instancia exclusiva
- **Servidor de Integracoes**: instancia exclusiva

Os componentes gerenciados (mitra-nuxt, backend WebSocket, Mitra Space) permanecem na infraestrutura da Mitra.

Nuvens homologadas para componentes dedicados: Azure, AWS, GCP. Specs e valores alinhados pelo time comercial.

### Backend Java (Mitra API) em on-premise

O Backend Java da Mitra e **dockerizado** — distribuido como imagem Docker, facilitando a implantacao em qualquer ambiente compativel.

**Pre-requisito de provedor de nuvem:**
A utilizacao de provedores consolidados de mercado — **AWS, Microsoft Azure ou Google Cloud Platform** — e obrigatoria, por oferecerem maior confiabilidade, escalabilidade e recursos nativos de seguranca.

**Requisitos minimos de infraestrutura da maquina:**

| Recurso | Especificacao |
|---------|--------------|
| **Processamento (CPU)** | Dimensionado com o time de suporte Mitra |
| **Memoria RAM** | Dimensionado com o time de suporte Mitra |
| **Armazenamento** | 500 GB NVMe |
| **Performance de disco** | Minimo de 250 MB/s para leitura e escrita sequencial |
| **Sistema Operacional** | Ubuntu Server 22.04 |

CPU e RAM devem ser dimensionados com o **time de suporte Mitra** conforme o volume esperado de uso, quantidade de usuarios simultaneos e perfil dos projetos a serem hospedados.

**Validacao da performance de disco:**

Para validar se o disco atende ao minimo de 250 MB/s, utilizar os comandos abaixo (requer a ferramenta `fio`):

Teste de leitura sequencial:
```bash
fio --name=teste-leitura-seq --ioengine=libaio --direct=1 --rw=read --bs=1M --size=2G --filename=teste.fio
```

Teste de escrita sequencial:
```bash
fio --name=teste-escrita-seq --ioengine=libaio --direct=1 --rw=write --bs=1M --size=2G --filename=teste.fio
```

Caso a performance medida seja inferior a 250 MB/s, sera necessario ajustar o tipo de disco (ex: gp3 com IOPS provisionados no AWS, Premium SSD no Azure, SSD com throughput configuravel no GCP).

---

## Backup, congelamento e restauracao

### Backup

Um projeto no Mitra Agent e composto por duas partes com estrategias de backup distintas:

| Componente | Metodo de backup | Retencao | RPO maximo |
|-----------|-----------------|----------|-----------|
| **Banco de dados** (dados, metadados) | Backup automatico diario (noturno) em S3 separado | Indefinida | 24 horas |
| **Codigo-fonte** (React + server functions) | GitHub com historico de commits | Permanente (historico de commits) | Segundos (cada alteracao gera commit) |
| **Build do frontend** (compilado Vite) | S3 com versionamento ativo | 15 dias de versoes | Minutos |

Isso significa que:
- Alteracoes no codigo-fonte (front + server functions) sao versionadas permanentemente no GitHub via commits, permitindo voltar a qualquer ponto historico
- O build compilado no S3 mantem 15 dias de versoes anteriores
- Alteracoes no banco de dados dependem do backup diario noturno (RPO de ate 24h)

### Congelamento automatico

Projetos sem acesso por 30 dias (baseado na tabela `INT_USERLOG`) sao elegiveis para congelamento automatico:

1. Backup completo e gerado antes do congelamento
2. Status do projeto muda para **Congelado**
3. Base de dados ativa e removida
4. Projeto deixa de estar disponivel para acesso

A tabela `INT_USERLOG` registra apenas acessos as telas de interface. Se usuarios nao acessarem telas, o projeto e considerado inativo mesmo que existam integracoes rodando.

### Exclusao de projetos

Quando um projeto e excluido:
- Backup adicional do banco e gerado no momento da exclusao
- O repositorio GitHub do projeto mantem o historico de commits
- O build permanece no S3 com versionamento por ate 15 dias
- Existe log interno de exclusao
- Em caso de exclusao acidental, acionar o suporte imediatamente

### Restauracao

**Restauracao so do codigo-fonte (front + server functions):**
Nao precisa acionar o suporte. Basta **solicitar a propria IA** — ela tem acesso ao repositorio GitHub do projeto e pode reverter para qualquer commit do historico.

**Restauracao completa (banco + codigo):**
Solicitar ao **Suporte Mitra** informando:
- Workspace ID, nome e ID do projeto
- O que precisa restaurar (banco, codigo ou ambos)
- Data/hora aproximada desejada

**Importante sobre sincronia entre codigo e banco:**
As retencoes sao independentes entre si. Exemplo: se o cliente solicita restauracao para o estado do projeto as **15h do dia X**, vale observar:
- O **ultimo backup do banco** disponivel sera o da **noite anterior** (dia X-1), pois o backup roda uma vez por noite
- O **codigo-fonte** pode ser restaurado com precisao de segundos via commit do GitHub

Portanto, em restauracoes completas, o banco sempre voltara para o estado da noite anterior a data desejada. Para recuperacoes mais granulares de codigo, a IA pode reverter commits especificos diretamente.

Formas de restauracao:
1. **Em um novo projeto** (recomendado) — para comparar ou recuperar sem sobrescrever
2. **Sobre o projeto original** — substitui completamente o estado atual (sem merge)
