# Gramática formal da linguagem de magias do CodeMage

Este apêndice apresenta a especificação formal, em notação BNF/EBNF, do subconjunto da linguagem Lua aceito pelo editor de magias do CodeMage, somada à interface de programação de domínio (`magia.*`) exposta pela *sandbox*.

O CodeMage não define uma linguagem nova: reaproveita o interpretador Lua 5.1 (LuaJIT) embarcado no framework LÖVE e o restringe a um ambiente fechado. A linguagem efetivamente aceita tem, portanto, duas camadas. A primeira é sintática: quais construções o texto do jogador pode conter — qualquer programa Lua válido compila, mas só um subconjunto é útil, porque a tabela de globais é reduzida. A segunda é semântica: quais nomes existem no ambiente. Tudo o que não estiver no ambiente montado pela *sandbox* resolve para `nil`, e o uso de um nome inexistente como função gera erro de execução, rejeitado na validação do grimório. A gramática a seguir formaliza a linguagem pretendida — o subconjunto idiomático que o jogo ensina —, e não toda a sintaxe de Lua; a contribuição do trabalho está justamente na restrição, não na linguagem hospedeira.

## Convenções de notação

| Símbolo | Significado |
|:---:|---|
| `::=` | definido como |
| &#124; | alternativa |
| `[ x ]` | opcional (zero ou uma ocorrência) |
| `{ x }` | repetição (zero ou mais ocorrências) |
| `( x )` | agrupamento |
| `"x"` | terminal literal |
| `<x>` | não terminal |

## Estrutura de um feitiço

O texto completo escrito pelo jogador é um bloco: não há declaração de função na linguagem pretendida, e o corpo da magia é o próprio *chunk* de topo, compilado de uma só vez pela *sandbox*.

```
<magia>        ::= <bloco>

<bloco>        ::= { <instrucao> }

<instrucao>    ::= <atribuicao>
                 | <decl-local>
                 | <chamada-stmt>
                 | <if>
                 | <for-num>
                 | <for-in>
                 | <while>
                 | <return>
                 | <comentario>
```

## Declarações e atribuições

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

Há uma restrição semântica relevante: as entidades obtidas por `magia.alvo()` e `magia.eu()` são expostas como *proxies*, e o campo `vida` é bloqueado tanto para escrita quanto para leitura — `entidade.vida = x` e a simples consulta a `entidade.vida` lançam erro. A vida só é alterada pelas funções de efeito da API (`magia.dano`, `magia.curar`, `magia.absorver`).

## Estruturas de controle

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

<return>       ::= "return" [ <lista-expr> ]
```

A forma idiomática do `<for-in>` ensinada pelo jogo é a iteração sobre a lista de status devolvida pela API: `for _, s in ipairs(magia.status(magia.alvo())) do ... end`. Os laços `while` e `for` são permitidos sem restrição sintática; a proteção contra laços infinitos é dinâmica, feita pelo *hook* de instruções descrito no \autoref{cap_prototipo}.

## Chamadas e expressões

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

## Terminais léxicos

```
<Nome>         ::= ( <letra> | "_" ) { <letra> | <digito> | "_" }
<Numero>       ::= <digito> { <digito> } [ "." { <digito> } ]
                 | "0x" <hexdigito> { <hexdigito> }
<string>       ::= '"' { <char-sem-aspas> } '"'
                 | "'" { <char-sem-aspas> } "'"
<comentario>   ::= "--" { <char-sem-quebra> }
<letra>        ::= "a" | ... | "z" | "A" | ... | "Z"
<digito>       ::= "0" | ... | "9"
```

## A API de domínio

A tabela global `magia` é o vocabulário de domínio da linguagem — o que distingue um programa Lua qualquer de um feitiço. A gramática das chamadas é a seguinte; o contrato semântico de cada função (retorno e efeito enfileirado) foi apresentado no \autoref{quadro_api}.

```
<chamada-api>  ::= <alvo-expr> | <eu-expr>
                 | <dano> | <curar> | <absorver>
                 | <carregar> | <dissipar>
                 | <status> | <log>

