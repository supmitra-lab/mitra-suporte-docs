# Governanca de IA (LLM)

Documentacao sobre como o Mitra utiliza modelos de linguagem (LLM), arquitetura de IA, privacidade, custos e controles. Aplica-se a todas as versoes.

---

## Modelo agnostico

A plataforma Mitra e **agnostica quanto ao provedor de LLM**. Disponibilizamos integracao com os modelos que estiverem melhor desempenhando nos benchmarks do mercado.

Provedores suportados atualmente:
- **Anthropic** (Claude) — via chave de API do cliente
- **OpenAI** (GPT) — via subscricao do cliente

O cliente conecta sua propria chave de API ou subscricao. A Mitra **nao e intermediaria** — a relacao contratual e direta entre o cliente e o provedor de LLM.

---

## Arquitetura de IA

| Aspecto | Resposta |
|---------|----------|
| Tecnica utilizada | **Prompt engineering puro** |
| Fine-tuning | Nao |
| RAG (Retrieval-Augmented Generation) | Nao |
| Indices vetoriais / embeddings | Nao |
| Modelo proprietario | Nao |
| Acesso a dados | Via **SDK nativa do Mitra** com autenticacao JWT (somente Agent) |

Nao ha embeddings dos dados do cliente armazenados em nenhum lugar. Cada sessao de chat com a IA comeca do zero, sem retencao de contexto de sessoes anteriores.

### Acesso a dados pela IA (apenas Agent)

Na versao Agent, a IA acessa dados do projeto em tempo real via **SDK nativa do Mitra com autenticacao JWT** durante o desenvolvimento. A SDK se comunica diretamente com o Backend Java da Mitra, que faz a leitura/escrita no banco do projeto.

Isso permite que a IA:

- Consulte estrutura e conteudo das tabelas
- Valide queries e transformacoes
- Teste o comportamento do codigo gerado com dados reais

O acesso respeita o modelo de permissoes do Mitra (RBAC + perfis de seguranca). A IA opera exclusivamente dentro do escopo autorizado para o usuario que esta desenvolvendo — nao tem acesso a dados fora do escopo dele.

Dados acessados via SDK ficam disponiveis apenas durante a sessao ativa do sandbox. Apos encerramento (por timeout sem interacoes), nenhum dado persiste — os sandboxes sao efemeros.

---

## Privacidade e treinamento

### Dados usados para treinamento?
**Nao.** A Mitra nao utiliza dados, prompts ou respostas dos clientes para treinamento ou fine-tuning de nenhum modelo em nenhuma hipotese.

### E o provedor de LLM?
O provedor e contratado diretamente pelo cliente. Para referencia:

**Anthropic (API comercial):**
- Dados de API nao sao usados para treinamento por padrao
- Retencao padrao: 7 dias (inputs e outputs), depois deletados
- Opcao de Zero Data Retention (ZDR) para clientes enterprise

**OpenAI (API comercial):**
- Dados de API nao sao usados para treinamento por padrao
- Retencao padrao: 30 dias para monitoramento de abuso
- Opcao de Zero Data Retention (ZDR) para clientes enterprise

Cabe ao cliente negociar as condicoes de retencao e uso de dados diretamente com o provedor.

### Retencao de conhecimento entre sessoes?
**Nao.** Como a arquitetura e prompt engineering puro (sem fine-tuning, sem RAG), a IA nao retém nenhum conhecimento do cliente entre sessoes. Ao encerrar o contrato, nao resta nenhum "aprendizado" sobre o negocio do cliente nos modelos.

---

## Controle de acesso da IA

A IA opera **dentro do escopo de permissao do usuario**:

- Um usuario **Business** com acesso restrito a determinadas telas nao consegue, via linguagem natural, acessar dados fora do seu escopo
- **Perfis de seguranca** com row-level security aplicam filtros automaticos e irremovíveis
- Nao e possivel gerar dashboards ou acessar dados fora do escopo autorizado pelo modelo de permissoes

### Segregacao de funcoes (SoD)
A ferramenta respeita a SoD. Um usuario com perfil de leitura nao consegue, via linguagem natural, gerar dashboards expondo dados aos quais normalmente nao teria acesso.

---

## Anonimizacao de PII

Nao ha camada automatica de mascaramento/anonimizacao de PII antes do envio ao LLM.

O controle e feito via:
- Modelo de permissoes (RBAC + perfis de seguranca)
- Configuracao do projeto: o desenvolvedor define quais tabelas e campos ficam acessiveis ao agente de IA

---

## Mitigacao de alucinacoes

O agente de IA opera dentro do contexto do projeto, com acesso a estrutura das tabelas e metadados configurados. O usuario pode orientar o comportamento da IA por meio de instrucoes personalizadas (modo operandi), incluindo a pratica de alinhar acoes com o usuario antes de executar.

Nao ha uma camada sistemica de validacao automatizada obrigatoria antes da execucao.

---

## Explicabilidade

- O usuario pode interagir com a IA de forma conversacional, solicitando explicacoes sobre o que sera feito antes da execucao
- O desenvolvedor pode revisar e ajustar qualquer componente criado pela IA antes de publicar para usuarios Business
- E possivel configurar o agente para sempre apresentar o plano de acao antes de executar

---

## Human-in-the-loop

As criacoes via linguagem natural sao aplicadas diretamente no ambiente de desenvolvimento. O fluxo natural e:

```
IA cria no ambiente de dev
  -> Desenvolvedor revisa e ajusta
  -> Publica para usuarios Business
```

O desenvolvedor tem controle total para revisar, ajustar ou reverter qualquer componente criado pela IA antes de disponibilizar para usuarios finais.

---

## Drift do modelo

Nao aplicavel. Nao ha fine-tuning nem modelo proprietario. O modelo e fornecido e atualizado pelo provedor de LLM (Anthropic/OpenAI). A Mitra acompanha os benchmarks e disponibiliza integracao com os modelos de melhor desempenho.

---

## Custos de IA

| Aspecto | Resposta |
|---------|----------|
| A Mitra cobra pelo uso da IA? | **Nao** |
| Modelo de contrato | Por quantidade de usuarios |
| Quem paga o consumo de tokens? | O **cliente**, diretamente ao provedor (Anthropic ou OpenAI) |
| Cotas/throttling | Gerenciados pelo proprio provedor de LLM, conforme plano do cliente |

Nao ha cobranca adicional da Mitra por volume de uso da IA.

---

## Duvidas frequentes

### "A IA pode acessar dados que o usuario nao deveria ver?"
Nao. A IA opera dentro do escopo de permissao do usuario. Se o usuario e Business com acesso restrito, a IA so ve o que ele ve.

### "Se eu cancelar o contrato, a IA ainda 'sabe' coisas sobre meu negocio?"
Nao. A arquitetura e prompt engineering puro — nao ha fine-tuning nem embeddings. A IA nao retém nenhum conhecimento entre sessoes.

### "Quem paga pelo uso da IA?"
O cliente paga diretamente ao provedor de LLM (Anthropic ou OpenAI) pelo consumo de tokens. A Mitra nao cobra nada adicional pelo uso do agente de IA.

### "Posso trocar de provedor de LLM?"
Sim. A plataforma e agnostica. Atualmente suporta Anthropic e OpenAI, e novos provedores podem ser adicionados conforme evoluam nos benchmarks.
