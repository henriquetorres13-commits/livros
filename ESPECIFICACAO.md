# Especificação funcional --- Aplicativo de Biblioteca e Diário de Leitura

**Versão:** 1.0 --- consolidação dos requisitos\
**Status:** levantamento encerrado; documento-base para desenvolvimento

## 1. Objetivo

Desenvolver um aplicativo pessoal para organizar livros, acompanhar
leituras e registrar anotações e personagens. O aplicativo deverá ser
simples, intuitivo, priorizar o uso em celular e funcionar
prioritariamente offline.

O escopo inicial contempla: - Biblioteca de livros próprios e
emprestados. - Lista de desejos. - Cadastro e consulta de autores. -
Organização por categorias e subcategorias. - Registro de uma ou mais
leituras por livro. - Histórico de progresso. - Anotações vinculadas a
leituras e referências a personagens. - Histórico de preços dos
livros. - Backup completo e restauração.

Não faz parte desta versão criar módulos de vendas, estoque ou gestão de
empréstimos com prazos.

## 2. Diretrizes gerais

1.  **Simplicidade:** poucas opções por tela e foco nas ações mais
    utilizadas.
2.  **Praticidade:** evitar campos e cadastros que não tragam benefício
    claro.
3.  **Uso móvel:** não poluir telas pequenas com informações
    secundárias.
4.  **Consistência:** mecanismos semelhantes devem ter experiência de
    uso semelhante.
5.  **Offline primeiro:** funções principais disponíveis sem conexão.
6.  **Evolução controlada:** não antecipar recursos sem necessidade
    comprovada.
7.  **Reaproveitamento:** aproveitar arquitetura, componentes e
    funcionalidades do aplicativo de preços existente sempre que
    possível.

## 3. Navegação e apresentação

### 3.1 Página inicial

O usuário poderá escolher nas configurações uma das seguintes páginas
iniciais: - **Leituras em andamento:** apresenta os livros que estão
sendo lidos e oferece acesso rápido ao progresso e às anotações. -
**Tela de atalhos:** acessos à Biblioteca, Lista de desejos, Leituras em
andamento e Pesquisa.

A preferência poderá ser alterada posteriormente. Não haverá painel de
estatísticas nesta versão.

### 3.2 Navegação principal

A navegação principal deverá conter quatro áreas: 1. Biblioteca. 2.
Lista de desejos. 3. Leituras. 4. Pesquisa.

Configurações e cadastros auxiliares deverão ficar em acesso secundário.

### 3.3 Identidade visual e temas

-   Identidade visual própria, adequada ao tema de leitura.
-   Reaproveitar padrões e componentes visuais do aplicativo de preços
    quando fizer sentido.
-   Disponibilizar tema claro e tema escuro, com escolha nas
    configurações.

### 3.4 Modelo de implementação

O novo aplicativo deverá seguir o mesmo modelo de implementação,
funcionamento e disponibilização do aplicativo de preços existente,
reaproveitando sua arquitetura e soluções técnicas sempre que possível.
Não detalhar nesta especificação tecnologias de instalação ou
distribuição.

## 4. Biblioteca e lista de desejos

### 4.1 Apresentação dos livros

Disponibilizar duas formas de visualização: - **Grade:** capas em
miniatura. - **Lista:** capa pequena e informações essenciais.

O usuário poderá definir o layout padrão nas configurações e alternar
entre grade e lista na própria tela. A mesma lógica poderá ser utilizada
na Lista de desejos.

Nas listagens, exibir somente: - Capa. - Título. - Percentual de
leitura, quando aplicável.

Não exibir autor, categoria/subcategoria ou status de leitura na
listagem inicial. Essas informações estarão disponíveis ao abrir o livro
ou utilizar filtros.

### 4.2 Visões predefinidas e personalizadas

Disponibilizar inicialmente estas visões: - **Meus livros:** livros
comprados e emprestados. - **Lista de desejos:** livros que o usuário
pretende comprar. - **Lidos:** livros cuja leitura foi concluída.

"Lidos" é uma visão/filtro, não uma situação de aquisição. Um livro lido
continua aparecendo em "Meus livros" se for próprio ou emprestado.

O usuário poderá: - Criar visualizações com nome próprio. - Combinar
vários filtros. - Excluir visualizações, inclusive as predefinidas. -
Reorganizar a ordem das visualizações. - Restaurar as visões originais.

Exemplo: "Ficção para ler" pode combinar livro próprio, não lido e
categoria Ficção.

