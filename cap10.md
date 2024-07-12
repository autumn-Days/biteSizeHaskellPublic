# 10. Parentização e composição de funções

## 10.1 Parentização

### 10.1.1 Motivação

Parentização é um termo que se refere a forma como os parênteses, ou a ausência deles, funcionam em uma linguagem de programação. 

A primeira vista, certamente, isso parece um tópico bastante impertinente de ser abordado nestes capítulos derradeiros, pois já usamos este conceito de forma exaustiva ao longo da disciplina. "Então por que se incomodar em estudá-lo agora?", você me pergunta.

Essa é uma excelente pergunta e é justamente por causa dela que _não vamos estudar o funcionamento dos parênteses_, mas sim a _ausência deles_. Portanto, nesta seção analisaremos a função cifrão (`$`) que tem como um de seus objetivos diminuir a quantidade de parênteses do código.

### 10.1.2 A função cifrão

#### 10.1.2.1 A ideia por trás

Como dito anteriormente, a função `$` diminui a quantidade de parênteses no código e ela faz isto dividindo-o em pequenas seções de tal forma que a porção do código que estiver a sua direita será avaliado primeiro, enquanto a porção esquerda será avaliada depois.

Esta precedência da porção direita sobre a esquerda, além de ter como efeito prático tornar o uso dos parênteses supérfluo, pois faz com que o GHC/GHCi saiba a ordem em que a expressão deve ser avaliada, ela também consegue atrasar a avaliação da porção esquerda, o que pode ser bastante útil dependendo do tipo de função que se deseja criar.

Redundante falar, mas, uma vez que `$` divide o código em partes, ela é uma _função infixa_, pois o seu primeiro parâmetro é a porção esquerda, enquanto o segundo é a direita.

#### 10.1.2.2 Utilização

```haskell
1.someMath1 = (3^)(3+4)
2.someMath2 = (3^)$3+4
>>someMath1 == someMath2 
True
```

Na linha 2, a utilização do cifrão fez com que `3+4` seja avaliado primeiro, fazendo com que `7` seja o expoente de `(3^)`, exatamente como na linha 1. Caso o cifrão tivesse sido omitido, a comparação da 3° linha retornaria `False`.

```
1.someMath1 = (3^)(3+4)
2.someMath2 = (3^)3+4
>>someMath1 == someMath2 
False
```

Isto acontece porque o expoente de `(3^)` passaria a ser `3`, o que retornaria 27 e faria com que `4` fosse adicionado por último à expressão.

Um exemplo ligeiramente mais elaborado poderia ser:

```haskell
>>(3*) $ (2^) $ 3+1
48
```

