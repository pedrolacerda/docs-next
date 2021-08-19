# Refs

> Esta seção usa a sintaxe [componente single-file](../guide/single-file-component.html) para os exemplos de código

## `ref`

Assume um valor interno e retorna um objeto referência (ref) reativo e mutável. O objeto referência tem uma única propriedade `.value` que aponta para o valor interno.

**Exemplo:**

```js
const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

Se um objeto for atribuído como um valor de `ref`, o objeto se tornará profundamente reativo pelo método [`reactive`](./basic-reactivity.html#reactive).

**Tipando:**

```ts
interface Ref<T> {
  value: T
}

function ref<T>(value: T): Ref<T>
```

Às vezes, podemos precisar especificar tipos complexos para o valor interno de uma referência. Podemos fazer isso de forma sucinta passando um argumento genérico ao chamar `ref` para substituir a inferência padrão:

```ts
const foo = ref<string | number>('foo') // tipo de foo: Ref<string | number>

foo.value = 123 // ok!
```

Se o tipo do genérico for desconhecido, é recomendado converter `ref` em ` Ref<T>`:

```js
function useState<State extends string>(initial: State) {
  const state = ref(initial) as Ref<State> // state.value -> State estende string
  return state
}
```

## `unref`

Retorna o valor interno se o argumento for um [`ref`](#ref), caso contrário, retorna o próprio argumento. Esta é uma _sugar function_ para `val = isRef(val) ? val.value : val`.

```js
function useFoo(x: number | Ref<number>) {
  const unwrapped = unref(x) // unwrapped é garantido ser número agora
}
```

## `toRef`

Pode ser usado para criar um [`ref`](#ref) de uma propriedade originária de um objeto reativo. A referência pode então ser transmitida, mantendo a conexão reativa com sua propriedade de origem.

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const fooRef = toRef(state, 'foo')

fooRef.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

`toRef` é útil quando você deseja passar a referência de uma propriedade para uma função de composição:

```js
export default {
  setup(props) {
    useSomeFeature(toRef(props, 'foo'))
  }
}
```

## `toRefs`

Converte um objeto reativo em um objeto simples onde cada propriedade do objeto resultante é um [`ref`](#ref) apontando para a propriedade correspondente do objeto original.

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)
/*
Tipo de stateAsRefs:

{
  foo: Ref<number>,
  bar: Ref<number>
}
*/

// A referência e a propriedade original estão "vinculadas"
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3
```

`toRefs` é útil ao retornar um objeto reativo de uma função de composição para que o componente consumidor possa desestruturar/espalhar o objeto retornado sem perder a reatividade:

```js
function useFeatureX() {
  const state = reactive({
    foo: 1,
    bar: 2
  })

  // lógica operando no estado

  // converter para referências ao retornar
  return toRefs(state)
}

export default {
  setup() {
    // pode desestruturar sem perder a reatividade
    const { foo, bar } = useFeatureX()

    return {
      foo,
      bar
    }
  }
}
```

## `isRef`

Verifica se um valor é um objeto referência.

## `customRef`

Cria uma referência personalizada com controle explícito sobre seu rastreamento de dependência e acionamento de atualizações. Ele espera uma função fábrica, que recebe as funções `track` e` trigger` como argumentos e deve retornar um objeto com `get` e` set`.

- Exemplo usando uma referência personalizada para implementar _debounce_ (prática que visa diminuir a quantidade de eventos disparados) com `v-model`:

  ```html
  <input v-model="text" />
  ```

  ```js
  function useDebouncedRef(value, delay = 200) {
    let timeout
    return customRef((track, trigger) => {
      return {
        get() {
          track()
          return value
        },
        set(newValue) {
          clearTimeout(timeout)
          timeout = setTimeout(() => {
            value = newValue
            trigger()
          }, delay)
        }
      }
    })
  }

  export default {
    setup() {
      return {
        text: useDebouncedRef('olá')
      }
    }
  }
  ```

**Tipando:**

```ts
function customRef<T>(factory: CustomRefFactory<T>): Ref<T>

type CustomRefFactory<T> = (
  track: () => void,
  trigger: () => void
) => {
  get: () => T
  set: (value: T) => void
}
```

## `shallowRef`

Cria uma referência que rastreia a mutação do seu próprio `.value`, mas não torna seu valor reativo.

```js
const foo = shallowRef({})
// alterar o valor da referência é reativo
foo.value = {}
// mas o "value" não será convertido.
isReactive(foo.value) // false
```

**Veja também**: [Criação de Valores Reativos Avulsos como `refs`](../guide/reactivity-fundamentals.html#criacao-de-valores-reativos-avulsos-como-refs)

## `triggerRef`

Execute quaisquer efeitos vinculados a um [`shallowRef`](#shallowref) manualmente.

```js
const shallow = shallowRef({
  greet: 'Olá, mundo!'
})

// Mostra "Olá, mundo!" só na primeira execução
watchEffect(() => {
  console.log(shallow.value.greet)
})

// Isso não acionará o efeito porque a referência é rasa (shallow)
shallow.value.greet = 'Olá, universo!'

// Mostra "Olá, universo!"
triggerRef(shallow)
```

**Veja também:** [Computado e Observado - watchEffect](./computed-watch-api.html#watcheffect)