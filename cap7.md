# 7. Funções de alta ordem - pt. 1

## 7.1 Introdução

Uma função de ordem superior, ou uma função de alta ordem, é uma função que recebe como parâmetro uma outra função ou que retorna uma função como saída. 

Tendo isto em mente, talvez você possa ter a impressão que já vimos funções de ordem superior várias vezes ao longo do nosso percurso, pois havíamos passado chamadas de funções como parâmetros para outras funções de forma exaustiva.

No entanto, este não é o caso, pois até então, haviámos visto funções que recebiam como parâmetros os _outputs_ de outras funções.

Assim, mesmo que tivéssemos passado alguma chamada de uma função `callBackFun arg1 ... argn` como parâmetro para uma função `fun`, ao fim do dia, o que havia sido passado para `fun` foi apenas o _output_ de `callBackFun`, não `callBackFun` em sí.

Quando falamos em funções de ordem superior, estamos falando em passar uma função em sí como parâmetro para uma outra função.

> **Note:** A título de simplificação, o fato de todas as funções do haskell serem funções curry foi ignorada.<br><br>
>Se considererássemos o processo de _currying_, então sim, todas as vezes que passamos  uma função `callBackFun arg1 ... argn` como parâmetro para uma função `fun`, `fun` poderia ser considerada uma função de alta ordem, pois teríamos que `fun` receberia `callBackFun` e retornaria `fun callBackFun`, que, por sua vez, receberia `arg1` e retornaria `(myFun callbackFun) arg1` e assim por diante.<br><br> Ademais, nem teríamos que ir tão longe ao ponto de ter que passar uma chamada de uma função `callBackFun arg1 ... argn` como parâmetro de uma `fun`, pois, se passassemos mais de uma parâmetro para `fun`, ela retornaria como output uma _função parcialmente aplicada_, então a condição de ela retornar uma função seria satisfeita, o que faria com que todas as funções com mais de 1 parâmetro sejam funções de ordem superior.<br><br>No entanto, é bastante oportuno que ignoremos isto por enquanto, pois isso complicaria bastante a terminologia. Porém, caso você não queira ignorar o funcionamento do _currying_, você pode interpretar que uma função de alta ordem é uma função cuja ao menos um dos seus argumentos é uma função e/ou retorna uma função de alta ordem como saída.

## 7.2 Motivação

Funções de alta ordem conseguem atender a vários propósitos, um dos mais notáveis é a reutilização de código. Por exemplo, suponha que você esteja escrevendo um programa que sirva como uma calculadora. Para tanto, você poderia escrever uma função para cada uma das quatro operações, como demonstrado abaixo:

```haskell
add:: Num a => a -> a -> a
add x y = x+y
	 
sub:: Num a => a -> a -> a
sub x y = x-y
	 
mult:: Num a => a -> a -> a
mult x y => x*y
	 
div:: Num a => a -> a -> a
mult x y = x/y
```

Perceba, porém, que, do ponto de vista estrutural, essas funções são praticamente iguais, a única coisa difere elas entre sí, é a operação que está sendo realizada! A repetição de código neste exemplo pode gerar algums empecilhos para o programador, como:

1. Mais tempo de escrita
1. Mais tempo de depuração
1. Mais erros
    - quanto mais você escreve código, maiores são as chances de você errar.

Portanto, ao invés de termos que escrever tudo isto, é preferível escrever uma única função que receberá uma função do tipo `Num a => a-> a->a` e 2 parâmetros numéricos, assim economizaremos tempo de escrita, tempo de depuração, e memória em armazenamento.

```haskell
tudoEmUm :: (t1 - > t2 -> t3) -> t1 -> t2 -> t3
tudoEmUm fun x y = (fun) x y
```

Conseguimos obeservar que `tudoEmUm` receberá uma função de nome `fun` e valores numéricos `x` e `y` e que, por sua vez, `fun` também receberá como parâmetros `x` e `y` e o seu retorno será atribuído a `tudoEmUm`.

Observe alguns exemplo de `tudoEmUm`:

```haskell
>>tudoEmUm (+) 2 4
6
>>tudoEmUm (-) 8 5
3
>>tudoEmUm (*) 4 2
8
>>tudoEmUm (/) 10 5 
2.0
```

### 7.2.1 Assinatura de tipo de `tudoEmUm`

Se lembrarmos do que foi dito no capítulo anterior, perceberemos que `tudoEmUm` não é uma função que recebe uma outra função e mais dois argumentos numéricos, mas sim ela é uma função  que recebe uma segunda função - que está expressa por ```(t1 -> t2 -> t3)``` -  e que retorna uma função `tudoEmUm (t1 -> t2 -> t3)`.

De forma semelhante, `tudoEmUm (t1 -> t2 -> t3)` receberá apenas um argumento, neste caso, um argumento do tipo `t1` e retornará uma função `(tudoEmUm (t1 -> t2 -> t3)) t1`.

