## Metodologia
\label{metodologia}

Este trabalho classifica-se, quanto à natureza, como 
aplicada, pois visa à produção de um artefato de software — o jogo CodeMage — destinado à solução de um problema concreto: o apoio ao ensino de lógica de programação.

Quanto aos objetivos, caracteriza-se como exploratória e descritiva, na medida em que investiga a viabilidade de uma abordagem (programação como mecânica de jogo) e descreve detalhadamente as decisões de projeto e implementação adotadas. Já à abordagem, é predominantemente qualitativa, centrada na análise das escolhas arquiteturais e de suas implicações pedagógicas e técnicas.

A metodologia base adotada é a da pesquisa de *Design Science* (ou *Design Science Research*), que orienta a construção e a avaliação de artefatos tecnológicos como meio de gerar conhecimento \cite{dresch2015design}. Sob essa ótica, o CodeMage é simultaneamente o objeto construído e o instrumento por meio do qual se examinam as questões de pesquisa.

O desenvolvimento está organizado nas seguintes etapas:

a. Levantamento e fundamentação: revisão da literatura sobre dificuldades no ensino de lógica de programação e pensamento computacional, gamificação e aprendizagem baseada em problemas, bem como o estudo de jogos correlatos que empregam código como mecânica;

b. Modelagem da arquitetura: definição de uma arquitetura modular que isola as regras de combate da camada de apresentação, atribuindo a cada módulo uma responsabilidade única;

c. Implementação do núcleo seguro: construção do ambiente isolado (*sandbox*) responsável por compilar e executar o código do jogador, com as defesas necessárias contra laços infinitos, exaustão de memória e acesso indevido a recursos;

d. Definição da linguagem de magias: especificação do subconjunto da linguagem Lua e da interface de programação de domínio (API) exposta ao jogador, formalizada por uma gramática em notação BNF/EBNF (apresentada no Apêndice A);

e. Implementação das mecânicas de jogo: desenvolvimento do modelo de combate, da progressão por círculos de magia e do sistema de imunidades dos adversários, que conferem propósito pedagógico ao avanço;

f. Validação técnica: verificação de que apenas código executável é incorporado ao jogo e de que o estado é persistido de forma confiável.

### Ferramentas e tecnologias

O artefato será desenvolvido na linguagem Lua, executada sobre o framework **LÖVE** (versão 11.5), que embarca o interpretador **LuaJIT** (compatível com a especificação Lua 5.1). 

A plataforma-alvo é o ambiente *desktop*, com possibilidade de empacotamento do jogo como executável independente. A resolução, o título e os demais parâmetros de inicialização são centralizados em um único módulo de configuração, e os recursos visuais (*sprites*) são gerados proceduralmente, dispensando dependências de arquivos externos obrigatórios.
