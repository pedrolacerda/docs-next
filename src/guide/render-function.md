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

## Criando VNodes de Componentes

Para criar um VNode para um componente, o primeiro argumento passado para `h` deve ser o próprio componente:

```js
render() {
  return h(ButtonCounter)
}
```

Se precisarmos resolver um componente pelo nome, podemos chamar `resolveComponent`:

```js
const { h, resolveComponent } = Vue

// ...

render() {
  const ButtonCounter = resolveComponent('ButtonCounter')
  return h(ButtonCounter)
}
```

`resolveComponent` é a mesma função que os _templates_ usam internamente para resolver componentes por nome.

Uma função `render` normalmente só precisa usar `resolveComponent` para componentes que são [registrados globalmente](/guide/component-registration.html#registro-global). Assim o [registro local de componentes](/guide/component-registration.html#registro-local) geralmente pode ser ignorado por completo. Considere o seguinte exemplo:

```js
// Podemos simplificar isso
components: {
  ButtonCounter
},
render() {
  return h(resolveComponent('ButtonCounter'))
}
```

Em vez de registrar um componente pelo nome e depois procurá-lo, podemos usá-lo diretamente:

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

Em um _template_ pode ser útil usar uma tag `<template>` para conter uma diretiva `v-if` ou `v-for`. Ao migrar para uma função `render`, a tag `<template>` não é mais necessária e pode ser descartada.

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

Os slots são passados ​​como funções, permitindo que o componente filho controle a criação do conteúdo de cada slot. Quaisquer dados reativos devem ser acessados ​​dentro da função do slot para garantir que sejam registrados como uma dependência do componente filho e não do pai. Por outro lado, chamadas ao `resolveComponent` devem ser feitas fora da função do slot, caso contrário, elas serão resolvidas em relação ao componente errado:

```js
// `<MyButton><MyIcon :name="icon" />{{ text }}</MyButton>`
render() {
  // Chamadas para resolveComponent devem estar fora da função slot
  const Button = resolveComponent('MyButton')
  const Icon = resolveComponent('MyIcon')

  return h(
    Button,
    null,
    {
      // Use uma arrow function para preservar o valor de `this`
      default: (props) => {
        // Propriedades reativas devem ser lidas dentro da função slot
        // para que se tornem dependências da renderização do filho
        return [
          h(Icon, { name: this.icon }),
          this.text
        ]
      }
    }
  )
}
```

Se um componente recebe slots de seu pai, eles podem ser passados ​​diretamente para um componente filho:

```js
render() {
  return h(Panel, null, this.$slots)
}
```

Eles também podem ser passados ​​individualmente ou envolvidos conforme apropriado:

```js
render() {
  return h(
    Panel,
    null,
    {
      // Se quisermos passar uma função de slot podemos
      header: this.$slots.header,

      // Se precisarmos manipular o slot de alguma forma
      // então precisamos envolvê-lo em uma nova função
      default: (props) => {
        const children = this.$slots.default ? this.$slots.default(props) : []

        return children.concat(h('div', 'Filho extra'))
      }
    }
  )
}
```

### `<component>` e `is`

Nos bastidores, os _templates_ usam `resolveDynamicComponent` para implementar o atributo `is`. Podemos usar a mesma função se precisarmos de toda a flexibilidade fornecida por `is` em nossa função `render`:

```js
const { h, resolveDynamicComponent } = Vue

// ...

// `<component :is="name"></component>`
render() {
  const Component = resolveDynamicComponent(this.name)
  return h(Component)
}
```

Assim como `is`, `resolveDynamicComponent` suporta a passagem de um nome de componente, um nome de elemento HTML ou um objeto de opções de componente.

No entanto, esse nível de flexibilidade geralmente não é necessário. Muitas vezes é possível substituir `resolveDynamicComponent` por uma alternativa mais direta.

Por exemplo, se precisarmos apenas oferecer suporte a nomes de componentes, então `resolveComponent` pode ser usado.

Se o VNode for sempre um elemento HTML, podemos passar seu nome diretamente para `h`:

```js
// `<component :is="bold ? 'strong' : 'em'"></component>`
render() {
  return h(this.bold ? 'strong' : 'em')
}
```

Da mesma forma, se o valor passado para `is` for um objeto de opções de componente, então não há necessidade de resolver nada, ele pode ser passado diretamente como o primeiro argumento de `h`.

Assim como uma tag `<template>`, uma tag `<component>` só é necessária em _templates_ como um espaço reservado sintático e deve ser descartada ao migrar para uma função `render`.

### Diretivas Personalizadas

Diretivas personalizadas podem ser aplicadas a um VNode usando [`withDirectives`](/api/global-api.html#withdirectives):

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

[`resolveDirective`](/api/global-api.html#resolvedirective) é a mesma função que os _templates_ usam internamente para resolver diretivas por nome. Isso só é necessário se você ainda não tiver acesso direto ao objeto de definição da diretiva.

### Componentes Integrados

[Componentes integrados](/api/built-in-components.html), como `<keep-alive>`, `<transition>`, `<transition-group>` e `<teleport>` por padrão não são registrados globalmente. Isso permite que empacotadores executem o _tree-shaking_ para que os componentes sejam incluídos na compilação apenas se forem usados. No entanto, isso também significa que não podemos acessá-los usando `resolveComponent` ou `resolveDynamicComponent`.

_Templates_ possuem tratamento especial para esses componentes, importando-os automaticamente quando utilizados. Quando estamos escrevendo nossas próprias funções `render`, precisamos importá-las nós mesmos:

```js
const { h, KeepAlive, Teleport, Transition, TransitionGroup } = Vue

// ...

render () {
  return h(Transition, { mode: 'out-in' }, /* ... */)
}
```

## Valores de Retorno para Funções de Renderização

Em todos os exemplos que vimos até agora, a função `render` retornou um único VNode raiz. No entanto, existem alternativas.

Retornar uma string criará um VNode de texto, sem nenhum elemento de encapsulamento:

```js
render() {
  return 'Olá mundo!'
}
```

Também podemos retornar um array de filhos, sem envolvê-los em um nó raiz. Isso cria um fragmento:

```js
// Equivalente a um template de `Olá<br>mundo!`
render() {
  return [
    'Olá',
    h('br'),
    'mundo!'
  ]
}
```

Se um componente não precisar renderizar nada, talvez porque os dados ainda estão sendo carregados, ele pode simplesmente retornar `null`. Isso será renderizado como um nó de comentário no DOM.

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

## Componentes Funcionais

Componentes funcionais são uma forma alternativa de componente que não possui nenhum estado próprio. Eles são renderizados sem criar uma instância de componente, ignorando o ciclo de vida normal do componente.

Para criar um componente funcional, usamos uma função simples, em vez de um objeto de opções. A função é efetivamente a função `render` para o componente. Como não há referência de `this` para um componente funcional, o Vue passará o `props` como primeiro argumento:

```js
const FunctionalComponent = (props, context) => {
  // ...
}
```

O segundo argumento, `context`, contém três propriedades: `attrs`, `emit` e `slots`. Elas são equivalentes às propriedades de instância [`$attrs`](/api/instance-properties.html#attrs), [`$emit`](/api/instance-methods.html#emit) e [`$slots`](/api/instance-properties.html#slots) respectivamente.

A maioria das opções de configuração usuais para componentes não está disponível para componentes funcionais. No entanto, é possível definir [`props`](/api/options-data.html#props) e [`emits`](/api/options-data.html#emits) adicionando-os como propriedades:

```js
FunctionalComponent.props = ['value']
FunctionalComponent.emits = ['click']
```

Se a opção `props` não for especificada, então o objeto `props` passado para a função conterá todos os atributos, o mesmo que `attrs`. Os nomes das props não serão normalizados para camelCase a menos que a opção `props` seja especificada.

Componentes funcionais podem ser registrados e consumidos como componentes normais. Se você passar uma função como primeiro argumento para `h`, ela será tratada como um componente funcional.

## Compilação de _Template_

Pode ser que você esteja interessando em saber que o Vue, na verdade, compila os _templates_ em funções de renderização. Isso é um detalhe de implementação que, geralmente, você não precisa saber, porém, se você quiser ver como recursos específicos de _templates_ são compilados, você pode achar interessante. Abaixo temos uma pequena demonstração de uso de `Vue.compile` para compilar em tempo real uma string de _template_:

<iframe src="https://vue-next-template-explorer.netlify.app/" width="100%" height="420"></iframe>
