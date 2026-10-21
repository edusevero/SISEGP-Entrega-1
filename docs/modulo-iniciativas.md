# Módulo Iniciativas do MVP

O SISEGP centraliza o cadastro e o acompanhamento das iniciativas do EGP/TJRS. Nesta entrega, criação, edição e consulta usam `InitiativeEditor.mount()` em `assets/js/initiative-editor.js`. São scripts clássicos, sem framework ou backend.

## Navegação e componentes

- `#/initiative/new`: exatamente cinco abas — Dados da Iniciativa, Classificação, Equipe, Documentos e Status Report.
- `#/initiative/:id` e `#/initiative/:id/edit`: os mesmos formulários, precedidos por Visão Geral. Essa aba é inicial e só aparece para registros existentes.
- Abas usam `role=tab`, painéis associados e teclado com setas, Home e End. Todas ficam no mesmo formulário; trocar de aba não recria os campos.
- Alterações são rascunhos em memória. Salvar iniciativa confirma o conjunto. Há indicação de pendências, confirmação para sair internamente e `beforeunload` na recarga/fechamento.

## Campos e distribuição

O grid tem 12 colunas. Em telas até 1250/980/720/430 px, os campos são redistribuídos, com uma coluna no celular.

| Card | Linhas no desktop |
| --- | --- |
| Dados da Iniciativa | Nome (8), Tipo (4); Departamento (2), Setor (2), Solicitante (3), Patrocinador (3), SEI (2); Escopo (6), Objetivo/Descrição (6); Benefício (6), Observações (6). |
| Administração | Status administrativo (4), conclusão numérica (2), progresso (6); Áreas Envolvidas (6), Partes Interessadas (6). |
| Anexos da Iniciativa | Card separado, abaixo da Administração, com seleção múltipla e tabela compacta. |
| Classificação | DITIC, Estratégica, Eixo; Biênio PDTIC, Ciclo, ENTICJUD; GP, Responsável, Objetivo Estratégico. Cada campo ocupa 4 colunas. |
| Priorização | Criticidade, Complexidade, Importância, Necessidade; Impacto no PE, Determinação Legal, Prazo, Orçamento. Cada campo ocupa 3 colunas. |
| Estimativa de Horas | HST numérica não negativa, independente da equipe, com unidade horas. |
| Equipe | Nome (5), Papel (3), HST (3), botão primário compacto (1). |
| Status Report | Tipo, Data, Status da Iniciativa, Responsável (3 cada); Realizações, Impedimentos, Próximas Atividades (4 cada). |

Nome, solicitante, patrocinador, responsável e SEI são textos livres; SEI não tem máscara ou validação de domínio definida. Os quatro textos longos de Dados usam 3 linhas; os três textos do Status Report usam 5 linhas, com redimensionamento vertical. Tipo, departamento, setor, classificações, GP, status e tipo/situação do relatório são selects provenientes dos cadastros auxiliares. Nome da iniciativa é obrigatório.

## Regras de interação

- Setor pertence ao departamento. Trocar departamento limpa setor incompatível. Setores inativos já vinculados são preservados; novos vínculos oferecem registros ativos.
- Áreas usam `details/summary` com lista de checkboxes, resumo compacto, até dois chips e contador adicional. Abrir mostra todos os ativos e vínculos inativos/legados; cada seleção pode ser removida. Escape fecha a lista. Partes interessadas permitem inclusão por botão/Enter e exclusão individual; nomes repetidos não são adicionados.
- Conclusão aceita valores finitos entre 0 e 100. A barra atualiza imediatamente. Ela não muda automaticamente o status.
- Equipe exige nome e HST numérica não negativa. O botão `+` tem tooltip/nome acessível; ao editar passa a `✓`/Salvar integrante. Após confirmar, limpa os campos. Não existe Cancelar edição. Duplicatas por mesmo nome e papel são impedidas; excluir exige confirmação.
- Status Report exige tipo, data válida, situação e GP. O label é exatamente **Status da Iniciativa**, mas seus valores são de acompanhamento. Não existe Cancelar edição; após confirmar inclusão/edição, os campos são limpos e o botão retorna ao modo de inclusão. A tabela é ordenada por data decrescente e permite visualizar, editar e excluir mediante confirmação.
- Integrantes e relatórios que ainda estão nos campos superiores devem ser confirmados nos botões próprios antes de salvar a iniciativa.

## Visão Geral

`MvpViews.overview()` apresenta uma capa azul institucional com nome, código, tipo, departamento/setor, badge de status e barra do percentual salvo. GP, HST e contagens de equipe, arquivos e relatórios usam cards claros separados com ícones. Mostra objetivo/descrição, escopo, benefício, DITIC, biênio, eixo, ciclo, SEI, áreas e partes interessadas. O último relatório é escolhido pela data, com resumos de até 240 caracteres e ação para o conteúdo completo. Ausências aparecem como Não informado ou estado vazio. Não calcula datas de cronograma, riscos ou decisões fictícias. Após salvar, o editor é reaberto na Visão Geral com os novos dados. Cores e padrões estão em [Interface do MVP](interface-mvp.md).

