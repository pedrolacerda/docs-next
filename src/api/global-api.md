---
sidebarDepth: 1
---

# API Global

Se você estiver usando uma versão CDN, as funções da API global são acessíveis através do objeto global `Vue`. ex.:

```js
const { createApp, h, nextTick } = Vue
```

Se você estiver usando módulos ES, eles podem ser importados diretamente:

```js
import { createApp, h, nextTick } from 'vue'
```

Funções globais que lidam com reatividade, como `reactive` e `ref`, são documentadas separadamente. Veja [API de Reatividade](/api/reactivity-api.html) para obter essas funções.

## createApp

Retorna uma instância de aplicativo que fornece um contexto de aplicativo. Toda a árvore de componentes montada pela instância do aplicativo compartilha o mesmo contexto.

```js
const app = createApp({})
```

Você pode encadear outros métodos após `createApp`, eles podem ser encontrados em [API da Aplicação](./application-api.html)

### Argumentos

A função recebe um objeto de opções do componente raiz como primeiro parâmetro:

```js
const app = createApp({
  data() {
    return {
      ...
    }
  },
  methods: {...},
  computed: {...}
  ...
})
```

Com o segundo parâmetro, podemos passar propriedades raiz para a aplicação:

```js
const app = createApp(
  {
    props: ['username']
  },
  { username: 'Evan' }
)
```

```html
<div id="app">
  <!-- Mostrará 'Evan' -->
  {{ username }}
</div>
```

