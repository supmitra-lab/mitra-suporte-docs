# SLA e Suporte

Documentacao sobre niveis de servico, canais de atendimento, gestao de vulnerabilidades e pen test. Aplica-se a todas as versoes.

---

## Canal de suporte

| Item | Detalhe |
|------|---------|
| **Canal oficial** | Portal de suporte Mitra: https://servicos.ai.mitralab.io/ |
| **Idioma** | Portugues |
| **Horario** | Comercial |

---

## SLA de resposta e resolucao

### Classificacao de severidade

| Severidade | Descricao | Exemplos |
|-----------|-----------|----------|
| **1 (critico)** | Indisponibilidade total ou impacto critico em operacao essencial | Plataforma fora do ar, perda de dados |
| **2 (alta)** | Impacto alto sem paralisacao total | Funcionalidade critica com erro, degradacao severa de performance |
| **3 (media)** | Impacto moderado | Funcionalidade secundaria com erro, comportamento inesperado |
| **4 (baixa)** | Baixo impacto ou duvida operacional | Duvidas de uso, ajustes cosmeticos, melhorias |

### Prazos

| Severidade | Tempo de resposta | Tempo de resolucao |
|-----------|-------------------|-------------------|
| 1 (critico) | 2h uteis | 8h uteis |
| 2 (alta) | 4h uteis | 2 dias uteis |
| 3 (media) | 1 dia util | 4 dias uteis |
| 4 (baixa) | 2 dias uteis | 5 dias uteis |

### Observacoes

- A Mitra reserva-se o direito de reavaliar a classificacao do chamado aberto
- Casos que dependam diretamente da participacao do cliente para resolucao nao terao o tempo de espera contado no prazo de resolucao

---

## Pen test

- A Mitra esta disponivel para que o cliente realize pen tests no ambiente
- Clientes anteriores ja realizaram pen tests na plataforma
- Os relatorios sao de propriedade dos respectivos clientes e nao sao compartilhados pela Mitra
- Caso o prospect/cliente deseje, pode realizar seu proprio pen test

---

## Gestao de incidentes de seguranca

Em caso de incidente de seguranca ou vazamento de dados, a Mitra se compromete a notificar o cliente conforme prazo definido em contrato (LGPD exige prazo razoavel).

O prazo especifico de notificacao e definido na clausula contratual entre Mitra e cliente.

---

## Plano de recuperacao de desastres (DRP)

Para detalhes completos sobre backup, RTO, RPO e cenarios de recuperacao, consulte o documento [geral-seguranca-lgpd-ia.md](geral-seguranca-lgpd-ia.md).

Resumo:
- **RTO**: ate 6 horas
- **RPO**: ate 24 horas
- Backups automaticos diarios
- Isolamento entre producao e backup em provedores distintos

---

## Certificacoes

A Mitra nao possui certificacoes ISO 27001, ISO 27701, SOC 2 Type II ou CSA STAR no momento.

A plataforma esta disponivel para auditorias e pen tests realizados pelo cliente ou auditor independente contratado.

---

## Duvidas frequentes

### "Qual o canal de suporte?"
Portal Mitra: https://servicos.ai.mitralab.io/ — atendimento em portugues, horario comercial.

### "Voces tem pen test pra compartilhar?"
Clientes anteriores ja realizaram pen tests, mas os relatorios sao de propriedade deles. Estamos disponiveis para que voce realize o seu.

### "Qual o SLA de disponibilidade?"
O indicador de disponibilidade (ex: 99,5%, 99,9%) e definido em contrato conforme negociacao entre Mitra e cliente. Consultar o time comercial.

### "E se o sistema ficar fora do ar?"
Severidade 1 (critico): resposta em ate 2h uteis, resolucao em ate 8h uteis. O DRP preve recuperacao total em ate 6 horas.
