Nesta seção, são apresentados trabalhos, jogos e ferramentas nos quais a escrita de código pelo usuário constitui o mecanismo central de interação. O objetivo é posicionar o CodeMage em relação ao estado da arte e evidenciar a lacuna que ele busca preencher: um jogo cuja mecânica é a construção de algoritmos voltado explicitamente ao exercício da lógica de programação. Os trabalhos foram organizados em três grupos: jogos que ensinam programação por meio de código, jogos de automação por programação e ferramentas criativas baseadas em código.

## Jogos que ensinam programação por meio de código

O CodeCombat \cite{codecombat} é uma plataforma na qual o jogador controla um personagem em um RPG de fantasia escrevendo código em Python ou JavaScript para movê-lo, coletar itens e derrotar inimigos. É amplamente adotado em contexto escolar, com uma edição voltada a professores. Aproxima-se do CodeMage por unir narrativa de RPG e escrita de código, mas dele se distingue por utilizar linguagens de propósito geral diretamente e por estruturar a progressão em fases lineares, e não em torno de adversários que forçam a generalização da solução.

O Screeps \cite{screeps} é um jogo de estratégia online massivo (MMO) no qual o jogador programa, em JavaScript, o comportamento autônomo de suas unidades, que continuam operando mesmo com o jogador desconectado. Diferencia-se por seu caráter persistente e competitivo entre jogadores reais; o foco recai sobre a arquitetura de sistemas autônomos, e não sobre o aprendizado guiado de lógica por um iniciante.

Os jogos da Tomorrow Corporation e da Zachtronics representam a vertente dos *puzzles* de programação. No Human Resource Machine \cite{humanresourcemachine}, o jogador resolve problemas combinando instruções de baixo nível em uma linguagem visual, exercitando fluxo de controle e manipulação de memória sem exigir conhecimento prévio de sintaxe. O TIS-100 \cite{tis100}, por sua vez, simula uma linguagem de montagem (*assembly*) e propõe desafios de programação de baixo nível. Ambos compartilham com o CodeMage a ideia de que cada desafio é um problema a ser resolvido por meio de código, porém operam em níveis de abstração (visual ou *assembly*) distintos do subconjunto de uma linguagem de alto nível adotado neste trabalho.

## Jogos de automação por programação

O The Farmer Was Replaced \cite{farmerwasreplaced} é um jogo de automação no qual o jogador programa um drone, em uma linguagem semelhante a Python, para executar tarefas agrícolas repetitivas de forma cada vez mais eficiente. À medida que automatiza a fazenda, o jogador progride por uma árvore tecnológica, e seu código é armazenado em arquivos editáveis externamente. É talvez o correlato mais próximo da proposta do CodeMage no que tange a fazer da escrita de código a fonte de progressão; a principal diferença está no domínio (automação contínua de uma fazenda *versus* combate por turnos) e no fato de o CodeMage adotar mecanismos — as imunidades dos adversários — desenhados especificamente para induzir a reescrita e a generalização do raciocínio.

## Ferramentas criativas baseadas em código

Fora do domínio dos jogos, o *live coding* musical ilustra como a escrita de código pode ser uma atividade expressiva e de retorno imediato. O Sonic Pi \cite{sonicpi}, criado por Sam Aaron e baseado na linguagem Ruby, permite criar e executar música ao vivo por meio de código, sendo amplamente empregado em contextos educacionais por sua baixa barreira de entrada. O TidalCycles \cite{tidalcycles}, escrito em Haskell, oferece uma linguagem de padrões para a composição algorítmica de ritmos e melodias. Embora não tenham finalidade lúdica nem objetivo direto de ensinar lógica de programação, essas ferramentas reforçam um princípio caro a este trabalho: o de que escrever código pode ser, em si, uma experiência envolvente e de causa e efeito imediatos.

## Síntese e posicionamento

O \autoref{quadro_relacionados} sintetiza a comparação entre os trabalhos analisados e o CodeMage.

Quadro quadro_relacionados: Comparação entre o CodeMage e os trabalhos relacionados

| Trabalho | Linguagem | Domínio | Foco principal |
|---|---|---|---|
| CodeCombat | Python / JavaScript | RPG de fantasia | Ensino de programação em fases |
| Screeps | JavaScript | Estratégia (MMO) | Sistemas autônomos e competição |
| Human Resource Machine | Visual (instruções) | Puzzle de escritório | Lógica e fluxo de controle |
| TIS-100 | Assembly (simulado) | Puzzle de baixo nível | Programação de montagem |
| The Farmer Was Replaced | Semelhante a Python | Automação de fazenda | Automação e eficiência |
| Sonic Pi / TidalCycles | Ruby / Haskell | Criação musical | Expressão artística (*live coding*) |
| **CodeMage** | **Subconjunto de Lua** | **RPG de combate por turnos** | **Lógica de programação** |

Fonte: Elaborado pelo autor.

Da análise depreende-se que, embora existam jogos que ensinam programação e jogos que usam código como mecânica de automação, há espaço para uma proposta que combine três características de forma integrada: (i) ter a construção de algoritmos como mecânica central; (ii) empregar um subconjunto restrito e formalizado de uma linguagem de alto nível, reduzindo a carga cognitiva do iniciante; e (iii) estruturar a progressão em torno de adversários que, por suas imunidades, exigem a reformulação e a generalização da solução. É nessa interseção que o CodeMage se posiciona.