As propriedades raiz são propriedades brutas, muito parecidas com aquelas passadas para [`h`](#h) para criar um VNode. Além de propriedades de componentes, eles também podem incluir atributos e escutadores de eventos a serem aplicados ao componente raiz.

### Tipando

```ts
interface Data {
  [key: string]: unknown
}

export type CreateAppFunction<HostElement> = (
  rootComponent: PublicAPIComponent,
  rootProps?: Data | null
) => App<HostElement>
```

## h

Retorna um "nó virtual", geralmente abreviado para **VNode**: um objeto simples que contém informações que descrevem ao Vue que tipo de nó ele deve renderizar na página, incluindo descrições de quaisquer nós filhos. Destina-se a [funções de renderização](../guide/render-function.md) escritas manualmente:

```js
render() {
  return h('h1', {}, 'Algum título')
}
```

### Argumentos

Aceita três argumentos: `type`, `props` e `children`

#### type

- **Tipo:** `String | Object | Function`

- **Detalhes:**

  Um nome de tag HTML, um componente, um componente assíncrono ou um componente funcional. Usar uma função que retorna `null` renderizaria um comentário. Este parâmetro é obrigatório

#### props

- **Tipo:** `Object`

- **Detalhes:**

  Um objeto correspondente aos atributos, propriedades e eventos que usaríamos em um _template_. Opcional

#### children

- **Tipo:** `String | Array | Object`

- **Detalhes:**

  VNodes filhos, construídos usando `h()`, ou usando strings para obter "VNodes de texto" ou um objeto com slots. Opcional

  ```js
  h('div', {}, [
    'Algum texto vem primeiro.',
    h('h1', 'Um título'),
    h(MyComponent, {
      someProp: 'foobar'
    })
  ])
  ```

## defineComponent

Em termos de implementação, o `defineComponent` não faz nada além de retornar o objeto passado para ele. No entanto, em termos de tipagem, o valor retornado tem um tipo sintético de um construtor para função de renderização manual, suporte a ferramentas TSX e IDE.

### Argumentos

Um objeto com opções de componente

```js
import { defineComponent } from 'vue'

const MyComponent = defineComponent({
  data() {
    return { count: 1 }
  },
  methods: {
    increment() {
      this.count++
    }
  }
})
```

Ou uma função `setup`, o nome da função será usado como nome do componente

```js
import { defineComponent, ref } from 'vue'

const HelloWorld = defineComponent(function HelloWorld() {
  const count = ref(0)
  return { count }
})
```

## defineAsyncComponent

Cria um componente assíncrono que será carregado somente quando necessário.

### Argumentos

Para uso básico, `defineAsyncComponent` pode aceitar uma função fabricadora retornando uma `Promise`. O _callback_ `resolve` da Promise deve ser chamado quando você tiver recuperado sua definição de componente do servidor. Você também pode chamar `reject(reason)` para indicar que o carregamento falhou.

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

app.component('async-component', AsyncComp)
```

Ao usar [registro local](../guide/component-registration.html#registro-local), você também pode fornecer diretamente uma função que retorna uma `Promise`:

```js
import { createApp, defineAsyncComponent } from 'vue'

createApp({
  // ...
  components: {
    AsyncComponent: defineAsyncComponent(() =>
      import('./components/AsyncComponent.vue')
    )
  }
})
```

Para uso avançado, `defineAsyncComponent` pode aceitar um objeto do seguinte formato:

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent({
  // A função fabricadora
  loader: () => import('./Foo.vue'),
  // Um ​​componente para usar enquanto o componente assíncrono está carregando
  loadingComponent: LoadingComponent,
  // Um ​​componente a ser usado se o carregamento falhar
  errorComponent: ErrorComponent,
  // Atraso antes de mostrar o componente de carregamento. Padrão: 200ms.
  delay: 200,
  // O componente de erro será exibido se um tempo limite for
  // fornecido e excedido. Padrão: Infinito.
  timeout: 3000,
  // Definindo se o componente é suspensível. Padrão: true.
  suspensible: false,
  /**
   *
   * @param {*} error Objeto da mensagem de erro
   * @param {*} retry Uma função que indica se o componente assíncrono deve tentar novamente quando a promise do carregador for rejeitada
   * @param {*} fail Fim da falha
   * @param {*} attempts Número máximo de tentativas permitidas
   */
  onError(error, retry, fail, attempts) {
    if (error.message.match(/fetch/) && attempts <= 3) {
      // tenta novamente em erros de 'fetch', 3 tentativas no máximo
      retry()
    } else {
      // Observe que retry/fail é como resolve/reject de uma promise:
      // um deles deve ser chamado para que o tratamento de erros continue.
      fail()
    }
  },
})
```

**Ver também**: [Componentes Dinâmicos e Assíncronos](../guide/component-dynamic-async.html)

## defineCustomElement <Badge text="3.2+" />

Este método aceita o mesmo argumento que [`defineComponent`](#definecomponent), mas retorna um [Elemento Personalizado](https://developer.mozilla.org/pt-BR/docs/Web/Web_Components/Using_custom_elements) nativo que pode ser usado com qualquer framework ou sem nenhum.

Exemplo de uso:

```html
<my-vue-element></my-vue-element>
```

```js
import { defineCustomElement } from 'vue'

const MyVueElement = defineCustomElement({
  // opções normais de componente Vue aqui
  props: {},
  emits: {},
  template: `...`,

  // somente do defineCustomElement: CSS a ser injetado na raiz sombra (shadow)
  styles: [`/* CSS embutido */`]
})

// Registra o elemento personalizado.
// Após o registro, todas as tags `<my-vue-element>` na página serão atualizadas.
customElements.define('my-vue-element', MyVueElement)

// Você também pode instanciar programaticamente o elemento:
// (só pode ser feito após o registro)
document.body.appendChild(
  new MyVueElement({
    // propriedades iniciais (opcional)
  })
)
```

Para obter mais detalhes sobre a construção de Web Components com Vue, especialmente com Componentes Single-File, veja [Vue e Web Components](/guide/web-components.html#building-custom-elements-with-vue).

## resolveComponent

:::warning Aviso
`resolveComponent` só pode ser usado dentro das funções `render` ou `setup`.
:::

Permite resolver um `component` por seu nome, se estiver disponível na instância atual do aplicativo.

Retorna um `Component` ou o argumento `name` quando não encontrado.

```js
const app = createApp({})
app.component('MyComponent', {
  /* ... */
})
```

```js
import { resolveComponent } from 'vue'
render() {
  const MyComponent = resolveComponent('MyComponent')
}
```

### Argumentos

Aceita um argumento: `name`

#### name

- **Tipo:** `String`

- **Detalhes:**

  O nome de um componente carregado.

## resolveDynamicComponent

:::warning Aviso
`resolveDynamicComponent` só pode ser usado dentro das funções `render` ou `setup`.
:::

Permite resolver um `component` pelo mesmo mecanismo que `<component :is="">` emprega.

Retorna o `Component` resolvido ou um `VNode` recém-criado com o nome do componente como a tag do nó. Irá emitir um aviso se o `Component` não for encontrado.

```js
import { resolveDynamicComponent } from 'vue'
render () {
  const MyComponent = resolveDynamicComponent('MyComponent')
}
```

### Argumentos

Aceita um argumento: `component`

#### component

- **Tipo:** `String | Object (objeto de opções do componente)`

- **Detalhes:**

  Para obter mais detalhes, consulte a documentação em [Componentes Dinâmicos](../guide/component-dynamic-async.html).

## resolveDirective

:::warning Aviso
`resolveDirective` só pode ser usado dentro das funções `render` ou `setup`.
:::

Permite resolver uma `directive` pelo seu nome, se estiver disponível na instância atual do aplicativo.

Retorna um `Directive` ou `undefined` quando não encontrado.

```js
const app = createApp({})
app.directive('highlight', {})
```

```js
import { resolveDirective } from 'vue'
render () {
  const highlightDirective = resolveDirective('highlight')
}
```

### Argumentos

Aceita um argumento: `name`

#### name

- **Tipo:** `String`

- **Detalhes:**

  O nome de uma diretiva carregada.

## withDirectives

:::warning Aviso
`withDirectives` só pode ser usado dentro das funções `render` ou `setup`.
:::

Permite aplicar diretivas a um **VNode**. Retorna um VNode com as diretivas aplicadas.

```js
import { withDirectives, resolveDirective } from 'vue'
const foo = resolveDirective('foo')
const bar = resolveDirective('bar')

return withDirectives(h('div'), [
  [foo, this.x],
  [bar, this.y]
])
```

### Argumentos

Aceita dois argumentos: `vnode` e `directives`.

#### vnode

- **Tipo:** `vnode`

- **Detalhes:**

  Um nó virtual, geralmente criado com `h()`.

#### directives

- **Tipo:** `Array`

- **Detalhes:**

  Um array de diretivas.

  Cada diretiva em si é um array, que permite que até 4 índices sejam definidos como visto nos exemplos a seguir.

  - `[directive]` - A directiva por si só. Requerido.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [[MyDirective]])
  ```

  - `[directive, value]` - O acima, mais um valor do tipo `any` a ser atribuído à diretiva

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [[MyDirective, 100]])
  ```

  - `[directive, value, arg]` - O acima, mais um argumento `String`, ex.: `click` em `v-on:click`

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [
    [MyDirective, 100, 'click']
  ])
  ```

  - `[directive, value, arg, modifiers]` - O acima, mais um par `key: value` de `Object` definindo quaisquer modificadores.

  ```js
  const MyDirective = resolveDirective('MyDirective')
  const nodeWithDirectives = withDirectives(h('div'), [
    [MyDirective, 100, 'click', { prevent: true }]
  ])
  ```

## createRenderer

A função createRenderer aceita dois argumentos genéricos:
`HostNode` e `HostElement`, correspondentes aos tipos Node e Element no
ambiente hospedeiro.

Por exemplo, para o _runtime-dom_, HostNode seria o DOM
A interface `Node` e HostElement seriam a interface DOM do `Element`.

Os renderizadores personalizados podem passar os tipos específicos da plataforma como este:

```ts
import { createRenderer } from 'vue'
const { render, createApp } = createRenderer<Node, Element>({
  patchProp,
  ...nodeOps
})
```

### Argumentos

Aceita dois argumentos: `HostNode` e `HostElement`

#### HostNode

- **Tipo:** `Node`

- **Detalhes:**

  O nó no ambiente hospedeiro.

#### HostElement

- **Tipo:** `Element`

- **Detalhes:**

  O elemento no ambiente hospedeiro.

## nextTick

Adie o _callback_ para ser executado após o próximo ciclo de atualização do DOM. Use-o imediatamente após alterar alguns dados para aguardar a atualização do DOM.

```js
import { createApp, nextTick } from 'vue'

const app = createApp({
  setup() {
    const message = ref('Olá!')
    const changeMessage = async newMessage => {
      message.value = newMessage
      await nextTick()
      console.log('Agora o DOM está atualizado')
    }
  }
})
```

**Ver também**: [método de instância `$nextTick`](instance-methods.html#nexttick)

## mergeProps

Pega vários objetos contendo propriedades de VNode e os mescla em um único objeto. Um objeto recém-criado é retornado, os objetos passados ​​como argumentos não são modificados.

Qualquer número de objetos pode ser passado, com as propriedades dos argumentos posteriores tendo precedência. Os escutadores de eventos são tratados especialmente, assim como `class` e `style`, com os valores dessas propriedades sendo mesclados em vez de sobrescritos.

```js
import { h, mergeProps } from 'vue'

export default {
  inheritAttrs: false,

  render() {
    const props = mergeProps(
      {
        // A classe será mesclada com qualquer classe de $attrs
        class: 'active'
      },
      this.$attrs
    )

    return h('div', props)
  }
}
```

## useCssModule

:::warning Aviso
`useCssModule` só pode ser usado dentro das funções `render` ou `setup`.
:::

Permite que módulos CSS sejam acessados ​​dentro da função [`setup`](/api/composition-api.html#setup) de um [componente single-file](/guide/single-file-component.html):

```vue
<script>
import { h, useCssModule } from 'vue'

export default {
  setup() {
    const style = useCssModule()

    return () =>
      h(
        'div',
        {
          class: style.success
        },
        'Tarefa concluída!'
      )
  }
}
</script>

<style module>
.success {
  color: #090;
}
</style>
```

Para obter mais informações sobre o uso de módulos CSS, consulte [Recursos de Estilo em SFC: `<style module>`](/api/sfc-style.html#style-module).

### Argumentos

Aceita um argumento: `name`

#### name

- **Tipo:** `String`

- **Detalhes:**

  O nome do módulo CSS. O padrão é `'$style'`.

## version

Fornece a versão instalada do Vue como uma string.

```js
const version = Number(Vue.version.split('.')[0])

if (version === 3) {
  // Vue 3
} else if (version === 2) {
  // Vue 2
} else {
  // Versões não suportadas do Vue
}
```

**Ver também**: [API da Aplicação - version](/api/application-api.html#version)
