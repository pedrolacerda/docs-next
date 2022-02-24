# Aprofundando-se na Reatividade

Agora é o momento de mergulhar fundo! Uma das características mais distintas do Vue é o seu discreto sistema de reatividade. Os modelos de dados são _proxies_ de objetos JavaScript. Quando você os modifica, a _view_ é atualizada. Isso faz com que a administração de estado seja simples e intuitiva, mas também é importante entender como isso funciona para evitar algumas pegadinhas. Nesta seção, vamos nos aprofundar em alguns dos detalhes de baixo nível do sistema de reatividade do Vue.

<VideoLesson href="https://www.vuemastery.com/courses/vue-3-reactivity/vue3-reactivity" title="Aprenda a fundo como a reatividade funciona com o Vue Mastery">Assista um vídeo gratuito sobre Aprofundando-se na Reatividade no Vue Mastery</VideoLesson>

## O Que é Reatividade?

Esse termo aparece na programação com uma certa frequência atualmente, mas o que realmente significa quando as pessoas o dizem? Reatividade é um paradigma da programação que permite nos ajustarmos à mudanças de uma maneira declarativa. O exemplo canônico geralmente mostrado, por ser ótimo, é uma planilha do Excel.

<video width="550" height="400" controls>
  <source src="/images/reactivity-spreadsheet.mp4" type="video/mp4">
  Seu navegador não possui suporte para a tag video.
</video>

Se você colocar o número 2 na primeira célula, e o número 3 na segunda e então utilizar o SUM, a planilha te dará o resultado. Até aqui nada demais. Mas se você atualizar o primeiro número, o SUM é automagicamente atualizado.

O JavaScript, geralmente, não funciona assim. Se escrevêssemos algo semelhante em Javascript:

```js
let val1 = 2
let val2 = 3
let sum = val1 + val2

console.log(sum) // 5

val1 = 3

console.log(sum) // Ainda 5
```

Ao atualizarmos o primeiro valor, a soma não é ajustada.

Então, como faríamos isso em JavaScript?

Como uma visão geral de alto nível, existem algumas coisas que precisamos ser capazes de fazer:

1. **Rastrear quando um valor é lido.** ex: `val1 + val2` lê `val1` e `val2`.
2. **Detectar quando um valor muda.** ex: Quando atribuímos `val1 = 3`.
3. **Execute novamente o código que leu o valor originalmente.** ex: Execute `sum = val1 + val2` novamente para atualizar o valor de `sum`.

Não podemos fazer isso diretamente usando o código do exemplo anterior, mas voltaremos a este exemplo mais tarde para ver como adaptá-lo para ser compatível com o sistema de reatividade do Vue.

Primeiro, vamos nos aprofundar um pouco mais em como o Vue implementa os requisitos básicos de reatividade descritos acima.

## Como Vue Sabe o Código Executando

Para poder executar nossa soma sempre que os valores mudarem, a primeira coisa que precisamos fazer é envolvê-la em uma função:

```js
const updateSum = () => {
  sum = val1 + val2
}
```

Mas como informamos o Vue sobre essa função?

O Vue mantém o controle de qual função está sendo executada usando um *efeito*. Um efeito é um _wrapper_ em torno da função que inicia o rastreamento logo antes de a função ser chamada. O Vue sabe qual efeito está sendo executado em um determinado ponto e pode executá-lo novamente quando necessário.

Para entender melhor, vamos tentar implementar algo semelhante nós mesmos, sem o Vue, para ver como pode funcionar.

O que precisamos é de algo que possa envolver nossa soma, assim:

```js
createEffect(() => {
  sum = val1 + val2
})
```

Precisamos de `createEffect` para acompanhar quando a soma está sendo executada. Podemos implementar algo assim:

```js
// Mantém uma pilha de efeitos em execução
const runningEffects = []

const createEffect = fn => {
  // Envolve o fn passado em uma função de efeito
  const effect = () => {
    runningEffects.push(effect)
    fn()
    runningEffects.pop()
  }

  // Executa automaticamente o efeito imediatamente
  effect()
}
```

Quando nosso efeito é chamado, ele se adiciona ao array `runningEffects`, antes de chamar `fn`. Qualquer coisa que precise saber qual efeito está sendo executado no momento pode verificar esse array.

Os efeitos atuam como ponto de partida para muitos recursos importantes. Por exemplo, tanto a renderização do componente quanto os dados computados usam efeitos internamente. Sempre que algo responde magicamente a alterações de dados, você pode ter certeza de que foi envolvido em um efeito.

