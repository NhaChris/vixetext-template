Este capítulo descreve o protótipo funcional do CodeMage, construído para demonstrar a viabilidade da proposta apresentada nos capítulos anteriores. A exposição parte da visão geral do artefato e do seu fluxo de uso, passa pela arquitetura de módulos e pelas duas decisões técnicas centrais — o modelo de execução em duas fases e o ambiente isolado que executa o código do jogador — e termina nas mecânicas que dão propósito pedagógico à progressão. A formalização da linguagem aceita pelo editor de magias é apresentada no Apêndice A.

## Visão geral do protótipo

O protótipo é um jogo *desktop* desenvolvido sobre o framework LÖVE, versão 11.5, que embarca o interpretador LuaJIT, compatível com a especificação Lua 5.1 \cite{love2d}. O jogo é distribuído como executável independente para Windows ou como pacote `.love` multiplataforma, sem exigir que o usuário instale a linguagem ou o framework. Os recursos visuais são gerados proceduralmente em tempo de execução; imagens externas, quando presentes, apenas se sobrepõem à geração procedural, de modo que nenhum arquivo de arte é obrigatório para o jogo funcionar.

Três telas compõem o fluxo de uso. No menu inicial, o jogador cria um novo jogo ou carrega o progresso salvo. O grimório é o ambiente de estudo: concentra o editor de código, a lista de magias salvas, um medidor de linhas e chamadas atualizado em tempo real e a referência da API. O combate, por fim, é onde a magia é posta à prova contra um chefão. O ciclo completo — escrever a magia, validá-la, equipá-la, combater e voltar ao editor para reescrevê-la — materializa o *loop* pedagógico discutido na fundamentação: a derrota diante de uma imunidade devolve o jogador ao código, e é a compreensão do porquê da falha que o faz avançar.

## Arquitetura de módulos

O critério que orientou a organização do código foi a separação entre regras e apresentação: o módulo que decide quanto dano um golpe causa não sabe desenhar na tela, e os módulos que desenham não decidem regra alguma. Cada arquivo do projeto tem uma responsabilidade única, como resume o \autoref{quadro_modulos}.

Quadro quadro_modulos: Módulos do protótipo e suas responsabilidades

| Módulo | Papel | Responsabilidade |
|---|---|---|
| `main.lua` | controle | orquestra telas, entrada de mouse e teclado e o estado global |
| `config.lua` / `conf.lua` | configuração | resolução, título e identidade de save centralizados |
| `combate.lua` | regras | modelo do combate: dano, imunidades, status, montagem da *timeline* |
| `dados/balanco.lua` | dados | tabelas de balanceamento (chefões, círculos, dano base) |
| `status.lua` | regras | tipos de dano e efeitos de status |
| `sandbox.lua` | segurança | compila e executa o código do jogador em ambiente isolado |
| `grimorio.lua` | regras | coleção de magias: valida, salva, equipa |
| `editor.lua` | apresentação | editor de texto multilinha (cursor, rolagem, clique) |
| `efeitos.lua` | apresentação | reprodução da *timeline* e efeitos visuais |
| `sprites.lua` | apresentação | geração procedural de *sprites* e carga opcional de imagens |
| `ui.lua` | apresentação | tema visual: painéis, botões, barras de vida |
| `save.lua` | persistência | gravação e leitura do progresso |

Fonte: Elaborado pelo autor.

Um detalhe dessa organização merece nota: as tabelas de balanceamento vivem em um módulo próprio, separado da lógica de combate. Os números que definem a dificuldade — vida dos chefões, dano base de cada tipo, limites de cada círculo — podem ser ajustados sem tocar no motor do jogo, o que barateia a iteração de design e deixa explícito o que é regra e o que é calibração.

## Execução em duas fases: do plano à *timeline*

A decisão arquitetural mais importante do protótipo nasce de um problema concreto. O código de uma magia pode produzir vários efeitos (dano, cura, aplicação de status) e pode falhar no meio da execução por um erro qualquer. Se cada chamada à API alterasse o estado do combate imediatamente, uma magia que quebrasse na terceira linha deixaria o jogo inconsistente: metade do feitiço aplicada, metade não.