Assim como as funções retornadas anteriormente,  `(tudoEmUm (t1 -> t2 -> t3)) t1` receberá apenas um argumento, desta vez, de tipo `t2` e, para acabar com a brincadeira, ele retornará um valor de tipo `t3`.

Para que isto fique ainda mais claro, considere o exemplo abaixo:

```haskell
1. tudoEmUm :: (t1 -> t2 -> t3) -> t1 -> t2 -> t3
2. tudoEmUm fun x y = (fun) x y
3. fstOutput = tudoEmUm (+)
4. sndOutput = fstOutput 2
5. trdOutput = sndOutput 4
```

```haskell
>>trdOutput
6
```
Também é muito curioso notar que, caso a assinatura de `tudoEmUm` fosse `t1 -> (t1 -> t2 -> t3) -> t2 -> t3` ela também funcionaria, apesar de soar muito estranho!

## 7.3 A função map

### 7.3.1 Motivação

Suponha que você tenha uma lista de $n$ elementos e você queira aplicar uma variedade de operações sobre essa lista conforme as suas necessidades. Ora, seria muito enfadonho se, para cada operação que você desejasse fazer, você tivesse que criar uma função exclusiva para realizá-la. Exemplo:

```haskell
--versão para soma
somaDois [] = []
somaDois (x:xs) = fun x : somaDois xs
  where fun = (+) 2
  
--versão para subtração
subtracaoDois [] = []
subtracaoDois (x:xs) = fun x : subtracao xs
  where fun = (-) 2
```

Poderíamos continuar imaginando exemplos por horas à fio, mas acredito que você já entendeu qual é a a problema, não é ? Exatamente! As funções apresentadas são praticamente iguais do ponto de vista estrutural. A única diferença de uma para outra reside na operação realizada por ```fun```!

Você não acha que seria mais simples e elegante se fôssemos capazes de criar uma função genérica que recebesse a operação que queremos realizar sobre a lista ? Afinal de contas, isto nos pouparia muito trabalho escrevendo código, diminuiria o tempo de manutenção e depuração, além de fornecer um aumento de legibilidade e diminuição de espaço gasto em disco.

Eu sei, fiz muito drama, é claro que podemos fazer isto ! Observe :

```haskell
1. transformaTodos :: (t -> a) -> [t] -> [a]
2. transformaTodos fun [] = []
3. transformaTodos fun (x:xs) = (fun) x : transformaTodos fun xs
```

```
>>transformaTodos (*2) [1,2,3,4,5]
[2,4,6,8,10]
```

Perceba que para implementar esta função, somente tivemos que passar como argumentos para `transformaTodos` a função parcialmente aplicada `(*2)` e a lista que queríamos e ela "transformasse" e, assim, todos os elementos da lista `[1,2,3,4,5]` foram multiplicados por 2.

### 7.3.2 Acerca da assinatura de `transformaTodos`

A função `transformaTodos` tem a seguinte assinatura:

```haskell
transformaTodos :: (t -> a) -> [t] -> [a]
```

Portanto, ela receberá dois argumentos como entrada :

1. Uma função `(t -> a)` que terá como entrada um tipo `t`, ou seja, um elemento da nossa lista, e produzirá como saída um valor do tipo `a` que, no exemplo anterior, é o elemento de tipo `t` multiplicado por dois.
2. Uma lista de elementos `[t]` composta por elementos do tipo `t`

Além disto, `transformaTodos` produzirá como saída uma lista `[a]` composta por elementos do tipo `a`.

 É muito importante notarmos que é possível que os tipos `t` e `a` podem ser iguais, mas caso a assinatura de `transformaTodos` fosse `transformaTodos :: (a -> a) -> [a] -> [a]` eles teriam que ser, _necessariamente_, iguais. 

### 7.3.3 Funcionamento da função `map`

A função `map` é uma função de alta ordem que faz exatamente o que a nossa função `transformaTodos` faz, mas ela é mais segura, então procure sempre usá-la ao invés de`transformaTodos`.

Observe o seu funcionamento comparado a `transformaTodos`.

```haskell
>>transformaTodos (*2) [1,2,3,4,5]
[2,4,6,8,10]

>>map (*2) [1,2,3,4,5]
[2,4,6,8,10]
```
## 7.4 Funções anônimas

### 7.4.1 Introdução

Uma função anônima, ou uma função _lambda_, é um tipo de função que foi idealizada pelo matemático estadunidense Alonzo Church e que foi exportada da matemática para o mundo da ciência da computação com o advento das linguagens funcionais. 

No contexto do haskell, existem apenas duas coisas que você precisa saber sobre esse tipo de funções:

1. Elas não tem nome, por isso, elas são ditas _anônimas_ e, uma vez que elas não tem nenhum tipo de identificador associado a elas, não é possível chamá-las multiplas vezes.<br><br>Portanto, todas as vezes que você desejar usar a mesma função anônima em diferentes partes do seu programa, você deve redeclará-la, logo é bastante nítido que a utilização de funções anônimas só será benéfico em casos pontuais.<br><br>Por causa disto, o uso mais comum das funções lambda é no interior de outras funções para que elas possam servir como funções auxiliares.

