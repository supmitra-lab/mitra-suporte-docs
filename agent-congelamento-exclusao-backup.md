# Congelamento, Exclusao e Backup de Projetos (Agent)

Documentacao sobre backup, restauracao e exclusao de projetos na versao Agent do Mitra. Esta versao tem diferencas arquiteturais em relacao ao MitraLab/MitraSheet.

---

## Diferenca arquitetural do Agent

Na versao Agent, um projeto e composto por **duas partes**:

| Componente | Onde fica | Backup |
|-----------|-----------|--------|
| **Banco de dados** (dados, metadados, server functions, configuracoes) | MySQL na nuvem | Backup automatico diario (noturno), armazenado em S3 separado |
| **Codigo do frontend** (telas, componentes visuais, layout) | AWS S3 com versionamento | Restauravel a qualquer momento dos ultimos **7 dias** |

Nas versoes MitraLab/MitraSheet, tudo ficava no banco de dados. No Agent, o frontend e armazenado separadamente no S3.

---

## Backup do banco de dados

- Backup automatico diario realizado durante a noite
- Armazenado em provedor independente (AWS S3), separado do ambiente de producao
- Nao existem snapshots ao longo do dia
- A restauracao considera o estado do banco no momento do backup noturno

O banco de dados contem: dados das tabelas, metadados, server functions (backend), Actions, integracoes, variaveis e configuracoes do projeto.

---

## Backup do codigo frontend

- O codigo do frontend fica em **AWS S3 com versionamento ativo**
- Cada alteracao no frontend gera uma nova versao automaticamente
- E possivel restaurar qualquer versao dos ultimos **7 dias** (politica de retencao)
- Apos 7 dias, as versoes anteriores sao removidas pela politica de lifecycle

Isso significa que:
- **Alteracoes no frontend** podem ser revertidas a qualquer momento dentro de 7 dias
- **Alteracoes no banco** dependem do backup diario noturno (RPO de ate 24h)

---

## Cenarios de restauracao

### Restaurar apenas o frontend (telas/layout)
- Possivel a qualquer momento dentro de 7 dias
- Solicitar ao suporte informando projeto e data/hora aproximada

### Restaurar apenas o banco (dados/configs)
- Possivel a partir do backup diario noturno
- Solicitar ao suporte informando projeto e data desejada

### Restauracao completa (frontend + banco)
- Combina os dois processos acima
- O frontend pode ser restaurado com mais granularidade (7 dias de versoes) do que o banco (backup diario)

---

## Exclusao de projetos

Quando um projeto e excluido:
- Backup adicional do banco e gerado no momento da exclusao
- O codigo do frontend permanece no S3 com versionamento por ate 7 dias
- Existe log interno de exclusao
- Em caso de exclusao acidental, acionar o suporte imediatamente

---

## Congelamento automatico

O processo de congelamento funciona da mesma forma que nas outras versoes:

1. Projetos sem acesso por 30 dias (baseado na tabela `INT_USERLOG`) sao elegiveis
2. Backup e gerado antes do congelamento
3. Base de dados ativa e removida
4. Status muda para **Congelado**

---

## Como solicitar restauracao

Entrar em contato com o **Suporte Mitra** informando:
- Workspace ID
- Nome e ID do projeto
- O que precisa restaurar (frontend, banco ou ambos)
- Data/hora aproximada desejada

### Formas de restauracao disponiveis

1. **Em um novo projeto** (recomendado) — ideal para comparar ou recuperar sem sobrescrever
2. **Sobre o projeto original** — substitui completamente o estado atual (sem merge)

---

## Resumo comparativo de backup

| Aspecto | Banco de dados | Frontend |
|---------|---------------|----------|
| Frequencia | Diario (noturno) | Cada alteracao (versionamento) |
| Retencao | Indefinida (backups armazenados) | 7 dias de versoes |
| Granularidade | Por dia | Por alteracao |
| RPO maximo | 24 horas | Minutos/horas |
| Server functions | Incluidas no banco | N/A |