A solução divide a execução em duas fases. Na primeira, o *cast*, o código do jogador roda dentro da sandbox e nenhuma função da API altera a vida de quem quer que seja: cada chamada calcula o efeito final — já considerando imunidades, fraquezas e carga acumulada — e enfileira a ação correspondente em um plano. O valor calculado é devolvido ao código do jogador, que pode usá-lo para ramificar (por exemplo, tentar outro tipo de dano quando o primeiro retorna zero). Na segunda fase, encerrado o *cast* sem erros, o combate monta a *timeline* completa do turno — as ações do jogador, os efeitos periódicos de status e o contra-ataque do chefão — e a camada de apresentação a reproduz ao longo do tempo, aplicando cada ação no instante agendado e disparando os efeitos visuais correspondentes.

Três propriedades decorrem desse desenho. A primeira é a atomicidade: se o código do jogador quebra, o plano é descartado e nada chegou a ser aplicado. A segunda é a reprodutibilidade: as decisões aleatórias do turno, como a chance de um golpe aplicar um status, usam um gerador de números pseudoaleatórios semeado a cada *cast* (RNG), de modo que a *timeline* montada é determinística. A terceira diz respeito à correção das interações: decisões que dependem do estado corrente do combate — o chefão ainda está vivo? o golpe foi esquivado? — são tomadas no momento da aplicação, e não numa previsão feita durante o *cast*. É isso que permite interromper um *multicast* (um laço `for` que dispara vários golpes) no instante em que o chefão morre, sem aplicar os golpes restantes nem sofrer um contra-ataque de um adversário já derrotado.

## O ambiente isolado de execução

Executar código arbitrário escrito pelo usuário dentro do próprio processo do jogo é o principal desafio técnico do trabalho. A abordagem adotada apoia-se no mecanismo de ambientes da linguagem: o texto do jogador é compilado por `load` com uma tabela de ambiente própria, que define todos os nomes globais visíveis. Esse ambiente contém apenas as bibliotecas seguras da linguagem (`math`, `string`, `table`), um conjunto pequeno de funções básicas (`pairs`, `ipairs`, `select`, `tonumber`, `tostring`, `type`, `pcall`, `error`, `assert`) e a tabela `magia`. Do ponto de vista do código do jogador, nada além disso existe: não há `os`, `io`, `require`, `load`, `debug` nem acesso ao estado global real do jogo. O \autoref{quadro_sandbox} relaciona os riscos mapeados e as defesas implementadas.

Quadro quadro_sandbox: Riscos do código arbitrário e defesas implementadas na sandbox

| Risco | Defesa implementada |
|---|---|
| Laço infinito (`while true do end`) | *hook* de instruções da VM, verificado a cada 100 instruções; aborta acima de cerca de um milhão |
| *Hook* inócuo sob compilação JIT | o compilador JIT é desligado durante o trecho do jogador e religado em seguida |
| Alocação gigante em uma instrução | `string.rep` substituído por versão limitada, válida também para a forma de método `("x"):rep(n)` |
| Crescimento descontrolado de memória | o mesmo *hook* compara o consumo corrente com um teto de 64 MB |
| Vazamento de *bytecode* | `string.dump` removido do ambiente |
| Alteração direta da vida | entidades expostas como *proxies* que bloqueiam ler e escrever o campo `vida` |

Fonte: Elaborado pelo autor.

Duas dessas defesas são pouco óbvias e merecem registro. A primeira é a interação entre o *hook* de instruções e o compilador JIT: no LuaJIT, *hooks* de contagem não disparam dentro de código compilado em *trace*, de modo que um laço infinito "quente" rodaria para sempre com o limite instalado e inócuo. A solução foi desligar o JIT somente durante a execução do código do jogador — interpretar um trecho de poucas linhas tem custo irrelevante, e o *hook* passa a contar de forma determinística. A segunda envolve o `string.rep`: limitar a função na tabela `string` do ambiente não basta, porque a forma de método `("x"):rep(n)` é resolvida pelo *metatable* global de strings; durante a execução da magia, esse *metatable* é redirecionado para a versão endurecida e restaurado ao final.

