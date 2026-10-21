# Orientações para agentes — SISEGP / TJRS

## Contexto e objetivo

Este repositório contém o protótipo funcional do Sistema do Escritório de Gestão de Projetos (SISEGP), destinado a apoiar o Escritório de Projetos do Tribunal de Justiça do Rio Grande do Sul (TJRS), também denominado EGP/PMO na interface.

O objetivo é validar a experiência de centralização e acompanhamento das iniciativas institucionais: portfólio, responsabilidades, planejamento, entregas, riscos, impedimentos, reuniões, documentos e informações para governança. Os principais públicos representados são gerentes de projetos (GPs), chefia do EGP e gestores que consultam informações executivas.

Trate o sistema como protótipo front-end com dados de exemplo. A aparência institucional, os nomes e os endereços presentes no código não comprovam integração, identidade autenticada nem uso em produção. Não acrescente dados reais de processos, documentos sigilosos, credenciais ou informações pessoais para demonstrar funcionalidades.

Estas orientações se aplicam a toda a pasta. Antes de alterar qualquer fluxo, leia a implementação correspondente; o README descreve intenções e pode divergir do comportamento atual.

## Arquitetura e arquivos

Não há framework, dependências de terceiros, gerenciador de pacotes, etapa de build, backend ou banco de dados. A aplicação usa HTML, CSS, JavaScript em scripts clássicos, SVG e armazenamento do navegador.

| Arquivo | Responsabilidade |
| --- | --- |
| `index.html` | Estrutura global, marca TJRS, menu, área `#conteudo`, pesquisa, notificações, modal e região de toasts. |
| `assets/js/data.js` | Catálogos, gerador de 200 iniciativas, modelos de artefatos, arquivos de exemplo e API `window.DemoData`. |
| `assets/js/charts.js` | Agregações, gráficos SVG/CSS, paleta e escape de HTML; expõe `window.AppCharts`. |
| `assets/js/app.js` | Estado, roteamento, renderização de telas, formulários, eventos, persistência, evolução e logs locais. |
| `assets/js/initiative-options.js` | Carga oficial de referência para os selects do MVP. |
| `assets/js/auxiliary-data.js`, `assets/js/auxiliary-view.js` | Persistência, vínculos e manutenção dos cadastros auxiliares. |
| `assets/js/initiative-editor.js`, `assets/js/mvp-views.js` | Formulários compartilhados, Visão Geral e consultas/indicadores do MVP. |
| `assets/css/styles.css` | Tokens visuais, componentes, layouts, responsividade e impressão. |
| `assets/images/` | Marca institucional e favicon locais. |
| `README.md` | Apresentação, execução, funcionalidades, limitações e possíveis evoluções. |

Preserve a ordem de carregamento: `data.js`, `charts.js`, `initiative-options.js`, `auxiliary-data.js`, `initiative-editor.js`, `demo-completion.js`, `mvp-views.js`, `auxiliary-view.js`, `app.js`. Os scripts usam IIFEs e `'use strict'`. Não converta para módulos ou introduza bibliotecas, CDN, bundler ou servidor sem necessidade expressa da tarefa.

## Navegação e módulos

O roteamento é feito por `location.hash` e pelo evento `hashchange`, sem reescrita de URL no servidor. Preserve links profundos, navegação voltar/avançar e a página de conteúdo não encontrado.

