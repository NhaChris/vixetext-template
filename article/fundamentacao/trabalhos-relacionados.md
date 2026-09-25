Neste capítulo, são apresentados trabalhos, jogos e ferramentas nos quais a escrita de código pelo usuário constitui o mecanismo central de interação. O objetivo é posicionar o CodeMage em relação ao estado da arte e evidenciar a lacuna que ele busca preencher: um jogo cuja mecânica é a construção de algoritmos, voltado explicitamente ao exercício da lógica de programação. Os trabalhos foram organizados em três grupos: jogos que ensinam programação por meio de código, jogos de automação por programação e ferramentas criativas baseadas em código. Em seguida, são discutidas as mecânicas que esses jogos têm em comum.

## Jogos que ensinam programação por meio de código

O CodeCombat \cite{codecombat} é uma plataforma na qual o jogador controla um personagem em um RPG de fantasia escrevendo código em Python ou JavaScript para movê-lo, coletar itens e derrotar inimigos. É amplamente adotado em contexto escolar, com uma edição voltada a professores. Aproxima-se do CodeMage por unir narrativa de RPG e escrita de código, mas dele se distingue por utilizar linguagens de propósito geral diretamente e por estruturar a progressão em fases lineares, e não em torno de adversários que forçam a generalização da solução.

O CodeSpells \cite{esper2013codespells}, desenvolvido na Universidade da Califórnia em San Diego, é o correlato acadêmico mais próximo desta proposta. Trata-se de um RPG tridimensional em primeira pessoa no qual o jogador é um aprendiz de mago e escreve feitiços em Java: cada feitiço é uma classe derivada de `Spell` que implementa o método `cast()`, editada em um ambiente de programação embutido no jogo. Os autores exploram a metáfora da magia como programação e projetam a API para que o jogador "sinta" o efeito do próprio código sobre o mundo, com chamadas como `getTarget()` para obter a entidade que o avatar está apontando. A semelhança com o CodeMage é grande, tanto na metáfora quanto na forma da API. As diferenças estão no domínio (exploração e missões em um mundo aberto, e não combate por turnos), na linguagem (Java completo, e não um subconjunto restrito) e na ausência de mecanismos desenhados para invalidar soluções anteriores.

Os jogos da Tomorrow Corporation e da Zachtronics representam a vertente dos *puzzles* de programação. No Human Resource Machine \cite{humanresourcemachine}, o jogador resolve problemas combinando instruções de baixo nível em uma linguagem visual, exercitando fluxo de controle e manipulação de memória sem exigir conhecimento prévio de sintaxe. O TIS-100 \cite{tis100}, por sua vez, simula uma linguagem de montagem (*assembly*) executada em uma grade de nós de processamento e propõe desafios de programação de baixo nível. Ambos compartilham com o CodeMage a ideia de que cada desafio é um problema a ser resolvido por meio de código, porém operam em níveis de abstração (visual ou *assembly*) distintos do subconjunto de uma linguagem de alto nível adotado neste trabalho.

O Screeps \cite{screeps} é um jogo de estratégia online massivo (MMO) no qual o jogador programa, em JavaScript, o comportamento autônomo de suas unidades, que continuam operando mesmo com o jogador desconectado. Diferencia-se por seu caráter persistente e competitivo entre jogadores reais. O foco recai sobre a arquitetura de sistemas autônomos, e não sobre o aprendizado guiado de lógica por um iniciante.

## Jogos de automação por programação

O The Farmer Was Replaced \cite{farmerwasreplaced} é um jogo de automação no qual o jogador programa um drone, em uma linguagem semelhante a Python, para executar tarefas agrícolas repetitivas de forma cada vez mais eficiente. À medida que automatiza a fazenda, o jogador progride por uma árvore tecnológica, e seu código é armazenado em arquivos editáveis externamente. É, entre os jogos comerciais, o correlato mais próximo da proposta do CodeMage no que tange a fazer da escrita de código a fonte de progressão. A principal diferença está no domínio (automação contínua de uma fazenda *versus* combate por turnos) e no fato de o CodeMage adotar mecanismos, as imunidades dos adversários, desenhados especificamente para induzir a reescrita e a generalização do raciocínio.

## Ferramentas criativas baseadas em código

