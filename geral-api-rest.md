# API REST do Mitra

Documentacao sobre a API REST da plataforma para integracao com sistemas externos. Aplica-se a todas as versoes.

---

## Visao geral

O Mitra disponibiliza uma API REST que permite integrar dados do projeto com qualquer sistema externo. Por meio da API e possivel:

- Consultar dados de tabelas do projeto (GET)
- Inserir novos registros (POST)
- Atualizar registros existentes (PUT)
- Remover registros (DELETE)
- Disparar Actions remotamente (POST)
- Gerenciar usuarios (via tabela INT_USER)

---

## Criacao de chaves de API

1. Acesse **Configuracoes Gerais** do projeto
2. Localize a secao **API**
3. Crie uma nova chave
4. O Mitra gera automaticamente a **URL base** e a **chave privada (token)**

### Controle de tabelas expostas

Nas configuracoes da API, o usuario define **quais tabelas** do banco estarao disponiveis via API. Somente tabelas explicitamente autorizadas podem ser acessadas. Isso e fundamental para seguranca e governanca de dados.

---

## Autenticacao

Todas as chamadas exigem o header:

```
Authorization: Bearer {chave_privada}
```

Sem esse header, a requisicao sera rejeitada.

---

## Consultar dados (GET)

```
GET {url_base}/rest/v0/{tableName}
```

### Filtros

Filtros sao passados como query parameters:

| Operador | Descricao | Exemplo |
|----------|-----------|---------|
| (sem sufixo) | Igual | `status=Ativo` |
| `_neq` | Diferente | `status_neq=Inativo` |
| `_gt` | Maior que | `horas_gt=4` |
| `_gte` | Maior ou igual | `horas_gte=8` |
| `_lt` | Menor que | `horas_lt=10` |
| `_lte` | Menor ou igual | `data_lte=2023-12-31` |
| `_like` | Contem | `nome_like=Joao` |

### Paginacao e ordenacao

| Parametro | Descricao | Default |
|-----------|-----------|---------|
| `page` | Pagina dos resultados | 0 |
| `size` | Registros por pagina | 10 |
| `sort` | Coluna de ordenacao | - |
| `order` | Direcao (`ASC` ou `DESC`) | ASC |

### Exemplo completo

```bash
curl -X GET "{url_base}/rest/v0/Task?status=Ativo&horas_gt=4&page=1&size=20&sort=horas&order=DESC" \
  -H "Authorization: Bearer {token}"
```

---

## Inserir dados (POST)

```
POST {url_base}/rest/v0/{tableName}
```

O body e um JSON array onde cada objeto e um registro:

```bash
curl -X POST "{url_base}/rest/v0/Task" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '[{
    "Descricao": "Nova tarefa",
    "ID Solicitante": "12345",
    "Horas": "10",
    "Data de Abertura": "2023-09-01"
  }]'
```

---

## Atualizar dados (PUT)

```
PUT {url_base}/rest/v0/{tableName}
```

O JSON deve conter os campos que identificam e atualizam o registro:

```bash
curl -X PUT "{url_base}/rest/v0/Task" \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '[{
    "Descricao": "Tarefa atualizada",
    "ID Solicitante": "12345",
    "Horas": "12"
  }]'
```

---

## Deletar dados (DELETE)

```
DELETE {url_base}/rest/v0/{tableName}/{id}
```

O `id` e o identificador unico do registro na tabela.

```bash
curl -X DELETE "{url_base}/rest/v0/Task/12345" \
  -H "Authorization: Bearer {token}"
```

---

## Executar Actions via API

Permite disparar Actions remotamente, util para integracoes evento-driven e automacoes externas.

```
POST {url_base}/rest/v0/action/init/{action_id}
```

```bash
curl -X POST "{url_base}/rest/v0/action/init/42" \
  -H "Authorization: Bearer {token}"
```

A Action e iniciada imediatamente apos a chamada.

---

## Gestao de usuarios via API

E possivel gerenciar usuarios do Mitra via API, liberando a tabela `INT_USER` nas configuracoes. Isso permite integrar a gestao de usuarios com sistemas externos (ex: criar/remover usuarios a partir do RH ou do AD).

A tabela INT_USER pode ser consultada e manipulada como qualquer outra tabela exposta (GET, POST, PUT, DELETE).

---

## Casos de uso comuns

- Sincronizacao bidirecional com ERP ou CRM
- Alimentacao de data lakes
- Automacao de processos externos
- Disparo de Actions a partir de eventos em outros sistemas
- Gestao centralizada de usuarios

---

## Boas praticas

- Exponha apenas as tabelas necessarias
- Trate a chave de API como informacao sensivel (nao versionar, nao expor em frontend)
- Use filtros e paginacao para evitar cargas excessivas
- Evite operacoes destrutivas (DELETE) sem validacao previa
- Registre e monitore chamadas no sistema externo