A administração dessas visões deverá ficar em local próprio, sem ocupar
espaço permanente na biblioteca.

Quando um livro da Lista de desejos for marcado como comprado, o mesmo
registro deverá passar a aparecer em Meus livros. Não duplicar o livro
nem perder histórico de preços ou outros dados.

## 5. Cadastro de livros

### 5.1 Formas de cadastro

Disponibilizar: 1. Leitura do código de barras com a câmera do celular,
normalmente EAN-13/ISBN-13. 2. Digitação de ISBN-10 ou ISBN-13. 3.
Cadastro manual.

Ao pesquisar por ISBN/código de barras, tentar obter informações online
de fontes como Google Books e Open Library. Poderão ser consultadas mais
de uma fonte e diferentes edições, quando necessário.

Dados que poderão ser obtidos, conforme disponibilidade: - Título e
subtítulo. - Autores. - Capa. - Quantidade de páginas. - Descrição. -
Outros dados bibliográficos úteis.

Os dados encontrados deverão ser apresentados para revisão. Nada será
salvo sem confirmação do usuário. Todos os campos pré-preenchidos
deverão poder ser editados, complementados ou apagados. Se a pesquisa
não encontrar dados, o cadastro manual deverá continuar disponível.

A imagem da capa é desejável, mas não obrigatória. As imagens deverão
ser compactas e adequadas apenas à identificação visual do livro.

### 5.2 Campos

Somente o título será obrigatório. Os demais campos serão opcionais,
incluindo: - Subtítulo. - ISBN. - Autores. - Capa. - Quantidade de
páginas. - Descrição. - Comentários pessoais gerais sobre o livro. -
Categoria e subcategoria. - Preço e histórico de preços.

O cadastro manual deverá funcionar normalmente sem ISBN.

### 5.3 Duplicidade de livros

-   Se o ISBN informado já existir, avisar e oferecer acesso ao registro
    existente.
-   A verificação por ISBN deve evitar duplicidade acidental.
-   Sem ISBN, não implementar regras complexas por título, pois livros
    diferentes podem ter o mesmo título.

### 5.4 Autores

Manter cadastro independente e reutilizável de autores, contendo apenas
o nome.

Um livro poderá ter vários autores. No cadastro do livro: - Sugerir
autores existentes conforme o usuário digita. - Permitir selecionar
vários autores. - Permitir cadastrar um novo autor sem sair da tela do
livro.

Para evitar duplicidades: - Comparar nomes ignorando
maiúsculas/minúsculas e acentos. - Alertar sobre nomes iguais ou
possivelmente semelhantes. - Nunca fundir registros automaticamente. -
Permitir ao usuário escolher entre reutilizar o autor existente ou criar
outro.

A semelhança é apenas um alerta; nomes parecidos podem pertencer a
pessoas diferentes.

### 5.5 Categorias e subcategorias

Manter cadastro independente de categorias e subcategorias para livros,
com classificação opcional no livro. O usuário poderá criar, editar e
excluir essas classificações. A organização poderá representar gêneros,
temas ou qualquer critério escolhido pelo usuário.

### 5.6 Empréstimos

Manter controle simples: - Identificar se o livro é próprio ou
emprestado. - Para livro emprestado, permitir informar o nome de quem
emprestou.

Não implementar datas, prazos, alertas ou gestão de devolução nesta
versão.

## 6. Leituras

### 6.1 Iniciar leitura

Dentro do cadastro do livro, disponibilizar a seção **Minhas leituras**,
com opção de iniciar uma nova leitura.

Ao iniciar: - Registrar automaticamente data e hora de início. - Definir
status "Em andamento". - Iniciar página atual em zero até o primeiro
registro de progresso. - Disponibilizar as personagens já cadastradas
para o livro. - Permitir registrar anotações.

Se já houver leitura em andamento para o mesmo livro, avisar antes de
permitir iniciar outra.

### 6.2 Múltiplas leituras do mesmo livro

Um livro poderá ter várias leituras ao longo do tempo: - A leitura em
andamento ficará em destaque. - Leituras finalizadas serão apresentadas
em lista. - Cada leitura manterá seu próprio progresso, datas e
anotações. - Personagens serão compartilhadas entre todas as leituras do
mesmo livro. - Iniciar uma nova leitura não modificará registros
anteriores.

### 6.3 Tela da leitura

