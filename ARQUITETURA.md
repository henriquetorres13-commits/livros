# ARQUITETURA.md — App "Livros"

Referência para quem for mexer no código (Claude ou pessoa). Ler inteiro antes de começar uma versão. Conferido contra o código da v1_09 em 2026-11-05; se algo divergir do `index.html`, o código vale e este documento deve ser corrigido no mesmo commit.

O escopo funcional está em `ESPECIFICACAO.md`; a ordem de entrega e o andamento estão em `PLANO.md`.

---

## 1. O que é

App pessoal e offline de biblioteca e diário de leitura (livros, autores, categorias, leituras, progresso, anotações, personagens, preços). Irmão do app **Preços** (repositório `pre`): mesma infraestrutura e mesmos padrões, **dados totalmente separados**.

- HTML único (`index.html`), sem servidor e sem dependências externas.
- Dados no IndexedDB do celular; PWA instalado pelo Chrome; GitHub Pages (branch `main`, raiz).
- Uso principal: celular, em pé, com uma mão. Telas pequenas, poucas opções por tela.

## 2. Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | Todo o app (HTML + CSS + JS). Manter único. |
| `sw.js` | Service worker (cache offline, rede com timeout de 3 s). |
| `manifest.webmanifest` | Manifesto do PWA. |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png` | Ícones. |
| `ESPECIFICACAO.md` | Especificação funcional v1.0 (escopo acordado). |
| `PLANO.md` | Fases de entrega e andamento. |
| `ARQUITETURA.md`, `CLAUDE.md` | Este documento e as instruções para o Claude. |

## 3. Regras de ouro (não quebrar)

1. **Isolamento do app Preços.** Os dois apps vivem no mesmo domínio (`henriquetorres13-commits.github.io`) e **compartilham o `localStorage`**. Por isso: banco `livros_db` e **toda chave de `localStorage` começa com `livros_`** (constante `LS_PREFIX`). Nunca usar o prefixo `precario_`. O backup só leva chaves `livros_*`.
2. **Nunca perder dados.** Mudança de esquema exige migração sem perda e incremento de `DB_VERSION`.
3. **Todo dado novo entra no backup `.json`.** O backup é genérico (todos os stores + todas as chaves `livros_*`). Store novo: criar em `onupgradeneeded` e na lista `STORES`. Chave nova de `localStorage`: prefixo `livros_`. Dado guardado em outro lugar se perde na restauração.
4. **Reaproveitar antes de criar:** mesmos helpers, camadas, diálogos, máscaras. Consistência entre telas tem prioridade sobre rapidez.
5. **HTML único.**
6. **Toda versão incrementa o número** (`v1_NN`), mesmo que pequena.
7. **Confirmação antes de qualquer exclusão**, com diálogos próprios (`confirmar`, `avisar`, `perguntar`): o diálogo nativo mostra o endereço do site.
8. **Nada é gravado sem confirmação** quando houver dados vindos de busca online (ISBN).

## 4. Versão e publicação

- A cada versão, trocar **juntos**: `APP_VERSION` (`index.html`) e `CACHE` (`sw.js`, formato `livros-vX_YY`).
- Uma branch por versão (`v1_NN`); commit; `git push -u origin <branch>`; **só mergear na `main` (ff-only) quando o usuário disser "pode publicar"**, porque a `main` publica o app instalado.
- Lembrar o usuário de fazer **backup `.json` antes de publicar**.
- Depois de publicar, criar a branch da próxima versão.
- A tela Mais → Backup mostra a versão instalada ("Sobre").

## 5. Verificação antes de entregar

1. Extrair o JS do `index.html` e rodar `node --check`.
2. Smoke test com Playwright (chromium, `file://`, viewport 390×780): semear dados com `putRecord` + `reloadAll`, navegar, voltar com `page.go_back()`; esperar (`wait_for_function`) após cliques que disparam código assíncrono.
3. Conferir ida e volta do dado novo no backup.
4. Reler o diff procurando o que a mudança pode ter quebrado.

## 6. Dados (IndexedDB `livros_db`, `DB_VERSION = 1`)

Todos os stores: `{keyPath:'id', autoIncrement:true}`. Acesso: `getAll`, `putRecord(store,obj)`, `deleteRecord(store,id)`, `reloadAll()` (recarrega as variáveis globais).