Embora a API pública do Vue não inclua nenhuma maneira de criar um efeito diretamente, ela expõe uma função chamada `watchEffect` que se comporta muito como a função `createEffect` do nosso exemplo. Discutiremos isso com mais detalhes [mais adiante no guia](/guide/reactivity-computed-watchers.html#watcheffect).

Mas saber qual código está sendo executado é apenas uma parte do quebra-cabeça. Como o Vue sabe quais valores o efeito usa e como ele sabe quando eles mudam?

## Como o Vue Rastreia Essas Mudanças

Não podemos rastrear reatribuições de variáveis ​​locais como aquelas em nossos exemplos anteriores, simplesmente não há mecanismo para fazer isso em JavaScript. O que podemos rastrear são as mudanças nas propriedades do objeto.

Quando retornamos um objeto JavaScript simples da função de `data` de um componente, O Vue envolverá esse objeto em um [Proxy](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Proxy) com manipuladores para `get` e `set`. Os proxies foram introduzidos no ES6 e permitem que o Vue 3 evite algumas das limitações de reatividade que existiam nas versões anteriores do Vue.

<div class="reactivecontent">
  <common-codepen-snippet title="Proxies e a Reatividade do Vue Explicados Visualmente" slug="VwmxZXJ" tab="result" theme="light" :height="500" :editable="false" :preview="false" />
</div>

A explicação anterior foi breve e requer algum conhecimento de [Proxies](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Proxy) para ser entendida! Então vamos nos aprofundar um pouco. Há muita literatura sobre Proxies, mas o que você realmente precisa saber é que um **Proxy é um objeto que encapsula um outro objeto ou função e permite que você o intercepte.**

Nós o utilizamos assim: `new Proxy(target, handler)`

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property) {
    console.log('interceptado!')
    return target[property]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// interceptado!
// tacos
```

Aqui, interceptamos tentativas de ler propriedades do objeto-alvo. Uma função manipuladora como essa também é conhecida como *armadilha* (*trap*). Existem muitos tipos diferentes de armadilhas disponíveis, cada uma lidando com um tipo diferente de interação.

Indo além de um `console.log`, nós poderíamos fazer aqui qualquer coisa que desejássemos. Poderíamos até mesmo _não_ retornar o valor real se quiséssemos. Isso é o que torna os Proxies tão poderosos para a criação de APIs.

Um desafio de usar um proxy é a vinculação de `this`. Gostaríamos que qualquer método fosse vinculado ao Proxy, em vez do objeto-alvo, para que também possamos interceptá-los. Felizmente, o ES6 introduziu outro novo recurso, chamado `Reflect`, que nos permite fazer esse problema desaparecer com o mínimo de esforço:

```js{7}
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// interceptado!
// tacos
```

A primeira etapa para implementar reatividade com um proxy é controlar quando uma propriedade é lida. Fazemos isso no manipulador, em uma função chamada `track`, onde passamos `target` e `property`:

```js{7}
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property, receiver) {
    track(target, property)
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// interceptado!
// tacos
```

A implementação de `track` não é mostrada aqui. Ele verificará qual *efeito* está sendo executado e registrará junto de `target` e `property`. É assim que Vue sabe que a propriedade é uma dependência do efeito.

Finalmente, precisamos executar novamente o efeito quando o valor da propriedade mudar. Para isso, vamos precisar de um manipulador `set` em nosso *proxy*:

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property, receiver) {
    track(target, property)
    return Reflect.get(...arguments)
  },
  set(target, property, value, receiver) {
    trigger(target, property)
    return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// interceptado!
// tacos
```

Lembra desta lista de anteriormente? Agora temos algumas respostas sobre como o Vue implementa essas etapas principais:

1. **Rastrear quando um valor for lido**: a função `track` no manipulador `get` do *proxy* registra a propriedade e o efeito atual.
2. **Detectar quando esse valor mudar**: o manipulador `set` é chamado no *proxy*.
3. **Execute novamente o código que leu o valor originalmente:** a função `trigger` procura quais efeitos dependem da propriedade e os executa.

O objeto com o _proxy_ aplicado é invisível para o usuário, mas por baixo dos panos ele possibilita que o Vue faça o rastreamento-de-dependência (_dependency-tracking_) e a notificação-de-mudança (_change-notification_) quando propriedades são acessadas ou modificadas. Um problema é que o _console_ de navegadores formatam diferentemente quando objetos de dados convertidos são registrados no _log_, então pode ser que você queria instalar o [vue-devtools](https://github.com/vuejs/vue-devtools) para uma interface mais amigável à inspeção.

Se tivéssemos de reescrever nosso exemplo original usando um componente, poderíamos fazer algo assim:

```js
const vm = createApp({
  data() {
    return {
      val1: 2,
      val2: 3
    }
  },
  computed: {
    sum() {
      return this.val1 + this.val2
    }
  }
}).mount('#app')

console.log(vm.sum) // 5

vm.val1 = 3

console.log(vm.sum) // 6
```

O objeto retornado por `data` será encapsulado em um proxy reativo e armazenado como `this.$data`. As propriedades `this.val1` e `this.val2` são apelidos para `this.$data.val1` e `this.$data.val2` respectivamente, então elas passam pelo mesmo proxy.

Vue envolverá a função para `sum` em um efeito. Quando tentamos acessar `this.sum`, ele executará esse efeito para calcular o valor. O proxy reativo em torno de `$data` rastreará se as propriedades `val1` e `val2` foram lidas enquanto o efeito está sendo executado.

A partir do Vue 3, nossa reatividade agora está disponível em um [pacote separado](https://github.com/vuejs/vue-next/tree/master/packages/reactivity). A função que envolve `$data` em um proxy é chamada de [`reactive`](/api/basic-reactivity.html#reactive). Podemos chamar isso diretamente nós mesmos, permitindo-nos envolver um objeto em um proxy reativo sem precisar usar um componente:

```js
const proxy = reactive({
  val1: 2,
  val2: 3
})
```

Exploraremos a funcionalidade exposta pelo pacote de reatividade ao longo das próximas páginas deste guia. Isso inclui funções como `reactive` e `watchEffect` que já conhecemos, bem como formas de usar outros recursos de reatividade, como `computed` e `watch`, sem a necessidade de criar um componente.

### Objetos com Proxy Aplicado

O Vue rastreia internamente todos os objetos que foram transformados em reativos, então ele sempre retorna o mesmo proxy de um mesmo objeto.

Quando um objeto aninhado é acessado através de um proxy reativo, esse objeto é _também_ convertido em um proxy antes de ser retornado:

```js{6-7}
const handler = {
  get(target, property, receiver) {
    track(target, property)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      // Envolve o objeto aninhado em seu próprio proxy reativo
      return reactive(value)
    } else {
      return value
    }
  }
  // ...
}
```

### Proxy vs. Identidade Original

O uso do Proxy de fato introduz um novo empecilho a ser considerado: o objeto com o *proxy* aplicado não é igual ao objeto original em termos de comparação de identidade (`===`). Por exemplo:

```js
const obj = {}
const wrapped = new Proxy(obj, handlers)

console.log(obj === wrapped) // false
```

Outras operações que dependem de comparações de igualdade estritas também podem ser afetadas, como `.includes()` ou `.indexOf()`.

A prática recomendada aqui é nunca manter uma referência ao objeto bruto original e trabalhar apenas com a versão reativa:

```js
const obj = reactive({
  count: 0
}) // nenhuma referência ao original
```

Isso garante que as comparações de igualdade e reatividade se comportem conforme o esperado.

Observe que o Vue não envolve valores primitivos, como números ou strings em um *proxy*, então você ainda pode usar `===` diretamente com esses valores:

```js
const obj = reactive({
  count: 0
})

console.log(obj.count === 0) // true
```

## Observadores

Toda instância de componente tem uma instância de observador correspondente, que registra quaisquer propriedades "tocadas" durante a renderização do componente como dependências. Depois, quando um setter de uma dependência é disparado, ele notifica o observador, que por sua vez faz o componente re-renderizar.

## Como a Renderização Reage às Mudanças

O _template_ para um componente é compilado em uma função [`render`](/guide/render-function.html). A função `render` cria os [VNodes](/guide/render-function.html#a-arvore-virtual-dom) que descrevem como o componente deve ser renderizado. Ele é envolvido em um efeito, permitindo que o Vue rastreie as propriedades que são 'tocadas' enquanto ele está em execução.

Uma função `render` é conceitualmente muito semelhante a uma propriedade `computed`. O Vue não rastreia exatamente como as dependências são usadas, ele apenas sabe que elas foram usadas em algum momento enquanto a função estava em execução. Se qualquer uma dessas propriedades for alterada posteriormente, o efeito será acionado novamente, executando novamente a função `render` para gerar novos VNodes. Estes são então usados ​​para fazer as alterações necessárias no DOM.

<div class="reactivecontent">
  <common-codepen-snippet title="Segundo Explicador da Reatividade com Proxies no Vue 3" slug="wvgqyJK" tab="result" theme="light" :height="500" :editable="false" :preview="false" />
</div>

> Se você está utilizando Vue v2.x ou mais antigo, pode estar interessado em alguns empecilhos da detecção de mudanças que existem nessas versões, [explorados em mais detalhes aqui](change-detection.md).
