# Onboarding de Parceiros Whitelabel (Agent)

Guia padrao para configuracao de novos parceiros whitelabel na versao Agent do Mitra. Cobre o que o parceiro precisa enviar, o que configuramos internamente e o fluxo completo ate o app ficar disponivel.

---

## O que e whitelabel no Mitra?

Um parceiro whitelabel recebe uma versao personalizada do Mitra Agent com:
- Dominio customizado (ex: `app.parceiro.com.br` no lugar de `agent.mitralab.io`)
- Logo e cores da marca do parceiro
- Nome personalizado do assistente de IA
- Experiencia de uso com a identidade visual do parceiro

---

## O que o parceiro precisa enviar

| Item | Especificacao | Obrigatorio |
|------|--------------|-------------|
| **Dominio customizado** | Ex: `app.parceiro.com.br` | Sim |
| **Logo principal** | Quadrada, proporcao 1:1 | Sim |
| **Logo do header** | Retangular, proporcao ~3:1 | Sim |
| **Logo da IA** | Quadrada, proporcao 1:1 | Sim |
| **Cor primaria** | Codigo hex (ex: `#2E5090`) | Sim |
| **Nome do assistente IA** | Substitui o nome padrao da IA | Sim |
| **Texto de entrada** | Texto inicial para criacao de projetos | Sim |

**Sobre o dominio:** alinhar com o parceiro se usara subdominio ou dominio raiz:
- Com subdominio: `app.parceiro.com.br`, `sistema.parceiro.com.br`
- Sem subdominio: `parceiro.com.br`

Se o parceiro informar apenas o dominio raiz, confirmar a preferencia.

---

## Fluxo completo

```
1. Parceiro envia informacoes e assets
          |
          v
2. Mitra configura no Firebase (colecao do ambiente correto)
          |
          v
3. Mitra testa o app (tema, login, navegacao)
          |
          v
4. Parceiro informa dominio desejado
          |
          v
5. Mitra cria a infraestrutura e repassa informacoes DNS ao parceiro
          |
          v
6. Parceiro cadastra DNS no provedor dele (ex: registro CNAME)
          |
          v
7. Mitra valida o vinculo do dominio
          |
          v
8. App disponivel no dominio do parceiro
```

---

## Configuracao interna (processo Mitra)

1. Criar documento na colecao Firebase (`whitelabel-apps-{env}` ou `custom-clients`)
2. Preencher campos do `WhitelabelApp` com as informacoes do parceiro
3. Testar deteccao de dominio e carregamento do tema
4. Validar login, SSO e fluxo de primeiro acesso

---

## Dominio customizado — detalhes tecnicos

1. Parceiro informa o dominio desejado
2. Mitra cria a infraestrutura necessaria
3. Mitra repassa ao parceiro as informacoes DNS (ex: registro CNAME, destino)
4. Parceiro cadastra o DNS no provedor de dominio dele
5. Mitra valida que o dominio esta apontando corretamente

---

## Troubleshooting

| Problema | Causa provavel | Verificacao |
|----------|---------------|-------------|
| **Tema nao carrega** | Nome do documento incorreto no Firebase | Verificar colecao e nome do doc |
| **Logo nao aparece** | URL nao publica ou problema de CORS | Verificar URL e headers |
| **Login falha** | Configuracao de SSO incorreta | Verificar `enableSSOAuth`, `autoLoginURL`, dominio no JWT |
| **Dominio nao resolve** | DNS nao cadastrado pelo parceiro | Verificar com parceiro se o registro foi criado |

---

## Checklist de onboarding

- [ ] Receber assets do parceiro (logos, cor, nome IA, texto)
- [ ] Confirmar dominio desejado (com ou sem subdominio)
- [ ] Criar documento no Firebase
- [ ] Testar tema, login e navegacao
- [ ] Criar infraestrutura de dominio
- [ ] Repassar informacoes DNS ao parceiro
- [ ] Parceiro cadastrar DNS
- [ ] Validar vinculo do dominio
- [ ] Confirmar app disponivel
