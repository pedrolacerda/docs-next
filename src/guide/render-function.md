# Funções de Renderização

Vue recomenda o uso de _templates_ para construir aplicações na grande maioria dos casos. No entanto, existem situações onde precisamos de todo o poder programático do JavaScript. É aí onde podemos utilizar a **função de renderização**.

Vamos mergulhar em um exemplo onde uma função `render()` seria prática. Digamos que queremos gerar um título ancorados:

```html
<h1>
  <a name="hello-world" href="#hello-world">
    Olá mundo!
  </a>
</h1>
```

Títulos ancorados são usados frequentemente, deveríamos criar um componente:

```vue-html
<anchored-heading :level="1">Olá mundo!</anchored-heading>
```

O componente deve gerar um título baseado na propriedade `level`, e nós rapidamente chegaríamos nisso:

```js
const { createApp } = Vue

const app = createApp({})

app.component('anchored-heading', {
  template: `
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-else-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-else-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-else-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-else-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-else-if="level === 6">
      <slot></slot>
    </h6>
  `,
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

Este _template_ não parece bom. Não apenas é verboso, como também estamos duplicando o `<slot></slot>` para cada nível de título. E quando adicionarmos o elemento de âncora, teríamos que duplicá-lo em cada ramo `v-if/v-else-if`.

Enquanto que _templates_ funcionam muito bem para a maioria dos componentes, fica claro que este não é um deles. Então, vamos tentar reescrevê-lo com uma função `render()`:

```js
const { createApp, h } = Vue

const app = createApp({})

