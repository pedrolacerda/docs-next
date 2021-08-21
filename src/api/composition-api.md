# API de Composição

> Esta seção usa a sintaxe de [Componentes Single-File](../guide/single-file-component.html) como exemplos de códigos

## `setup`

Uma opção de componente que é executada **antes** do componente ser criado, uma vez que as `props` são resolvidas, e serve como um ponto de entrada para a *API* de composição

- **Argumentos:**

  - `{Data} props`
  - `{SetupContext} context`

- **Tipagem**:

```ts
interface Data {
  [key: string]: unknown
}

interface SetupContext {
  attrs: Data
  slots: Slots
  emit: (event: string, ...args: unknown[]) => void
}

function setup(props: Data, context: SetupContext): Data
```

::: tip Dica
Para obter a inferência de tipos para os argumentos passados para o `setup()`, é necessário o uso do [defineComponent](global-api.html#definecomponent).
:::

- **Exemplo**

  Com o *template*:

  ```vue-html
  <!-- MyBook.vue -->
  <template>
    <div>{{ readersNumber }} {{ book.title }}</div>
  </template>

  <script>
    import { ref, reactive } from 'vue'

    export default {
      setup() {
        const readersNumber = ref(0)
        const book = reactive({ title: 'Guia do Vue 3' })

        // Expõe para o template
        return {
          readersNumber,
          book
        }
      }
    }
  </script>
  ```

  Com função de renderização:

  ```js
  // MyBook.vue

  import { h, ref, reactive } from 'vue'

  export default {
    setup() {
      const readersNumber = ref(0)
      const book = reactive({ title: 'Guia do Vue 3' })
      // Observe que precisamos expor explicitamente o valor do "ref" aqui.
      return () => h('div', [readersNumber.value, book.title])
    }
  }
  ```

- **Veja também**: [`setup` da API de Composição](../guide/composition-api-setup.html)

## Gatilhos de Ciclo de Vida

Os gatilhos de ciclo de vida podem ser registrados com funções `onX` importadas diretamente:

```js
import { onMounted, onUpdated, onUnmounted } from 'vue'

const MyComponent = {
  setup() {
    onMounted(() => {
      console.log('Montado!')
    })
    onUpdated(() => {
      console.log('Atualizado!')
    })
    onUnmounted(() => {
      console.log('Desmontado!')
    })
  }
}
```

Essas funções de registro de gatilhos de ciclo de vida somente podem ser usadas de forma síncrona durante o [`setup()`](#setup), já que elas dependem do estado global para localizar a instância ativa atual (a instância do componente onde o `setup()` está sendo chamado agora). Chamá-los sem uma instância ativa resultará em um erro.

O contexto da instância do componente também é definido durante a execução síncrona dos gatilhos de ciclo de vida. Como resultado, os observadores e os dados computados criados de forma síncrona dentro dos gatilhos de ciclo de vida também são destruídos automaticamente quando o componente é desmontado.

- **Mapeamento das Opções de Ciclo de Vida entre a API de Opções e API de Composição**
  - ~~`beforeCreate`~~ -> use `setup()`
  - ~~`created`~~ -> use `setup()`
  - `beforeMount` -> `onBeforeMount`
  - `mounted` -> `onMounted`
  - `beforeUpdate` -> `onBeforeUpdate`
  - `updated` -> `onUpdated`
  - `beforeUnmount` -> `onBeforeUnmount`
  - `unmounted` -> `onUnmounted`
  - `errorCaptured` -> `onErrorCaptured`
  - `renderTracked` -> `onRenderTracked`
  - `renderTriggered` -> `onRenderTriggered`

- **Veja também**: [Gatilhos de ciclo de vida da API de Composição](../guide/composition-api-lifecycle-hooks.html)

## Prover / Injetar

`provide` e `inject` ativam a injeção de dependência. Ambos só podem ser chamados durante o [`setup()`](#setup) com uma instância atual ativa.

- **Tipagem**:

```ts
interface InjectionKey<T> extends Symbol {}

function provide<T>(key: InjectionKey<T> | string, value: T): void

// Sem valor padrão
function inject<T>(key: InjectionKey<T> | string): T | undefined
// Com valor padrão
function inject<T>(key: InjectionKey<T> | string, defaultValue: T): T
```

O Vue fornece uma interface `InjectionKey` que é um tipo genérico que estende de `Symbol`. Isso pode ser usado para sincronizar o tipo do valor injetado entre o provedor e consumidor:

```ts
import { InjectionKey, provide, inject } from 'vue'

const key: InjectionKey<string> = Symbol()

provide(key, 'foo') // Prover um valor que não seja string resultará em um erro.

const foo = inject(key) // tipo de foo: string | undefined
```

Se estiver usando chaves do tipo String ou Symbols não tipados, o tipo do valor injetado precisará ser declarado explicitamente:

```ts
const foo = inject<string>('foo') // string | undefined
```

- **Veja também**:
  - [Prover e Injetar Dados](../guide/component-provide-inject.html)
  - [Prover e Injetar Dados na API de Composição](../guide/composition-api-provide-inject.html)

## `getCurrentInstance`

`getCurrentInstance` permite acesso a uma instância interna do componente, útil para usos mais avançados ou para criadores de bibliotecas.

```ts
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties // acesso a globalProperties
  }
}
```

`getCurrentInstance` funciona **apenas** durante o [setup](#setup) ou [Gatilhos de Ciclo de vida](#gatilhos-de-ciclo-de-vida)

> Quando usado fora do [setup](#setup) ou [Gatilhos de Ciclo de vida](#gatilhos-de-ciclo-de-vida), chame o `getCurrentInstance()` no `setup` e use a instância em seu lugar.

```ts
const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance() // Funciona

    const id = useComponentId() // Funciona

    const handleClick = () => {
      getCurrentInstance() // Não funciona
      useComponentId() // Não funciona

      internalInstance // Funciona
    }

    onMounted(() => {
      getCurrentInstance() // Funciona
    })

    return () =>
      h(
        'button',
        {
          onClick: handleClick
        },
        `uid: ${id}`
      )
  }
}

// Também funciona se chamado em uma composição
function useComponentId() {
  return getCurrentInstance().uid
}
```