A última linha do quadro tem natureza diferente das demais: não protege o sistema, protege as regras do jogo. O código do jogador recebe as entidades do combate — o chefão e ele próprio — na forma de uma "vista", um *proxy* cuja *metatable* intercepta leituras e escritas e bloqueia o campo `vida` nas duas direções. Sem essa barreira, `magia.alvo().vida = 0` venceria qualquer combate sem exercitar raciocínio algum; a vida só muda pelas funções de efeito da API, que passam pela cadeia de imunidades.

## A linguagem de magias e a API de domínio

O jogador do CodeMage não escreve "um programa Lua qualquer": escreve um feitiço. O que distingue um do outro é a tabela `magia`, o vocabulário de domínio injetado no ambiente da sandbox. O \autoref{quadro_api} apresenta o contrato de cada função — o que ela devolve ao código do jogador e o que ela enfileira no plano do *cast*.

Quadro quadro_api: Contrato das funções da API de domínio

| Chamada | Retorno | Efeito enfileirado |
|---|---|---|
| `magia.alvo()` | vista do chefão | nenhum (leitura) |
| `magia.eu()` | vista do jogador | nenhum (leitura) |
| `magia.dano(alvo, tipo)` | dano efetivo | dano e eventual status no chefão |
| `magia.curar(quem, n)` | valor curado | cura no jogador |
| `magia.absorver(tipo)` | dano efetivo | dano no chefão e cura de metade do valor |
| `magia.carregar()` | nível de carga | amplifica e garante o próximo golpe |
| `magia.dissipar(quem)` | nome removido ou `nil` | remove um efeito negativo da entidade |
| `magia.status(quem)` | lista de nomes | nenhum (leitura) |
| `magia.log(texto)` | — | mensagem no registro de combate |

Fonte: Elaborado pelo autor.

A assinatura de `magia.dano` carrega uma decisão pedagógica: a potência não é um argumento. O jogador escolhe apenas o tipo do dano (fogo, gelo, ácido, entre os nove disponíveis); o valor vem de uma tabela interna de balanceamento, indexada pelo tipo e pelo círculo da magia. Se a potência fosse um parâmetro livre, escrever `magia.dano(alvo, 999999)` seria a estratégia ótima e nenhum raciocínio seria exercitado. Ao retirar o número da mão do jogador, o jogo desloca a otimização para onde ela interessa: a estrutura do código.

Dois exemplos ilustram os extremos do que a linguagem comporta. O feitiço mínimo é uma única sentença:

```lua
magia.dano(magia.alvo(), "fogo")
```

Já uma magia de círculo mais alto pode consultar o próprio estado, iterar sobre ele e reagir:

```lua
for _, s in ipairs(magia.status(magia.eu())) do
  if s == "maldicao" then
    magia.dissipar(magia.eu())
  end
end
magia.dano(magia.alvo(), "sagrado")
```

O segundo exemplo exercita iteração, condicional e consulta ao estado do combate — exatamente as construções que uma disciplina introdutória de lógica pretende ensinar. A gramática do subconjunto aceito, em notação BNF/EBNF, está formalizada no Apêndice A.

## Mecânicas de progressão

### Círculos de magia

A progressão estrutural usa a metáfora dos círculos de magia. O jogador começa no círculo 1 e sobe um círculo a cada chefão derrotado; o círculo vigente impõe dois limites ao código de cada magia: um máximo de linhas lógicas e um máximo de chamadas à API, conforme o \autoref{quadro_circulos}. A medição ignora comentários e o conteúdo de strings, e não conta quebras de linha dentro de parênteses como linhas novas — sem isso, espalhar uma chamada por várias linhas físicas inflaria artificialmente a magia.

Quadro quadro_circulos: Limites de código por círculo de magia

| Círculo | Máximo de linhas lógicas | Máximo de chamadas à API |
|:---:|:---:|:---:|
| 1 | 2 | 3 |
| 2 | 3 | 4 |
| 3 | 4 | 4 |
| 4 | 5 | 6 |
| 5 | 6 | 8 |
| 6 | 8 | 10 |
| 7 | 10 | 12 |

Fonte: Elaborado pelo autor.