| Rota | Área |
| --- | --- |
| `#/dashboard` | Painel executivo exclusivamente com cards numéricos e gráficos, sem listas, tabelas ou filtros. |
| `#/initiatives` | Portfólio com pesquisa, filtros, ordenação, paginação de 20 itens e modos cards/tabela. |
| `#/initiative/new` | Cadastro de iniciativa. |
| `#/initiative/INI-001` | Detalhes de uma iniciativa; o identificador é variável. |
| `#/initiative/INI-001/edit` | Edição da iniciativa. |
| `#/artifacts` | Rota removida; exibe conteúdo não encontrado. |
| `#/repository` | Repositório geral do EGP. |
| `#/reports`, `#/gp-report` | Relatórios e carteira por GP, somente com campos do MVP. |
| `#/critical`, `#/impediments` | Indisponibilidade informativa; fora do menu ativo do MVP. |
| `#/logs` | Histórico local de eventos, com filtros e paginação. |
| `#/managers`, `#/settings`, `#/governance` | GPs, configurações e visão de governança. |
| `#/settings/catalog/:nome`, `#/settings/organization`, `#/settings/organization/sectors` | Manutenção específica dos auxiliares e estrutura organizacional; nome do cadastro é codificado na URL. |
| `#/action/...` | Telas auxiliares; algumas dependem de contexto mantido em memória. |

O cadastro MVP usa cinco abas, e a edição/consulta usa seis com Visão Geral inicial. As antigas áreas de RACI, riscos, marcos, cronograma, evolução e reuniões permanecem no código legado, fora das telas ativas de iniciativas. Consulte a seção MVP e os documentos específicos antes de criar links.

## Dados, regras e persistência

- A entidade central é a iniciativa, identificada por `id` no padrão inicial `INI-001`. Preserve identificadores únicos e referências ao alterar cadastro ou duplicação.
- O código público do MVP usa `AuxiliaryData.initiativeCode(item)`: departamento atual + número global do ID (ex.: `DSUP-068`). Cards, Visão Geral, tabelas e CSV usam esse código; pesquisa aceita código e ID legado. IDs internos, URLs e referências permanecem estáveis. Mudança de departamento ou renomeação do catálogo altera o prefixo exibido, sem renumerar nem desvincular registros. Sem departamento, mostrar SEM-DEPARTAMENTO como prefixo.
- Preserve os campos consumidos pelas telas e as coleções `risks`, `milestones`, `activities`, `evolutions`, `meetings`, `files`, `raci` e `statusReports`. Novos campos precisam de valores padrão para registros antigos.
- Use `AuxiliaryData` como fonte dos selects e filtros do MVP. `InitiativeOptions` é a carga de referência; os catálogos antigos de `DemoData` permanecem apenas para compatibilidade e geração demonstrativa. Não manter listas ativas divergentes.
- Os 200 registros pertencem à carga inicial, não a uma contagem fixa após cadastros ou arquivamentos. Há nove pessoas no catálogo de gestores; o gerador distribui iniciativas entre as oito primeiras.
- O gerador usa sementes para diversos atributos, mas também usa `new Date()`: reuniões, prazos de risco e alguns estados variam com a data de execução. Não assuma que a carga inteira é independente do relógio.
- Mantenha progresso numérico entre 0 e 100. Preserve a coerência com a situação, especialmente conclusão, ao alterar regras.
- Arquivamento usa `archived`; as consultas de portfólio ativo devem excluir esses registros. Não substitua arquivamento por exclusão definitiva.
- Datas de domínio são normalmente `YYYY-MM-DD`; a apresentação é brasileira (`DD/MM/AAAA`). Preserve o tratamento local das datas, evitando deslocamentos por conversão UTC.
- Não usar indicadores de atraso, vencimento, risco ou desatualização nas consultas ativas do MVP. Os helpers `isDelayed`, `isStale` e `isDueSoon` permanecem no legado; sua existência não autoriza reintroduzir campos fora do cadastro.
- Criticidade e nível de risco são dimensões distintas; risco também contém probabilidade, impacto e pontuação. Não intercambie esses campos por semelhança do texto.
- Alterações persistentes de iniciativas devem passar pelo fluxo `persist()`/`DemoData.save()`, atualizar os indicadores pertinentes e informar falhas de gravação quando o fluxo for modificado.