A porção mais a direita da expressão, no caso, `3+1` é avaliada primeiro e retorna `4`. Nisto, `4` é aplicado a `2^`, se tornado o expoente da expressão e retornando `16`. Por último, `(3*)` recebe `16` como argumento e retorna `48.

## 10.2 Composição de funções

### 10.2.1 Motivação

Na matemática, é possível que, dado uma função $f:A \shortrightarrow B$ e uma função $g:B \shortrightarrow C$, $f$ seja "passada como argumento" para $g$, uma vez que o contradomínio de $f$ corresponde ao domínio de $g$. Isto pode ser expresso por $g \circ f$ e é chamado de _função composta_.

No haskell, não é novidade que isto seja possível, pois estamos fazendo isto já fazem nove capítulo. No entanto, ainda assim, este conceito pode ser bastante útil para nós. Imagine, por exemplo, que você tenha as duas funções abaixo:

```haskell
reverse :: [a] -> [a]
sort :: Ord a => [a] -> [a] --orderna a lista em ordem crescente
```

Caso você quisesse criar uma função que ordena uma lista em ordem decrescente, você poderia muito bem fazer algo do tipo:

```haskell
descendingSort :: Ord a => [a] -> [a]
descendingSort _list = (reverse) $ sort _list
```

No entanto, caso pudéssemos usar funções compostas, poderíamos definir `descendingSort` como, simplesmente, `reverse`$\circ$`sort`, pois o `reverse` esperaria a saída do `sort` e, `sort`, por sua vez, estaria sendo parcialmente aplicada, ou seja, ele estaria à espera de uma lista. Lista essa que seria passada para `descendingSort` e, imediatamente, repassada para `sort`.

Realmente, seria muito bom se pudéssemos usa a composição de funções, pois isto deixaria a implementação de `descendingSort` mais concisa... que bom que podemos!

O haskell possui um operador ponto(`.`) que funciona exatamente como o $\circ$. 

Na prática, a função à direita do ponto será avaliada primeiro e a sua saída será fornecida para a função à esquerda do ponto.

Por causa disto, é fundamental que a função à direita do ponto produza como saída um valor de tipo `x` e que a função a esquerda do ponto receba como entrada um valor de mesmo tipo `x`.

Desta forma, `descendingSort` poderia ser implementada da seguinte forma:

```haskell
1.descendingSort:: Ord a => [a] -> [a]
2.descendingSort = reverse.sort --parcialmente aplicada
>> descendingSort [31,532,14,87]
[532,87,31,14]
```

> **Note:** `(reverse) $ sort _list` é o mesmo que `reverse (sort _list)`. Apesar de a utilização do cifrão não deixar o código mais legível neste caso, é um bom recurso didático para que você se acostume com esse novo recurso.

> **Note:** A função `sort` está definida na biblioteca `Data.List`. Para utilizá-lo, basta fazer `import Data.List (sort)`


### 10.2.2 Exemplos

```haskell
1.mult3 = (*3)
2.add7 = (+7)
3.funnyOperation = add7.mult3
>>funnyOperation 1
10
```

Neste exemplo, temos duas funções parcialmente aplicadas:

1. mult3
2. add7

Portanto, quando fazemos `add7.mult3` na terceira linha, estamos definindo uma função parcialmente aplicada que está a espera de um valor numérico `y` que sirva como entrada para `mult3. Logo, assim que `mult3` terminar de processar a entrada, a sua saída será fornecida para a função parcialmente aplicada `add7` que será responsável por produzir o resultado final.

Essas funções foram divertidas, mas podemos fazer algo mais emocionante.

```
{--soma os cubos impares--}
1. somaCubosImpares :: [Integer] -> Integer
2. somaCubosImpares = sum . map (^3) . filter (not . isEven)
  where isEven x = if x `mod` 2 == 0 then True else False
```

Para entendermos bem está função, é altamente recomendável que façamos a sua leitura da direita para à esquerda. Com isto analisamos que temos as seguintes funções  

1. `filter (not . isEven)`<br> Aqui temos uma aplicação parcial da função `filter`. Ela está a espera de uma lista `[a]` e, assim que recebê-la, filtrará os seus elementos impares de acordo com o predicado `not.isEven`.<br><br>Como já sabemos `filter` retornará uma lista `[b]`, que é justamente tipo esperado pela função `map (^3)`. 
2. `map (^3)`<br>Receberá a lista com todos os elementos impares de `[a]` e aplicará a função parcialmente aplicada `^3` em cada um deles, fazendo com que o cubo desses elementos seja obtido. Por fim, retornará uma lista `[c]`.
3. `sum`<br> Receberá uma lista `[c]` e realizará o somatório de todos os seus elementos, com isto, o resultado final será gerado.


> **Note:** O predicado `not.isEven` da função `filter (not.isEven)`funciona recebendo um valor de tipo numérico `x` e analisa se ele é par ou não, o valor booleano da saída é então encaminhado para a função `not`, que inverte os valores booleanos.

> **Note:** `(f.g)` é o equivalente à `f (g x)`, onde `x` é o argumento passado.

## Referências

1. https://wiki.haskell.org/Function_composition
2. Allen, Christopher; Moronuki, Julie. Haskell programming from its principles:Pure functional programming without fear or frustration.1 ed. Lorepub, 2016. p. 30-31.

## Sugestões de leitura

1. Allen, Christopher; Moronuki, Julie. Haskell programming from its principles:Pure functional programming without fear or frustration.1 ed. Lorepub, 2016. p. 30-32.