| Store | Variável | Uso previsto |
|---|---|---|
| `livros` | `LIVROS` | título (obrigatório), subtítulo, isbn, `autorIds[]`, `capa` (data URL), páginas, descrição, comentários, `categoriaId`/`subcategoriaId`, `situacao` (`desejo`\|`proprio`\|`emprestado`), `emprestadoDe` |
| `autores` | `AUTORES` | só `nome` (v1_03). Excluir um autor tira o id de `autorIds` dos livros; os livros ficam |
| `categorias`, `subcategorias` | `CATS`, `SUBCATS` | classificação de livros (subcategoria tem `categoriaId`) |
| `leituras` | `LEITURAS` | `livroId`, `inicio` e `fim` (ISO/UTC; `fim` null em andamento), `status` (`andamento`\|`finalizada`). A página atual **não** é gravada aqui (ver 7e) |
| `progressos` | `PROGRESSOS` | `leituraId`, `pagina`, `dataHora` (ISO/UTC; preserva histórico, aceita regressão) |
| `anotacoes` | `ANOTACOES` | `leituraId`, `texto`, `dataHora` (ISO/UTC, editável), `pagina` (número ou `null`), `personagemIds[]` (vínculos reais; v1_08) |
| `personagens` | `PERSONAGENS` | `livroId`, `nome`, `comentario` (compartilhadas entre as leituras do livro; v1_08) |
| `precos` | `PRECOS` | `livroId`, preço, data, `localId`, `comprou` (pago × observado) |
| `locais` | `LOCAIS` | lojas/livrarias |
| `visoes` | `VISOES` | visões e combinações de filtros salvas (um único mecanismo), com `ordem` |

Os campos acima são o **desenho previsto**; cada versão que usar um store confirma/ajusta o esquema aqui. A situação "Lido" **não** é campo do livro: é derivada de existir leitura finalizada.

Capas: imagem redimensionada em data URL no próprio registro do livro (vai no backup). Usar `resizeImage(file, maxSize, quality)` com tamanho menor que o do Preços (capas ~360 px) para manter o backup leve.

Configurações: `localStorage` `livros_cfg_*`, lidas por `loadConfig()` (padrões em `CONFIG_DEFAULTS`): `telaInicial` (`leituras`\|`atalhos`), `layoutPadrao` (`grade`\|`lista`). Tema: `livros_theme` (`light`\|`dark`). Outras chaves: `livros_ultimo_backup` (AAAA-MM-DD), `livros_primeiro_uso`.

## 7. Navegação e camadas de UI (herdado do Preços)

- **Pilha única `navStack`** (telas + modais). O botão voltar do celular desempilha; sem nada empilhado, pergunta se quer sair do app.
- Camadas de sheet (`criarCamada`): principal (`openSheet`), sub (`openSubSheet`), sub2 (`openSub2Sheet`); têm `dirty`/`salvando`/`confirmarDescarte` (sair de edição com alterações pede confirmação). Seletor sobreposto `pickerOverlay` (z-index 65): `escolherNoPicker(html)` mostra botões com `data-v="valor"` e devolve o valor tocado, ou `null` se o usuário voltar/tocar fora (entra na `navStack` como modal; o voltar do celular fecha só ele). Usado nas escolhas de autor parecido e de ISBN repetido.
- **Só empilha** por atalhos dentro das telas. Barra inferior e itens do Mais são **raiz** e zeram a pilha: `switchView(name, {raiz:true})`.
- Barra inferior: Biblioteca, Desejos, Leituras, Pesquisa, Mais. Em Mais: Tela de atalhos, Categorias de livros, Backup e restauração, Configurações. O botão flutuante `+` (`fabAdd`) aparece só em Biblioteca e Desejos. `VIEWS_DO_MAIS` lista as telas que acendem a aba Mais.
- Página inicial configurável: Leituras (`view-leituras`) ou Atalhos (`view-inicio`).
- `renderCurrent()` é o ponto único de redesenho; cada tela nova ganha seu `render*()` chamado ali.
- Diálogos: `avisar(msg)`, `confirmar(msg)` → boolean, `perguntar(msg, valor)` → texto|null.

### Livros (v1_02)

