# Dados Computados e Observadores

> Esta seção usa a sintaxe de [Componentes Single-File](../guide/single-file-component.html) para exemplos de código

## `computed`

Recebe uma função _getter_ e retorna um objeto [ref](./refs-api.html#ref) reativo imutável para o valor retornado do _getter_.

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // erro
```

Alternativamente, ele pode usar um objeto com as funções `get` e `set` para criar um objeto ref gravável.

```js
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: val => {
    count.value = val - 1
  }
})

plusOne.value = 1
console.log(count.value) // 0
```

**Tipando:**

```ts
// somente leitura
function computed<T>(
  getter: () => T,
  debuggerOptions?: DebuggerOptions
): Readonly<Ref<Readonly<T>>>

// gravável
function computed<T>(
  options: {
    get: () => T
    set: (value: T) => void
  },
  debuggerOptions?: DebuggerOptions
): Ref<T>

interface DebuggerOptions {
  onTrack?: (event: DebuggerEvent) => void
  onTrigger?: (event: DebuggerEvent) => void
}

interface DebuggerEvent {
  effect: ReactiveEffect
  target: any
  type: OperationTypes
  key: string | symbol | undefined
}
```

## `watchEffect`

Executa uma função imediatamente enquanto rastreia reativamente suas dependências e a executa novamente sempre que as dependências são alteradas.

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> loga 0

setTimeout(() => {
  count.value++
  // -> loga 1
}, 100)
```

**Tipando:**

```ts
function watchEffect(
  effect: (onInvalidate: InvalidateCbRegistrator) => void,
  options?: WatchEffectOptions
): StopHandle

interface WatchEffectOptions {
  flush?: 'pre' | 'post' | 'sync' // default: 'pre'
  onTrack?: (event: DebuggerEvent) => void
  onTrigger?: (event: DebuggerEvent) => void
}

interface DebuggerEvent {
  effect: ReactiveEffect
  target: any
  type: OperationTypes
  key: string | symbol | undefined
}

type InvalidateCbRegistrator = (invalidate: () => void) => void

type StopHandle = () => void
```

**Ver também**: [Guia do `watchEffect`](../guide/reactivity-computed-watchers.html#watcheffect)

## `watchPostEffect` <Badge text="3.2+" />

Apelido de `watchEffect` com a opção `flush: 'post'`.

## `watchSyncEffect` <Badge text="3.2+" />

Apelido de `watchEffect` com a opção `flush: 'sync'`.

## `watch`

A API do `watch` é o equivalente exato da API de Opções [this.\$watch](./instance-methods.html#watch) (e a opção [watch](./options-data.html#watch) correspondente). `watch` requer a observação de uma fonte de dados específica e aplica efeitos colaterais em uma função _callback_ separada. Também é preguiçoso por padrão - ou seja, o _callback_ só é chamado quando a fonte monitorada é alterada.

- Comparado com [watchEffect](#watcheffect), `watch` nos permite:

  - Executar o efeito colateral preguiçosamente;
  - Ser mais específico sobre qual estado deve fazer com que o observador seja executado novamente;
  - Acessar o valor anterior e o atual do estado observado.

### Observando uma Única Fonte

A fonte de dados do observador pode ser uma função _getter_ que retorna um valor ou diretamente um [ref](./refs-api.html#ref):

```js
// observando um getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// observando diretamente uma ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

### Observando Várias Fontes

Um observador também pode observar várias fontes ao mesmo tempo usando um array:

```js
watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
  /* ... */
})
```

### Comportamento Compartilhado com `watchEffect`

`watch` compartilha comportamento com [`watchEffect`](#watcheffect) em termos de [interrupção manual](../guide/reactivity-computed-watchers.html#parando-o-observador), [invalidação de efeito colateral](../guide/reactivity-computed-watchers.html#invalidacao-de-efeito-colateral) (com `onInvalidate` passado para o _callback_ como o terceiro argumento), [momento de limpeza](../guide/reactivity-computed-watchers.html#momento-de-limpeza-do-efeito) e [depuração](../guide/reactivity-computed-watchers.html#depuracao-do-observador).

**Tipando:**

```ts
// observando uma única fonte
function watch<T>(
  source: WatcherSource<T>,
  callback: (
    value: T,
    oldValue: T,
    onInvalidate: InvalidateCbRegistrator
  ) => void,
  options?: WatchOptions
): StopHandle

// observando várias fontes
function watch<T extends WatcherSource<unknown>[]>(
  sources: T
  callback: (
    values: MapSources<T>,
    oldValues: MapSources<T>,
    onInvalidate: InvalidateCbRegistrator
  ) => void,
  options? : WatchOptions
): StopHandle

type WatcherSource<T> = Ref<T> | (() => T)

type MapSources<T> = {
  [K in keyof T]: T[K] extends WatcherSource<infer V> ? V : never
}

// veja a tipagem de `watchEffect` para opções compartilhadas
interface WatchOptions extends WatchEffectOptions {
  immediate?: boolean // default: false
  deep?: boolean
}
```

**Ver também**: [Guia do `watch`](../guide/reactivity-computed-watchers.html#watch)