Organizar a tela em três abas: 1. **Progresso:** página atual e
histórico. 2. **Anotações:** registros da leitura, com ações para
adicionar e editar. 3. **Personagens:** cadastro e consulta das
personagens do livro.

### 6.4 Registro de progresso

Na aba Progresso, destacar o campo para informar a página atual e o
botão **Registrar página**.

Ao registrar: - Data e hora serão automáticas. - Preservar todos os
registros anteriores. - Atualizar percentual de conclusão quando o total
de páginas estiver cadastrado. - Aceitar página maior, igual ou menor
que o registro anterior. - Se a página for menor que a última
registrada, avisar com a página anterior e pedir confirmação antes de
salvar. - Se o usuário cancelar, não alterar o histórico.

Valores inválidos, como páginas negativas ou acima do total conhecido,
poderão ser impedidos com validação simples.

### 6.5 Histórico de progresso

Disponibilizar duas visualizações: - **Gráfico**, padrão: evolução
acumulada das páginas ao longo do tempo. - **Lista:** registros
individuais.

O usuário poderá alternar entre elas. O gráfico deverá representar
também registros regressivos, quando o usuário registrar uma página
menor.

Na lista, ao selecionar um registro, oferecer discretamente as opções de
editar ou excluir, sem manter botões de ação permanentemente visíveis.

### 6.6 Finalizar leitura

Disponibilizar o botão **Finalizar leitura**.

Ao finalizar: - Registrar automaticamente data e hora de conclusão. -
Alterar status para "Finalizada". - Preservar histórico de progresso e
anotações. - Permitir iniciar uma nova leitura do mesmo livro
posteriormente.

Se o livro tiver total de páginas cadastrado e o progresso estiver
abaixo de 100%, perguntar: - **Manter página/percentual atual:**
finalizar sem alterar o progresso. - **Alterar para 100%:** registrar
também a página final no histórico de progresso.

Se o total de páginas não estiver cadastrado, finalizar normalmente, sem
essa pergunta.

## 7. Anotações e personagens

### 7.1 Anotações

Cada leitura poderá ter várias anotações. Cada anotação terá: - Texto
livre. - Data e hora automáticas. - Página opcional. - Referências
opcionais a uma ou mais personagens.

O usuário poderá editar anotações existentes. Data e hora serão
preenchidas automaticamente, mas poderão ser alteradas manualmente.

As anotações pertencem à leitura. O campo de comentários gerais do livro
é independente.

### 7.2 Organização das anotações

Disponibilizar: 1. **Todas:** ordem cronológica, da mais recente para a
mais antiga. 2. **Por personagem:** selecionar uma personagem e
visualizar somente as anotações que fazem referência a ela.

### 7.3 Personagens

As personagens pertencem ao livro, não a uma leitura específica, e ficam
compartilhadas entre todas as leituras daquele livro.

Cadastro simples: - Nome obrigatório. - Comentário opcional.

Permitir cadastrar, editar e consultar personagens, inclusive
diretamente durante a escrita de uma anotação.

Ao digitar `@` na anotação: - Apresentar personagens já cadastradas para
seleção. - Se o texto não corresponder a uma personagem existente,
oferecer a opção de cadastrar uma nova. - Antes de criar, perguntar
explicitamente se o usuário deseja cadastrar a nova personagem. - Se
confirmar, criar o cadastro e vincular à anotação. - Se não confirmar,
manter o texto digitado sem criar personagem.

As menções deverão ser vínculos reais com os cadastros, permitindo
localizar anotações por personagem. Uma anotação poderá referenciar
nenhuma, uma ou várias personagens.

Ao excluir uma personagem que tenha referências, a aplicação deverá
avisar sobre os vínculos e pedir confirmação para evitar referências
quebradas.

## 8. Preços dos livros

O módulo de preços deverá aproveitar os recursos já existentes no
aplicativo de preços.

**Não desenvolver novamente como funcionalidades novas** os recursos já
disponíveis, tais como cadastro de preços, histórico, gráficos e
mecanismos relacionados. A especificação deverá identificá-los como
componentes reaproveitados e indicar somente adaptações necessárias para
integrá-los aos livros.

O histórico poderá registrar preços efetivamente pagos ou preços apenas
observados, preservando a distinção entre as duas situações. Os detalhes
funcionais deverão seguir o comportamento já implementado no aplicativo
de preços, salvo adaptações indispensáveis à integração.

## 9. Pesquisa e filtros

### 9.1 Pesquisa geral