| Armazenamento | Chave | Conteúdo |
| --- | --- | --- |
| `localStorage` | `sisegp-data-v2` | Iniciativas e seus registros associados. |
| `localStorage` | `sisegp-audit-v1` | Eventos locais, limitados aos 2.500 mais recentes. |
| `localStorage` | `sisegp-view` | Preferência de cards/tabela. |
| `localStorage` | `sisegp-catalogs-v1` | Cadastros auxiliares, IDs, situação, hierarquia, aliases e pontuações opcionais. |
| `sessionStorage` | `sisegp-access-logged` | Controle de registro de acesso da sessão. |
| `sessionStorage` | `sisegp-authenticated` | Acesso simulado da aba; removido pela opção Sair no menu do usuário. |

O menu do usuário no cabeçalho permite acessar o perfil e sair. Sair encerra apenas a sessão simulada e fecha overlays; não limpa iniciativas nem logs. O visual adapta o layout e as marcas de `modelo-app-master` em HTML/CSS, sem incorporar Angular ou a biblioteca externa `@ditic/modelo-ui-lib`.

Preserve compatibilidade com os dados já gravados. Há normalizações de coleções, status reports, reuniões e nomes legados no início de `app.js`; considere-as ao alterar o esquema. Não renomeie chaves nem limpe armazenamento como solução para incompatibilidade sem tratar a migração. Restauração de iniciativas e limpeza dos logs são operações distintas; confira o manipulador de cada ação.

Para ações relevantes, reutilize os mecanismos `recordAutomatic()` e `addAuditLog()` quando aplicáveis. Diferencie evolução manual e automática, preservando `source`, `automatic` e a ligação `auditId`. Os logs são demonstrações locais editáveis pelo navegador, sem garantia de auditoria institucional.

## Convenções de implementação

- Faça alterações focadas no pedido. Evite reformatar arquivos inteiros ou reorganizar a arquitetura junto de uma mudança pontual.
- Use português do Brasil na interface e documentação, preservando acentuação e UTF-8. Os identificadores existentes são predominantemente em inglês; siga o contexto local.
- Reutilize os helpers de componentes, formulários, badges, gráficos e ícones antes de criar alternativas.
- Os eventos são majoritariamente delegados no `document`, por atributos `data-*`. Ao adicionar controles, conecte renderização e manipulador sem duplicar listeners após cada renderização.
- Conteúdo de usuário e armazenamento local deve passar por `escapeHtml()` ao entrar em templates HTML, inclusive atributos. Valide separadamente URLs, identificadores, números e valores usados em estilos; escape de HTML não substitui essas validações.
- Após inserir componentes dinâmicos com `data-icon`, considere `hydrateIcons()`. Novas rotas devem preservar o título da página, o estado ativo do menu e o foco do conteúdo.
- Evite formulários aninhados e preserve validação e vínculos entre rótulos e campos. Não exiba mensagem de gravação real para uma ação que apenas altera a apresentação.

## Interface, acessibilidade e impressão

Mantenha a identidade visual existente: azuis institucionais, superfícies claras, tipografia do sistema, logo local e cores semânticas de sucesso, alerta e risco. Reutilize variáveis de `:root`, componentes e convenções como `.button`, `.panel`, `.field` e `.badge`.

Preserve o funcionamento em desktop, tablet e celular. O CSS contém pontos de adaptação em 1250, 980, 720 e 430 pixels. Confira tabelas, Gantt, filtros, cards e menu lateral em telas estreitas.

Mantenha HTML semântico, link de salto, foco visível, labels, navegação por teclado e atributos ARIA coerentes. Botões apenas com ícone precisam de nome acessível. Situações e riscos precisam de texto além de cor. Confira abertura, fechamento por Escape e gerenciamento de foco em overlays quando estes forem alterados.

A exportação CSV usa BOM UTF-8, separador `;`, aspas escapadas e o resultado de `filteredInitiatives()`. Preserve a compatibilidade com conteúdo acentuado e a correspondência entre filtros e exportação.