<alvo-expr>    ::= "magia" "." "alvo"     "(" ")"
<eu-expr>      ::= "magia" "." "eu"       "(" ")"
<dano>         ::= "magia" "." "dano"     "(" <expr-entidade> "," <tipo-dano> ")"
<curar>        ::= "magia" "." "curar"    "(" <expr-entidade> "," <expr> ")"
<absorver>     ::= "magia" "." "absorver" "(" <tipo-dano> ")"
<carregar>     ::= "magia" "." "carregar" "(" ")"
<dissipar>     ::= "magia" "." "dissipar" "(" <expr-entidade> ")"
<status>       ::= "magia" "." "status"   "(" <expr-entidade> ")"
<log>          ::= "magia" "." "log"      "(" <expr> ")"

<expr-entidade> ::= <alvo-expr> | <eu-expr> | <expr>

<tipo-dano>    ::= '"neutro"'  | '"fogo"'   | '"eletrico"' | '"acido"'
                 | '"sagrado"' | '"veneno"' | '"gelo"'     | '"glitch"'
                 | '"trevas"'
```

Duas observações semânticas. Primeira: `<tipo-dano>` lista os nove tipos válidos, mas uma string fora da lista não é erro — em tempo de execução ela é normalizada para `"neutro"`. Segunda: a potência do dano não aparece na gramática porque não é argumento; ela vem da tabela interna de balanceamento, indexada pelo tipo e pelo círculo da magia, como discutido no \autoref{cap_prototipo}.

## Ambiente da *sandbox*

O conjunto de nomes globais visíveis a um feitiço é fechado e pode ser descrito como um alfabeto:

```
<global>       ::= "math"  | "string" | "table"
                 | "pairs" | "ipairs" | "select"
                 | "tonumber" | "tostring" | "type"
                 | "pcall" | "error" | "assert"
                 | "magia"
```

Qualquer `<Nome>` usado como global fora desse conjunto resolve para `nil`. Não existem `os`, `io`, `love`, `require`, `load`, `dofile`, `package`, `debug`, `_G`, `getfenv`/`setfenv` nem `collectgarbage`; além disso, `string.dump` é removido e `string.rep` é limitado. Essa cláusula é a fronteira pedagógica da linguagem: o ambiente reduzido define exatamente o que o jogador pode aprender, e a gramática somada ao ambiente constitui a linguagem de ensino.

## Exemplos derivados da gramática

```lua
-- (1) feitiço mínimo: uma sentença <chamada-stmt>
magia.dano(magia.alvo(), "fogo")
```

```lua
-- (2) <decl-local> + <chamada-stmt>: usa o retorno da API
local d = magia.dano(magia.alvo(), "acido")
magia.log("Causei " .. d .. " de dano")
```

```lua
-- (3) <for-num> com chamada no corpo (multicast)
for i = 1, 3 do
  magia.dano(magia.alvo(), "gelo")
end
```

```lua
-- (4) <for-in> sobre magia.status + <if>: decisão a partir do estado
for _, s in ipairs(magia.status(magia.eu())) do
  if s == "maldicao" then
    magia.dissipar(magia.eu())
  end
end
```

```lua
-- (5) canalizar em um turno, liberar no seguinte
magia.carregar()
magia.dano(magia.alvo(), "sagrado")
```

Os cinco exemplos compilam e executam dentro do ambiente descrito.

## Restrição métrica dos círculos

Além da gramática, cada magia está sujeita a uma restrição métrica imposta pela progressão do jogador, não expressável em BNF pura por depender de contagem: o número de linhas lógicas e o número de chamadas `magia.*` não podem exceder os limites do círculo vigente (\autoref{quadro_circulos}, no \autoref{cap_prototipo}). A medição ignora comentários e o conteúdo de strings e não conta quebras de linha dentro de parênteses como linhas novas. Trata-se, formalmente, de uma gramática com atributos contáveis sobrepostos à gramática livre de contexto aqui apresentada.
