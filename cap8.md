# 8. Funções de alta ordem - pt. 2

## 8.1 Introdução

No capítulo anterior foi lhe apresentado o conceito de funções de ordem superior e, para exemplificar este novo conceito, foi introduzido a função `map`, além de ter sido tratado sobre as funções lambda.

Neste capítulo vamos nos aprofundar ainda mais neste tópico ao exploramos mais funções de alta ordem que, assim como o map, operam sobre listas.

## 8.2 Função filter

A função `filter` tem como objetivo selecionar todos os elementos de uma lista que atendem a um determinado predicado, isto é, que possuem uma certa característica.

### 8.2.1 Assinatura

```haskell
filter :: (a -> Bool) -> [a] -> [a]
```

Serão fornecidos para `filter` uma lista `[a]` e uma função `fun` que receberá como entrada um elemento `a` de `[a]`, de tal forma que, `fun` avaliará se `a` atende ou não ao predicado. Caso ele atenda, `fun` retornará `True` e  `a` será adicionado a `[a]'` que corresponde a lista de todos os elementos que atendem ao predicado. Caso ele não atenda, `fun` retornará `False` e, por conseguinte, `a` não será adicionado a `[a]'`.

### 8.2.2 Exemplos

#### Seleciona divisíveis

```haskell
selectDivisibles:: Integral a => a -> [a] -> [a]
selectDivisibles divisor _list = filter (\a -> a `mod` divisor == 0) _list 
```
```haskell
>> selectDivisibles 2 [1,2,3,4,5,6,7,8,9,10]
[2,4,6,8,10]
```

#### Seleciona minúsculos

```haskell
1.someCharacters = ['A' .. 'z']
>> someCharacters
"ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz"
```
```haskell
2.justLower = filter (\char -> char >= 'a' && char <= 'z') someCharacters
>>justLower
"abcdefghijklmnopqrstuvwxyz"
```

#### Seleciona as listas em ordem

As funções anteriores serviram tão somente para que pudessemos sentir o aroma do `filter`, para que possamos sentir um pouco mais de emoção, podemos tentar implementar uma função que 

> dado uma matriz $M_{n \times m}$, faça uma função `selectInOrder` que selecione todas as matrizes $M´_{1 \times m} \in M_{n \times m}$, de tal forma que, para cada elemento $m'_{1 \times j} \in M'_{1 \times m}$, $m'_{1 \times j} < m'_{1 \times j+1}$ 

E, caso você não tenha entendido o enunciado da versão da questão da segunda chamada, permita-me apresentar a versãa da primeira chamada:

> dada uma lista de listas, faça uma função `selectInOrder` que seleciona as listas cujos elementos estão em ordem crescente.

Tendo entendido melhor o enunciado, ainda é necessário que, antes de começarmos a pensar no `filter` ou em todos os casos que `selectInOrder` tenha que lidar com uma excessão, temos que nos perguntar uma coisa: 

>_Qual a alma desta função?_

 Certamente, ela reside na função `fun`, a qual é responsável por determinar se o elemento `[a]` pertencente a `[[a]]` é crescente ou não.

Tendo isto como premissa, basta que analisemos se, dado uma lista $(x:xs)$ de tamanho $m$ é verdade que para todo $x_{i}$ $\in$ $(x:xs)$, $x_{i}$ < $x_{i+1}$. Observe:

```haskell
1. selectInOrder (x:y:xs) = x <= y && selectInOrder (y:xs)
```

Este trecho de código é a _alma_ de `selectInOrder`, pois ela é responsável por comparar $x_{i}$ com $x_{i+1}$, já que ocorre a comparação `x <= y`. Caso $x_{i}$ seja maior que $x_{i+1}$ é retornado falso e a recursão acaba. Caso contrário, a recursão continua.
 

No entanto, `selectInOrder` ainda está incompleto, pois ainda é necessário que consideremos o caso em que $x_{i}$ é igual a $x_{m}$, ou seja, o caso em que $x_{i}$ é o último elemento, pois não haverá um elemento $x_{m+1}$ que possa ser comparado a $x_{i}$.

Portanto, devemos fazer uma pequeno _casamento de padrão_ para evitar este problema.

```haskell
1. selectInOrder (x:[]) = True
2. selectInOrder (x:y:xs) = x <= y && selectInOrder (y:xs)
```

A linha 1 se traduz como, "caso a lista seja unitária, então ela está ordenada". Perceba que `selectInOrder` só retornará `True` no caso em que a lista é unitária, ou seja, quando os primeiros $x_{m-1}$ elementos já tenham sido analisados e reste apenas um elemento a ser analisado, no caso, $x_{m}$. 

Além disto, não podemos esquecer o caso da cardinalidade de  $(x:xs)$ ser igual a zero, assim:

```haskell
selectInOrder [] = True
selectInOrder (x:[]) = True
selectInOrder (x:y:xs) = x <= y && selectInOrder (y:xs)
```

Tendo a função `fun` em mãos, agora só temos que aplicar ela ao `filter`.

```haskell
whichSorted :: Ord a => [[a]] -> [[a]]
whichSorted _listOfLists = filter selectInOrder _listOfLists
```
```haskell
>>whichSorted [[1,2,3], [10,5,2],[50,75,76,100]]
[[1,2,3],[50,75,76,100]
```
## 8.3 span e break

As funções `span` e `break` são bastante similares entre sí, sendo uma o contrário da primeira.

### 8.3.1 span

Basicamente, `span` receberá como parâmetros uma função $p$ - que servirá de predicado - e uma lista $[a]$ de tamanho $n$ de tal modo que ela repartirá $[a]$ em duas porções $L_{1}$ e $L_{2}$.

A porção $L_{1}$ tem todos os elementos de $[a]$ que satisfazem ao predicado até o momento da avaliação de um elemento $a_{k}$ que não o segue.

Por sua vez, a porção $L_{2}$ tem todo o restante dos elementos de $[a]$ a partir de $a_{k}$

As porções $L_{1}$ e $L_{2}$ são retornadas por uma tupla, logo elas podem ser selecionadas facilmente com as funções `fst` e `snd`.

Observe a sua assinatura para entender melhor:

```haskell
span :: (a -> Bool) -> [a] -> ([a], [a])
```

#### 8.3.1.1 Exemplo

```haskell
1. breakThisList _list = fst (span (< 3) _list)
2. breakThisList [1,2,3,4,1,2,3,4]
3. [1,2]
```

Este exemplo se traduz como "dado uma lista `list` selecione os elementos menores que 3 até que seja encontrado ume elemento maior que 3".

Perceba que `span` em sí retorna

```haskell
([1,2],[3,4,1,2,3,4])
```

### 8.3.2 break

A função `break` é apenas o contrário de `span`, ou seja, $L_{1}$ se tratá de todos os elemento de $[a]$ que não atendem ao predicado fornecido até o momento do surgimento de um elemento $a_{k}$ que atenda. De forma análoga, $L_{2}$ se trara de todos os elementos de $[a]$ a partir de $a_{k}$. Abaixo, consta a sua assinatura.

```haskell
break :: (a -> Bool) -> [a] -> ([a], [a])
```

#### 8.3.1.2 Exemplo

```haskell
1. breakThisOtherList _list = break (> 3) _list
2. breakThisOtherList [1,2,3,7,1,2,3,4]
3. ([1,2,3],[7,1,2,3,4])
```

Neste exemplo, break recebe uma função parcialmente aplicada definida por `(> 3)` que retornará `True` caso ela receba um elemento $x$ pertencente a `_list` que seja maior que 3.

Portanto, $L_{1}$ se trata de todos os elementos de `_list`  que não satisfazem ao predicado de ser maior que 3 até a avaliação de um elemento que satisfaça. A partir do instante que é verificado que `7` atende a propriedade estabelecida por `(> 3)`, todos os elementos de `_list` a partir dele são adicionados a $L_{2}$.
<br><br>
> **Note**: Perceba que a função `(> 3)` é diferente da função `((>) 3)`. A primeira é uma aplicação parcial de `>` e isto faz com que o parâmetro `3` corresponda ao seu segundo operando, de tal modo que as "iterações" do exemplo anterior funcionem da seguinte maneira:<br><br> `1 > 3 -> False`<br>`2 > 3 -> False`<br>`3 > 3 -> False`<br>`7 > 4 -> True`<br><br> Caso a função fornecida a break tivesse sido a segunda, `3` corresponderia ao primeiro operando de `>`, pois `>` está sendo descrita em sua forma prefixa. Observe como as comparações funcionariam neste caso para uma lista arbitrária $L$ de tamanho $n > 4$.<br><br> $3 > l_{0} \rightarrow Bool$<br>$3 > l_{1} \rightarrow Bool$<br>$3 > l_{2} \rightarrow Bool$<br>...<br>$3 > l_{n} \rightarrow Bool$<br>

### 8.4 takeWhile e dropWhile

As funções `takeWhile` e `dropWhile` podem ser entendidas como variações da função `break` com a única diferença que primeira faz o serviço de selecionar $L_{1}$ e a segunda de selecionar $L_{2}$ de forma automatica.

## Referências

1. Thomson, Simon. Haskell - The craft of a functional programming language. 3ed. 247

1. https://hackage.haskell.org/package/base-4.20.0.1/docs/Prelude.html#v:span

1. https://hackage.haskell.org/package/base-4.20.0.1/docs/Prelude.html#v:break