- Biblioteca = livros com `situacao` `proprio`/`emprestado`; Desejos = `desejo`. Marcar como comprado só troca `situacao` para `proprio` (mesmo registro, nada se perde).
- Listagem: `renderLivros(aba)`; grade/lista alternável na tela (`layoutTela`, reinicia com `CONFIG.layoutPadrao`, não é gravado). Capa, título e, quando há leitura em andamento e o livro tem total de páginas, a barra com o % (v1_06).
- Detalhe `abrirLivro(id)` (sheet principal, modo leitura) e formulário `abrirFormLivro(id, situacaoInicial)`; salvar/cancelar de edição voltam ao detalhe sem empilhar nova entrada de histórico.
- `autorIds` é gravado como `[]` (autores na v1_03); duplicidade de ISBN também na v1_03.
- Exclusão `excluirLivro`: apaga em cascata leituras, progressos, anotações, personagens e preços do livro, avisando as quantidades.
- Categorias: tela Mais → Categorias (`renderCategorias`); `criarCategoria`/`criarSubcategoria` também são usadas pelo botão `+` do formulário. Nome repetido (ignorando acento/caixa) é recusado. Excluir categoria apaga suas subcategorias e deixa os livros sem categoria.

## 7b. Autores e ISBN (v1_03)

- **Formulário do livro:** autores escolhidos viram chips; o campo de busca sugere existentes (sem diferenciar acento/caixa) e sempre oferece "+ Criar autor". Texto digitado e não adicionado bloqueia o salvar com aviso. Estado local `autorIds`, gravado em `livro.autorIds`.
- **Criar autor:** `criarAutorComChecagem(nome)` compara com `normalizeNome` + `nomesSemelhantes` (mesmas letras sem espaços/pontuação, ordem trocada, erro de digitação por `levenshtein`, nome contido em nome mais completo). Se houver igual/parecido, o seletor oferece "Usar existente" / "Criar como outro autor" / "Cancelar". **Nunca une autores sozinho.** Renomear na tela Autores usa `confirmar` quando o novo nome é parecido.
- **Tela Mais → Autores** (`view-autores`, `renderAutores`): lista com contagem de livros; tocar no nome abre os livros do autor (`abrirAutor`); renomear; excluir (avisa quantos livros perdem o autor).
- **ISBN repetido:** `isbnChave` limpa hífens/espaços e converte ISBN-10 válido em ISBN-13 (mesma chave = mesmo livro); `livroComIsbn(isbn, ignorarId)`. O formulário mostra aviso na hora; ao salvar, **só se o ISBN é novo ou mudou**, o seletor oferece "Abrir o livro existente" (descarta o formulário) / "Salvar mesmo assim (outro exemplar)" / "Voltar". Sem ISBN, nenhuma regra por título.
- Sem mudança de esquema: store `autores` já existia (`DB_VERSION` segue 1) e o backup é genérico.

## 7c. ISBN digitado e busca online (v1_04)

- **Botão "🔍 Buscar"** ao lado do campo ISBN do formulário (`lv-isbn-busca`). Valida o ISBN (`isbnValido`, com dígito verificador, ISBN-10 e ISBN-13), exige internet e consulta **Google Books** (`consultaGoogleBooks`) e **Open Library** (`consultaOpenLibrary`) em paralelo, 8 s de limite cada (`fetchComTimeout`).
- `buscarIsbnOnline(isbn)` devolve `{dados, fontes}`, `{naoEncontrado}` ou `{erroRede}`. **Google tem prioridade; a Open Library só completa campo vazio** (não mistura listas: se o Google trouxe autores, os da Open Library são ignorados). Descrição passa por `limpaTexto` (remove HTML).
- **Capa:** `baixarCapa(urls)` tenta, em ordem, a miniatura do Google, a da Open Library e `covers.openlibrary.org/b/isbn/<ISBN13>-M.jpg`; descarta respostas que não são imagem ou com menos de 2 KB (o Google devolve um "sem imagem" minúsculo); reduz com `resizeImage(blob, 360, 0.7)` e guarda como data URL (vai no backup). Se nenhuma baixar (CORS), a revisão avisa e o usuário usa foto/galeria.
- **Tela de revisão** `abrirRevisaoIsbn(res, atual, aoAplicar)` (sub-sheet, ids `isb-*` — sem `-save`, então não pede confirmação de descarte): uma linha com caixa por campo achado e diferente do formulário. **Marcado por padrão só onde o formulário está vazio** (campo já preenchido mostra "Atual: …" e vem desmarcado; capa atual idem). "Usar marcados" só **preenche o formulário**; **nada é gravado até Salvar** (regra de ouro 8). Autores achados são somados aos já escolhidos: nome igual (sem acento/caixa) reaproveita o cadastro, senão `criarAutorComChecagem` (pergunta se houver nome parecido; nunca une sozinho).
- O service worker já ignora requisições de outros domínios, então as APIs não passam pelo cache. Sem mudança de esquema (`DB_VERSION` segue 1).
- Fora do escopo desta versão: outras fontes de busca. O scanner de câmera veio na v1_05 (seção 7d).