O PDF usa `window.print()` e a opção de salvar do navegador. O CSS de impressão configura A4 paisagem e oculta controles globais. Alterações em relatórios precisam considerar impressão, quebras e legibilidade do documento.

## Limites e divergências conhecidos

- Não há autenticação, autorização efetiva, colaboração simultânea nem integração ativa com SEI, GIE, Microsoft 365 ou IA.
- Uploads guardam metadados, sem preservar os bytes dos arquivos. Não descreva esse fluxo como armazenamento documental completo.
- Algumas telas auxiliares, cadastros administrativos e ações rápidas exibem formulários ou toasts sem implementar toda a operação sugerida. Verifique a mutação efetiva e a persistência antes de afirmar que uma função está pronta.
- O download auxiliar de documento produz texto `.txt`; os formatos DOCX/XLSX/PPTX apresentados nos catálogos não significam geração desses arquivos.
- Notificações automáticas estão indisponíveis no MVP. Calendário e gráficos históricos ilustrativos permanecem somente no código legado, fora do dashboard ativo. Indicadores ativos derivam dos dados cadastrados.
- Por solicitação do usuário, os cards de aviso demonstrativo foram removidos do portfólio, consultas e editor; `demoBanner()` retorna vazio. A natureza fictícia dos dados permanece documentada, sem qualquer integração ou informação oficial.
- O README menciona `.htaccess`, mas esse arquivo não está presente na pasta analisada. Não presuma que os cabeçalhos citados estão configurados.
- O README usa o caminho de publicação `/tjrs/sisegp/`. Servindo diretamente esta raiz de trabalho, a URL local é `/`; o subcaminho só existe se o servidor o disponibilizar.

Esses pontos são contexto para manutenção, não uma solicitação para corrigir todos eles em cada tarefa. Atualize a documentação quando uma mudança alterar esses limites.

## Execução e validação

Abra `index.html` diretamente ou execute um servidor estático nesta pasta, se o interpretador estiver disponível:

```powershell
python -m http.server 8080 --bind 127.0.0.1
```

Acesse `http://127.0.0.1:8080/`. Não há comandos `npm install` ou `npm run build`. Os testes do editor MVP usam apenas Node.js: `node --test tests/initiative-editor.test.cjs`. Eles verificam lógica com um adaptador DOM mínimo; não substituem validação visual e interativa no navegador. O armazenamento pode variar entre `file://`, endereços, portas e perfis do navegador; use a mesma origem ao verificar persistência.

Se Node.js estiver disponível, confira a sintaxe dos scripts alterados:

```powershell
node --check assets/js/data.js
node --check assets/js/charts.js
node --check assets/js/app.js
```

Adapte a validação ao impacto da mudança:

1. Confira carregamento dos recursos e console; ao alterar navegação global, percorra as rotas do menu, links diretos e endereço inexistente.
2. Para portfólio, combine pesquisa/filtros, ordenação, paginação, cards/tabela, métricas e exportação.
3. Para dados, confira criação/edição, recarga da página, duplicação e arquivamento em um perfil de teste. Não restaure dados do usuário para realizar testes.
4. Para áreas da iniciativa, verifique o registro alterado, o histórico automático/manual, logs e indicadores relacionados.
5. Para interface, confira desktop e celular, teclado, overlays e console; para relatórios, verifique a prévia de impressão.
6. Para mudanças de esquema, verifique carga inicial e registros previamente persistidos, incluindo coleções ausentes.

Uma alteração apenas documental exige revisão do conteúdo e dos caminhos, sem necessidade de instalar ferramentas ou criar testes. Informe o que foi efetivamente verificado e qualquer validação de navegador não realizada; não repita como evidência própria a seção histórica de validação do README.

Ao entregar mudanças, resuma o resultado, os arquivos relevantes e as verificações realizadas. Atualize este guia quando arquitetura, comandos, armazenamento ou fluxos mudarem de forma material.

## SISEGP — Módulo Iniciativas (MVP)

