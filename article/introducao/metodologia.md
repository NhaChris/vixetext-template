## Metodologia

Este trabalho classifica-se, quanto à natureza, como aplicada, pois visa à produção de um artefato de software, denominado CodeMage, destinado à solução de um problema concreto: o apoio ao ensino de lógica de programação.

Quanto aos objetivos, caracteriza-se como exploratória e descritiva, na medida em que explora a construção de uma plataforma em que a programação é a mecânica de jogo e descreve detalhadamente as decisões de projeto e implementação adotadas. Quanto à abordagem, é predominantemente qualitativa, centrada na análise das escolhas arquiteturais e de suas implicações pedagógicas e técnicas.

A metodologia base adotada é a da pesquisa de *Design Science* (ou *Design Science Research*), que orienta a construção e a avaliação de artefatos tecnológicos como meio de gerar conhecimento \cite{dresch2015design}. Sob essa ótica, o CodeMage é simultaneamente o objeto construído e o instrumento por meio do qual se examina a questão de pesquisa. \citeonline{lacerda2013design} descrevem o método em cinco etapas principais: conscientização, sugestão, desenvolvimento, avaliação e conclusão, às quais se acrescenta a comunicação dos resultados.

As atividades deste trabalho foram organizadas de acordo com essas etapas. As atividades de (a) a (f) foram realizadas na primeira fase do Trabalho de Conclusão de Curso (TCC) e resultaram na primeira versão da plataforma, descrita no \autoref{cap_plataforma}. As atividades (g) e (h) serão conduzidas na segunda fase, o TCC 2.

a. Levantamento e fundamentação (conscientização): revisão da literatura sobre dificuldades no ensino de lógica de programação e pensamento computacional, gamificação e aprendizagem baseada em problemas, bem como o estudo de jogos correlatos que empregam código como mecânica;

b. Modelagem da arquitetura (sugestão): definição de uma arquitetura modular que isola as regras de combate da camada de apresentação, atribuindo a cada módulo uma responsabilidade única;

c. Implementação do núcleo seguro (desenvolvimento): construção do ambiente isolado (*sandbox*) responsável por compilar e executar o código do jogador, com as defesas necessárias contra laços infinitos, exaustão de memória e acesso indevido a recursos;

d. Definição da linguagem de magias (desenvolvimento): especificação do subconjunto da linguagem Lua e da interface de programação de domínio (API) exposta ao jogador, formalizada por uma gramática escrita na forma de Backus-Naur (*Backus-Naur Form*, BNF), em sua versão estendida (*Extended Backus-Naur Form*, EBNF), apresentada na \autoref{sec_gramatica};

e. Implementação das mecânicas de jogo (desenvolvimento): desenvolvimento do modelo de combate, da progressão por círculos de magia e do sistema de imunidades dos adversários, que conferem propósito pedagógico ao avanço;

f. Validação técnica (avaliação): verificação, por meio de uma suíte de testes automatizados, de que as regras de combate se comportam como projetado, de que apenas código executável é incorporado ao jogo e de que o estado é persistido de forma confiável;

g. Evolução da plataforma (desenvolvimento, TCC 2): ampliação do conteúdo pedagógico com novos chefões e lições, melhorias no ambiente de programação e registro do processo de aprendizagem do jogador, detalhados na \autoref{sec_proximas_etapas};

h. Avaliação técnica (avaliação, TCC 2): ampliação da suíte de testes para cobrir as novas funcionalidades e verificação de que a plataforma atende aos objetivos específicos definidos neste trabalho.

As etapas de conclusão e comunicação correspondem, respectivamente, à análise dos resultados de cada fase e à redação deste documento.

### Ferramentas e tecnologias

O artefato é desenvolvido na linguagem **Lua**, executada sobre o framework **LÖVE**[^site_love] (versão 11.5), que embarca o interpretador **LuaJIT**[^site_luajit] (compatível com a especificação Lua 5.1). As características da linguagem e os motivos da escolha do framework são discutidos na fundamentação teórica (\autoref{sec_lua} e \autoref{sec_love}).

A plataforma-alvo é o ambiente *desktop*, com empacotamento do jogo como executável independente. A resolução, o título e os demais parâmetros de inicialização são centralizados em um único módulo de configuração, e os recursos visuais (*sprites*) são gerados proceduralmente, dispensando dependências de arquivos externos obrigatórios.

[^site_love]: Disponível em: <https://love2d.org/>. Acesso em: 16 jun. 2026.

[^site_luajit]: Disponível em: <https://luajit.org/luajit.html>. Acesso em: 25 set. 2026.
