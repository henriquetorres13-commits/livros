# ARQUITETURA.md — App "Livros"

Referência para quem for mexer no código (Claude ou pessoa). Ler inteiro antes de começar uma versão. Conferido contra o código da v1_01 em 2026-11-03; se algo divergir do `index.html`, o código vale e este documento deve ser corrigido no mesmo commit.

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
| `autores` | `AUTORES` | só `nome` |
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
- Camadas de sheet (`criarCamada`): principal (`openSheet`), sub (`openSubSheet`), sub2 (`openSub2Sheet`); têm `dirty`/`salvando`/`confirmarDescarte` (sair de edição com alterações pede confirmação). Seletor sobreposto `pickerOverlay` (z-index 65) reservado para seletores (autores, categorias).
- **Só empilha** por atalhos dentro das telas. Barra inferior e itens do Mais são **raiz** e zeram a pilha: `switchView(name, {raiz:true})`.
- Barra inferior: Biblioteca, Desejos, Leituras, Pesquisa, Mais. Em Mais: Tela de atalhos, Backup e restauração, Configurações. `VIEWS_DO_MAIS` lista as telas que acendem a aba Mais.
- Página inicial configurável: Leituras (`view-leituras`) ou Atalhos (`view-inicio`).
- `renderCurrent()` é o ponto único de redesenho; cada tela nova ganha seu `render*()` chamado ali.
- Diálogos: `avisar(msg)`, `confirmar(msg)` → boolean, `perguntar(msg, valor)` → texto|null.

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
- Novo: scanner de código de barras, busca por ISBN, leituras/progresso, anotações com `@personagem`, visões salvas, alerta de backup.

## 11. Armadilhas conhecidas

- Mesmo domínio que o Preços: nunca ler/gravar chaves fora de `livros_*`.
- Chrome instalado como PWA: após publicar, o service worker só atualiza se `CACHE` mudou.
- Hook de "unpushed" pode reclamar: `git fetch origin <branch>:refs/remotes/origin/<branch>`.
- No Playwright, handlers `async` (backup, restauração) exigem espera antes de verificar o resultado.