## 7d. Scanner de código de barras (v1_05)

- **Botão 📷** no formulário do livro, ao lado do campo ISBN (`lv-isbn-scan`). Abre `#scanOverlay` (tela cheia preta, z-index 70: acima das camadas, abaixo da foto ampliada e dos diálogos), que é uma entrada de modal na `navStack` (o voltar do celular fecha só o scanner: `closeScanDom` na cadeia do `popstate`; `camadaTopoAtiva()` devolve `null` com ele aberto, então o voltar não pergunta de descarte do formulário).
- **Leitura:** `escanearIsbn()` → `getUserMedia` (câmera traseira, 1280×720) + `BarcodeDetector({formats:['ean_13']})` num laço de ~180 ms (`loopLeitura`, encerrado por `_scanId`). Só aceita `97[89]` + 10 dígitos com dígito verificador (`isbnValido`); EAN-13 que não é de livro mostra dica e continua lendo; após 15 s sem leitura mostra dica de uso. Vibra ao ler.
- **Resultado:** `escanearIsbn()` devolve o ISBN-13 lido, `''` (botão "Digitar à mão": o formulário foca o campo ISBN) ou `null` (voltar/cancelar/sem câmera). `concluirScan(valor)` fecha o scanner e entrega o valor; a câmera é sempre solta (`pararCamera`), inclusive ao ir para segundo plano (`visibilitychange`).
- **Depois de ler:** preenche o ISBN, mostra o aviso de duplicado e, se o ISBN já existe em outro livro, abre `escolherIsbnRepetido(rep,'leitura')` (abrir o existente / continuar cadastrando / voltar; o mesmo seletor é usado ao salvar). Se não há duplicado (ou o usuário escolhe continuar), dispara a busca online da v1_04 (`btnBusca.click()`), que só mostra a revisão: **nada é gravado até Salvar**.
- **Lanterna:** botão "🔦 Luz" só aparece se a câmera declarar `torch` (`applyConstraints`).
- **Sem suporte:** sem `BarcodeDetector` (ou sem EAN-13), sem contexto seguro ou sem `mediaDevices`, o app avisa e o usuário digita o ISBN. Permissão negada, câmera ausente/ocupada têm mensagens próprias (`erroCamera`). **Não há leitor em JavaScript embutido** (o Chrome do Android traz o `BarcodeDetector`); se algum aparelho não tiver, decidir o fallback.
- Sem mudança de esquema (`DB_VERSION` segue 1); o service worker não precisa de ajuste.

## 7e. Leituras (v1_06)