O ponto central do desenho está no cálculo da potência: o círculo de uma magia é o menor círculo cujos limites comportam o seu código, e é esse círculo — não o do jogador — que indexa a tabela de dano. Uma magia de uma linha é sempre de círculo baixo e, portanto, fraca; uma que aproveita as dez linhas do círculo 7 atinge a potência máxima. Programar melhor e ficar mais forte tornam-se, deliberadamente, a mesma coisa. Os limites funcionam ainda como andaime no sentido discutido na fundamentação: nos primeiros círculos, o jogador só consegue (e só precisa) escrever duas linhas, e a complexidade permitida cresce junto com o seu domínio.

### Imunidades: cada chefão é um problema

Se os círculos regulam o quanto se pode escrever, as imunidades dos chefões determinam o que vale a pena escrever. Cada chefão intercepta uma classe de estratégia, e as imunidades são acumulativas — o chefão seguinte mantém, na prática, as lições de todos os anteriores, pois a estratégia bloqueada antes continua sendo um mau caminho. O \autoref{quadro_chefoes} resume a progressão.

Quadro quadro_chefoes: Chefões do protótipo e a estratégia que cada um bloqueia

| Chefão | Mecânica | O que força o jogador a fazer |
|---|---|---|
| Boneco de Treino | nenhuma (tutorial) | escrever e validar a primeira magia |
| Golem de Pedra | reflete o excesso de dano acima de um teto por golpe | dosar o dano em vez de maximizá-lo |
| Sanguessuga Etérea | corta a cura pela metade e amaldiçoa quem cura | reavaliar a estratégia defensiva; dissipar status |
| Reflexo Veloz | esquiva escalante por golpe; em 100%, interrompe a sequência | abandonar o *multicast* indiscriminado |
| Daemon Firewall | adapta-se ao tipo de dano recebido e bloqueia o uso seguinte | alternar tipos, condicionar o código ao estado |
| Lich de Cromo | devolve ao jogador todo status que recebe | prever efeitos colaterais; dissipar-se |
| Compilador Supremo | anula um feitiço repetido em sequência (*cache*) | generalizar: nenhuma solução única resolve tudo |

Fonte: Elaborado pelo autor.

A tabela deixa visível a filiação à Aprendizagem Baseada em Problemas: o chefão é o problema, a magia é a solução proposta, e a imunidade é o que torna a solução anterior insuficiente. O Reflexo Veloz, por exemplo, pune o padrão `for i = 1, n do magia.dano(...) end` — cada golpe da sequência tem chance crescente de ser esquivado, e a sequência inteira é interrompida quando a esquiva satura. O Daemon Firewall bloqueia o tipo de dano ao qual acabou de se adaptar, o que empurra o jogador para código condicional que alterna elementos. O chefão final anula qualquer feitiço repetido, cobrando de uma vez o repertório inteiro construído até ali. Nenhuma dessas informações é anunciada ao jogador: o registro de combate relata o que aconteceu ("o golpe foi esquivado", "a cura foi corrompida"), e cabe a ele formular a hipótese, testá-la e refinar o código — o ciclo de depuração como mecânica de jogo.

## Validação e persistência

Uma regra simples sustenta a confiabilidade do conjunto: só entra no grimório magia que compila e executa. Ao salvar, o código é compilado e rodado uma vez contra um combate de teste descartável — um "combate-fantasma" contra o chefão de treino —, e qualquer erro de sintaxe ou de execução é reportado imediatamente, no momento em que o aprendiz ainda tem o contexto do que escreveu. Além do valor pedagógico do retorno imediato, a validação garante uma invariante do sistema: toda magia equipada é executável, e o combate real nunca encontra código quebrado.

O progresso do jogador — círculo, chefões vencidos e o grimório completo, com o código-fonte de cada magia — é persistido como um arquivo de dados em sintaxe Lua, relido em um ambiente vazio: o arquivo carregado é somente dado e não tem acesso a função alguma, de modo que um save adulterado não executa nada. Na serialização, os textos das magias são escapados caractere a caractere, o que preserva quebras de linha e caracteres de controle sem corromper o arquivo. Ao carregar, cada magia é revalidada contra a sandbox e as inválidas são descartadas — a invariante do grimório sobrevive, portanto, até à edição manual do arquivo de save.
