## Objetivo

Neste tópico são apresentados os objetivos desta pesquisa, divididos em geral e específicos.

### Objetivo Geral

Desenvolver o CodeMage, uma plataforma gamificada na forma de um jogo de combate por turnos, na qual a construção de algoritmos constitui a mecânica central, com o propósito de apoiar o ensino de lógica de programação, utilizando a linguagem Lua como meio de expressão dessa lógica.

### Objetivos Específicos

a. Realizar um mapeamento sistemático da literatura sobre plataformas gamificadas e jogos voltados ao ensino de programação, identificando abordagens, lacunas e tendências;
b. Buscar e analisar ferramentas e jogos correlatos nos quais a escrita de código seja o mecanismo de construção ou de resolução de problemas, posicionando o CodeMage em relação a elas;
c. Projetar uma arquitetura de software que separe claramente as regras do jogo de sua apresentação visual, favorecendo a manutenibilidade e a extensibilidade;
d. Implementar um ambiente de execução isolado (*sandbox*) capaz de compilar e executar com segurança o código arbitrário escrito pelo jogador, prevenindo laços infinitos, exaustão de memória e acesso a recursos do sistema;
e. Definir uma linguagem de domínio restrita, sobre o subconjunto idiomático da linguagem Lua, e formalizá-la por meio de uma gramática livre de contexto (notação BNF/EBNF);
f. Conceber mecânicas de progressão — círculos de magia e imunidades dos adversários — que incentivem o jogador a exercitar o raciocínio algorítmico, reescrevendo, depurando e generalizando suas soluções como forma de avançar.