- **Dados:** `leitura = {livroId, inicio, fim, status}` e `progresso = {leituraId, pagina, dataHora}`; datas em `new Date().toISOString()` (UTC), exibidas no horário local por `fmtDataHora`/`fmtDataCurta`. **A página atual é derivada**: página do último progresso (ordenado por `dataHora`, depois `id`), ou 0 se não há registro (`paginaAtualLeitura`). Assim editar/excluir registros (v1_07) nunca deixa a leitura inconsistente. Sem mudança de esquema (`DB_VERSION` segue 1); o backup genérico já leva os dois stores (testado ida e volta).
- **Helpers:** `leiturasDoLivro`, `leiturasAndamento` (mais recente primeiro; normalmente 0 ou 1), `progressosDaLeitura`, `ultimoProgresso`, `percentualLeitura(lt, livro)` (arredonda **para baixo**: 349 de 350 = 99%; `null` sem total de páginas; teto 100) e `percentualLivro(livroId)` (só leitura em andamento; usado nas listagens).
- **Detalhe do livro:** seção "Minhas leituras" (`minhasLeiturasHTML`): em andamento em destaque, finalizadas em lista (mais nova primeiro, "Leitura N" pela ordem de início) e botão "Iniciar leitura" (vira "Iniciar nova leitura", em contorno se já há uma em andamento). Vale para qualquer situação do livro (inclusive desejo). `iniciarLeitura`: se já há uma em andamento, `confirmar` antes; ao iniciar abre a tela da leitura.
- **Tela da leitura** `abrirLeitura(id, origem)` (sheet **principal**, substitui o conteúdo como o formulário do livro; origem `'livro'` volta ao detalhe, `'lista'` fecha; o voltar do celular fecha a sheet): título, status, datas, página atual grande, % e barra, último registro, campo "Página atual" + "Registrar página" (Enter também), "Finalizar leitura" e "Excluir leitura". Três **abas** (v1_08, ver 7g): Progresso (progresso, histórico da 7f, finalizar/reabrir, excluir), Anotações e Personagens.
- **Registrar:** inteiro ≥ 0; acima do total de páginas (se houver) é impedido; menor que a última página registrada pede `confirmar` citando a anterior (cancelar não grava); igual ou maior grava direto. Data/hora automáticas.
- **Finalizar** (`finalizarLeitura`): com total de páginas e progresso < 100% abre o seletor (`escolherNoPicker`): manter página atual / alterar para 100% (grava também um progresso com a última página) / voltar; nos demais casos só `confirmar` (que lembra que dá para reabrir; ver 7f). Grava `fim` e `status:'finalizada'`; leitura finalizada não aceita novos registros.
- **Excluir leitura** (`excluirLeitura`, não está na especificação; necessária para desfazer início ou finalização por engano): `confirmar` com contagem; apaga a leitura, seus progressos e suas anotações (`leituraId`); o livro e as outras leituras ficam.
- **Aba Leituras** (`renderLeituras`, `#leituras-lista`): só leituras em andamento, a de atividade mais recente primeiro (último progresso, senão início); card com capa, título, página/%, barra e data de início; toque abre `abrirLeitura(id,'lista')`. É também a página inicial padrão.
- **Armadilha:** o CSS `.det-sec-t` usa `text-transform:uppercase`, então `innerText` devolve maiúsculas; nos testes Playwright conferir textos com `textContent`.

## 7f. Histórico de progresso e reabrir leitura (v1_07)

- **Seção "Histórico"** na tela da leitura (`#lt-hist`, `desenharHistorico(id, origem)`), com alternância **Gráfico** (padrão; cada leitura abre no gráfico: `_histLeitura`/`_histModo`) e **Lista**. Sem registros: só a mensagem "Nenhuma página registrada ainda".
- **Gráfico** (`graficoProgressoSVG`): SVG inline (cores por variáveis CSS, serve nos dois temas). Eixo X proporcional ao tempo, do início da leitura ao último registro (rótulos em hora se o intervalo é < 24 h, senão dd/mm); eixo Y de 0 ao maior entre total de páginas e maior página registrada. O **início da leitura (página 0) é só uma âncora tracejada**, não um registro. Registro **regressivo** (página menor que a do registro anterior, `registrosComRegressao`) aparece com ponto vazado e segmento na cor de perigo; a legenda só cita isso quando existe.
- **Lista:** registros do mais recente ao mais antigo, com página, % (se há total) e data/hora; os regressivos levam "↓ menor que a anterior". **Tocar no registro** mostra Editar/Excluir (`_histSel`; tocar de novo esconde): nenhum botão fica visível o tempo todo.
- **Editar registro** (`editarProgresso`, sub-sheet com ids `pg-*`, `pg-save` ativa o aviso de descarte): página (0 até o total, se houver), data e hora. A data/hora **só é regravada se mudou** (mantém os segundos originais). Recusa: anterior ao início da leitura, posterior ao fim (se finalizada) e futura. Vale também para leituras finalizadas.
- **Excluir registro** (`excluirProgresso`): `confirmar` citando página e data; a página atual passa a ser a do registro anterior (é derivada, ver 7e).
- **Reabrir leitura finalizada** (`reabrirLeitura`, botão "↺ Reabrir leitura" só em leitura finalizada; **não está na especificação**, pedido do usuário): volta `status` a `andamento` e `fim` a `null`; histórico e anotações ficam. Se o livro já tem **outra** leitura em andamento, avisa antes (mesma regra de iniciar); senão pede confirmação simples citando o fim que será apagado.
- Sem mudança de esquema (`DB_VERSION` segue 1); o backup genérico já leva tudo (testado ida e volta com registro editado).

## 7g. Anotações e personagens (v1_08)

