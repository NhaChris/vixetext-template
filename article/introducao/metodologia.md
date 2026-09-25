## Metodologia

Este trabalho classifica-se, quanto à natureza, como aplicada, pois visa à produção de um artefato de software, denominado CodeMage, destinado à solução de um problema concreto: o apoio ao ensino de lógica de programação.

Quanto aos objetivos, caracteriza-se como exploratória e descritiva, na medida em que investiga a viabilidade de uma abordagem (programação como mecânica de jogo) e descreve detalhadamente as decisões de projeto e implementação adotadas. Quanto à abordagem, é predominantemente qualitativa, centrada na análise das escolhas arquiteturais e de suas implicações pedagógicas e técnicas.

A metodologia base adotada é a da pesquisa de *Design Science* (ou *Design Science Research*), que orienta a construção e a avaliação de artefatos tecnológicos como meio de gerar conhecimento \cite{dresch2015design}. Sob essa ótica, o CodeMage é simultaneamente o objeto construído e o instrumento por meio do qual se examinam as questões de pesquisa. \citeonline{lacerda2013design} descrevem o método em cinco etapas principais: conscientização, sugestão, desenvolvimento, avaliação e conclusão, às quais se acrescenta a comunicação dos resultados.

As atividades deste trabalho foram organizadas de acordo com essas etapas. As atividades de (a) a (f) foram realizadas nesta primeira fase do TCC e resultaram no protótipo descrito no \autoref{cap_prototipo}. A atividade (g) será conduzida na segunda fase.

a. Levantamento e fundamentação (conscientização): revisão da literatura sobre dificuldades no ensino de lógica de programação e pensamento computacional, gamificação e aprendizagem baseada em problemas, bem como o estudo de jogos correlatos que empregam código como mecânica;

b. Modelagem da arquitetura (sugestão): definição de uma arquitetura modular que isola as regras de combate da camada de apresentação, atribuindo a cada módulo uma responsabilidade única;

c. Implementação do núcleo seguro (desenvolvimento): construção do ambiente isolado (*sandbox*) responsável por compilar e executar o código do jogador, com as defesas necessárias contra laços infinitos, exaustão de memória e acesso indevido a recursos;

d. Definição da linguagem de magias (desenvolvimento): especificação do subconjunto da linguagem Lua e da interface de programação de domínio (API) exposta ao jogador, formalizada por uma gramática em notação BNF/EBNF, apresentada na \autoref{sec_gramatica};

e. Implementação das mecânicas de jogo (desenvolvimento): desenvolvimento do modelo de combate, da progressão por círculos de magia e do sistema de imunidades dos adversários, que conferem propósito pedagógico ao avanço;

f. Validação técnica (avaliação): verificação, por meio de uma suíte de testes automatizados, de que as regras de combate se comportam como projetado, de que apenas código executável é incorporado ao jogo e de que o estado é persistido de forma confiável;

g. Avaliação com estudantes (avaliação, prevista para o TCC 2): aplicação do CodeMage junto a estudantes de disciplinas introdutórias de programação, observando engajamento, desempenho na resolução dos desafios e percepção dos participantes.

As etapas de conclusão e comunicação correspondem, respectivamente, à análise dos resultados de cada fase e à redação deste documento.

### Ferramentas e tecnologias

O artefato é desenvolvido na linguagem **Lua**, executada sobre o framework **LÖVE** (versão 11.5), que embarca o interpretador **LuaJIT** (compatível com a especificação Lua 5.1). As características da linguagem e os motivos da escolha do framework são discutidos na fundamentação teórica (\autoref{sec_lua} e \autoref{sec_love}).

A plataforma-alvo é o ambiente *desktop*, com empacotamento do jogo como executável independente. A resolução, o título e os demais parâmetros de inicialização são centralizados em um único módulo de configuração, e os recursos visuais (*sprites*) são gerados proceduralmente, dispensando dependências de arquivos externos obrigatórios.