Consulte estas regras e [docs/modulo-iniciativas.md](docs/modulo-iniciativas.md) antes de alterar o módulo central do SISEGP. A finalidade é centralizar iniciativas e registros associados para acompanhamento do EGP/TJRS. A documentação dos cadastros está em [docs/cadastros-auxiliares.md](docs/cadastros-auxiliares.md); padrões visuais, Dashboard, cores e Configurações estão em [docs/interface-mvp.md](docs/interface-mvp.md).

- **Criação:** exatamente cinco abas — Dados da Iniciativa, Classificação, Equipe, Documentos, Status Report. **Edição e consulta de existentes:** os mesmos componentes e campos, precedidos por Visão Geral (seis abas), sem duas implementações independentes.
- **Dados da Iniciativa:** grid responsivo de 12 colunas. Nome/início/previsão de término em 8/2/2; tipo/departamento/setor/solicitante/patrocinador/SEI em seis campos de 2 colunas; escopo/descrição e benefício/observações em pares 6/6. Datas opcionais usam `start`/`end` (YYYY-MM-DD), validando término igual ou posterior ao início sem substituir `originalEnd`. Textos longos têm 3 linhas. Nome é obrigatório; SEI é texto livre. Trocar departamento limpa setor incompatível.
- **Administração:** status/conclusão/progresso em 4/2/6; áreas/partes interessadas em 6/6. Conclusão finita de 0 a 100 atualiza a barra, sem mudar automaticamente status. Áreas usam lista suspensa compacta com checkboxes, chips removíveis e resumo adicional; partes interessadas permitem múltiplos nomes sem duplicação. **Anexos** ficam em terceiro card separado, abaixo da Administração.
- **Classificação:** três linhas de três campos (4 colunas): DITIC/estratégica/eixo; biênio/ciclo/ENTICJUD; GP/responsável/objetivo. GP é select dos ativos; responsável é texto livre. Priorização contém oito selects em duas linhas de quatro (3 colunas). HST é numérica não negativa e independente das horas dos integrantes. Não inventar pesos ou calcular score.
- **Equipe:** nome/papel/HST/botão em 5/3/3/1. Nome obrigatório, HST não negativa; adicionar, editar, excluir com confirmação. Botão compacto + com nome acessível vira Salvar/✓ na edição e retorna à inclusão após confirmar, limpando campos. Não há Cancelar edição, RACI nova ou disponibilidade. Impedir duplicação involuntária de nome/papel.
- **Status Report:** tipo/data/Status da Iniciativa/responsável em quatro colunas iguais; realizações/impedimentos/próximas atividades em três textareas iguais de 5 linhas. Label exatamente Status da Iniciativa. Tipo, data válida, situação e GP são obrigatórios. Tabela por data decrescente com visualizar/editar/excluir confirmada. Sem Cancelar edição; após confirmar, limpar e voltar à inclusão.
- **Estados separados:** `administrativeStatus` é ciclo administrativo. `statusReports[].monitoringStatus` é situação de acompanhamento (inicialmente Em dia, Atrasado, Em risco). Preservar o campo legado `status` sem inferir ciclo a partir de Atrasada. Não intercambiar criticidade com risco.
- **Visão Geral:** somente existentes. A capa tem fundo azul institucional, texto contrastante, código/nome, informações principais, badge de status e barra do único percentual salvo. Indicadores claros separados usam ícones para GP/HST/contagens. Manter informações principais e último relatório por data com conteúdo completo acessível. Ausências usam Não informado/estado vazio. Atualizar após salvamento, sem valor de progresso paralelo.
- **Fonte única:** `AuxiliaryData` em `assets/js/auxiliary-data.js` atende selects e filtros. Configurações permite administrar todos os cadastros; Cadastro de GP reutiliza o mesmo componente. A carga em `initiative-options.js` foi conferida com o DOCX local. Preservar grafia oficial, inclusive Seção ERP INTEGRS E DADOS e VALORIZAÇÃO REAL DAS PESSOAL. Não inventar CPF/e-mail.
- **Integridade auxiliar:** IDs estáveis, setores vinculados ao departamento, aliases para renomeações, sem exclusão física. Inativos continuam nos vínculos existentes e não são oferecidos em novos. Duplicidade é impedida dentro do cadastro (e do departamento para setores). Carga idempotente complementa somente valores ausentes, considerando aliases/parentesco, sem reativar ou sobrescrever nomes, cores válidas, pontuações ou dados manuais. Dados locais ilegíveis não são sobrescritos. Pontuações são opcionais/null; sem score. Não transferir setor existente entre departamentos.
- **Persistência:** manter `sisegp-data-v2`, logs e demais chaves existentes. Auxiliares usam `sisegp-catalogs-v1`. `catalogRefs` associa IDs preservando os campos textuais legados; relatórios têm referências próprias. `resolveRecord()` lê nomes atuais e `bindRecord()` reconstrói IDs após alterações explícitas. Preservar campos e coleções não expostos; não limpar armazenamento para migrar esquema.
- **Rascunhos:** trocar abas não perde valores nem altera dados persistidos. Confirmar integrantes/relatórios nos botões próprios e depois salvar a iniciativa. Tratar falhas mantendo o formulário e revertendo a mutação do estado. Reutilizar logs/evolução automática quando aplicável. Manter avisos de alterações pendentes, Escape, foco e ARIA coerentes.
- **Arquivos:** documentos e anexos compartilham `files`, com origem/categoria. Exibir nome/extensão/tamanho/data; evitar duplicação por nome/tamanho, permitir remoção confirmada. Persistir só metadados; File/bytes ficam em Map na sessão, sem base64/binários no localStorage. Download/prévia dependem desse conteúdo temporário; após recarga selecionar novamente. Sem versionamento nem integração externa.
- **Consultas:** Dashboard, portfólio, relatórios e governança usam a mesma projeção `MvpViews.project()` dos registros não arquivados. Somente campos do MVP e derivações legítimas. Portfólio/relatórios preservam pesquisa, filtros, ordenação e exportação; CSV usa exatamente os filtrados com BOM/;/aspas. Não mostrar cronograma, prazo, riscos, impedimentos estruturados ou decisões simuladas. A natureza demonstrativa é documentada; os cards de aviso foram removidos por solicitação do usuário.
- **Dashboard:** exclusivamente cards numéricos e gráficos; sem pesquisa, filtros, formulários, tabelas ou listas de iniciativas. Indicadores: total, abertas, planejamento, execução, finalizadas, suspensas e conclusão média. Não apresentar HST, vínculos de equipe ou documentos/anexos nesse painel. Donuts, barras horizontais/verticais e faixas de conclusão usam `AppCharts` SVG/CSS existente, sem nova biblioteca. Legendas/tooltips, estados vazios e responsividade são obrigatórios; não inventar valores para preencher gráficos.
- **Cards de iniciativas:** grid responsivo com altura consistente, fundo claro, bordas/sombras discretas, código/título, status, departamento/setor, tipo, GP, ciclo informado, progresso e ações existentes. Limitar título a duas linhas com tooltip completo; ícones discretos não substituem labels. Reutilizar `AppUI.icon()` e `hydrateIcons()`. Badges/chips são usados seletivamente para estados, tipo/ciclo e situação, sem poluição visual.
- **Cores de status:** somente `AuxiliaryData` guarda o mapa inicial `initialStatusColors` e o campo persistente `color`. O mapeamento exato está em docs/interface-mvp.md; EM AJUSTES e FINALIZADO começam ambos em `#6699CC`. Validar `#RRGGBB`. Picker e hexadecimal são administráveis em Configurações. Todos os componentes e gráficos de status leem `statusStyle()`/`statusBadge()`, calculando contraste sem alterar o HEX armazenado. Status Report é independente e não herda essas cores.
- **Configurações:** painel de cards agrupados em Administração, Classificação, Priorização e Status Report, com ícone/descrição/contagem. Cada cadastro tem rota própria e manutenção genérica com breadcrumb, voltar, Novo Cadastro, pesquisa, situação e ações. Departamentos/Setores têm navegação relacionada. Criticidade é coleção única; GPs reutilizam a estrutura existente. Confirmar ativação/inativação e tratar falhas de gravação. Preservar links diretos e voltar/avançar.
- **Navegação:** Ajuda e tour guiado removido do menu; Modo Apresentação removido do dashboard. Relatórios integralmente fora do MVP não aparecem no menu e suas rotas exibem indisponibilidade. Preservar implementações futuras com prefixo legacy e dados sem reativar telas incompatíveis.
- **Arquitetura e futuro:** preservar HTML/CSS/JS clássicos, identidade institucional, responsividade, componentes, foco e impressão. A modelagem de banco é preliminar; futura API/banco deve incorporar classificação adicional, criticidade, HST e relacionamentos com equipe/documentos/relatórios. N:N usa relações próprias, nunca IDs concatenados. Backend, autenticação/autorização, colaboração e armazenamento documental real permanecem pendentes.
- **Documentação permanente:** toda tarefa futura que modificar campos, regras, cadastros, relacionamentos, navegação, arquitetura ou comportamento deve atualizar o AGENTS.md na mesma tarefa, além dos documentos específicos quando necessário. Manter interfaces, entidades e futura API consistentes, sem instruções redundantes/conflitantes nem regras de negócio inventadas.