- **Abas da tela da leitura** (`#lt-abas`; painéis `#lt-pane-progresso|anotacoes|personagens`): `mostrarAbaLeitura`, `iniciarAbasLeitura`, `atualizarAbasLeitura(id, origem)` (redesenha só Anotações/Personagens e as contagens, sem reabrir a tela). Estado do módulo: `_ltAba` e `_anotFiltro`, reiniciados (Progresso, sem filtro) ao abrir outra leitura. Tudo que já existia (progresso, histórico, finalizar/reabrir, excluir leitura) ficou dentro da aba Progresso.
- **Personagens** (`desenharPersonagens`, `abrirFormPersonagem`): são **do livro** (`livroId`), compartilhadas entre as leituras dele. Lista com comentário resumido e nº de anotações citando; form em sub-sheet (ids `pn-*`): nome obrigatório, comentário, "Ver anotações desta leitura sobre X" (aplica o filtro e vai à aba Anotações), excluir. **Nome igual** (sem acento/caixa) no mesmo livro é recusado; **parecido** (`personagensParecidas`, mesmo `nomesSemelhantes` dos autores) pede confirmação. **Excluir personagem citada**: `confirmar` com a quantidade de anotações; as anotações ficam e só perdem o vínculo (`personagemIds` é regravado sem ela): nunca sobra referência quebrada.
- **Anotações** (`desenharAnotacoes`, `abrirFormAnotacao`): pertencem à **leitura** (também as finalizadas). Lista da mais recente à mais antiga (`dataHora`, depois id), cartão com texto (preserva quebras de linha), data/hora, página (se houver) e etiquetas das personagens; tocar abre a edição. Filtro "Mostrar": Todas ou "Sobre <personagem>" (só personagens do livro; some se a personagem for excluída). Form em sub-sheet (ids `an-*`, `an-save` ativa o aviso de descarte): texto obrigatório, página opcional (inteiro ≥ 0, não acima do total do livro), personagens por chips + botão "+ Personagem", data e hora **automáticas mas editáveis** (não pode ser futura; se o usuário não mexe, a anotação nova guarda a hora exata do salvamento e a editada mantém a original), excluir com `confirmar`.
- **Personagem dentro da anotação** (`escolherPersonagemParaAnotacao`): o seletor lista as personagens do livro ainda não escolhidas e "+ Nova personagem" (pergunta o nome; `criarPersonagemComChecagem`: nome igual/parecido oferece usar a existente, criar outra ou cancelar). A personagem nova **já é gravada** ao criar (como autor no formulário do livro), mesmo que a anotação não seja salva depois.
- `@menções` no texto vieram na v1_09 (seção 7h). Excluir leitura e excluir livro já apagavam anotações (e personagens, no caso do livro) desde a v1_06.
- Sem mudança de esquema (`DB_VERSION` segue 1); os stores `anotacoes` e `personagens` já existiam e o backup genérico os leva (testado ida e volta).

## 7h. @menções nas anotações (v1_09)

- **Modelo:** o texto guarda `@Nome` em texto puro; o vínculo real continua sendo `personagemIds`. Regra: **personagem citada com `@` no texto fica vinculada** (`idsMencionados`: nome mais longo primeiro, e o trecho já usado é mascarado, então "@Ana Maria" não vincula "Ana"; o `@` precisa vir no começo ou depois de algo que não é letra/dígito, então "a@b" não conta; sem diferenciar maiúsculas). Se a menção é apagada do texto, o vínculo some. Personagens vinculadas **só pelo botão "+ Personagem"** (v1_08) continuam, com ✕. Ao salvar: `personagemIds` = menções do texto + as do botão.
- **Chips do formulário:** menções aparecem como "@Nome" **sem ✕** (para remover, apague do texto); as do botão, com ✕. Ao abrir uma anotação para editar, as já vinculadas que o texto cita contam como menção e as outras como "do botão".
- **Autocompletar** (`contextoMencao`, `atualizarSug`, `#an-sug`): ao digitar `@` (até 30 caracteres, sem quebra de linha) aparecem até 6 personagens do livro cujo nome contém o que foi digitado (as que começam com o texto primeiro; `@` sozinho mostra todas). Tocar numa sugestão troca o trecho por `@Nome ` e leva o cursor para depois (a sugestão usa `mousedown` com `preventDefault`, então o teclado do celular não fecha). "@Nome " com o nome completo seguido de espaço encerra a menção (só continua sugerindo se existir nome maior que começa igual, ex.: "@Ana " → Ana Maria).
- **Nova personagem pela menção:** se o texto digitado não é de nenhuma personagem, a lista oferece "+ Cadastrar “texto”". **Antes de criar, `confirmar` pergunta explicitamente.** Confirmou: `historicoAssentar()` e `criarPersonagemComChecagem` (nome igual/parecido oferece usar a existente), grava a personagem e insere a menção. Recusou (ou cancelou o seletor): **o texto digitado fica como está, nada é criado** e o autocompletar não volta a oferecer para aquele mesmo `@` (`ignorarPos`) até a menção mudar.
- **Cartão da anotação:** `textoComMencoesHTML` destaca (`.an-men`) as menções das personagens vinculadas; `@algo` sem personagem fica como texto comum.
- **Renomear personagem** (`renomearMencoes`): as menções `@Antigo` nas anotações que a citam passam a `@Novo` (o formulário avisa). **Excluir personagem:** as anotações ficam, o `@Nome` permanece no texto sem vínculo, e o aviso de exclusão diz isso.
- Sem mudança de esquema (`DB_VERSION` segue 1); o backup genérico já cobre tudo (o texto e `personagemIds` são os mesmos campos da v1_08).

