# App "Livros" — instruções para o Claude

**Antes de qualquer alteração, leia `ARQUITETURA.md` inteiro e o `PLANO.md`** (a fase atual e o que já foi entregue). O escopo funcional está em `ESPECIFICACAO.md`: não ampliar sem aprovação. Se o código divergir da documentação, o código vale: corrija o documento no mesmo commit.

App irmão do "Preços" (repositório `pre`, mesma infraestrutura). Se precisar reaproveitar código dele, leia **só o trecho necessário** (o `index.html` do Preços tem ~7.800 linhas): use `grep -n` para localizar funções e `sed -n` para ler faixas.

## Fluxo de versão (resumo)

- Uma branch por versão (`v1_NN`). Toda versão incrementa o número, mesmo que pequena.
- Trocar **juntos** `APP_VERSION` (`index.html`) e `CACHE` (`sw.js`, formato `livros-vX_YY`).
- Antes de entregar: extrair o JS e rodar `node --check`; smoke test com Playwright; conferir que o dado novo vai e volta no backup `.json`.
- **Só mergear na `main` (ff-only) quando o usuário disser "pode publicar"**, porque a `main` publica o app instalado. Lembrar do backup `.json` antes.
- Depois de publicar, criar a branch da próxima versão e atualizar o `PLANO.md`.

## Regras que não podem quebrar

- Mesmo domínio do Preços: banco `livros_db`; toda chave de `localStorage` começa com `livros_` (nunca `precario_`).
- Nunca perder dados: mudança de esquema exige migração sem perda e novo `DB_VERSION`.
- Todo dado novo entra no backup. Store novo: criar em `onupgradeneeded` e acrescentar em `STORES`.
- HTML único. Confirmação (diálogo próprio) antes de qualquer exclusão. Nada vindo de busca online é salvo sem confirmação.

## Sobre o usuário

Não conhece GitHub: conduza o Git por ele, em português simples, e não peça que rode comandos. Usa o plano gratuito do Claude, com limite de uso: entregue **uma fase do `PLANO.md` por sessão**, resuma em poucas frases (o que mudou, como testar no celular, o que falta) e pare.
