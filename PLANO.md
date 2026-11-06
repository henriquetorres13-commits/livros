# PLANO.md — fases de entrega do app Livros

Uma fase = uma versão = uma sessão (cabe no limite do plano gratuito). A ordem pode mudar por decisão do usuário. Atualizar a coluna "Situação" ao fechar cada versão.

| Versão | Conteúdo | Situação |
|---|---|---|
| v1_01 | Base: esqueleto, banco/prefixo próprios, 4 abas + Mais, tema claro/escuro, página inicial configurável, backup completo + restauração, alerta de 7 dias, documentação | publicada |
| v1_02 | Livros manuais: cadastro, detalhe, grade/lista (+ layout padrão), categorias e subcategorias, próprio/emprestado/desejo, desejo → comprado, exclusão com aviso | publicada |
| v1_03 | Autores (sugestão, vários por livro, criar sem sair da tela, alerta de semelhança), duplicidade de ISBN (avisa e oferece abrir o existente; ISBN-10 = ISBN-13), tela Mais → Autores; aba Lista de desejos revisada e testada (já vinha da v1_02) | publicada |
| v1_04 | ISBN digitado + busca online (Google Books / Open Library), tela de revisão, capa | publicada |
| v1_05 | Scanner de código de barras (câmera) | publicada |
| v1_06 | Leituras: iniciar, registrar página (aviso de página menor), finalizar, várias leituras, aba Leituras, % nas listagens (inclui excluir leitura) | publicada |
| v1_07 | Histórico de progresso: gráfico (com regressões) e lista com editar/excluir discreto; **inclui reabrir leitura finalizada** (pedido do usuário) | publicada |
| v1_08 | Personagens e anotações (abas da leitura, todas / por personagem, página, data editável) | publicada |
| v1_09 | `@menções` em anotações (autocompletar, confirmar criação, vínculos reais, exclusão com aviso) | publicada |
| v1_10 | Preços dos livros (adaptado do Preços: preço, data, local, pago × observado, histórico, gráfico); inclui Mais → Locais | publicada |
| v1_11 | Pesquisa geral, filtros e visões salvas (um único mecanismo; feita numa sessão só) | publicada |
| v1_12 | Acabamento: identidade visual, ícones, limpeza de CSS herdado, revisão contra a especificação | |

## Pontos a validar no aparelho do usuário

- v1_06 a v1_10: **validadas pelo usuário em 2026-11-05** (leituras, histórico/reabrir, abas, @menções, preços e locais).
- v1_11: aba Pesquisa (texto, filtros, visões, salvar/atualizar) e Mais → Visões e filtros salvos (reordenar, renomear, excluir, restaurar originais) ainda a validar no celular.
- v1_04 (busca online por ISBN real) e v1_05 (scanner): validadas em 2026-11-04.

## Itens pedidos para versões futuras

- **Reabrir leitura finalizada** (pedido do usuário em 2026-11-04; fora da especificação): **entregue na v1_07** (botão na leitura finalizada; avisa se já há outra em andamento).