## 8. Backup (reaproveitado do Preços, adaptado)

- Arquivo `{app:'livros', versao:1, exportadoEm, stores, configuracoes}`; nome `livros_backup_AAAA-MM-DD_HHhMM.json`.
- Exportação genérica: percorre `db.objectStoreNames` e as chaves `livros_*`. Backup do app Preços (`app:'precario'`) é recusado com mensagem clara.
- Restauração: **apaga os dados atuais** e substitui (uma transação; falhou = nada muda), só nos stores que o banco conhece.
- **Alerta de 7 dias** (novo; o Preços não tem): `renderAvisoBackup()` mostra faixa no topo se há dados e o último backup (ou o primeiro uso, se nunca houve) tem 7 dias ou mais. Sai assim que o backup é feito.
- `navigator.storage.persist()` ativo.

## 9. Padrões visuais

- CSS herdado do Preços (tokens `--paper`, `--card`, `--ink`, `--accent`, `--line`...), com identidade própria sobreposta no final do `<style>` (bordô `#7A2E3A`, dourado no logo). Modo escuro: classe `html.dark`.
- Tipografia dos modos de leitura: 3 estilos — título `.view-heading`, texto `.leitura` (14 px), nota `.view-nota`/`.hint`/`.obs` (12 px cinza).
- Listagens de livros mostram **só capa, título e % de leitura**; autor, categoria e status ficam no detalhe e nos filtros.
- Há CSS herdado sem uso (receitas etc.); pode ser removido aos poucos.

## 10. Reaproveitado do Preços × novo

- Reaproveitado (copiado): CSS base, máscaras de valor e quantidade, diálogos, pilha de navegação e camadas, lightbox, `resizeImage`, SW, estrutura do backup, tema.
- Adaptar nas fases seguintes: módulo de preços (sem quantidade de referência/conversões), gráfico em canvas (progresso de leitura), filtros "+ Filtros".
- Novo: scanner de código de barras (v1_05), busca por ISBN, leituras/progresso, anotações com `@personagem`, visões salvas, alerta de backup.

## 11. Armadilhas conhecidas

- Mesmo domínio que o Preços: nunca ler/gravar chaves fora de `livros_*`.
- Chrome instalado como PWA: após publicar, o service worker só atualiza se `CACHE` mudou.
- Hook de "unpushed" pode reclamar: `git fetch origin <branch>:refs/remotes/origin/<branch>`.
- **Modais em sequência:** quando um seletor/diálogo fecha e outro abre logo em seguida (ex.: seletor → `perguntar` → seletor), o `history.back()` do primeiro ainda está em andamento e o histórico do navegador se corrompe (a página chegou a ir para `about:blank`). Entre os dois, usar `await historicoAssentar()` (espera os `history.back()` pendentes, via `executarAposHistorico`).
- No Playwright, `page.evaluate` de função que abre diálogo e espera a resposta trava o teste: chamar sem aguardar (`excluirLivro(id); return 1`) e clicar no diálogo depois.
- No Playwright, handlers `async` (backup, restauração) exigem espera antes de verificar o resultado.
