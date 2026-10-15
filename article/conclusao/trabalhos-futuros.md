## Próximas etapas {#sec_proximas_etapas}

A segunda fase do TCC dará continuidade ao desenvolvimento da plataforma em três frentes, acompanhadas da ampliação da suíte de testes que verifica cada nova funcionalidade.

### Ampliação do conteúdo pedagógico

Os sete chefões atuais exercitam sequência, condicionais, laços, tabelas e consulta ao estado. Construções importantes ficaram fora do recorte da versão atual, como a definição de funções pelo próprio jogador, a recursão e o uso mais rico de tabelas. A próxima etapa prevê novos adversários cujas imunidades tornem essas construções necessárias, acompanhados das respectivas lições de tutorial, mantendo o princípio de que cada conceito entra no jogo como solução de um problema, e não como item de um currículo.

### Melhorias no ambiente de programação

O editor embutido é funcional, porém espartano. Realce de sintaxe, indicação visual da linha do erro e mensagens de validação reescritas em linguagem didática (hoje elas expõem o texto do interpretador) reduzirão o atrito nos primeiros contatos. Para as mensagens, um analisador sintático próprio, gerado com ANTLR a partir da gramática apresentada na \autoref{sec_gramatica}, permitiria identificar a construção em que o erro ocorreu e explicá-lo nos termos da linguagem de ensino. Os tutoriais atuais ensinam a sintaxe sem revelar a solução de cada chefão, e poderão ser complementados com dicas progressivas sobre o problema, ativadas apenas após um número de tentativas frustradas, de modo a preservar o espaço de descoberta.

### Registro do processo de aprendizagem

Hoje o jogo registra o resultado das ações no log de combate, mas não guarda histórico do processo: quantas tentativas uma magia exigiu, quais erros de validação apareceram, quanto tempo separa uma reescrita da outra. Instrumentar a plataforma com esse registro (telemetria) transformará cada sessão de jogo em um retrato do percurso de aprendizagem do jogador. Esse histórico poderá alimentar um painel do professor, que identificaria em qual conceito cada turma está travada, e servirá de base para avaliações futuras com estudantes.

## Trabalhos futuros

Fora do escopo do TCC, dois desdobramentos ficam indicados para a continuidade da pesquisa.

### Avaliação com estudantes

Este trabalho se propõe a construir a plataforma, e não a medir seus efeitos sobre a aprendizagem. Avaliar esse efeito exige aplicar o CodeMage junto a estudantes de disciplinas introdutórias de programação, observando engajamento, desempenho na resolução dos desafios e percepção dos participantes. Interessa, em particular, verificar se o ciclo de reescrita induzido pelas imunidades se traduz em ganho observável de raciocínio algorítmico e em que medida a restrição da linguagem reduz de fato a carga cognitiva relatada pelos iniciantes. O registro do processo de aprendizagem previsto para o TCC 2 fornecerá parte dos dados necessários.

### Versão para navegador

A plataforma roda em *desktop* e é empacotada como executável para Windows. Levar o jogo ao navegador facilitaria a adoção em laboratórios escolares, onde a instalação de software costuma ser restrita, e ampliaria o alcance de uma futura avaliação com estudantes.