## Preenchimento demonstrativo e portfólio

Os cards abrem a iniciativa pelo link nativo do título, expandido por CSS à superfície do card, com foco de teclado. Não exibem botões de abrir, editar ou duplicar. As oito informações ocupam duas linhas de quatro colunas iguais no desktop, duas colunas em tablet e uma em celular. A barra de progresso ocupa a largura disponível e tem 14px de altura. Exibir a data válida mais recente de Status Report e dias de calendário até hoje (data local; cálculo sem efeito de horário de verão); informar ausência ou data futura explicitamente.

No card, o responsável usa `mvp.classification.owner` ao lado do GP. Último SR e Biênio / Ciclo são entradas da mesma lista de metadados, com rótulo, valor e ícone; não usar um chip separado para ciclo. A explicação dos cálculos não aparece no portfólio; as regras continuam documentadas aqui e nos relatórios.

Os cards também exibem `start` e `end` como Data de início e Previsão de término em DD/MM/AAAA. Destacar em vermelho o último Status Report quando tiver estritamente mais de 15 dias de calendário; não destacar relatório ausente, inválido ou futuro. Esse destaque visual não altera a situação cadastrada do SR nem o indicador SR Atrasados.

`demo-completion.js` complementa os campos ausentes do cadastro MVP das iniciativas existentes, inclusive classificação, equipe, anexos (somente metadados) e Status Reports. Usa os catálogos auxiliares e preserva valores preenchidos, IDs e coleções legadas. O marcador `demoCompletionVersion: 1` evita recompletar registros em recargas posteriores; cadastros salvos pelo editor recebem o marcador e continuam permitindo campos opcionais vazios. A base complementada passa pelo armazenamento habitual e alimenta dashboard, portfólio e relatórios. Não são criados bytes de documentos nem dados pessoais reais.

No portfólio, filtros ocupam duas linhas de seis células no desktop, com adaptação em telas menores. Cada card ocupa uma linha completa e a paginação contém 20 registros. Os indicadores são Iniciativas, Conclusão média, Status Reports e SR Atrasados. O último conta todos os relatórios com situação Atrasado nas iniciativas filtradas, incluindo o histórico e reconhecendo renomeações do catálogo. A ação de arquivar foi retirada dos cards. Os avisos demonstrativos em cards foram removidos por solicitação do usuário; o sistema continua sendo um protótipo local.
