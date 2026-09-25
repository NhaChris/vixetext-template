Este capítulo descreve o protótipo funcional do CodeMage, construído nesta primeira etapa do trabalho para demonstrar a viabilidade da proposta. A exposição parte da visão geral do artefato e do seu fluxo de uso e passa pela arquitetura de módulos e pelas duas decisões técnicas centrais, que são o modelo de execução em duas fases e o ambiente isolado que executa o código do jogador. Em seguida, apresenta a linguagem de magias, com sua API de domínio e a gramática formal do subconjunto aceito, e termina nas mecânicas que dão propósito pedagógico à progressão e nos mecanismos de validação do artefato.

## Visão geral do protótipo

O protótipo é um jogo *desktop* desenvolvido sobre o framework LÖVE, versão 11.5, que embarca o interpretador LuaJIT, compatível com a especificação Lua 5.1 \cite{love2d}. O jogo é distribuído como executável independente para Windows ou como pacote `.love` multiplataforma, sem exigir que o usuário instale a linguagem ou o framework. Os recursos visuais são gerados proceduralmente em tempo de execução. Imagens externas, quando presentes, apenas se sobrepõem à geração procedural, de modo que nenhum arquivo de arte é obrigatório para o jogo funcionar.

Três telas compõem o fluxo de uso. No menu inicial, o jogador cria um novo jogo ou carrega o progresso salvo. O grimório é o ambiente de estudo: concentra o editor de código, a lista de magias salvas, um medidor de linhas e chamadas atualizado em tempo real e a referência da API. O combate, por fim, é onde a magia é posta à prova contra um chefão. O ciclo completo (escrever a magia, validá-la, equipá-la, combater e voltar ao editor para reescrevê-la) materializa o *loop* pedagógico discutido na fundamentação: a derrota diante de uma imunidade devolve o jogador ao código, e é a compreensão do porquê da falha que o faz avançar.

