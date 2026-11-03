# PLANO.md — fases de entrega do app Livros

Uma fase = uma versão = uma sessão (cabe no limite do plano gratuito). A ordem pode mudar por decisão do usuário. Atualizar a coluna "Situação" ao fechar cada versão.

| Versão | Conteúdo | Situação |
|---|---|---|
| v1_01 | Base: esqueleto, banco/prefixo próprios, 4 abas + Mais, tema claro/escuro, página inicial configurável, backup completo + restauração, alerta de 7 dias, documentação | feita; aguardando publicação |
| v1_02 | Livros manuais: cadastro, detalhe, grade/lista (+ layout padrão), categorias e subcategorias, próprio/emprestado/desejo, desejo → comprado, exclusão com aviso | |
| v1_03 | Autores (sugestão, vários por livro, criar sem sair da tela, alerta de semelhança), duplicidade de ISBN, aba Lista de desejos | |
| v1_04 | ISBN digitado + busca online (Google Books / Open Library), tela de revisão, capa | |
| v1_05 | Scanner de código de barras (câmera) | |
| v1_06 | Leituras: iniciar, registrar página (aviso de página menor), finalizar, várias leituras, aba Leituras, % nas listagens | |
| v1_07 | Histórico de progresso: gráfico (com regressões) e lista com editar/excluir discreto | |
| v1_08 | Personagens e anotações (abas da leitura, todas / por personagem, página, data editável) | |
| v1_09 | `@menções` em anotações (autocompletar, confirmar criação, vínculos reais, exclusão com aviso) | |
| v1_10 | Preços dos livros (adaptado do Preços: preço, data, local, pago × observado, histórico, gráfico) | |
| v1_11 | Pesquisa geral, filtros e visões salvas (um único mecanismo; pode precisar de 2 sessões) | |
| v1_12 | Acabamento: identidade visual, ícones, limpeza de CSS herdado, revisão contra a especificação | |

## Pontos a validar no aparelho do usuário

- v1_04: as APIs de ISBN respondem ao navegador (CORS) e a capa pode ser baixada; senão, capa por foto/galeria.
- v1_05: `BarcodeDetector` funciona no Chrome dele; senão, decidir fallback.
