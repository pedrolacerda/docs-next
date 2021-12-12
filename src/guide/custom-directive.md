# Diretivas customizadas (Custom Directives)

## Introdução

Fora o conjunto de diretivas padrão existente por padrão no core (tipo `v-model` ou `v-show`), o Vue também possibilita registrar diretivas customizadas. Note que no Vue, a forma primária de reuso e abstração de código são os componentes. No entanto, pode haver casos em que você precise um acesso de nível mais baixo aos elementos no DOM, e é aqui que as diretivas customizadas são úteis. Um exemplo seria acionar o foco em um elemento de input, como esse:

<common-codepen-snippet title="Diretivas customizadas: exemplo básico" slug="JjdxaJW" :preview="false" />

Quando a página carregar , o elemento é focado (nota: o `autofocus` não funciona no Safari de mobile). Na verdade, se você não clicou em nada desde que visitou essa página, o input acima deve estar focado agora. Além disso, você pode clicar no botão`Rerun` e o input vai ganhar foco.

Agora vamos construir a diretiva que faz isso:

```js
const app = Vue.createApp({})
// Registar uma diretiva customizada e global chamada `v-focus`
app.directive('focus', {
  // Quando o elemento é montado ao DOM
  mounted(el) {
    // Foca o elemento
    el.focus()
  }
})
```
Se você quer registrar uma diretiva local, os componentes também aceitam a opção `directives`:

```js
directives: {
  focus: {
    // definição da diretiva
    mounted(el) {
      el.focus()
    }
  }
}
```

Então no template, você pode usar o novo atributo `v-focus` em qualquer elemento, tipo assim:

```html
<input v-focus />
```

## Funções de hook (Hook Functions)

As funções de hook são as funções que são executadas conforme o estado da diretiva e há uma variedade disponível para uso (todas opcionais):

- `created`: called before the bound element's attributes or event listeners are applied. This is useful in cases where the directive needs to attach event listeners that must be called before normal `v-on` event listeners.

- `beforeMount`: Executa quando a diretiva é ligada pela primeira vez ao elemento e antes que o componente pai seja montado. Aí é onde você pode fazer o trabalho de configuração inicial.

- `mounted`: Executa quando o componente pai do elemento ligado é montado.

- `beforeUpdate`: Executa antes que os VNode contido no componente são atualizados.

> Nota:
> Vamos cobrir VNodes com mais detalhes [depois](render-function.html#the-virtual-dom-tree), quando discutirmos as funções de renderização (render functions).

- `updated`: Executado após os VNodes contidos no componente **e os filhos desses VNodes** terem sido atualizados.

- `beforeUnmount`: Executado antes do pai dos elementos ligados sejam desmontados.

- `unmounted`: Executado apenas uma vez, quando a diretiva é desligada do elemento e o componente pai é desmontado

Você pode checar os argumentos que foram passados para esses hooks (ex: `el`, `binding`, `vnode` e `prevNode`) em [API de diretivas customizadas](../api/application-api.html#directive)

### Argumentos dinâmicos da diretiva

Os argumentos da diretiva podem ser dinâmicos. Por exemplo, em `v-minhadiretiva:[argumento]="valor"`, o `argumento` pode ser atualizado baseada nas propriedades de dados na nossa instância do componente! Isso faz nossas diretivas customizadas flexíveis para utilizar na aplicação.

Digamos que você queira fazer uma diretiva customizada que permite "pregar" elementos na sua página utilizando posicionamento fixo. Nós poderiamos criar uma diretiva customizada onde o valor atualiza a posição vertical em pixels, desse jeito:

```vue-html
<div id="exemplo-argumentos-dinamicos" class="demo">
  <p>Role para baixo</p>
  <p v-pin="200">Me pregue 200px do topo da página</p>
</div>
```

```js
const app = Vue.createApp({})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.value é o valor que vamos passar para a diretiva - nesse caso, é 200.
    el.style.top = binding.value + 'px'
  }
})

app.mount('#exemplo-argumentos-dinamicos')
```

Isso iria pregar o elemento 200 px do topo da página. Mas o que acontece quando encontramos um cenário que precisamos pregar um elemento da esquerda, ao invés do topo? É aqui que os argumentos dinâmicos que podem ser atualizados por instância de componente são úteis:

```vue-html
<div id="exemplodinamico">
  <h3>Role para baixo nessa seção ↓</h3>
  <p v-pin:[direction]="200">Estou pregado na página a 200 px para a esquerda.</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      direction: 'right'
    }
  }
})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.arg é um argumento que passamos para a diretiva
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})

app.mount('#exemplo-argumentos-dinamicos')
```

Resultado:

<common-codepen-snippet title="Diretivas customizadas: argumentos dinâmicos" slug="YzXgGmv" :preview="false" />

A nossa diretiva customizada agora está flexível o suficiente para atender a vários casos diferentes. Para deixá-lo mais dinâmico, podemos possibilitar alterar um valor ligado. Vamos criar uma propriedade adicional `pinPadding` e ligar ao `<input type="range">`

```vue-html{4}
<div id="exemplodinamico">
  <h2>Role a página para baixo</h2>
  <input type="range" min="0" max="500" v-model="pinPadding">
  <p v-pin:[direction]="pinPadding">Me pregue {{ pinPadding + 'px' }} da {{ direction || 'top' }} da página</p>
</div>
```

```js{5}
const app = Vue.createApp({
  data() {
    return {
      direction: 'right',
      pinPadding: 200
    }
  }
})
```

Agora vamos incrementar a lógica da diretiva para recalcular a distância dos elementos pregados quando atualizar o componente:

```js{7-10}
app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  },
  updated(el, binding) {
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})
```

Resultado:

<common-codepen-snippet title="Diretivas customizadas: argumentos dinâmicos + vínculo dinâmico" slug="rNOaZpj" :preview="false" />

## Atalho de função (Function Shorthand)

No exemplo anterior, você pode querer o mesmo comportamento no `mounted` e `updated`, mas não liga para os outros hooks. Você pode fazer isso passando a callback para a diretiva:

```js
app.directive('pin', (el, binding) => {
  el.style.position = 'fixed'
  const s = binding.arg || 'top'
  el.style[s] = binding.value + 'px'
})
```

## Objetos literais (Object Literals)

Se a sua diretiva precisa de múltiplos valores, você também pode passar um objeto literal do javascript. Lembre-se que as diretivas pode receber qualquer expressão válida em javascript.

```vue-html
<div v-demo="{ color: 'white', text: 'ola!!' }"></div>
```

```js
app.directive('demo', (el, binding) => {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text) // => "ola!"
})
```

## Utilização nos componentes

When used on components, custom directive will always apply to component's root node, similarly to [non-prop attributes](component-attrs.html).

```vue-html
<my-component v-demo="test"></my-component>
```

```js
app.component('my-component', {
  template: `
    <div> // v-demo directive will be applied here
      <span>My component content</span>
    </div>
  `
})
```

Unlike attributes, directives can't be passed to a different element with `v-bind="$attrs"`.

With [fragments](/guide/migration/fragments.html#overview) support, components can potentially have more than one root nodes. When applied to a multi-root component, directive will be ignored and the warning will be thrown.
