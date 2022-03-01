# API da Aplicação

No Vue 3, APIs que modificam globalmente o comportamento do Vue foram movidas para instâncias da aplicação, criadas pelo novo método `createApp`. Além disso, seus efeitos agora têm como escopo a instância dessa aplicação específica:

```js
import { createApp } from 'vue'

const app = createApp({})
```

Chamar `createApp` retorna uma instância da aplicação. Esta instância fornece um contexto da aplicação. Toda a árvore de componentes montada pela instância da aplicação compartilha o mesmo contexto, fornecendo as configurações que eram "globais" anteriormente no Vue 2.x.

Além disso, uma vez que o método `createApp` retorna a própria instância da aplicação, você pode encadear outros métodos após ele, que podem ser encontrados nas seções seguintes.

## component

- **Argumentos:**

  - `{string} name`
  - `{Function | Object} definition (opcional)`

- **Retorna:**

  - A instância da aplicação se o argumento `definition` é passado
  - A definição do componente se o argumento `definition` não é passado

- **Uso:**

  Registrar ou recuperar um componente global. O registro também atribui automaticamente o `name` do componente com o parâmetro `name` fornecido. 

- **Exemplo:**

```js
import { createApp } from 'vue'

const app = createApp({})

// registrar um objeto de opções
app.component('my-component', {
  /* ... */
})

// recuperar um componente registrado
const MyComponent = app.component('my-component')
```

- **Ver também:** [Componentes](../guide/component-basics.html)

## config

- **Uso:**

Um objeto contendo configurações da aplicação.

- **Exemplo:**

```js
import { createApp } from 'vue'
const app = createApp({})

app.config = {...}
```

- **Ver também:** [Configuração da Aplicação](./application-config.html)

## directive

- **Argumentos:**

  - `{string} name`
  - `{Function | Object} definition (opcional)`

- **Retorna:**

  - A instância da aplicação se o argumento `definition` é passado
  - A definição da diretiva se o argumento `definition` não é passado

- **Uso:**

  Registrar ou recuperar uma diretiva global.

- **Exemplo:**

```js
import { createApp } from 'vue'
const app = createApp({})

// registro
app.directive('my-directive', {
  // A diretiva tem um conjunto de gatilhos de ciclo de vida:
  // chamado antes que os atributos do elemento vinculado (bound) ou escutas de evento sejam aplicados
  created() {},
  // chamado antes que o componente pai do elemento vinculado seja montado
  beforeMount() {},
  // chamado quando o componente pai do elemento vinculado é montado
  mounted() {},
  // chamado antes que o VNode do componente contido seja atualizado
  beforeUpdate() {},
  // chamado depois que o VNode do componente contido e de seus filhos
  // tenha atualizado
  updated() {},
  // chamado antes que o componente pai do elemento vinculado seja desmontado
  beforeUnmount() {},
  // chamado quando o componente pai do elemento vinculado é desmontado
  unmounted() {}
})

// registro (diretiva de função)
app.directive('my-directive', () => {
  // isso será chamado como `mounted` e `updated`
})

// getter, retorna a definição da diretiva se registrada
const myDirective = app.directive('my-directive')
```

Os gatilhos de diretiva recebem estes argumentos:

#### el

O elemento ao qual a diretiva está vinculada. Pode ser usado para manipular diretamente o DOM.

#### binding

Um objeto contendo as seguintes propriedades.

- `instance`: A instância do componente onde a diretiva é usada.
- `value`: O valor passado para a diretiva. Por exemplo em `v-my-directive="1 + 1"`, o valor seria `2`.
- `oldValue`: O valor anterior, disponível somente em `beforeUpdate` e `updated`. Está disponível se o valor foi alterado ou não.
- `arg`: O argumento passado para a diretiva, se houver. Por exemplo em `v-my-directive:foo`, o argumento seria `"foo"`.
- `modifiers`: Um objeto contendo modificadores, se houver. Por exemplo em `v-my-directive.foo.bar`, o objeto `modifiers` seria `{ foo: true, bar: true }`.
- `dir`: um objeto, passado como um parâmetro quando a diretiva é registrada. Por exemplo, na diretiva:

```js
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})
```

`dir` seria o seguinte objeto:

```js
{
  mounted(el) {
    el.focus()
  }
}
```

#### vnode

Um desenho (_blueprint_) do elemento DOM real recebido como o argumento `el` acima.

#### prevNode

O nó virtual anterior, somente disponível nos gatilhos `beforeUpdate` e `updated`.

:::tip Nota
Além de `el`, você deve tratar esses argumentos como somente leitura e nunca modificá-los. Se você precisar compartilhar informações entre gatilhos, é recomendado fazê-lo por meio do elemento [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset).
:::

