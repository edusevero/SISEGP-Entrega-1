# Padrões de interface do MVP

## Dashboard executivo

`#/dashboard` apresenta exclusivamente cards numéricos e cards com gráficos, além do cabeçalho identificador do protótipo. Não há filtros, pesquisa, formulários, tabelas, listas ou ações sobre iniciativas. Não apresentar HST, vínculos de equipe ou documentos/anexos nesse painel. Esses dados permanecem disponíveis nas demais telas em que são pertinentes.

Os sete indicadores são total de iniciativas, ABERTO, EM PLANEJAMENTO, EM EXECUÇÃO, FINALIZADO, SUSPENSO e conclusão média. Contagens por status consideram a identidade do registro auxiliar, incluindo aliases após renomeações. Média é a soma dos percentuais dividida pela quantidade de iniciativas não arquivadas, arredondada para exibição. Ausência de registros não produz média fictícia.

Seis gráficos complementares usam os mesmos dados não arquivados: status (donut com cores configuradas), departamentos (barras horizontais), tipos (donut), ciclos (barras verticais), GPs (barras horizontais) e conclusão (barras por faixa). As faixas são 0–25%, >25–50%, >50–75% e >75–100%, cobrindo também valores fracionários sem lacunas ou sobreposição. Categorias ausentes são identificadas como Não informado; gráficos sem dados têm estado vazio.

Reutilizar `AppCharts` em `assets/js/charts.js`: SVG/CSS, sem biblioteca externa, CDN ou etapa de instalação. Há legendas, valores visíveis, nomes acessíveis e tooltips nativos quando apropriado. Gráficos ficam em duas colunas no desktop e uma no celular. Não adicionar séries ilustrativas nem usar datas/riscos fora do MVP.

## Cards e Visão Geral

O portfólio mantém pesquisa, filtros, ordenação, paginação, cards/tabela, exportação e ações de abrir, editar, duplicar e arquivar. `MvpViews.initiativeCard()` organiza código, título, badge de status, departamento/setor, tipo, GP, ciclo informado, progresso e rodapé. O título ocupa até duas linhas; tooltip preserva o texto completo. Metadados truncados também têm tooltip. Ações somente com ícone têm nome acessível e tooltip. Cards têm fundo branco, borda/sombra discretas e grid de três, duas ou uma coluna conforme a largura.

A edição mantém seis abas e a criação cinco. A capa da Visão Geral usa azul institucional `--navy-900`, texto claro, código/nome, tipo, departamento/setor e badge de status. O percentual e a barra usam o único valor salvo `progress`, sem indicador paralelo. GP, HST, equipe, arquivos e relatórios ficam em cards claros separados, com ícones discretos. O restante das informações e o último Status Report preservam seu conteúdo e suas ações.

Reutilizar `AppUI.icon()`/`hydrateIcons()` e o catálogo SVG existente. Não introduzir outra biblioteca de ícones. Ícones não substituem labels; badges se limitam a estados, tipos/ciclos e situações que beneficiam a leitura. Tipografia do sistema, espaçamento consistente, foco visível e pontos responsivos existentes devem ser preservados. A impressão mantém legibilidade e cores significativas.

## Cores administrativas

A fonte de execução é exclusivamente o campo `color` do registro auxiliar de status administrativo em `sisegp-catalogs-v1`. A carga/migração inicial está em `initialStatusColors`, somente em `assets/js/auxiliary-data.js`:

| Status | Cor inicial |
| --- | --- |
| ABERTO | `#DDDDDD` |
| EM PLANEJAMENTO | `#009966` |
| EM EXECUÇÃO | `#EEE600` |
| EM HOMOLOGAÇÃO | `#FF6400` |
| EM AJUSTES | `#6699CC` |
| FINALIZADO | `#6699CC` |
| SUSPENSO | `#ACAA59` |
| CANCELADO | `#D8D7B1` |

Não alterar esse mapeamento inicial nem duplicá-lo nos componentes. Novos status começam com cor neutra editável. O campo aceita somente `#RRGGBB`; estilos nunca recebem texto arbitrário. `AuxiliaryData.statusStyle()` obtém a cor e calcula texto preto/branco com contraste mínimo de 4,5:1. `statusBadge()` produz a apresentação compartilhada. A cor armazenada não é clareada; cores complementares servem apenas ao contraste e ao contexto.

Mudanças salvas em Configurações são usadas no próximo render dos gráficos, indicadores de status, cards, tabelas, Visão Geral, prévia de Dados da Iniciativa, relatórios e governança. IDs/aliases preservam o vínculo após renomeação ou inativação. O Status Report tem outro cadastro e não herda as cores administrativas.

## Configurações por grupos

`#/settings` é o painel de cards dos grupos Administração das Iniciativas, Classificação da Iniciativa, Priorização da Iniciativa e Status Report. Cada card tem ícone, título, descrição, contagem e indicação de navegação. Criticidade tem uma única coleção, acessada em Priorização; GPs reutilizam a mesma manutenção de `#/managers`.

Cada cadastro abre `#/settings/catalog/:nome-codificado`. Departamentos e Setores usam `#/settings/organization` e `#/settings/organization/sectors`, com navegação interna que preserva o relacionamento. Os links funcionam com voltar/avançar e acesso direto.

A página específica mostra breadcrumb, título, descrição, voltar, Novo Cadastro, pesquisa, filtro de situação e tabela. O editor aparece após Novo Cadastro ou Editar e usa um único componente genérico. Em status, há picker e texto hexadecimal sincronizados, amostra e coluna Cor. Ativação/inativação exige confirmação. Gravações mostram sucesso/erro; não há exclusão física ou troca dos IDs.

O complemento da referência adiciona somente registros ausentes, considerando nomes anteriores/aliases e parentesco. Não reativa inativos, não substitui nomes/cores válidas ou pontos já informados e não remove registros manuais. Pontuações indefinidas continuam `null`; nenhum score foi introduzido. Dados locais ilegíveis são preservados.

## Validação

Os testes de `tests/initiative-editor.test.cjs` verificam estrutura renderizada, cálculos, cores, migração, vínculos e manutenção com DOM mínimo e armazenamento isolado. `node --check` verifica sintaxe. Esses checks não comprovam aparência, navegação por teclado real, console do navegador ou impressão: realizar essas conferências em desktop, tablet e celular antes de publicação.