1. No trabalho original de Alonzo Church, as funções lambda eram expressas pela letra grega $\lambda$. <br><br>Deste modo, se você quisesse uma função que multiplicasse um número por 2, você precisaria fazer algo _parecido_ com isto: $\lambda x= x \times 2$. Você pode interpretar que "$\lambda$" é o símbolo utilizado para declarar uma função anônima e que "$x$" é o argumento que ela recebe. <br><br> Como seria muito inconveniente ter que usar um caractere tão diferente quanto o "$\lambda$" para programar, os criadores do haskell optaram por expressá-las pelo caractere "\\", pois ele lembra um pouco um "$\lambda$" sem uma das pernas.

### 7.4.2 Exemplos

Funções lambda são muito usadas como auxiliares de funções de alta ordem, como o map. Observe este exemplo simples:

```haskell
1. map (\x -> x*2) [1,2,3,4,5]
>> [2,4,6,8,10]
```
Neste exemplo a função `map` recebe uma função lambda (a qual é declarada por "\\") que, por sua vez, recebe como argumento um parâmetro `x` e retorna como output `x*2`. Perceba que a seta (`->`) representa o retorna da função _lambda_.

Uma vez que a função map está sendo aplicada sobre a lista `[1,2,3,4,5]`, o `x` da primeira "iteração" será `1`, o `x` da segunda, será `2` e assim será até $5$.

Caso quiséssemos reutilizar esta operação várias vezes, com várias listas diferentes e deixá-la ligeiramente mais genérica, poderíamos fazer o seguinte:

```haskell
1. operateInAll :: Num a => a -> [a] -> [a] 
2. operateInAll multiplicand _list = map (\x->x*multiplicand) _list
```

E, se realmente quiséssemos desbloquear todo o potencial do haskell, poderíamos ir além:

```haskell
1. operateInAll :: Num a => (a -> a -> a) -> a -> [a] -> [a]
2. operateInAll operation operand _list = map (\x-> (operation) x operand) _list
```

```haskell
>>operateInAll (+) 2 [1,2,3,4]
[3,4,5,6]

>>operateInAll (-) 2 [1,2,3,4]
[-1,0,1,2]

>>operateInAll (*) 2 [1,2,3,4]
[2,4,6,8]

>> operateInAll (/) 2 [1,2,3,4]
[0.5,1.0,1.5,2.0]
```

Até agora só tivemos diversão e jogos, mas poderíamos deixar tudo isso mais emocianante se fizermos uma pergunta simples:

> Como declarar uma função lambda com mais de um parâmetro?

Que bom que você perguntou! Realmente não existe muito mistério, pois basta que declaremos todos os parâmetros que serão passados a função anônima após a barra invertida (`\`) e implementar a função normalmente após a seta (`->`). Vejamos um exemplo:

```haskell
1. add :: Num a => a -> a -> a
2. add = \a b -> a+b
```

No entanto, no contexto da função `map`, se quiséssemos usar uma função lambda $\lambda$ que operasse sobre mais de um valor, teríamos que passar uma tupla como argumento para $\lambda$, além de fornecer uma lista de tuplas para `map`. Isto se deve pelo fato do `map` aplicar a operação de $\lambda$ em cada elemento da lista e, uma vez que, obviamente, cada elemento é unário, caso não estivéssemos trabalhando com tuplas, não seriam fornecidos argumentos suficientes para $\lambda$.
Observe o exemplo abaixo para entender melhor:

```haskell
1. calculateDiscriminants :: Num a => [(a,a,a)] -> [a]
2. calculateDiscriminants tupleList= map (\(a,b,c) -> b*b - 4*a*c) tupleList
```
```haskell
>> calculateDiscriminants [(1,3,1), (1,5,6),(2,4,1), (1,6,5)]
[5,1,8,16]
```

Redudante dizer, mas, apesar de só termos mostrado exemplos numéricos até agora, é possível usar funções anônimas que trabalham com outros tipos, como strings, por exemplo.

```haskell
1.transformWords someWords = map (\xs -> 'r':xs) someWords
```

```haskell
>>transformWords ["ato","iso","amo"]
["rato","riso","ramo"]
```

> **NOTE**: Como as funções anônimas carecem de uma assinatura para denotar os tipos dos seus parâmetros, o haskell se encarrega de inferí-los baseando-se nas operações realizadas sobre eles.

___
## Referências

1. Apprenez Haskell Language, RIP tutorial, 2019, p. 109-110
	- versão web: https://learntutorials.net/fr/haskell/ 
1. Stotts, David. Overview of Lambda Calculus. Disponível em: https://www.cs.unc.edu/~stotts/723/Lambda/overview.html. Acesso em: 17/06/2024.

## Sugestões de leitura

1. https://stackoverflow.com/questions/75724067/are-all-curried-functions-considered-higher-order-functions
