Este capítulo reúne os desdobramentos previstos para a continuidade do trabalho. O primeiro deles constitui o núcleo da segunda etapa do TCC; os demais são extensões do artefato que a experiência de construção do protótipo indicou como promissoras.

## Avaliação empírica com estudantes

O protótipo demonstra a viabilidade técnica e de design da proposta, mas a pergunta de pesquisa só se responde por completo com dados de uso real. A etapa seguinte do trabalho prevê a aplicação do CodeMage junto a estudantes de disciplinas introdutórias de programação, observando engajamento, desempenho na resolução dos desafios e percepção dos participantes. Interessa, em particular, verificar se o ciclo de reescrita induzido pelas imunidades se traduz em ganho observável de raciocínio algorítmico, e em que medida a restrição da linguagem reduz de fato a carga cognitiva relatada pelos iniciantes.

## Telemetria do processo de aprendizagem

Hoje o jogo registra o resultado das ações no log de combate, mas não guarda histórico do processo: quantas tentativas uma magia exigiu, quais erros de validação apareceram, quanto tempo separa uma reescrita da outra. Instrumentar o protótipo com essa telemetria transformaria cada sessão de jogo em um registro do percurso de aprendizagem do jogador — insumo valioso tanto para a avaliação empírica quanto para um eventual painel do professor, que poderia identificar em qual conceito cada turma está travada.

## Ampliação do conteúdo pedagógico

Os sete chefões atuais exercitam sequência, condicionais, laços e consulta ao estado. Construções importantes ficaram fora do recorte do protótipo, como a definição de funções pelo próprio jogador, recursão e o uso mais rico de tabelas. Uma expansão natural é desenhar novos adversários cujas imunidades tornem essas construções necessárias — mantendo o princípio de que cada conceito entra no jogo como solução de um problema, e não como item de um currículo.

## Melhorias no ambiente de programação

O editor embutido é funcional, porém espartano. Realce de sintaxe, indicação visual da linha do erro e mensagens de validação reescritas em linguagem didática (hoje elas expõem o texto do interpretador) reduziriam o atrito nos primeiros contatos. Outra direção é oferecer dicas progressivas por chefão, ativadas apenas após um número de tentativas frustradas, preservando o espaço de descoberta.

## Portabilidade e distribuição

O protótipo roda em desktop e é empacotado como executável para Windows. Levar o jogo ao navegador facilitaria a adoção em laboratórios escolares, onde a instalação de software costuma ser restrita, e ampliaria o alcance da avaliação empírica prevista para a próxima etapa.