app.component('anchored-heading', {
  render() {
    return h(
      'h' + this.level, // nome da tag
      {}, // propriedades/atributos
      this.$slots.default() // array de filhos
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

A implementação da função `render()` é muito mais simples, mas também requer mais familiaridade com as propriedades das instâncias dos componentes. Nesse caso, você deve saber que quando você passar filhos sem uma diretiva `v-slot` para um componente, como o `Olá mundo!` dentro do `anchored-heading`, esses filhos serão armazenados na instância do componente em `$slots.default()`. Se você já não tiver feito ainda, **é recomendado ler a [API de propriedades de instância](../api/instance-properties.html) antes de mergulhar nas funções de renderização.**

## A Árvore DOM

Antes de mergulharmos nas funções de renderização, é importante conhecer um pouco sobre como os navegadores funcionam. Veja esse HTML como exemplo:

```html
<div>
  <h1>Meu título</h1>
  Algum conteúdo em texto
  <!-- TODO: Adicionar slogan -->
</div>
```

Quando um navegador lê este código, ele compila uma [árvore de "nós DOM"](https://javascript.info/dom-nodes) para ajudá-lo a acompanhar tudo.

A árvore de nós DOM para o HTML acima se parece com isso:

![Visualização da Árvore DOM](/images/dom-tree.png)

Cada elemento é um nó. Cada trecho de texto é um nó. Até mesmo comentários são nós! Cada nó pode possuír filhos (i.e. cada nó pode conter outros nós).

Atualizar todos esses nós eficientemente pode ser difícil, mas felizmente, nós nunca precisamos fazê-lo manualmente. Ao invés disso, nós dizemos ao Vue qual HTML nós queremos na página, em um _template_:

```html
<h1>{{ blogTitle }}</h1>
```

Ou em uma função de renderização:

```js
render() {
  return h('h1', {}, this.blogTitle)
}
```

E em ambos os casos, o Vue automaticamente mantém a página atualizada, até mesmo quando o `blogTitle` muda.

## A Árvore Virtual DOM

Vue mantém a página atualizada compilando um **DOM virtual** para acompanhar as mudanças que necessita para fazer o DOM real. Olhando a seguinte linha mais de perto:

```js
return h('h1', {}, this.blogTitle)
```

O que a função `h()` retorna? Não é _exatamente_ um elemento DOM real. Ela retorna um objeto que contém informações que descrevem para o Vue que tipo de nó deve ser renderizado na página, incluíndo descrições de qualquer nó filho. Chamamos essa descrição do nó de "nó virtual", geralmente abreviado para **_VNode_**. "Virtual DOM" é como chamamos toda a árvore de _VNodes_, constituída de uma árvore de componentes Vue.

## Argumentos do `h()`

A função `h()`é um utilitário para criar _VNodes_. Poderia, talvez, ser nomeado com mais precisão como `createVNode()`, mas é chamada `h()` devido ao uso frequente e por brevidade. Ela aceita três argumentos:

```js
// @returns {VNode}
h(
  // {String | Object | Function} tag
  // O nome de uma tag HTML, um componente, um componente assíncrono ou um
  // componente funcional.
  //
  // Obrigatório.
  'div',

  // {Object} props
  // Um objeto correspondente aos atributos, propriedades e eventos
  // que utilizaríamos em um template.
  //
  // Opcional.
  {},

  // {String | Array | Object} children
  // VNodes filhos, construídos usando `h()`,
  // ou usando strings para obter 'VNodes de texto' ou
  // um objeto com slots.
  //
  // Opcional.
  [
    'Algum texto vem primeiro.',
    h('h1', 'Um título'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ]
)
```

Se não houver props, os filhos geralmente podem ser passados como segundo argumento. Nos casos em que isso for ambíguo, `null` pode ser passado como o segundo argumento para manter os filhos como terceiro argumento.

## Exemplo Completo

Com este conhecimento, podemos agora finalizar o componente que começamos:

```js
const { createApp, h } = Vue

const app = createApp({})

/** Recupera o texto dos nós filhos recursivamente */
function getChildrenTextContent(children) {
  return children
    .map(node => {
      return typeof node.children === 'string'
        ? node.children
        : Array.isArray(node.children)
        ? getChildrenTextContent(node.children)
        : ''
    })
    .join('')
}

app.component('anchored-heading', {
  render() {
    // cria um id em kebab-case a partir do texto dos filhos
    const headingId = getChildrenTextContent(this.$slots.default())
      .toLowerCase()
      .replace(/\W+/g, '-') // substitui caracteres não-texto por traços
      .replace(/(^-|-$)/g, '') // remove os traços iniciais e finais

    return h('h' + this.level, [
      h(
        'a',
        {
          name: headingId,
          href: '#' + headingId
        },
        this.$slots.default()
      )
    ])
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

## Restrições

### VNodes Devem Ser Únicos

Todos os _VNodes_ na árvore de componentes devem ser únicos. Isso significa que a função de renderização a seguir é inválida:

```js
render() {
  const myParagraphVNode = h('p', 'hi')
  return h('div', [
    // Eita - VNodes duplicados!
    myParagraphVNode, myParagraphVNode
  ])
}
```

Se você realmente quiser duplicar o mesmo elemento/componente várias vezes, você pode fazê-lo com uma função fábrica (_factory function_). Por exemplo, a função de renderização a seguir é uma forma perfeitamente válida de renderizar 20 parágrafos idênticos:

```js
render() {
  return h('div',
    Array.from({ length: 20 }).map(() => {
      return h('p', 'hi')
    })
  )
}
```

## Creating Component VNodes

To create a VNode for a component, the first argument passed to `h` should be the component itself:

```js
render() {
  return h(ButtonCounter)
}
```

If we need to resolve a component by name then we can call `resolveComponent`:

```js
const { h, resolveComponent } = Vue

// ...

render() {
  const ButtonCounter = resolveComponent('ButtonCounter')
  return h(ButtonCounter)
}
```

`resolveComponent` is the same function that templates use internally to resolve components by name.

A `render` function will normally only need to use `resolveComponent` for components that are [registered globally](/guide/component-registration.html#global-registration). [Local component registration](/guide/component-registration.html#local-registration) can usually be skipped altogether. Consider the following example:

```js
// We can simplify this
components: {
  ButtonCounter
},
render() {
  return h(resolveComponent('ButtonCounter'))
}
```

Rather than registering a component by name and then looking it up we can use it directly instead:

```js
render() {
  return h(ButtonCounter)
}
```

## Substituíndo Recursos de _Templates_ com JavaScript Simples

### `v-if` e `v-for`

Sempre que algo for facilmente realizado usando JavaScript simples, as funções de renderização do Vue não são uma alternativa apropriada. Por exemplo, em um _template_ usando `v-if` e `v-for`:

```html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>Não foram encontrados itens.</p>
```

Pode ser rescrito usando `if`/`else` e `map()` com JavaScript em uma função de renderização:

```js
props: ['items'],
render() {
  if (this.items.length) {
    return h('ul', this.items.map((item) => {
      return h('li', item.name)
    }))
  } else {
    return h('p', 'Não foram encontrados itens.')
  }
}
```

In a template it can be useful to use a `<template>` tag to hold a `v-if` or `v-for` directive. When migrating to a `render` function, the `<template>` tag is no longer required and can be discarded.

### `v-model`

A diretiva `v-model` é expandida para as propriedades `modelValue`e `onUpdate:modelValue` durante a compilação do _template_ - nós mesmos teremos que prover essas propriedades:

```js
props: ['modelValue'],
emits: ['update:modelValue'],
render() {
  return h(SomeComponent, {
    modelValue: this.modelValue,
    'onUpdate:modelValue': value => this.$emit('update:modelValue', value)
  })
}
```

### `v-on`

Temos que prover um nome de propriedade adequado para o manipulador do evento, e.g., para manipular um evento de `click`, o nome da propriedade deve ser `onClick`.

```js
render() {
  return h('div', {
    onClick: $event => console.log('clicked', $event.target)
  })
}
```

#### Modificadores de Eventos

Os modificadores de evento `.passive`, `.capture` e `.once`, podem ser concatenados após o nome do evento usando _camel case_.

Por exemplo:

```js
render() {
  return h('input', {
    onClickCapture: this.doThisInCapturingMode,
    onKeyupOnce: this.doThisOnce,
    onMouseoverOnceCapture: this.doThisOnceInCapturingMode
  })
}
```

Para todos os outros modificadores de evento, não é necessária nenhuma API especial, pois podemos usar métodos de evento no manipulador:

| Modificador(es)                                                | Equivalente no manipulador                                                                                                        |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `.stop`                                                        | `event.stopPropagation()`                                                                                                         |
| `.prevent`                                                     | `event.preventDefault()`                                                                                                          |
| `.self`                                                        | `if (event.target !== event.currentTarget) return`                                                                                |
| Teclas:<br>ex: `.enter`                                        | `if (event.key !== 'Enter') return`<br><br>Mude `'Enter'` para a [tecla](http://keycode.info/) apropriada                               |
| Modificadores de teclas:<br>`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return`<br><br>Da mesma forma para `altKey`,` shiftKey` e `metaKey`                                          |

Aqui temos um exemplo de todos esses modificadores sendo usados juntos:

```js
render() {
  return h('input', {
    onKeyUp: event => {
      // Aborta se o elemento emitindo o evento não é
      // o elemento em qual o evento está ligado
      if (event.target !== event.currentTarget) return
      // Aborta se a tecla que foi pressionada não é a tecla enter
      // e a tecla shift não está sendo segurada
      // ao mesmo tempo
      if (!event.shiftKey || event.key !== 'Enter') return
      // Para a propagação de eventos
      event.stopPropagation()
      // Previne o manipulador padrão de teclas para este elemento
      event.preventDefault()
      // ...
    }
  })
}
```

### Slots

Podemos acessar os conteúdos de slots como arrays de _VNodes_ através de [`this.$slots`](../api/instance-properties.html#slots):

```js
render() {
  // `<div><slot></slot></div>`
  return h('div', this.$slots.default())
}
```

```js
props: ['message'],
render() {
  // `<div><slot :text="message"></slot></div>`
  return h('div', this.$slots.default({
    text: this.message
  }))
}
```

Para VNodes de componente, precisamos passar os filhos para `h` como um objeto em vez de um array. Cada propriedade é usada para preencher o slot de mesmo nome:

```js
render() {
  // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
  return h('div', [
    h(
      resolveComponent('child'),
      null,
      // passa `slots` como objetos filhos
      // na forma de { name: props => VNode | Array<VNode> }
      {
        default: (props) => h('span', props.text)
      }
    )
  ])
}
```

The slots are passed as functions, allowing the child component to control the creation of each slot's contents. Any reactive data should be accessed within the slot function to ensure that it's registered as a dependency of the child component and not the parent. Conversely, calls to `resolveComponent` should be made outside the slot function, otherwise they'll resolve relative to the wrong component:

```js
// `<MyButton><MyIcon :name="icon" />{{ text }}</MyButton>`
render() {
  // Calls to resolveComponent should be outside the slot function
  const Button = resolveComponent('MyButton')
  const Icon = resolveComponent('MyIcon')

  return h(
    Button,
    null,
    {
      // Use an arrow function to preserve the `this` value
      default: (props) => {
        // Reactive properties should be read inside the slot function
        // so that they become dependencies of the child's rendering
        return [
          h(Icon, { name: this.icon }),
          this.text
        ]
      }
    }
  )
}
```

If a component receives slots from its parent, they can be passed on directly to a child component:

```js
render() {
  return h(Panel, null, this.$slots)
}
```

They can also be passed individually or wrapped as appropriate:

```js
render() {
  return h(
    Panel,
    null,
    {
      // If we want to pass on a slot function we can
      header: this.$slots.header,

      // If we need to manipulate the slot in some way
      // then we need to wrap it in a new function
      default: (props) => {
        const children = this.$slots.default ? this.$slots.default(props) : []

        return children.concat(h('div', 'Extra child'))
      }
    }
  )
}
```

### `<component>` and `is`

Behind the scenes, templates use `resolveDynamicComponent` to implement the `is` attribute. We can use the same function if we need all the flexibility provided by `is` in our `render` function:

```js
const { h, resolveDynamicComponent } = Vue

// ...

// `<component :is="name"></component>`
render() {
  const Component = resolveDynamicComponent(this.name)
  return h(Component)
}
```

Just like `is`, `resolveDynamicComponent` supports passing a component name, an HTML element name, or a component options object.

However, that level of flexibility is usually not required. It's often possible to replace `resolveDynamicComponent` with a more direct alternative.

For example, if we only need to support component names then `resolveComponent` can be used instead.

If the VNode is always an HTML element then we can pass its name directly to `h`:

```js
// `<component :is="bold ? 'strong' : 'em'"></component>`
render() {
  return h(this.bold ? 'strong' : 'em')
}
```

Similarly, if the value passed to `is` is a component options object then there's no need to resolve anything, it can be passed directly as the first argument of `h`.

Much like a `<template>` tag, a `<component>` tag is only required in templates as a syntactical placeholder and should be discarded when migrating to a `render` function.

### Custom Directives

Custom directives can be applied to a VNode using [`withDirectives`](/api/global-api.html#withdirectives):

```js
const { h, resolveDirective, withDirectives } = Vue

// ...

// <div v-pin:top.animate="200"></div>
render () {
  const pin = resolveDirective('pin')

  return withDirectives(h('div'), [
    [pin, 200, 'top', { animate: true }]
  ])
}
```

[`resolveDirective`](/api/global-api.html#resolvedirective) is the same function that templates use internally to resolve directives by name. That is only necessary if you don't already have direct access to the directive's definition object.

### Built-in Components

[Built-in components](/api/built-in-components.html) such as `<keep-alive>`, `<transition>`, `<transition-group>`, and `<teleport>` are not registered globally by default. This allows bundlers to perform tree-shaking, so that the components are only included in the build if they are used. However, that also means we can't access them using `resolveComponent` or `resolveDynamicComponent`.

Templates have special handling for those components, automatically importing them when they are used. When we're writing our own `render` functions, we need to import them ourselves:

```js
const { h, KeepAlive, Teleport, Transition, TransitionGroup } = Vue

// ...

render () {
  return h(Transition, { mode: 'out-in' }, /* ... */)
}
```

## Return Values for Render Functions

In all of the examples we've seen so far, the `render` function has returned a single root VNode. However, there are alternatives.

Returning a string will create a text VNode, without any wrapping element:

```js
render() {
  return 'Hello world!'
}
```

We can also return an array of children, without wrapping them in a root node. This creates a fragment:

```js
// Equivalent to a template of `Hello<br>world!`
render() {
  return [
    'Hello',
    h('br'),
    'world!'
  ]
}
```

If a component needs to render nothing, perhaps because data is still loading, it can just return `null`. This will be rendered as a comment node in the DOM.

## JSX

Se estivermos escrevendo muitas funções `render`, pode ficar doloroso escrever algo assim:

```js
h(
  resolveComponent('anchored-heading'),
  {
    level: 1
  },
  {
    default: () => [h('span', 'Olá'), ' mundo!']
  }
)
```

Especialmente quando a versão usando _template_ é mais concisa em comparação:

```vue-html
<anchored-heading :level="1"> <span>Olá</span> mundo! </anchored-heading>
```

É por isso que existe um [_plugin_ Babel](https://github.com/vuejs/jsx-next) para usar JSX com Vue, nos colocando em uma sintaxe que é mais próxima dos _templates_:

```jsx
import AnchoredHeading from './AnchoredHeading.vue'

const app = createApp({
  render() {
    return (
      <AnchoredHeading level={1}>
        <span>Olá</span> mundo!
      </AnchoredHeading>
    )
  }
})

app.mount('#demo')
```

Para saber mais sobre como JSX mapeia para o JavaScript, veja a [documentação de uso](https://github.com/vuejs/jsx-next#installation).

## Functional Components

Functional components are an alternative form of component that don't have any state of their own. They are rendered without creating a component instance, bypassing the usual component lifecycle.

To create a functional component we use a plain function, rather than an options object. The function is effectively the `render` function for the component. As there is no `this` reference for a functional component, Vue will pass in the `props` as the first argument:

```js
const FunctionalComponent = (props, context) => {
  // ...
}
```

The second argument, `context`, contains three properties: `attrs`, `emit`, and `slots`. These are equivalent to the instance properties [`$attrs`](/api/instance-properties.html#attrs), [`$emit`](/api/instance-methods.html#emit), and [`$slots`](/api/instance-properties.html#slots) respectively.

Most of the usual configuration options for components are not available for functional components. However, it is possible to define [`props`](/api/options-data.html#props) and [`emits`](/api/options-data.html#emits) by adding them as properties:

```js
FunctionalComponent.props = ['value']
FunctionalComponent.emits = ['click']
```

If the `props` option is not specified, then the `props` object passed to the function will contain all attributes, the same as `attrs`. The prop names will not be normalized to camelCase unless the `props` option is specified.

Functional components can be registered and consumed just like normal components. If you pass a function as the first argument to `h`, it will be treated as a functional component.

## Compilação de _Template_

Pode ser que você esteja interessando em saber que o Vue, na verdade, compila os _templates_ em funções de renderização. Isso é um detalhe de implementação que, geralmente, você não precisa saber, porém, se você quiser ver como recursos específicos de _templates_ são compilados, você pode achar interessante. Abaixo temos uma pequena demonstração de uso de `Vue.compile` para compilar em tempo real uma string de _template_:

<iframe src="https://vue-next-template-explorer.netlify.app/" width="100%" height="420"></iframe>
