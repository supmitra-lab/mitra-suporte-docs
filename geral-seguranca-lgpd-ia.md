# Seguranca, LGPD e Privacidade

Documentacao sobre controles de seguranca, adequacao a LGPD, privacidade de dados e politicas relacionadas ao uso de IA. Aplica-se a todas as versoes.

---

## Criptografia

### Em transito
- Todas as comunicacoes utilizam HTTPS/TLS
- Versao do TLS gerenciada pelo provedor de nuvem

### Em repouso
- Criptografia de disco (server-side encryption do provedor de nuvem, que utiliza AES-256 por padrao)
- Senhas de usuarios: criptografadas com BCrypt
- Credenciais de conexoes JDBC: criptografadas no banco

### Gestao de chaves
- Chaves gerenciadas pelo provedor de nuvem (Microsoft-managed keys)
- Nao ha suporte a BYOK (Bring Your Own Key) ou HYOK (Hold Your Own Key) no momento

---

## Autenticacao

O Mitra oferece tres formas de autenticacao:

1. **Cadastro direto**: email e senha (senha criptografada com BCrypt)
2. **SSO com Google**: OAuth 2.0
3. **SSO com Microsoft**: OAuth 2.0 com Microsoft Entra ID (Azure AD)

Para detalhes completos sobre autenticacao, SSO e permissoes, consulte o documento [geral-login-permissoes-acessos.md](geral-login-permissoes-acessos.md).

### MFA
- Nao ha MFA nativo no cadastro direto da plataforma
- Quando o login e via SSO (Microsoft ou Google), o MFA e gerenciado pelo provedor de identidade do proprio cliente (ex: Azure AD com MFA habilitado, politicas de acesso condicional)
- O Mitra respeita e nao contorna essas politicas

### SAML 2.0
- Nao ha suporte nativo. Pode ser desenvolvido sob demanda como customizacao

### SCIM
- Nao ha provisionamento/desprovisionamento automatico via SCIM
- A gestao de usuarios e manual (convite por email, remocao pelo administrador)
- Quando usando SSO com Azure AD, usuarios desativados no AD perdem acesso automaticamente (token JWT deixa de ser validado)

---

## LGPD e privacidade

### Papeis LGPD
- **Controlador**: o cliente (contratante) e o controlador dos dados pessoais
- **Operador**: a Mitra atua como operadora dos dados, nos termos da LGPD

### Propriedade dos dados
- Todos os dados inseridos, gerados ou processados pelo cliente na plataforma sao de **propriedade exclusiva do cliente**
- O cliente pode solicitar exportacao completa de seus dados a qualquer momento, em formato acessivel e estruturado

### Localizacao dos dados
- Infraestrutura hospedada em nuvens globais com datacenter no **Brasil**
- Backups armazenados em provedor independente, tambem no Brasil

### Transferencia internacional
- Os dados da plataforma (banco, projetos, dashboards) ficam no Brasil
- Os prompts enviados a IA podem ser processados fora do Brasil, dependendo da infraestrutura do provedor de LLM escolhido pelo cliente (Anthropic/OpenAI)
- A relacao contratual com o provedor de IA e direta entre o cliente e o provedor

### Subprocessadores
| Subprocessador | Funcao | Localizacao |
|---------------|--------|-------------|
| Provedor de nuvem (atualmente Azure) | Infraestrutura da plataforma | Brasil |
| AWS S3 | Armazenamento de backups | Brasil |
| Provedor de LLM (Anthropic ou OpenAI) | Processamento de IA | Definido pelo cliente |

A Mitra se compromete a notificar o cliente em caso de alteracao de subprocessadores.

### Retencao e descarte
- Em caso de encerramento do contrato, a Mitra disponibiliza exportacao completa em ate **5 (cinco) dias uteis**
- Apos confirmacao de recebimento pelo cliente, procede com **exclusao definitiva** dos dados dos servidores

### Direitos do titular (LGPD Art. 18)
- O cliente, como controlador, e responsavel pelo atendimento aos direitos dos titulares
- A Mitra, como operadora, fornece os meios tecnicos para acesso, correcao e eliminacao de dados

### Uso de dados para treinamento de IA
- A Mitra **nao utiliza** dados, prompts ou respostas dos clientes para treinamento ou fine-tuning de nenhum modelo, proprio ou de terceiros, em nenhuma hipotese
- O provedor de LLM e contratado diretamente pelo cliente
- Ambos provedores suportados (Anthropic e OpenAI) garantem por padrao que dados de API comercial nao sao usados para treinamento
- Ambos oferecem opcoes de Zero Data Retention para clientes enterprise

### Anonimizacao e mascaramento de PII
- Nao ha camada automatica de mascaramento/anonimizacao de PII antes do envio ao LLM
- O controle sobre quais dados sao acessiveis pela IA e feito via modelo de permissoes da plataforma (RBAC + perfis de seguranca com row-level security)
- Cabe ao cliente definir, na configuracao do projeto, quais tabelas e campos ficam acessiveis ao agente de IA

---

## Certificacoes

A Mitra nao possui certificacoes ISO 27001, ISO 27701, SOC 2 Type II ou CSA STAR no momento.

A plataforma esta disponivel para:
- Auditorias realizadas pelo cliente ou auditor independente contratado
- Pen tests realizados pelo cliente

Clientes anteriores ja realizaram pen tests, porem os relatorios sao de propriedade dos respectivos clientes e nao sao compartilhados pela Mitra.

---

## Plano de recuperacao de desastres (DRP)

### Estrategia de backup
- Backups automaticos diarios do banco de dados
- Armazenamento em provedor independente do ambiente de producao
- Principio de isolamento: producao em um provedor, backups em outro

### Cenario 1: indisponibilidade do ambiente de producao
- Backups permanecem disponiveis no provedor de backup
- Procedimento: provisionamento de nova instancia + restauracao do banco
- **RTO**: ate 6 horas
- **RPO**: ate 24 horas (perda maxima dos dados gerados apos o ultimo backup diario)

### Cenario 2: indisponibilidade do provedor de backup
- Aplicacao continua operando normalmente
- Nao ha impacto para usuarios finais
- Apos restabelecimento, backups sao retomados automaticamente

---

## Logs e auditoria

| Tipo de log | O que registra |
|------------|----------------|
| Acesso ao sistema | Usuario, data/hora, tempo de sessao |
| Execucao de acoes | Usuario, input, output das acoes de backend |
| Historico de chats | Conversas com a IA, exportavel por usuario |

- Nao ha integracao nativa com SIEM (ex: Splunk, Elastic)
- Os logs podem ser exportados sob demanda

---

## Gestao de vulnerabilidades

SLA de correcao por severidade:

| Severidade | Descricao | Tempo de resposta | Tempo de resolucao |
|-----------|-----------|-------------------|-------------------|
| 1 (critico) | Indisponibilidade total ou impacto critico | 2h uteis | 8h uteis |
| 2 (alta) | Impacto alto sem paralisacao total | 4h uteis | 2 dias uteis |
| 3 (media) | Impacto moderado | 1 dia util | 4 dias uteis |
| 4 (baixa) | Baixo impacto ou duvida operacional | 2 dias uteis | 5 dias uteis |

- A Mitra reserva-se o direito de reavaliar a classificacao do chamado
- Casos que dependam de participacao do cliente nao contam no prazo de resolucao