## Modelo, compatibilidade e arquivos

As iniciativas continuam em `sisegp-data-v2`. `mvp.classification` contém `ditic`, `strategic`, `biennium`, `axis`, `cycle`, `owner`, `enticjud`, `objective`, `criticality`, `complexity`, `importance`, `necessity`, `impact`, `legal`, `deadline`, `budget`, `hst`. `manager` identifica o GP. `mvp.members` contém objetos com nome/papel/HST; `mvp.areas` e `mvp.stakeholders` são arrays. As coleções legadas permanecem sem exclusão.

`administrativeStatus` guarda o ciclo administrativo. O campo legado `status` recebe a forma compatível quando há equivalência conhecida, ou o texto do novo status cadastrado. A situação legada Atrasada não implica um ciclo administrativo; preserve-a até escolha explícita. `statusReports[].monitoringStatus` é independente; relatórios também têm `type`, `owner`, `date`, `achievements`, `impediments`, `nextSteps`. Não inventar migração entre situações distintas.

`catalogRefs` guarda IDs dos auxiliares, incluindo referências de classificação e áreas. Relatórios têm referências próprias. Aliases e IDs permitem ler rótulos renomeados sem substituir indiscriminadamente texto livre. Ao salvar uma seleção alterada, `bindRecord()` recalcula as referências para não restaurar a opção antiga.

Anexos e documentos compartilham `files`, diferenciados por `origin`. Nome original, extensão, tamanho e data são exibidos quando disponíveis. Não se duplica arquivo de mesmo nome/tamanho; selecionar novamente reanexa o conteúdo em memória. Apenas metadados são persistidos; `File`/bytes/base64 nunca vão para localStorage. Conteúdo fica em um Map até recarga/fechamento. Há download na sessão e prévia de PDF/imagens raster comuns; outros formatos devem ser baixados. Excluir metadados exige confirmação. Não há versionamento.

## Consultas e indicadores

Dashboard, portfólio, relatórios gerais, carteira por GP e governança usam a mesma projeção `MvpViews.project()` das iniciativas não arquivadas. O Dashboard é exclusivamente executivo: cards numéricos e gráficos, sem tabelas/listas ou filtros, conforme [Interface do MVP](interface-mvp.md). Nas demais consultas, filtros usam os catálogos auxiliares e os valores legados efetivamente utilizados: status administrativo, departamento, GP, tipo, ciclo, DITIC, áreas envolvidas, criticidade e iniciativa estratégica, além da pesquisa textual. Há ordenação e, no portfólio, cards/tabela e paginação de 20 itens. Os cards organizam código/título, status colorido, departamento/setor, tipo, GP, ciclo, progresso e ações. HST ausente em todos os registros é apresentada como Não informado onde esse indicador é pertinente, nunca no Dashboard.

Indicadores fora do Dashboard podem incluir total, média dos percentuais, HST preenchidas e contagens de integrantes/documentos/relatórios. O Dashboard limita-se aos totais/status e conclusão média, com gráficos derivados do cadastro. Não usar prazo, risco, impedimento estruturado, cronograma, disponibilidade, score ou decisões simuladas. Consultas filtráveis calculam indicadores sobre os filtrados; o Dashboard usa todos os não arquivados. CSV usa o mesmo conjunto filtrado, BOM UTF-8, `;` e aspas escapadas. Impressão usa `window.print()` com CSS A4 paisagem. Status administrativos sempre usam `AuxiliaryData.statusStyle()`/`statusBadge()`, inclusive em relatórios e governança; a situação do Status Report permanece separada.

`#/critical` e `#/impediments` mostram indisponibilidade e não aparecem no menu ativo. Implementações futuras permanecem no código com prefixo `legacy`. A consulta detalhada usa as seis abas, sem reexpor as antigas áreas fora do MVP. Dados de exemplo estão explicitamente identificados como demonstrativos. Notificações automáticas permanecem indisponíveis, sem contagens fictícias. Ajuda e tour guiado foi removido do menu; Modo Apresentação não aparece no dashboard.

## Evolução e validação

Backend/API REST, autenticação, autorização, auditoria institucional, colaboração e conteúdo documental persistente continuam pendentes. A modelagem preliminar deve incluir os novos campos de classificação, criticidade e HST e relações apropriadas para equipe, documentos, relatórios, áreas e partes interessadas. Para N:N, usar tabelas associativas, nunca IDs concatenados. Pesos/score e cálculo de HST por equipe não foram definidos nem implementados.

Execute `node --test tests/initiative-editor.test.cjs` e `node --check` para os scripts alterados. Os testes usam armazenamento isolado e um adaptador DOM mínimo; não substituem conferência de navegador, teclado real, responsividade, console e impressão.
