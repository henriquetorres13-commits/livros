# ARQUITETURA.md — App "Livros"

Referência para quem for mexer no código (Claude ou pessoa). Ler inteiro antes de começar uma versão. Conferido contra o código da v1_05 em 2026-11-04; se algo divergir do `index.html`, o código vale e este documento deve ser corrigido no mesmo commit.

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
| `leituras` | `LEITURAS` | `livroId`, início, fim, `status` (`andamento`\|`finalizada`) |
| `progressos` | `PROGRESSOS` | `leituraId`, página, data/hora (preserva histórico, aceita regressão) |
| `anotacoes` | `ANOTACOES` | `leituraId`, texto, data/hora, página, referências a personagens |
| `personagens` | `PERSONAGENS` | `livroId`, `nome`, comentário (compartilhadas entre leituras do livro) |
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
- Listagem: `renderLivros(aba)`; grade/lista alternável na tela (`layoutTela`, reinicia com `CONFIG.layoutPadrao`, não é gravado). Só capa e título (o % de leitura entra na v1_06).
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
- No Playwright, handlers `async` (backup, restauração) exigem espera antes de verificar o resultado.