- **Ver também:** [Diretivas Personalizadas](../guide/custom-directive.html)

## mixin

- **Argumentos:**

  - `{Object} mixin`

- **Retorna:**

  - A instância da aplicação

- **Uso:**

  Aplicar um mixin em todo o escopo da aplicação. Uma vez registrado eles podem ser usados no _template_ de qualquer componente da aplicação atual. Pode ser usado por autores de plugins para injetar comportamento personalizado nos componentes. **Não recomendado no código da aplicação**.

- **Ver também:** [Mixin Global](../guide/mixins.html#mixin-global)

## mount

- **Argumentos:**

  - `{Element | string} rootContainer`
  - `{boolean} isHydrate (opcional)`

- **Retorna:**

  - A instância do componente raiz

- **Uso:**

  O `innerHTML` do elemento DOM fornecido será substituído pelo _template_ renderizado do componente raiz do aplicativo.

- **Exemplo:**

```html
<body>
  <div id="my-app"></div>
</body>
```

```js
import { createApp } from 'vue'

const app = createApp({})
// faça alguns preparativos necessários
app.mount('#my-app')
```

- **Ver também:**
  - [Diagrama do Ciclo de Vida](../guide/instance.html#diagrama-do-ciclo-de-vida)

## provide

- **Argumentos:**

  - `{string | Symbol} key`
  - `value`

- **Retorna:**

  - A instância da aplicação

- **Uso:**

  Define um valor que pode ser injetado em todos os componentes dentro da aplicação. Componentes devem usar `inject` para receber os valores providos.

  De uma perspectiva `provide`/`inject`, a aplicação pode ser considerada como um ancestral do nível raiz, com o componente raiz como seu filho único.

  Este método não pode ser confundido com a [opção `provide` de componente](options-composition.html#provide-inject) ou a [função `provide`](composition-api.html#prover-injetar) da API de composição. Embora também façam parte do mesmo mecanismo `provide`/`inject`, eles são usados para configurar valores providos por um componente ao invés da aplicação.

  Prover valores por meio da aplicação é especialmente útil quando ao escrever plugins, pois os plugins normalmente não seriam capazes de prover valores usando componentes. É uma alternativa ao uso de [globalProperties](application-config.html#globalproperties).

  :::tip Nota
  Os vínculos `provide` e `inject` NÃO são reativos. Isto é intencional. No entanto, se você passar um objeto reativo, as propriedades desse objeto permanecerão reativas.
  :::

- **Exemplo:**

  Injetando uma propriedade no componente raiz, com um valor provido pela aplicação:

```js
import { createApp } from 'vue'

const app = createApp({
  inject: ['user'],
  template: `
    <div>
      {{ user }}
    </div>
  `
})

app.provide('user', 'administrator')
```

- **Ver também:**
  - [Prover / Injetar](../guide/component-provide-inject.md)

## unmount

- **Uso:**

  Desmonta um componente raiz da instância da aplicação.

- **Exemplo:**

```html
<body>
  <div id="my-app"></div>
</body>
```

```js
import { createApp } from 'vue'

const app = createApp({})
// faça algumas preparações necessárias
app.mount('#my-app')

// Aplicação será desmontada 5 segundos após a montagem
setTimeout(() => app.unmount(), 5000)
```

## use

- **Argumentos:**

  - `{Object | Function} plugin`
  - `...options (opcional)`

- **Retorna:**

  - A instância da aplicação

- **Uso:**

  Instalar um plugin Vue.js. Se o plugin é um objeto, deve expor um método `install`. Se for uma função em si, esta será tratada como o método `install`.

  O método `install` será chamado com a aplicação como seu primeiro argumento. Quaisquer `options` passadas para `use` serão passadas em argumentos subsequentes.

  Quando esse método é chamado no mesmo plugin várias vezes, o plugin será instalado apenas uma vez.

- **Exemplo:**

  ```js
  import { createApp } from 'vue'
  import MyPlugin from './plugins/MyPlugin'

  const app = createApp({})

  app.use(MyPlugin)
  app.mount('#app')
  ```

- **Ver também:** [Plugins](../guide/plugins.html)

## version

- **Uso:**

  Fornece a versão instalada do Vue como uma string. Isso é especialmente útil para [plugins](/guide/plugins.html) da comunidade, onde você pode usar estratégias diferentes para versões diferentes.

- **Exemplo:**

  ```js
  export default {
    install(app) {
      const version = Number(app.version.split('.')[0])

      if (version < 3) {
        console.warn('Este plugin requer o Vue 3')
      }

      // ...
    }
  }
  ```

- **Ver também:**: [API Global - version](/api/global-api.html#version)
