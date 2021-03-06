# Uma Suave Introdução ao JavaScript Funcional: Parte 4

* **Artigo Original**: [A GENTLE INTRODUCTION TO FUNCTIONAL JAVASCRIPT: PART 4](http://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-style/)
* **Tradução**: [Eric Douglas](https://github.com/ericdouglas)

> *Escrito por James Sinclair em 11 de Fevereiro de 2016* 

Essa é a parte 4 de uma série de 4 artigos sobre introdução a programação funcional no JavaScript. No último artigo vamos ver sobre *high-order functions* (funções de ordem superior): funções para criar funções. Neste artigo, nós discutimos como usar essas novas ferramentas com estilo. 

- [Parte 1: Blocos fundamentais e motivação](009-uma-suave-introducao-ao-javascript-parte-1.md)
- [Parte 2: Trabalhando com Arrays e Listas](010-uma-suave-introducao-ao-javascript-parte-2.md)
- [Parte 3: Funções para fazer funções](011-uma-suave-introducao-ao-javascript-parte-3.md)
- Parte 4: Fazendo isso com estilo (esse artigo)

## Fazendo Isso Com Estilo
No artigo anterior vimos sobre `partial`, `compoese`, `curry`, `pipe`, e também como podemos usá-las para juntar pequenas, simples funções e criar outras grandes e complicadas. Mas o que isso faz pra gente? Vale a pena se importar quando já estamos escrevendo código perfeitamente válido?

Parte da resposta é que sempre é útil ter mais ferramentas disponíveis para realizar o trabalho - desde que você saiba como usá-las - e programação funcional certamente nos dá um conjunto útil de ferramentas para escrever JavaScript. Mas eu penso que há mais do que isso. Programação Funcional nos abre um diferente *estilo* de programação. Isso por sua vez nos permite conceitualizar problemas e soluções de formas diferentes.

Existem duas funcionalidades chave para programação funcional:

1. Escrever funções puras, o que é importante se você deseja testar programação funcional; e
1. Estilo de programação *pointfree* (sem pontos), o que não é *tão* importante mas bom de se entender.

### Pureza
Se você ler sobre programação funcional, você eventualmente vai se deparar com o conceito de funções *puras* e *impuras*. Funções puras são funções que preenchem dois critérios:

1. Chamar a função com as mesmas entradas sempre *retorna* as mesmas saídas.
1. Chamar a função não produz efeitos colaterais: Sem chamadas na rede (*network*); nenhuma leitura ou escrita de arquivos; nenhuma consulta em banco de dados; nenhum elemento DOM modificado; nenhuma modificação de variáveis globais; e nenhuma saída no console. Nada.

Funções impuras deixam programadores funcionais desconfortáveis. Tão desconfortáveis que eles as evitam o máximo possível. O problema com isso é que o grande ponto de escrever programas de computador *são* os efeitos colaterais. Fazer uma chamada na rede e renderizar elementos DOM está no núcleo do que uma aplicação web faz; esse foi o motivo pelo qual o JavaScript foi inventado.

Então o que um aspirante a programador funcional faz? Bom, a chave é que nós não evitamos funções impuras inteiramente, nós apenas damos a elas uma boa quantidade de respeito, e deixá-mos de lidar com elas até que nós absolutamente precisemos. Elaboramos um plano claro e testado para o que queremos fazer *antes* de tentar fazer isso. Como Eric Elliot colocou em [*The Dao of Immutability*](https://medium.com/javascript-scene/the-dao-of-immutability-9f91a70c88cd):

> **Separação:** Lógica é pensamento. Efeitos são ações. Portanto, o sábio pensa antes de agir, e age somente quando o pensamento está terminado. Se você tentar executar efeitos e lógica ao mesmo tempo, você vai criar efeitos colaterais ocultos que causam *bugs* (problemas) na lógica. Mantenha as funções pequenas. Faça uma coisa de cada vez, e faça isso bem.

Em outras palavras, com programação funcional, nós geralmente trabalhamos a lógica do que estamos tentando fazer primeiro, antes de fazer qualquer coisa que potencialmente tenha efeitos colaterais.

Outra forma de pensar sobre isso é a diferença de usar uma metralhadora e um *rifle sniper*. Com a metralhadora você dispara o máximo de balas possível, contando com o fato que se você continuar atirando, eventualmente você vai acertar algo. Mas você pode também acertar coisas que você não queria. Porém um rifle sniper é direfente. Você escolhe o lugar mais vantajoso, alinha o tiro, toma em conta a velocidade do vento e a distância do alvo. Você pacientemente, metódicamente, cuidadosamente configura todas as coisas e no momento certo, puxa o gatilho. Muito menos munição, e um efeito muito mais preciso.

Então como tornamos nossas funções puras? Vamos ver um exemplo:

```js
var myGlobalMessage = '{{verb}} me';

var impureInstruction = function(verb) {
    return myGlobalMessage.replace('{{verb}}', verb);
}

var eatMe = impureInstruction('Eat');
//=> 'Eat me'
var drinkMe = impureInstruction('Drink');
//=> 'Drink me'
```

Essa função é impura pois depende da variável global `myGlobalMessage`. Se a variável mudar, se torna difícil dizer o que `impureInstruction` vai fazer. Uma forma de torná-la pura é mover a variável para dentro:

```js
var pureInstruction = function (verb) {
    var message =  '{{verb}} me';
    return message.replace('{{verb}}', verb);
}
```

Essa função agora vai sempre retornar o mesmo resultado dado um mesmo conjunto de entradas. Mas algumas vezes não podemos usar essa ténica. Por exemplo:

```js
var getHTMLImpure = function(id) {
    var el = document.getElementById(id);
    return el.innerHTML;
}
```

Essa função é impura pois depende do objeto `document` para acessar o DOM. Se o DOM mudar isso *pode* produzir resultados diferentes. Mas nós não podemos definir `document` dentro de nossa função porque isso é uma API do navegador, mas nós *podemos* passar isso como um parâmetro:

```js
var getHTML = function(doc, id) {
    var el = doc.getElementById(id);
    return el.innerHTML;
}
```

Isso pode parecer trivial e sem sentido, mas é uma técnica útil. Imagine você estava tentando fazer um teste unitário nessa função. Normalmente, teríamos que configurar algum tipo de browser para pegar o objeto `document` e poder testar isso. Uma vez que temos o parâmetro `doc`, é fácil passar um objeto *stub* - uma simulação do objeto real - ao invés disso:

```js
var stubDoc = {
    getElementById: function(id) {
        if (id === 'jabberwocky') {
            return {
                innerHTML: '<p>Twas brillig…'
            };
        }
    }
};

assert.equal(getHTML('jabberwocky'), '<p>Twas brillig…');
//=> test passes
```

Escrever esse *stub* pode parecer um pouco trabalhoso, mas agora podemos testar essa função sem precisar de um navegador. Se quisermos, podemos rodar isso na linha de comando sem um navegador *headless*. E como um bônus, o teste vai rodar muito, muito mais rápido do que um que tenho o objeto `document` inteiro.

Uma outra forma de ter uma função pura é retornar outra função que eventualmente irá fazer algo impuro quando a chamarmos. Isso inicialmente parece um truque sujo, mas é totalmente legítimo. Por exemplo:

```js
var htmlGetter = function(id) {
    return function() {
        var el = document.getElementById(id);
        return el.innerHTML;
    }
}
```

 A função `htmlGetter` é pura porque rodando-a não acessamos uma variável global - ao invés disso, ela sempre retorna exatamente a mesma função.

 Fazer coisas dessa forma não é muito útil para testes unitários, e isso não remove a impureza completamente - apenas a posterga. E isso não é necessariamente uma coisa ruim. Lembre-se, queremos lidar com toda a lógica previamente nas funções puras e depois puxar o gatilho em quaisquer efeitos colaterais.

### *Pointfree* (sem pontos)
Programação *Pointfree* ou programação *tácita* é um estilo particular de programação que funções de ordem superior como `curry` e `compose` tornam possível. Para explicar isso, vamos olhar novamente para o exemplo do poema do último artigo:

> **Nota do tradutor:** Programação tácita, também chamada estilo **point-free** (sem pontos), é um paradigma de programação em que definições de função não identificam os argumentos (ou "pontos") em que elas opearam. Ao invés disso, as definições meramente compõem outras funções, onde essas são combinadores que manipulam os argumentos. [Fonte](https://en.wikipedia.org/wiki/Tacit_programming).

```js
var poem = 'Twas brillig, and the slithy toves\n' + 
    'Did gyre and gimble in the wabe;\n' +
    'All mimsy were the borogoves,\n' +
    'And the mome raths outgrabe.';

var replace = curry(function(find, replacement, str) {
    var regex = new RegExp(find, 'g');
    return str.replace(regex, replacement);
});

var wrapWith = curry(function(tag, str) {
    return '<' + tag + '>' + str + '</' + tag + '>'; 
});

var addBreaks      = replace('\n', '<br/>\n');
var replaceBrillig = replace('brillig', wrapWith('em', 'four o’clock in the afternoon'));
var wrapP          = wrapWith('p');
var wrapBlockquote = wrapWith('blockquote');

var modifyPoem = compose(wrapBlockquote, wrapP, addBreaks, replaceBrillig);
```

Note que `compose` espera que cada função passada receba exatamente um parâmetro. So, we use curry to change