<!-- TODO(autor): inserir aqui os prints do grimório e do combate quando os arquivos existirem, por exemplo:
![Tela do grimório](imagens/grimorio.png){#fig_grimorio largura=90%}

Fonte: Captura de tela do protótipo CodeMage.
-->

## Arquitetura de módulos

O critério que orientou a organização do código foi a separação entre regras e apresentação: o módulo que decide quanto dano um golpe causa não sabe desenhar na tela, e os módulos que desenham não decidem regra alguma. Cada arquivo do projeto tem uma responsabilidade única, como resume o \autoref{quadro_modulos}.

Quadro quadro_modulos: Módulos do protótipo e suas responsabilidades

| Módulo | Papel | Responsabilidade |
|---|---|---|
| `main.lua` | controle | orquestra telas, entrada de mouse e teclado e o estado global |
| `config.lua` / `conf.lua` | configuração | resolução, título e identidade de save centralizados |
| `combate.lua` | regras | modelo do combate: dano, imunidades, status, montagem da *timeline* |
| `dados/balanco.lua` | dados | tabelas de balanceamento (chefões, círculos, dano base, limites) |
| `dados/tutoriais.lua` | dados | textos e exemplos dos tutoriais de linguagem |
| `status.lua` | regras | tipos de dano e efeitos de status |
| `sandbox.lua` | segurança | compila e executa o código do jogador em ambiente isolado |
| `grimorio.lua` | regras | coleção de magias: valida, salva, equipa |
| `editor.lua` | apresentação | editor de texto multilinha (cursor, rolagem, clique) |
| `efeitos.lua` | apresentação | reprodução da *timeline* e efeitos visuais |
| `sprites.lua` | apresentação | geração procedural de *sprites* e carga opcional de imagens |
| `ui.lua` | apresentação | tema visual: painéis, botões, barras de vida |
| `save.lua` | persistência | gravação e leitura do progresso |

Fonte: Elaborado pelo autor.

Um detalhe dessa organização merece nota: os dados vivem em módulos próprios, separados da lógica. Os números que definem a dificuldade (vida dos chefões, dano base de cada tipo, limites de cada círculo) podem ser ajustados sem tocar no motor do jogo, o que barateia a iteração de design e deixa explícito o que é regra e o que é calibração. O mesmo vale para os textos dos tutoriais, que podem ser revistos sem alterar o código que os exibe.

## Execução em duas fases: do plano à *timeline*

A decisão arquitetural mais importante do protótipo nasce de um problema concreto. O código de uma magia pode produzir vários efeitos (dano, cura, aplicação de status) e pode falhar no meio da execução por um erro qualquer. Se cada chamada à API alterasse o estado do combate imediatamente, uma magia que quebrasse na terceira linha deixaria o jogo inconsistente, com metade do feitiço aplicada e a outra metade não.

A solução divide a execução em duas fases, representadas na \autoref{fig_duas_fases}. Na primeira, o *cast*, o código do jogador roda dentro da sandbox e nenhuma função da API altera a vida de quem quer que seja. Cada chamada calcula o efeito final, já considerando imunidades, fraquezas e carga acumulada, e enfileira a ação correspondente em um plano. O valor calculado é devolvido ao código do jogador, que pode usá-lo para ramificar, por exemplo, tentando outro tipo de dano quando o primeiro retorna zero. Na segunda fase, encerrado o *cast* sem erros, o combate monta a *timeline* completa do turno, com as ações do jogador, os efeitos periódicos de status e o contra-ataque do chefão, e a camada de apresentação a reproduz ao longo do tempo, aplicando cada ação no instante agendado e disparando os efeitos visuais correspondentes.

![Execução de um feitiço em duas fases](imagens/execucao-duas-fases.pdf){#fig_duas_fases largura=100%}

Fonte: Elaborado pelo autor.

Três propriedades decorrem desse desenho. A primeira é a atomicidade: se o código do jogador quebra, o plano é descartado e nada chegou a ser aplicado. A segunda é a reprodutibilidade. As decisões aleatórias do turno, como a chance de um golpe aplicar um status, usam um gerador de números pseudoaleatórios (RNG) semeado a cada *cast*, de modo que a *timeline* montada é determinística. A terceira diz respeito à correção das interações. Decisões que dependem do estado corrente do combate, como saber se o chefão ainda está vivo ou se o golpe foi esquivado, são tomadas no momento da aplicação, e não numa previsão feita durante o *cast*. É isso que permite interromper um *multicast* (um laço `for` que dispara vários golpes) no instante em que o chefão morre, sem aplicar os golpes restantes nem sofrer um contra-ataque de um adversário já derrotado.

## O ambiente isolado de execução

Executar código arbitrário escrito pelo usuário dentro do próprio processo do jogo é o principal desafio técnico do trabalho. A abordagem adotada apoia-se no mecanismo de ambientes da linguagem, discutido na \autoref{sec_lua}. O texto do jogador é compilado pela função `load` em modo texto, que recusa *bytecode* pré-compilado, e com uma tabela de ambiente própria, que define todos os nomes globais visíveis. Esse ambiente contém apenas as bibliotecas seguras da linguagem (`math`, `string`, `table`), um conjunto pequeno de funções básicas (`pairs`, `ipairs`, `select`, `tonumber`, `tostring`, `type`, `pcall`, `error`, `assert`) e a tabela `magia`. Do ponto de vista do código do jogador, nada além disso existe: não há `os`, `io`, `require`, `load`, `debug` nem acesso ao estado global real do jogo. O \autoref{quadro_sandbox} relaciona os riscos mapeados e as defesas implementadas.

Quadro quadro_sandbox: Riscos do código arbitrário e defesas implementadas na sandbox

| Risco | Defesa implementada |
|---|---|
| Laço infinito (`while true do end`) | *hook* de instruções da VM, verificado a cada 100 instruções; aborta acima de cerca de um milhão |
| *Hook* inócuo sob compilação JIT | o compilador JIT é desligado durante o trecho do jogador e religado em seguida |
| Alocação gigante em uma instrução | `string.rep` substituído por versão limitada, válida também para a forma de método `("x"):rep(n)` |
| Crescimento descontrolado de memória | o mesmo *hook* compara o crescimento do consumo desde o início da magia com um teto de 64 MB |
| Carga de *bytecode* forjado | `load` em modo texto; `string.dump` removido do ambiente |
| Alteração direta da vida | entidades expostas como *proxies* que bloqueiam ler e escrever o campo `vida` |

Fonte: Elaborado pelo autor.

Duas dessas defesas são pouco óbvias e merecem registro. A primeira é a interação entre o *hook* de instruções e o compilador JIT. No LuaJIT, *hooks* de contagem não disparam dentro de código compilado em *trace*, de modo que um laço infinito "quente" rodaria para sempre com o limite instalado e inócuo. A solução foi desligar o JIT somente durante a execução do código do jogador. Interpretar um trecho de poucas linhas tem custo irrelevante, e o *hook* passa a contar de forma determinística. A segunda envolve o `string.rep`. Limitar a função na tabela `string` do ambiente não basta, porque a forma de método `("x"):rep(n)` é resolvida pela *metatable* global de strings. Durante a execução da magia, essa *metatable* é redirecionada para a versão endurecida e restaurada ao final.

A última linha do quadro tem natureza diferente das demais: não protege o sistema, protege as regras do jogo. O código do jogador recebe as entidades do combate (o chefão e ele próprio) na forma de uma "vista", um *proxy* cuja *metatable* intercepta leituras e escritas e bloqueia o campo `vida` nas duas direções. Sem essa barreira, `magia.alvo().vida = 0` venceria qualquer combate sem exercitar raciocínio algum. A vida só muda pelas funções de efeito da API, que passam pela cadeia de imunidades.

## A linguagem de magias {#sec_linguagem}

O jogador do CodeMage não escreve "um programa Lua qualquer", escreve um feitiço. O que distingue um do outro é a tabela `magia`, o vocabulário de domínio injetado no ambiente da sandbox. Esta seção apresenta primeiro esse vocabulário, que define as entidades e as ações disponíveis, e depois a gramática que formaliza o subconjunto da linguagem em que ele é usado.

### A API de domínio

O \autoref{quadro_api} apresenta o contrato de cada função: o que ela devolve ao código do jogador e o que ela enfileira no plano do *cast*.

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
| `magia.log(texto)` | nenhum | mensagem no registro de combate |

Fonte: Elaborado pelo autor.

As duas primeiras funções devolvem as entidades do combate, isto é, as "vistas" descritas na seção anterior. É sobre elas que as demais funções operam. A assinatura de `magia.dano` carrega uma decisão pedagógica: a potência não é um argumento. O jogador escolhe apenas o tipo do dano (fogo, gelo, ácido, entre os nove disponíveis), e o valor vem de uma tabela interna de balanceamento, indexada pelo tipo e pelo círculo da magia. Se a potência fosse um parâmetro livre, escrever `magia.dano(alvo, 999999)` seria a estratégia ótima e nenhum raciocínio seria exercitado. Ao retirar o número da mão do jogador, o jogo desloca a otimização para onde ela interessa, que é a estrutura do código.

Dois exemplos ilustram os extremos do que a linguagem comporta. O feitiço mínimo é uma única sentença:

```
magia.dano(magia.alvo(), "fogo")
```

Já uma magia de círculo mais alto pode consultar o próprio estado, iterar sobre ele e reagir:

```
for _, s in ipairs(magia.status(magia.eu())) do
  if s == "maldicao" then
    magia.dissipar(magia.eu())
  end
end
magia.dano(magia.alvo(), "sagrado")
```

O segundo exemplo exercita iteração, condicional e consulta ao estado do combate, exatamente as construções que uma disciplina introdutória de lógica pretende ensinar.

### Gramática do subconjunto {#sec_gramatica}

O CodeMage não define uma linguagem nova. Ele reaproveita o interpretador Lua 5.1 embarcado no LÖVE e o restringe a um ambiente fechado, de modo que a linguagem efetivamente aceita tem duas camadas, uma sintática e outra semântica.

A camada sintática define quais construções o texto do jogador pode conter. Como o texto é compilado pelo próprio LuaJIT, qualquer programa Lua sintaticamente válido é aceito, mas só um subconjunto é útil, já que o ambiente oferece poucos nomes. A camada semântica define justamente quais nomes existem nesse ambiente. Tudo o que não estiver no ambiente montado pela sandbox é resolvido como `nil`, e usar um nome inexistente como função provoca um erro de execução. Esse erro é detectado quando o jogador tenta salvar a magia no grimório, que a executa uma vez em um combate de teste e a rejeita se algo falhar, como descrito na \autoref{sec_validacao}.

A gramática a seguir formaliza a linguagem pretendida, isto é, o subconjunto idiomático que o jogo ensina, e não toda a sintaxe de Lua. Ela é escrita em EBNF, com as convenções do \autoref{quadro_notacao}, e cumpre o papel de especificação discutido na \autoref{sec_especificacao}.

Quadro quadro_notacao: Convenções da notação EBNF adotada

| Símbolo | Significado |
|:---:|---|
| `::=` | definido como |
| &#124; | alternativa |
| `[ x ]` | opcional (zero ou uma ocorrência) |
| `{ x }` | repetição (zero ou mais ocorrências) |
| `( x )` | agrupamento |
| `"x"` | terminal literal |
| `<x>` | não terminal |

Fonte: Elaborado pelo autor.

**Estrutura de um feitiço.** Um feitiço é todo o texto escrito pelo jogador, e esse texto forma um único bloco. Não há declaração de função na linguagem pretendida: o corpo da magia é o próprio trecho de topo (*chunk*), compilado de uma só vez pela sandbox. Como em Lua, a instrução `return`, quando presente, só pode ser a última do bloco.

```
<magia>        ::= <bloco>

<bloco>        ::= { <instrucao> } [ <return> ]

<instrucao>    ::= <atribuicao>
                 | <decl-local>
                 | <chamada-stmt>
                 | <if>
                 | <for-num>
                 | <for-in>
                 | <while>
                 | <comentario>

<return>       ::= "return" [ <lista-expr> ]
```

**Declarações e atribuições.** Uma declaração local cria uma ou mais variáveis visíveis apenas dentro do bloco em que aparecem, com valores iniciais opcionais. É a forma de guardar um resultado para usá-lo depois, como o dano devolvido por `magia.dano`. Uma atribuição altera o valor de variáveis já existentes ou de campos de uma tabela. Sem a palavra `local`, a atribuição a um nome novo cria uma variável global, mas essa global pertence ao ambiente isolado da magia e desaparece quando a execução termina. Os campos das entidades devolvidas por `magia.alvo()` e `magia.eu()` também podem ser atribuídos, com uma exceção: o campo `vida` é bloqueado tanto para escrita quanto para leitura, e as expressões `entidade.vida = x` e a simples consulta a `entidade.vida` lançam erro. A vida só é alterada pelas funções de efeito da API (`magia.dano`, `magia.curar` e `magia.absorver`).

```
<decl-local>   ::= "local" <lista-nomes> [ "=" <lista-expr> ]

<atribuicao>   ::= <lista-var> "=" <lista-expr>

<lista-nomes>  ::= <Nome> { "," <Nome> }
<lista-var>    ::= <var> { "," <var> }
<lista-expr>   ::= <expr> { "," <expr> }

<var>          ::= <Nome>
                 | <prefixo> "[" <expr> "]"
                 | <prefixo> "." <Nome>
```

**Estruturas de controle.** O subconjunto oferece a condicional e três formas de laço.

```
<if>           ::= "if" <expr> "then" <bloco>
                   { "elseif" <expr> "then" <bloco> }
                   [ "else" <bloco> ]
                   "end"

<for-num>      ::= "for" <Nome> "=" <expr> "," <expr> [ "," <expr> ] "do"
                       <bloco>
                   "end"

<for-in>       ::= "for" <lista-nomes> "in" <lista-expr> "do"
                       <bloco>
                   "end"

<while>        ::= "while" <expr> "do" <bloco> "end"
```

A forma idiomática do `<for-in>` ensinada pelo jogo é a iteração sobre a lista de status devolvida pela API, como em `for _, s in ipairs(magia.status(magia.alvo())) do ... end`. Os laços são permitidos sem restrição sintática, e a proteção contra laços infinitos é dinâmica, feita pelo *hook* de instruções. As construções `repeat ... until`, `do ... end` e `break` foram deixadas fora da linguagem pretendida. Elas continuam válidas, porque o compilador as aceita, mas nenhum dos problemas propostos pelos chefões depende delas e nenhum tutorial as utiliza, o que mantém pequeno o vocabulário que o iniciante precisa conhecer.

**Chamadas e expressões.** Toda ação do feitiço é uma chamada de função, e toda chamada é também uma expressão, cujo valor pode ser guardado ou comparado.

```
<chamada-stmt> ::= <chamada>

<chamada>      ::= <prefixo> <args>
                 | <prefixo> ":" <Nome> <args>

<prefixo>      ::= <Nome>
                 | "(" <expr> ")"
                 | <prefixo> "." <Nome>
                 | <prefixo> "[" <expr> "]"
                 | <chamada>

<args>         ::= "(" [ <lista-expr> ] ")"
                 | <string>

<expr>         ::= "nil" | "true" | "false"
                 | <Numero>
                 | <string>
                 | <tabela>
                 | <prefixo>
                 | <expr> <op-bin> <expr>
                 | <op-un> <expr>

<op-bin>       ::= "+" | "-" | "*" | "/" | "%" | "^" | ".."
                 | "==" | "~=" | "<" | ">" | "<=" | ">="
                 | "and" | "or"
<op-un>        ::= "-" | "not" | "#"

<tabela>       ::= "{" [ <campo> { ("," | ";") <campo> } [ "," | ";" ] ] "}"
<campo>        ::= "[" <expr> "]" "=" <expr>
                 | <Nome> "=" <expr>
                 | <expr>
```

A produção `<expr> <op-bin> <expr>` é ambígua quando lida isoladamente, pois não diz qual operador se aplica primeiro. A ambiguidade é resolvida pelas regras de precedência da linguagem \cite{lua51manual}, reproduzidas no \autoref{quadro_precedencia}.

Quadro quadro_precedencia: Precedência dos operadores, da menor para a maior

| Nível | Operadores | Associatividade |
|:---:|---|:---:|
| 1 | `or` | esquerda |
| 2 | `and` | esquerda |
| 3 | `<` `>` `<=` `>=` `~=` `==` | esquerda |
| 4 | `..` | direita |
| 5 | `+` `-` | esquerda |
| 6 | `*` `/` `%` | esquerda |
| 7 | `not` `#` `-` (unário) | não se aplica |
| 8 | `^` | direita |

Fonte: Adaptado de \citeonline{lua51manual}.

**Terminais léxicos.** Os nomes, números, strings e comentários seguem as regras de Lua. As sequências de escape dentro de strings (como `\n` e `\"`) são aceitas, mas não são detalhadas aqui.

```
<Nome>            ::= ( <letra> | "_" ) { <letra> | <digito> | "_" }
<Numero>          ::= <digito> { <digito> } [ "." { <digito> } ]
                    | "0x" <hexdigito> { <hexdigito> }
<string>          ::= '"' { <char-sem-aspas> } '"'
                    | "'" { <char-sem-aspas> } "'"
<comentario>      ::= "--" { <char-sem-quebra> }
<letra>           ::= "a" | ... | "z" | "A" | ... | "Z"
<digito>          ::= "0" | ... | "9"
<hexdigito>       ::= <digito> | "a" | ... | "f" | "A" | ... | "F"
<char-sem-aspas>  ::= qualquer caractere, exceto a aspa delimitadora
                      e a quebra de linha
<char-sem-quebra> ::= qualquer caractere, exceto a quebra de linha
```

**Chamadas da API.** A tabela global `magia` é o vocabulário de domínio da linguagem. Cada `<chamada-api>` é um caso particular de `<chamada>`, e a produção a seguir apenas detalha os argumentos que cada função espera. O contrato semântico de cada uma foi apresentado no \autoref{quadro_api}.

```
<chamada-api>   ::= <alvo-expr> | <eu-expr>
                  | <dano> | <curar> | <absorver>
                  | <carregar> | <dissipar>
                  | <status> | <log>

<alvo-expr>     ::= "magia" "." "alvo"     "(" ")"
<eu-expr>       ::= "magia" "." "eu"       "(" ")"
<dano>          ::= "magia" "." "dano"     "(" <expr-entidade> "," <tipo-dano> ")"
<curar>         ::= "magia" "." "curar"    "(" <expr-entidade> "," <expr> ")"
<absorver>      ::= "magia" "." "absorver" "(" <tipo-dano> ")"
<carregar>      ::= "magia" "." "carregar" "(" ")"
<dissipar>      ::= "magia" "." "dissipar" "(" <expr-entidade> ")"
<status>        ::= "magia" "." "status"   "(" <expr-entidade> ")"
<log>           ::= "magia" "." "log"      "(" <expr> ")"

<expr-entidade> ::= <alvo-expr> | <eu-expr> | <Nome>

<tipo-dano>     ::= '"neutro"'  | '"fogo"'   | '"eletrico"' | '"acido"'
                  | '"sagrado"' | '"veneno"' | '"gelo"'     | '"glitch"'
                  | '"trevas"'
```

Duas observações semânticas completam a especificação. A primeira é que `<tipo-dano>` lista os nove tipos válidos, mas uma string fora da lista não é erro: em tempo de execução ela é normalizada para `"neutro"`. A segunda é que a potência do dano não aparece na gramática porque não é argumento, pelos motivos já discutidos.

**Ambiente da sandbox.** O conjunto de nomes globais visíveis a um feitiço é fechado e pode ser descrito como um alfabeto:

```
<global>       ::= "math"  | "string" | "table"
                 | "pairs" | "ipairs" | "select"
                 | "tonumber" | "tostring" | "type"
                 | "pcall" | "error" | "assert"
                 | "magia"
```

Qualquer `<Nome>` usado como global fora desse conjunto é resolvido como `nil`. Não existem `os`, `io`, `love`, `require`, `load`, `dofile`, `package`, `debug`, `_G`, `getfenv`, `setfenv` nem `collectgarbage`; além disso, `string.dump` é removido e `string.rep` é limitado. Essa cláusula é a fronteira pedagógica da linguagem: o ambiente reduzido define exatamente o que o jogador pode usar, e a gramática somada ao ambiente constitui a linguagem de ensino.

**Exemplos derivados da gramática.** Os trechos a seguir compilam e executam dentro do ambiente descrito.

```
-- (1) feitiço mínimo: uma sentença <chamada-stmt>
magia.dano(magia.alvo(), "fogo")
```

```
-- (2) <decl-local> + <chamada-stmt>: usa o retorno da API
local d = magia.dano(magia.alvo(), "acido")
magia.log("Causei " .. d .. " de dano")
```

```
-- (3) <for-num> com chamada no corpo (multicast)
for i = 1, 3 do
  magia.dano(magia.alvo(), "gelo")
end
```

```
-- (4) <for-in> sobre magia.status + <if>: decisão a partir do estado
for _, s in ipairs(magia.status(magia.eu())) do
  if s == "maldicao" then
    magia.dissipar(magia.eu())
  end
end
```

```
-- (5) canalizar e liberar: a carga amplifica o golpe seguinte
magia.carregar()
magia.dano(magia.alvo(), "sagrado")
```

### Restrições além da gramática

Algumas regras da linguagem não podem ser expressas em uma gramática livre de contexto, porque dependem de contagem ou do comportamento do programa durante a execução. A primeira é a restrição métrica dos círculos de magia. O número de linhas lógicas e o número de chamadas `magia.*` de cada feitiço não podem exceder os limites do círculo do jogador, apresentados na \autoref{sec_circulos}. A medição ignora comentários e o conteúdo de strings e não conta quebras de linha dentro de parênteses como linhas novas. Formalmente, trata-se de atributos contáveis sobrepostos à gramática apresentada.

A segunda restrição é operacional e limita a quantidade de ações que um único feitiço pode enfileirar no plano. Os limites de círculo contam linhas e chamadas, mas não o número de voltas de um laço: `for i = 1, 10000 do magia.dano(...) end` ocupa uma linha e uma chamada e cabe no primeiro círculo. Como cada ação é reproduzida na *timeline* em cerca de 0,7 segundo, esse feitiço travaria o combate por horas. O protótipo aceita no máximo 60 ações por feitiço. O excedente é descartado e o registro de combate avisa o jogador, o que serve também de retorno pedagógico sobre o custo de um laço desmedido. É uma restrição que nenhuma gramática captura, da mesma classe do problema da parada que motiva o limite de instruções da sandbox.

A terceira restrição envolve a função `magia.carregar`. Cada chamada acumula um nível de carga, que multiplica o próximo golpe (por dois, três ou quatro) e o torna garantido contra esquivas. Sem um teto, o laço `for i = 1, 10000 do magia.carregar() end` multiplicaria o dano por mais de dez mil dentro dos limites do primeiro círculo. O teto é de três níveis, mas, em vez de simplesmente proibir o excesso, o jogo o transforma em mecânica. Passar do teto causa um *overflow*: a carga acumulada se perde e a energia explode no próprio jogador, com dano equivalente a um quarto da sua vida máxima. A escolha é deliberadamente temática, pois o castigo por estourar um acumulador é o mesmo que as linguagens reais aplicam, e o jogador que tenta explorar o sistema encontra uma consequência dentro da ficção, e não uma mensagem de erro.

## Mecânicas de progressão

### Círculos de magia {#sec_circulos}

A progressão estrutural usa a metáfora dos círculos de magia. O jogador começa no círculo 1 e sobe um círculo a cada chefão derrotado, com exceção do Boneco de Treino, que serve apenas de tutorial. O círculo vigente impõe dois limites ao código de cada magia, um máximo de linhas lógicas e um máximo de chamadas à API, e define também quantas magias podem ser equipadas ao mesmo tempo para o combate, conforme o \autoref{quadro_circulos}.

Quadro quadro_circulos: Limites de código e de magias equipadas por círculo

| Círculo | Máximo de linhas lógicas | Máximo de chamadas à API | Magias equipadas |
|:---:|:---:|:---:|:---:|
| 1 | 2 | 3 | 2 |
| 2 | 3 | 4 | 3 |
| 3 | 4 | 4 | 5 |
| 4 | 5 | 6 | 7 |
| 5 | 6 | 8 | 9 |
| 6 | 8 | 10 | 11 |
| 7 | 10 | 12 | 13 |

Fonte: Elaborado pelo autor.

O ponto central do desenho está no cálculo da potência. O círculo de uma magia é o menor círculo cujos limites comportam o seu código, e é esse círculo, e não o do jogador, que indexa a tabela de dano. Uma magia de uma linha é sempre de círculo baixo e, portanto, fraca, enquanto uma que aproveita as dez linhas do círculo 7 atinge a potência máxima. Programar melhor e ficar mais forte tornam-se, deliberadamente, a mesma coisa. Os limites funcionam ainda como andaime no sentido discutido na fundamentação: nos primeiros círculos, o jogador só consegue (e só precisa) escrever duas linhas, e a complexidade permitida cresce junto com o seu domínio.

### Imunidades: cada chefão é um problema

Se os círculos regulam o quanto se pode escrever, as imunidades dos chefões determinam o que vale a pena escrever. Cada chefão intercepta uma classe de estratégia, e as imunidades são acumulativas, pois a estratégia bloqueada por um chefão continua sendo um mau caminho diante dos seguintes. O \autoref{quadro_chefoes} resume a progressão.

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

O quadro deixa visível a filiação à Aprendizagem Baseada em Problemas: o chefão é o problema, a magia é a solução proposta, e a imunidade é o que torna a solução anterior insuficiente. O Reflexo Veloz, por exemplo, pune o padrão `for i = 1, n do magia.dano(...) end`, pois cada golpe da sequência tem chance crescente de ser esquivado e a sequência inteira é interrompida quando a esquiva satura. A resposta projetada para ele é a canalização, já que o golpe carregado por `magia.carregar` é garantido e troca a quantidade de golpes por um golpe certeiro. O Daemon Firewall bloqueia o tipo de dano ao qual acabou de se adaptar, o que empurra o jogador para código condicional que alterna elementos. O chefão final anula qualquer feitiço repetido e cobra, de uma vez, o repertório inteiro construído até ali. Nenhuma dessas informações é anunciada ao jogador. O registro de combate relata o que aconteceu ("o golpe foi esquivado", "a cura foi corrompida"), e cabe a ele formular a hipótese, testá-la e refinar o código, o que faz do ciclo de depuração uma mecânica de jogo.

### Tutoriais: a ferramenta sem a solução

Um jogo que exige escrever código precisa, em algum momento, ensinar a linguagem. A referência da API, disponível no grimório, documenta as funções da tabela `magia`, mas não as construções de Lua: quem não sabe escrever um laço não tem como descobrir que precisa de um. Para isso o protótipo inclui um balão de tutorial, exibido na primeira vez em que o jogador enfrenta cada chefão e que pode ser reaberto depois pelo botão "? Dica", tanto no combate quanto no grimório. Cada balão apresenta uma construção da linguagem, com explicação e exemplos, na sequência resumida no \autoref{quadro_tutoriais}.

Quadro quadro_tutoriais: Sequência de lições dos tutoriais de linguagem

| Lição | Tema | Construções apresentadas |
|:---:|---|---|
| 1 | A chamada de função | nome, parênteses, argumentos e ordem de avaliação |
| 2 | Laços de repetição | `for i = 1, N do ... end` |
| 3 | Variáveis e valor de retorno | `local d = magia.dano(...)` |
| 4 | Condicionais | `if`, `then`, `else`, `end` e operadores de comparação |
| 5 | Tabelas e listas | `{...}`, `#` e `ipairs` |
| 6 | Ler o próprio estado | iteração sobre `magia.status`, comparação de strings, `magia.dissipar` |
| 7 | Composição | combinação das construções anteriores em um só feitiço |

Fonte: Elaborado pelo autor.

A regra que define o conteúdo dos tutoriais é que eles ensinam apenas a linguagem. Nenhum texto cita o chefão enfrentado, suas imunidades ou suas fraquezas, e um teste automatizado falha se o nome de um chefão aparecer em algum tutorial. A separação distingue duas coisas que antes estavam misturadas: a ferramenta, isto é, como se escreve um laço, que é ensinada no momento exato em que passa a ser necessária; e o uso da ferramenta diante daquele inimigo, que continua sendo descoberto por experimentação e pela leitura do registro de combate. O andaime passa a ter duas camadas. Os círculos controlam quanta complexidade o jogador pode escrever, e os tutoriais entregam a sintaxe sem entregar a solução. Os exemplos de cada lição respeitam os limites do círculo em que o jogador estará ao vê-la, e todos são compilados na sandbox pela suíte de testes.

<!-- TODO(autor): inserir aqui o print do balão de tutorial quando o arquivo existir, por exemplo:
![Balão de tutorial](imagens/tutorial.png){#fig_tutorial largura=90%}

Fonte: Captura de tela do protótipo CodeMage.
-->

## Validação, testes e persistência {#sec_validacao}

Uma regra simples sustenta a confiabilidade do conjunto: só entra no grimório magia que compila e executa. Ao salvar, o código é compilado e rodado uma vez contra um combate de teste descartável, um "combate-fantasma" contra o Boneco de Treino, e qualquer erro de sintaxe ou de execução é reportado imediatamente, no momento em que o aprendiz ainda tem o contexto do que escreveu. Além do valor pedagógico do retorno imediato, a validação garante uma invariante do sistema: toda magia equipada é executável, e o combate real nunca encontra código quebrado.

As regras do jogo são verificadas por uma suíte de testes automatizados que roda fora da interface gráfica, sobre o próprio LÖVE, e reporta cada verificação como aprovada ou reprovada. São cerca de noventa verificações, que cobrem a tabela de dano, o comportamento de cada chefão, os efeitos de status, a canalização e o *overflow*, o teto de ações por feitiço, o bloqueio do campo `vida`, o grimório, a gravação e a leitura do progresso, o gerador de números pseudoaleatórios e os tutoriais. A suíte corresponde à etapa de validação técnica prevista na metodologia.

O progresso do jogador (círculo, chefões vencidos, lições de tutorial já vistas e o grimório completo, com o código-fonte de cada magia) é persistido como um arquivo de dados em sintaxe Lua, relido em um ambiente vazio. O arquivo carregado é somente dado e não tem acesso a função alguma, de modo que um save adulterado não executa nada, e um save corrompido é ignorado sem interromper o jogo. Na serialização, os textos das magias são escapados caractere a caractere, o que preserva quebras de linha e caracteres de controle sem corromper o arquivo. Ao carregar, cada magia é revalidada contra a sandbox, as inválidas são descartadas e as equipadas são limitadas à quantidade permitida pelo círculo. A invariante do grimório sobrevive, portanto, até à edição manual do arquivo de save.
