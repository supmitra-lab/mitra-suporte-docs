# Congelamento, Exclusao e Backup de Projetos (MitraLab / MitraSheet)

Documentacao sobre congelamento automatico, exclusao, backup, restauracao e duplicacao de projetos nas versoes MitraLab e MitraSheet.

---

## Backup automatico diario

Todos os projetos possuem backup automatico diario realizado durante a noite.

- O backup e feito **uma vez por dia** (noturno)
- Nao existem snapshots ao longo do dia
- A restauracao considera o estado do projeto no momento do backup noturno
- Nao e possivel restaurar um horario especifico dentro do mesmo dia

Nestas versoes, **tudo sobre o projeto esta no banco de dados** (dados, metadados, telas, componentes, Actions, server functions, integracoes, configuracoes). Por isso, o backup do banco e suficiente para restauracao completa.

---

## Congelamento automatico

O congelamento e um processo automatico que libera recursos quando um projeto fica muito tempo sem uso.

### Quando um projeto e congelado?

Todas as condicoes abaixo devem ser atendidas:
1. O projeto **nao foi acessado por 30 dias** (sem registros na tabela `INT_USERLOG`)
2. O projeto **nao e recente** (projetos recem-criados nao sao congelados)

### O que acontece ao congelar

1. Backup completo e gerado
2. Status do projeto muda para **Congelado**
3. Base de dados ativa e removida
4. Projeto deixa de estar disponivel para acesso

### Como o Mitra identifica inatividade

O sistema usa a tabela **INT_USERLOG**, que registra acessos dos usuarios as telas de interface. Se nao houver acessos as telas, o projeto e considerado inativo — mesmo que existam integracoes rodando em segundo plano.

### Processo tecnico

1. **Verificacao** (a cada hora): identifica projetos sem acesso recente
2. **Fila de congelamento**: projetos elegiveis sao marcados (nenhum dado removido ainda)
3. **Congelamento efetivo** (entre 00:00 e 05:00): ultima validacao + backup + congelamento

### Uso da INT_USERLOG para dashboards

A tabela INT_USERLOG tambem pode ser usada para criar dashboards de utilizacao: acessos por usuario, por tela, por data, engajamento ao longo do tempo.

---

## Exclusao de projetos

Quando um projeto e excluido:
- Um **backup adicional** e gerado no momento da exclusao
- Existe log interno de exclusao
- Nao ha disparo automatico de email ao excluir
- Em caso de exclusao acidental, acionar o suporte o mais rapido possivel

---

## Restauracao de backup

A restauracao deve ser solicitada ao **Suporte Mitra** informando:
- Workspace ID
- Nome e ID do projeto (quando conhecido)
- Data aproximada desejada

### Formas de restauracao

**1. Em um novo projeto (recomendado)**
O backup e restaurado como um novo projeto (mesmo workspace ou outro). Ideal para analisar dados, comparar versoes ou recuperar funcionalidades sem sobrescrever o projeto atual.

**2. Preservando ou restringindo usuarios**
- Restaurar com todos os usuarios originais
- Ou restaurar apenas para administradores do workspace

**3. Sobre o projeto original**
Sobrescreve o projeto atual. Tudo desenvolvido apos a data do backup sera **substituido completamente**. Nao ha merge. Usar com cautela.

### O que o backup contem

O backup e do projeto inteiro: dados, metadados, telas, componentes, Actions, dbactions, variaveis, integracoes, configuracoes e permissoes. Nao existe backup parcial.

---

## Duplicacao de projetos

O Mitra permite duplicar projetos pela interface:

1. Clicar em **Duplicar Projeto**
2. Selecionar o workspace de destino (apenas workspaces com permissao de criacao)
3. O sistema gera backup instantaneo e restaura no destino

A duplicacao inclui tudo: dados, telas, componentes, Actions, variaveis, integracoes e configuracoes.

---

## Boas praticas

- Evitar exclusao de projetos em producao sem validacao previa
- Manter controle interno de responsaveis por projetos
- Em caso de exclusao acidental, acionar suporte imediatamente
- Para restauracoes criticas, preferir restaurar em projeto paralelo antes de sobrescrever producao