Disponibilizar pesquisa geral por: - Título. - Subtítulo. - Autor. -
Personagem.

A pesquisa deverá ignorar diferenças entre maiúsculas/minúsculas e
acentuação.

### 9.2 Filtros adicionais

Disponibilizar filtros contextuais, conforme a tela, incluindo: -
Categoria. - Subcategoria. - Autor. - Situação do livro, como
próprio/comprado, emprestado, na lista de desejos, lido ou em andamento.

### 9.3 Combinações salvas

Permitir salvar combinações de filtros com nomes definidos pelo usuário,
para reutilização.

A experiência de criação, edição, exclusão e organização dessas
combinações deverá ser visual e operacionalmente semelhante à
administração das visualizações da biblioteca. O objetivo é manter um
padrão único de utilização.

## 10. Backup, importação e restauração

Reaproveitar o mecanismo de backup e restauração do aplicativo de
preços, com as adaptações necessárias.

Disponibilizar somente: - Exportação completa de todos os dados. -
Importação completa para restauração.

O arquivo completo deverá preservar: - Livros e seus dados. - Autores. -
Categorias e subcategorias. - Leituras e históricos de progresso. -
Anotações e personagens. - Registros de preços. - Imagens. -
Configurações. - Visualizações e filtros personalizados. -
Relacionamentos entre todos os registros.

A restauração deverá permitir transferir os dados para outro aparelho,
preservando sua integridade.

Manter alerta quando não houver backup realizado nos últimos 7 dias.

Não incluir exportações parciais ou importação de bibliotecas de outros
aplicativos nesta versão.

## 11. Funcionamento e proteção

-   O aplicativo deverá funcionar prioritariamente offline.
-   Consulta e cadastro de livros, progresso, anotações e personagens
    deverão funcionar sem internet.
-   A internet será necessária para serviços externos, como busca de
    dados por ISBN e obtenção de informações online.
-   Os dados ficarão armazenados localmente no aparelho.
-   Não haverá sincronização automática entre aparelhos, contas de
    usuário ou armazenamento em nuvem nesta versão.
-   A transferência entre aparelhos será feita por exportação e
    importação completas.
-   Seguir o mesmo modelo de acesso e proteção do aplicativo de preços,
    sem criar login ou conta próprios.

## 12. Exclusão de livros

Ao excluir um livro: - Verificar se existem leituras, anotações,
personagens, histórico de preços ou outros dados associados. - Se houver
dados relacionados, informar que eles também serão excluídos. - Exigir
confirmação explícita do usuário.

Não implementar lixeira ou recuperação de registros excluídos nesta
versão.

## 13. Fora do escopo da primeira versão

Não desenvolver nesta etapa: - Módulo de vendas. - Controle de estoque
ou inventário. - Gestão de empréstimos com datas, prazos, alertas ou
devoluções. - Sincronização automática entre dispositivos. - Contas de
usuário ou armazenamento em nuvem. - Estatísticas e painéis de
leitura. - Importação de bibliotecas provenientes de outros
aplicativos. - Exportações parciais. - Lixeira ou recuperação de livros
excluídos.

## 14. Reaproveitamento do aplicativo de preços

A implementação deverá começar pela avaliação do aplicativo de preços
existente, identificando arquitetura, componentes e funcionalidades que
possam ser reaproveitados.

A especificação distingue: - **Funcionalidades novas:** recursos
próprios da biblioteca, do acompanhamento de leitura, das anotações e
personagens. - **Funcionalidades reaproveitadas:** recursos já
implementados no aplicativo de preços, como o módulo de preços e o
mecanismo de backup/restauração, além da arquitetura e soluções técnicas
compatíveis.

O esforço de desenvolvimento deverá considerar como novas apenas as
funcionalidades inéditas e as adaptações efetivamente necessárias. Não
contabilizar como desenvolvimento do zero recursos já existentes e
reaproveitáveis.

## 15. Orientação para desenvolvimento

Antes de implementar, avaliar o aplicativo de preços existente para
confirmar quais componentes podem ser reutilizados diretamente e quais
exigem adaptação. Priorizar entregas simples e funcionais, mantendo a
experiência intuitiva e evitando complexidade desnecessária.

Esta especificação registra o escopo funcional acordado para a primeira
versão. Decisões técnicas de implementação deverão respeitar o modelo já
utilizado no aplicativo de preços e não ampliar o escopo sem necessidade
ou aprovação.