Fora do domínio dos jogos, o *live coding* musical ilustra como a escrita de código pode ser uma atividade expressiva e de retorno imediato. O Sonic Pi \cite{sonicpi}, criado por Sam Aaron e baseado na linguagem Ruby, permite criar e executar música ao vivo por meio de código, sendo amplamente empregado em contextos educacionais por sua baixa barreira de entrada. O TidalCycles \cite{tidalcycles}, escrito em Haskell, oferece uma linguagem de padrões para a composição algorítmica de ritmos e melodias. Embora não tenham finalidade lúdica nem objetivo direto de ensinar lógica de programação, essas ferramentas reforçam um princípio caro a este trabalho: o de que escrever código pode ser, em si, uma experiência envolvente e de causa e efeito imediatos.

## Mecânicas comuns em jogos de programação

A análise dos trabalhos acima revela um conjunto de mecânicas recorrentes, que ajudam a situar as escolhas de projeto do CodeMage.

A primeira é o **editor embutido com retorno imediato**. CodeCombat, CodeSpells, Human Resource Machine e TIS-100 oferecem o ambiente de programação dentro do próprio jogo, e o efeito do código é observado logo após sua execução, sem que o jogador precise sair da ficção para compilar ou depurar. O CodeMage segue o mesmo caminho com o grimório, e o registro de combate cumpre o papel de mensagem de depuração.

A segunda é a **restrição de recursos**. No Screeps, a execução do código de cada jogador é limitada por um orçamento de tempo de CPU por ciclo do jogo, e o excedente não utilizado é acumulado em uma reserva que pode ser gasta depois \cite{screepscpu}. No TIS-100, a computação é distribuída entre doze nós de capacidade reduzida. Em ambos os casos, a restrição transforma a eficiência do código em parte do desafio. No CodeMage, a restrição equivalente são os limites de linhas e de chamadas impostos por cada círculo de magia.

A terceira é o **desbloqueio progressivo da linguagem**. No The Farmer Was Replaced, construções como laços, variáveis, operadores e funções não estão disponíveis desde o início e precisam ser desbloqueadas na árvore tecnológica \cite{farmerwasreplaced}. O conceito entra no jogo quando o jogador já tem motivo para usá-lo. O CodeMage adota uma variação dessa ideia: todas as construções estão disponíveis, mas o espaço para usá-las cresce com os círculos, e cada nova construção é apresentada por um tutorial no momento em que passa a ser necessária.

A quarta são as **métricas de otimização**. O Human Resource Machine propõe, para cada fase, um desafio de tamanho, medido pelo número de instruções do programa, e um desafio de velocidade, medido pelo número de passos executados \cite{humanresourcemachine}. O TIS-100 mede ciclos, instruções e nós utilizados e compara o resultado com o de outros jogadores \cite{tis100}. Essas métricas incentivam o jogador a revisitar uma solução que já funciona. O CodeMage persegue o mesmo efeito por outra via: em vez de pontuar a solução, cria adversários contra os quais a solução anterior deixa de funcionar.

## Síntese e posicionamento

O \autoref{quadro_relacionados} sintetiza a comparação entre os trabalhos analisados e o CodeMage.

Quadro quadro_relacionados: Comparação entre o CodeMage e os trabalhos relacionados

| Trabalho | Linguagem | Domínio | Foco principal |
|---|---|---|---|
| CodeCombat | Python / JavaScript | RPG de fantasia | Ensino de programação em fases |
| CodeSpells | Java | RPG 3D em primeira pessoa | Ensino de programação por imersão |
| Human Resource Machine | Visual (instruções) | Puzzle de escritório | Lógica e fluxo de controle |
| TIS-100 | Assembly (simulado) | Puzzle de baixo nível | Programação de montagem |
| Screeps | JavaScript | Estratégia (MMO) | Sistemas autônomos e competição |
| The Farmer Was Replaced | Semelhante a Python | Automação de fazenda | Automação e eficiência |
| Sonic Pi / TidalCycles | Ruby / Haskell | Criação musical | Expressão artística (*live coding*) |
| **CodeMage** | **Subconjunto de Lua** | **RPG de combate por turnos** | **Lógica de programação** |

Fonte: Elaborado pelo autor.

Da análise depreende-se que, embora existam jogos que ensinam programação e jogos que usam código como mecânica de automação, há espaço para uma proposta que combine três características de forma integrada: (i) ter a construção de algoritmos como mecânica central; (ii) empregar um subconjunto restrito e formalizado de uma linguagem de alto nível, reduzindo a carga cognitiva do iniciante; e (iii) estruturar a progressão em torno de adversários que, por suas imunidades, exigem a reformulação e a generalização da solução. O CodeSpells é o trabalho que mais se aproxima dessa combinação, pois compartilha a primeira característica e a metáfora da magia, mas adota uma linguagem de propósito geral completa e não possui mecanismos equivalentes ao terceiro. É nessa interseção que o CodeMage se posiciona.
