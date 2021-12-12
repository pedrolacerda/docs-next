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

console.log(sum) // Still 5
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

But how do we tell Vue about this function?

Vue keeps track of which function is currently running by using an *effect*. An effect is a wrapper around the function that initiates tracking just before the function is called. Vue knows which effect is running at any given point and can run it again when required.

To understand that better, let's try to implement something similar ourselves, without Vue, to see how it might work.

What we need is something that can wrap our sum, like this:

```js
createEffect(() => {
  sum = val1 + val2
})
```

We need `createEffect` to keep track of when the sum is running. We might implement it something like this:

```js
// Maintain a stack of running effects
const runningEffects = []

const createEffect = fn => {
  // Wrap the passed fn in an effect function
  const effect = () => {
    runningEffects.push(effect)
    fn()
    runningEffects.pop()
  }

  // Automatically run the effect immediately
  effect()
}
```

When our effect is called it pushes itself onto the `runningEffects` array, before calling `fn`. Anything that needs to know which effect is currently running can check that array.

Effects act as the starting point for many key features. For example, both component rendering and computed properties use effects internally. Any time something magically responds to data changes you can be pretty sure it has been wrapped in an effect.

While Vue's public API doesn't include any way to create an effect directly, it does expose a function called `watchEffect` that behaves a lot like the `createEffect` function from our example. We'll discuss that in more detail [later in the guide](/guide/reactivity-computed-watchers.html#watcheffect).

But knowing what code is running is just one part of the puzzle. How does Vue know what values the effect uses and how does it know when they change?

## Como o Vue Rastreia Essas Mudanças

Não podemos rastrear reatribuições de variáveis ​​locais como aquelas em nossos exemplos anteriores, simplesmente não há mecanismo para fazer isso em JavaScript. O que podemos rastrear são as mudanças nas propriedades do objeto.

Quando retornamos um objeto JavaScript simples da função de `data` de um componente, O Vue envolverá esse objeto em um [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) com manipuladores para `get` e `set`. Os proxies foram introduzidos no ES6 e permitem que o Vue 3 evite algumas das limitações de reatividade que existiam nas versões anteriores do Vue.

<div class="reactivecontent">
  <common-codepen-snippet title="Proxies e a Reatividade do Vue Explicados Visualmente" slug="VwmxZXJ" tab="result" theme="light" :height="500" :editable="false" :preview="false" />
</div>

A explicação anterior foi breve e requer algum conhecimento de [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) para ser entendida! Então vamos nos aprofundar um pouco. Há muita literatura sobre Proxies, mas o que você realmente precisa saber é que um **Proxy é um objeto que encapsula um outro objeto ou função e permite que você o intercepte.**

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

The object returned by `data` will be wrapped in a reactive proxy and stored as `this.$data`. The properties `this.val1` and `this.val2` are aliases for `this.$data.val1` and `this.$data.val2` respectively, so they go through the same proxy.

Vue will wrap the function for `sum` in an effect. When we try to access `this.sum`, it will run that effect to calculate the value. The reactive proxy around `$data` will track that the properties `val1` and `val2` were read while that effect is running.

As of Vue 3, our reactivity is now available in a [separate package](https://github.com/vuejs/vue-next/tree/master/packages/reactivity). The function that wraps `$data` in a proxy is called [`reactive`](/api/basic-reactivity.html#reactive). We can call this directly ourselves, allowing us to wrap an object in a reactive proxy without needing to use a component:

```js
const proxy = reactive({
  val1: 2,
  val2: 3
})
```

We'll explore the functionality exposed by the reactivity package over the course of the next few pages of this guide. That includes functions like `reactive` and `watchEffect` that we've already met, as well as ways to use other reactivity features, such as `computed` and `watch`, without needing to create a component.

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

## How Rendering Reacts to Changes

The template for a component is compiled down into a [`render`](/guide/render-function.html) function. The `render` function creates the [VNodes](/guide/render-function.html#the-virtual-dom-tree) that describe how the component should be rendered. It is wrapped in an effect, allowing Vue to track the properties that are 'touched' while it is running.

A `render` function is conceptually very similar to a `computed` property. Vue doesn't track exactly how dependencies are used, it only knows that they were used at some point while the function was running. If any of those properties subsequently changes, it will trigger the effect to run again, re-running the `render` function to generate new VNodes. These are then used to make the necessary changes to the DOM.

<div class="reactivecontent">
  <common-codepen-snippet title="Second Reactivity with Proxies in Vue 3 Explainer" slug="wvgqyJK" tab="result" theme="light" :height="500" :editable="false" :preview="false" />
</div>

> Se você está utilizando Vue v2.x ou mais antigo, pode estar interessado em alguns empecilhos da detecção de mudanças que existem nessas versões, [explorados em mais detalhes aqui](change-detection.md).
