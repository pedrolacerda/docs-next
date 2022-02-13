# Suporte ao TypeScript

> O [Vue CLI](https://cli.vuejs.org) fornece ferramentas integradas de suporte ao TypeScript.

## Declaração Oficial em Pacotes NPM

Um sistema de tipagem estática pode ajudar à prevenir diversos potenciais erros de _runtime_ conforme as aplicações crescem, é por isso que o Vue 3 é escrito em TypeScript. Isso significa que você não precisa de nenhum ferramental adicional para usar TypeScript com Vue - temos um suporte de primeiro mundo.

## Configuração Recomendada

```js
// tsconfig.json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    // isso permite uma inferência mais estrita para propriedades de dados no `this`
    "strict": true,
    "jsx": "preserve",
    "moduleResolution": "node"
  }
}
```

Observe que você precisa incluir `strict: true` (ou ao menos `noImplicitThis: true`, que é uma parte da _flag_ `strict`) para aumentar a checagem de tipos do `this` em métodos de componentes, caso contrário, será sempre tratado como tipo `any`.

Veja a [documentação das opções do compilador TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html) para mais detalhes.

## Configuração do Webpack

Se você estiver usando uma configuração do Webpack personalizada, o `ts-loader` precisa ser configurado para poder analisar os blocos `<script lang="ts">` em arquivos` .vue`:

```js{10}
// webpack.config.js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/],
        },
        exclude: /node_modules/,
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
      }
      ...
```

## Ferramentas de Desenvolvimento

### Criação de Projeto

O [Vue CLI](https://github.com/vuejs/vue-cli) pode gerar novos projetos que usam TypeScript. Para começar:

```bash
# 1. Instale o Vue CLI, se não estiver instalado
npm install --global @vue/cli

# 2. Crie um novo projeto e então escolha a opção "Manually select features"
vue create my-project-name

# Se você já tem um projeto com Vue CLI sem TypeScript, por favor adicione o plugin apropriado:
vue add typescript
```

Certifique-se que a parte `script` do componente tem o TypeScript definido como linguagem:

```html
<script lang="ts">
  ...
</script>
```

Ou, se você deseja combinar TypeScript com uma [função `render` do JSX](/guide/render-function.html#jsx):

```html
<script lang="tsx">
  ...
</script>
```

### Suporte do Editor

Para o desenvolvimento de aplicações Vue com TypeScript, nós recomendamos fortemente o uso do [Visual Studio Code](https://code.visualstudio.com/), que fornece um excelente suporte para o TypeScript. Se você está usando [componentes Single File](./single-file-component.html) (SFCs), obtenha a incrível [extensão Volar](https://github.com/johnsoncodehk/volar), que provê a inferência do TypeScript dentro de SFCs e muitos outros ótimos recursos.

O [WebStorm](https://www.jetbrains.com/webstorm/) também fornece um suporte pronto para uso de ambos, TypeScript e Vue.

## Definindo Componentes Vue

Para permitir ao TypeScript inferir corretamente os tipos dentro das opções do componente Vue, você precisa definir os componentes com o método global `defineComponent`:

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  // inferência de tipos habilitada
})
```

Se você estiver usando [componentes single-file](/guide/single-file-component.html), isso normalmente seria escrito assim:

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  // inferência de tipo habilitada
})
</script>
```

## Usando com a API de Opções

O TypeScript deve ser capaz de inferir a maioria dos tipos sem defini-los explicitamente. Por exemplo, se você tem um componente com uma propriedade numérica `count`, você receberá um erro se tentar chamar um método específico de _string_ nela.

```ts
const Component = defineComponent({
  data() {
    return {
      count: 0
    }
  },
  mounted() {
    const result = this.count.split('') // => A propriedade 'split' não existe no tipo 'number'
  }
})
```

Se você tem um tipo complexo ou uma interface, pode convertê-lo(a) usando a [afirmação de tipo](https://www.typescriptlang.org/docs/handbook/basic-types.html#type-assertions):

```ts
interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  data() {
    return {
      book: {
        title: 'Guia do Vue 3',
        author: 'Time do Vue',
        year: 2020
      } as Book
    }
  }
})
```

### Augmenting Types for `globalProperties`

Vue 3 provides a [`globalProperties` object](../api/application-config.html#globalproperties) that can be used to add a global property that can be accessed in any component instance. For example, a [plugin](./plugins.html#writing-a-plugin) might want to inject a shared global object or function.

```ts
// User Definition
import axios from 'axios'

const app = Vue.createApp({})
app.config.globalProperties.$http = axios

// Plugin for validating some data
export default {
  install(app, options) {
    app.config.globalProperties.$validate = (data: object, rule: object) => {
      // check whether the object meets certain rules
    }
  }
}
```

In order to tell TypeScript about these new properties, we can use [module augmentation](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#module-augmentation).

In the above example, we could add the following type declaration:

```ts
import axios from 'axios'

declare module '@vue/runtime-core' {
  export interface ComponentCustomProperties {
    $http: typeof axios
    $validate: (data: object, rule: object) => boolean
  }
}
```

We can put this type declaration in the same file, or in a project-wide `*.d.ts` file (for example, in the `src/typings` folder that is automatically loaded by TypeScript). For library/plugin authors, this file should be specified in the `types` property in `package.json`.

::: warning Make sure the declaration file is a TypeScript module
In order to take advantage of module augmentation, you will need to ensure there is at least one top-level `import` or `export` in your file, even if it is just `export {}`.

[In TypeScript](https://www.typescriptlang.org/docs/handbook/modules.html), any file containing a top-level `import` or `export` is considered a 'module'. If type declaration is made outside of a module, it will overwrite the original types rather than augmenting them.
:::

For more information about the `ComponentCustomProperties` type, see its [definition in `@vue/runtime-core`](https://github.com/vuejs/vue-next/blob/2587f36fe311359e2e34f40e8e47d2eebfab7f42/packages/runtime-core/src/componentOptions.ts#L64-L80) and [the TypeScript unit tests](https://github.com/vuejs/vue-next/blob/master/test-dts/componentTypeExtensions.test-d.tsx) to learn more.

### Anotando Tipos de Retorno

Por causa da natureza circular dos arquivos de declaração do Vue, o TypeScript pode ter dificuldades para inferir os tipos de dados computados. Por esse motivo, você precisa anotar o tipo do retorno de propriedades computadas.

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  data() {
    return {
      message: 'Hello!'
    }
  },
  computed: {
    // precisa de uma anotação
    greeting(): string {
      return this.message + '!'
    },

    // em uma computada com um setter, o getter precisa ser anotado
    greetingUppercased: {
      get(): string {
        return this.greeting.toUpperCase()
      },
      set(newValue: string) {
        this.message = newValue.toUpperCase()
      }
    }
  }
})
```

### Anotando Propriedades

O Vue faz uma validação em runtime em propriedades com um `type` definido. Para fornecer esses tipos para o TypeScript, precisamos converter o construtor com `PropType`:

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  props: {
    name: String,
    id: [Number, String],
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>
    },
    book: {
      type: Object as PropType<Book>,
      required: true
    },
    metadata: {
      type: null // metadata is typed as any
    }
  }
})
```

::: warning
Because of a [design limitation](https://github.com/microsoft/TypeScript/issues/38845) in TypeScript when it comes
to type inference of function expressions, you have to be careful with `validator` and `default` values for objects and arrays:
:::

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  year?: number
}

const Component = defineComponent({
  props: {
    bookA: {
      type: Object as PropType<Book>,
      // Make sure to use arrow functions
      default: () => ({
        title: 'Arrow Function Expression'
      }),
      validator: (book: Book) => !!book.title
    },
    bookB: {
      type: Object as PropType<Book>,
      // Or provide an explicit this parameter
      default(this: void) {
        return {
          title: 'Function Expression'
        }
      },
      validator(this: void, book: Book) {
        return !!book.title
      }
    }
  }
})
```

### Anotando Tipos de Eventos Emitidos

Podemos anotar o tipo do *payload* de um evento emitido. Além disso, todos os eventos emitidos não declarados lançarão um erro de tipo quando chamados:

```ts
const Component = defineComponent({
  emits: {
    addBook(payload: { bookName: string }) {
      // executa validação em tempo de execução
      return payload.bookName.length > 0
    }
  },
  methods: {
    onSubmit() {
      this.$emit('addBook', {
        bookName: 123 // Erro de tipo!
      })

      this.$emit('non-declared-event') // Erro de tipo!
    }
  }
})
```

## Usando com a API de Composição

Na função `setup()`, você não precisa passar um tipo para o parâmetro `props`, pois os tipos serão inferidos da opção `props` do componente.

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  props: {
    message: {
      type: String,
      required: true
    }
  },

  setup(props) {
    const result = props.message.split('') // correto, 'message' é tipada como uma string
    const filtered = props.message.filter(p => p.value) // um erro será disparado: Propriedade 'filter' não existe no tipo 'string'
  }
})
```

### Tipando `refs`

Refs inferem o tipo do valor inicial:

```ts
import { defineComponent, ref } from 'vue'

const Component = defineComponent({
  setup() {
    const year = ref(2020)

    const result = year.value.split('') // => Propriedade 'split' não existe no tipo 'number'
  }
})
```

Algumas vezes, podemos precisar especificar tipos complexos para o valor interno de uma ref. Podemos fazer isso simplesmente passando um argumento genérico ao chamar ref para sobrescrever a inferência padrão:

```ts
const year = ref<string | number>('2020') // tipo do year: Ref<string | number>

year.value = 2020 // ok!
```

::: tip Nota
Se o tipo do genérico é desconhecido, é recomendado converter `ref` para `Ref<T>`.
:::

### Typing Template Refs

Sometimes you might need to annotate a template ref for a child component in order to call its public method. For example, we have a `MyModal` child component with a method that opens the modal:

```ts
import { defineComponent, ref } from 'vue'

const MyModal = defineComponent({
  setup() {
    const isContentShown = ref(false)
    const open = () => (isContentShown.value = true)

    return {
      isContentShown,
      open
    }
  }
})
```

We want to call this method via a template ref from the parent component:

```ts
import { defineComponent, ref } from 'vue'

const MyModal = defineComponent({
  setup() {
    const isContentShown = ref(false)
    const open = () => (isContentShown.value = true)

    return {
      isContentShown,
      open
    }
  }
})

const app = defineComponent({
  components: {
    MyModal
  },
  template: `
    <button @click="openModal">Open from parent</button>
    <my-modal ref="modal" />
  `,
  setup() {
    const modal = ref()
    const openModal = () => {
      modal.value.open()
    }

    return { modal, openModal }
  }
})
```

While this will work, there is no type information about `MyModal` and its available methods. To fix this, you should use `InstanceType` when creating a ref:

```ts
setup() {
  const modal = ref<InstanceType<typeof MyModal>>()
  const openModal = () => {
    modal.value?.open()
  }

  return { modal, openModal }
}
```

Please note that you would also need to use [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) or any other way to check that `modal.value` is not undefined.

### Tipando `reactive`

Ao tipar uma propriedade `reactive`, podemos usar interfaces:

```ts
import { defineComponent, reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Vue 3 Guide' })
    // ou
    const book: Book = reactive({ title: 'Vue 3 Guide' })
    // ou
    const book = reactive({ title: 'Vue 3 Guide' }) as Book
  }
})
```

### Tipando `computed`

Valores computados irão inferir automaticamente o tipo do valor retornado

```ts
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  name: 'CounterButton',
  setup() {
    let count = ref(0)

    // apenas leitura
    const doubleCount = computed(() => count.value * 2)

    const result = doubleCount.value.split('') // => Propriedade 'split' não existe no tipo 'number'
  }
})
```

### Typing Event Handlers

When dealing with native DOM events, it might be useful to type the argument we pass to the handler correctly. Let's take a look at this example:

```vue
<template>
  <input type="text" @change="handleChange" />
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  setup() {
    // `evt` will be of type `any`
    const handleChange = evt => {
      console.log(evt.target.value) // TS will throw an error here
    }

    return { handleChange }
  }
})
</script>
```

As you can see, without annotating the `evt` argument correctly, TypeScript will throw an error when we try to access the value of the `<input>` element. The solution is to cast the event target with a correct type:

```ts
const handleChange = (evt: Event) => {
  console.log((evt.target as HTMLInputElement).value)
}
```
