#  Capítulo 6

## 6.1 Funções curry e funções parcialmente aplicadas

### 6.1.1 Intuição

Antes de te apresentar propriamente o que são as funções curry e como elas se comportam, gostaria que fizéssemos um pequeno exercício mental: suponha que, por algum motivo que foge da realidade do brasileiro moderno, você queira sacar dinheiro físico em um caixa eletrônico. Caso você não tenha vivido na época das cavernas, permita-me explicar como este procedimento funciona: tudo o que é necessário fazer é providenciar três informações simples ao caixa eletrônico.


1. Seu cartão de crédito, ou seja, você deve inseri-lo no caixa. 
2. A senha do seu cartão.
3. A quantia que você quer sacar.

A partir desta enumeração, podemos fazer uma constatação simples e que pode parecer bastante óbvia: Não podemos fornecer essas informações ao mesmo tempo, mas uma de cada vez e, necessariamente, nesta ordem.

Tudo bem, você agora deve estar inquieto e se perguntando "Tudo bem, chega de caixas eletrônicos,
eu quero o haskell!". Atendendo ao seu pedido, suponha que desejemos implementar uma função  bastante simples para simular o comportamento do nosso caixa!

```haskell
caixaEletronico:: (String, Int, Int) -> Int
caixaEletronico (cartao, senha, quantiaSaque) = quantiaSaque

>>caixaEletronico ("123.456.789.123.4", 123456, 100)
>>100
```
Apesar da função que implementamos estar certa, existe algo intrisicamente contraintuitivo nela: fornecemos todas as três informações ao mesmo tempo através de uma tupla, não sequencialmente, como é de se esperar na vida real!

Então, apesar deste detalhe ser largamente desimportante, seria legal se tivéssemos uma forma de passar essas informaçãos sequencialmente, como ocorre na realidade. Felizmente, o haskell nos permite fazer isto!

A primeira constatação que precisamos fazer é simples: temos uma função ```caixaEletronico``` e três argumentos ```cartao```,```senha```, ```quantiaSaque```. Ora, para podermos inserir um parâmetro de cada vez no caixa bastaria que fizéssemos o seguinte:

1. Forneceríamos ```cartao``` ao ```caixaEletronico``` e ele nos retornaria uma outra função, neste caso, uma que ainda está incompleta, ou seja, que não gera um output definitivo por ter parâmetros insuficientes. Chamemos esta função que seria gerada de ```\fun1```
2. Agora seria necessário que fornecéssemos ```senha``` para ```\fun1```. Se fizéssemos isso, essa função também nos retornaria uma outra função que, assim como a anterior, estaria incompleta. Chamemo-la de ```\fun2```
3. Por sua vez,  ```\fun2``` receberia ```quantiaSaque```, o que faria com que esse ciclo vicioso de geração de funções incompletas acabasse, pois agora que todos os argumentos foram fornecidos, ```\fun2``` é capaz de gerar o output final que, nesse caso, é igual à ```quantiaSaque```.

Pondo de uma forma mais concreta, teríamos algo parecido com isso em cada chamada de parâmetros:

```haskell
caixaEletronico cartao --"caixaEletronico":função "original"; "cartao":parâmetro
(caixaEletronico cartao) senha -- "(caixaEletronico cartao)":fun1;"senha":parâmetro
((caixaEletronico cartao) senha) quantiaSaque --"((caixaEletronico cartao) senha)":fun2; "quantiaSaque":parâmetro
```

Pondo de uma forma que realmente compile, ```caixaEletronico``` poderia ser escrito da seguinte forma:

```haskell
((caixaEletronico cartao) senha) quantiaSaque = quantiaSaque
```

Com certeza você deve estar bem surpreso com o rumo que a nossa conversa sobre caixas eletrônicos tomou não, é? Olhe pelo lado positivo, agora, finalmente, ```caixaEletronico``` funciona de uma maneira intuitiva, pois agora ele recebe as informações sequencialmente, ou seja, uma de cada vez e não de uma vez só através de uma tupla!

E é exatamente isso que uma **função curry** é: uma função que recebe apenas um parâmetro e retorna apenas um valor.

Além disto, se observarmos com bastante cautela a implementação desta última versão de ```caixaEletronico``` conseguimos notar algo interessante: ela é exatamente o equivalente a:

```haskell
caixaEletronico cartao senha quantiaSaque = quantiaSaque
```

Com isso podemos uma coisa muito simples:

- Todas as funções em haskell são funções curry:
    - quando escrevemos a assinatura de uma função como, por exemplo, ```div :: Int -> Int -> Int```, para os olhos destreinados, _pode até parecer_ que ela significa que ```div``` receberá dois ```Int```'s e retornará um valor do tipo ```Int```.
    - No entanto, o que ela realmente diz é que ela receberá um ```Int``` e retornará uma função que recebe um ```Int``` e que, por sua vez, essa função que recebe um ```Int``` produzirá como output um valor do tipo ```Int```
    - Portanto, sem perda de generalidade, poderíamos reescrever a assinatura de ```div``` como ```div :: Int -> (Int -> Int)```
        - Tal reescrita é permitida justamente porque o primeiro ```Int``` é justamente o parâmetro que ```div``` receberá, de tal forma que a seta (```->```) indica o valor que será gerado.
        - Neste caso "```div Int```" retornará uma função, neste caso ```(Int -> Int)```
        - O que é mais curioso sobre a notação "```(Int -> Int)```" é que ela, não somente representa a função retornada por "```div Int```", mas também denota qual é o tipo de valor a própria função retornada . Observe bem:
            - Preste atenção nos valores entre parênteses: ```(Int -> Int)```, o ```Int``` antes da seta aponta que a função retornada receberá um valor inteiro e, de forma análoga, o ```Int``` depois da seta aponta que o valor retornado por ```(Int -> Int)``` é um inteiro.
    - Eu sei, eu sei, eu tagarelei bastante e deixei você por muito tempo com a terrível dúvida de _"Como será possível que uma função que foi feita para trabalhar com _x_ quantidade de parâmetros possa ser capaz de gerar qualquer tipo de output, de tal modo que o seu output seja uma função que receberá o próximo input e assim sucessivamente até que o output final seja produzido ?"_
        - Pois é, eu sei, à princípio, pode ser difícil de aceitar, mas você não deve se preocupar com este tipo de detalhe, pois isto é uma preocupação para o GHC e o GHCI.
        - Portanto, nós, programadores haskell, podemos confiar que o compilador conseguirá receber funções ditas _parcialmente aplicadas_ (ou seja, funções com um número de parâmetros insuficiente para produzir um output definitivo) e, ao longo das etapas de _currying_, conseguirá gerar o resultado final sem problemas

## 6.2 Funções parcialmente aplicadas

A capacidade de poder trabalhar com funções incompletas, isto é, parcialmente aplicadas,  é algo intrinsecamente ligado a programação funcional então é pertinente que investiguemos mais atentamente este recurso.

Para começarmos, suponha que você tenha a seguinte função

```haskell
myPartiallyAppliedFun :: Num a => a -> a 
myPartiallyAppliedFun = (+) 5
```

Perceba que a `myPartiallyAppliedFun` faz uso da função `+`, a qual possui a seguinte assinatura:

```
(+) :: Num a => a -> a -> a
```

Como vimos anteriormente, Esta assinatura poderia ser melhor descrita como:

```
(+) :: Num a => a -> (a -> a)
```

O que é interessante notarmos neste caso é que a assinatura de ```myPartiallyAppliedFun``` é exatamente igual a assinatura da função gerada por ```(+) a```, ou seja, `a -> a` e isto se deve justamente pelo fato de ```myPartiallyAppliedFun``` guardar o output gerado pela função parcialmente aplicada `(+) 5`.

Sendo assim, para usarmos `myPartiallyAppliedFun` bastaria que fizéssemos:

```haskell
myPartiallyAppliedFun 7
>> 12
```

E ao observarmos o tipo de ```myPartiallyAppliedFun 7``` notamos que ele tem a seguinte anotação de tipo (que costumamos chamar de "assinatura")

```haskell
:t (myPartiallyAppliedFun 7)
(myPartiallyAppliedFun 7) :: Num a => a
```

A ausência de setas significa meramente que ele não retorna informação alguma, portanto, está anotação de tipo tão somente denota que a expressão `(myPartiallyAppliedFun 7)` é ```Num```, ou seja, numérica.

Note, porém, que, caso tentássemos chamar `myPartiallyAppliedFun` sem passar nenhum parâmetro a ela, ganharíamos um erro similar a este:

```haskell
<interactive>:y:x: error:
    •No instance for (Show (Integer -> Integer))
        arising from a use of ‘print’
        (maybe you haven't applied a function to enough arguments?)   
    •In a stmt of an interactive GHCi command: print it 
```    

Que nos diz, um tanto timidamente entre parênteses, que "talvez não tenhamos aplicado a função a um número suficiente de argumentos".


## Referências do capítulo

1. Hudak, Paul; Peterson, John; Fasel, Joseph. A Gentle Introduction to Haskell 98. haskell.org, 1999, p. 10.
1. Apprenez la programmation fonctionnelle avec Haskell; 2.ed. Zeste de Savoir, 2019. p. 68
1. Apprenez Haskell Language, RIP tutorial, 2019, p. 134  
1. Sullivan, Bryan; Stewart, Don; Goerzen, John; O'reill2008.Real World Haskell, chapter 4 "Partial function application and currying"
1. https://wiki.haskell.org/Currying
1. https://wiki.haskell.org/Partial_application

### Sugestões de leituras

1. Thomson, Simon. Haskell - The craft of a functional programming language. 3ed. 249-253

### Sugestões de vídeos

1. Curried Functions - Computerphile

