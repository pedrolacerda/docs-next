# API's Básicas de Reatividade

> Esta seção usa sintaxe de [Componente Single-File](../guide/single-file-component.html) para exemplos de código

## `reactive`

Retorna uma cópia reativa do objeto.

```js
const obj = reactive({ count: 0 })
```

A conversão reativa é "profunda" - ela afeta todas as propriedades aninhadas. Na implementação baseada no [Proxy da ES2015](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), o *proxy* retornado **não** é igual ao objeto original. Recomenda-se trabalhar exclusivamente com o *proxy* reativo e evitar depender do objeto original.

**Tipagem:**

```ts
function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
```

::: tip Nota
`reactive` irá desempacotar profundamente todas as [refs](./refs-api.html#ref), enquanto mantém a reativida da ref

```ts
const count = ref(1)
const obj = reactive({ count })

// A ref será desempacotada
console.log(obj.count === count.value) // true

// Isto vai atualizar `obj.count`
count.value++
console.log(count.value) // 2
console.log(obj.count) // 2

// Isto vai apenas atualizar a ref `count`
obj.count++
console.log(obj.count) // 3
console.log(count.value) // 3
```

:::

::: warning Importante
Ao atribuir um [ref](./refs-api.html#ref) a uma propriedade `reactive`, este ref será automaticamente desempacotado.

```ts
const count = ref(1)
const obj = reactive({})

obj.count = count

console.log(obj.count) // 1
console.log(obj.count === count.value) // true
```

:::

## `readonly`

Captura um objeto (reativo ou simples) ou um [ref](./refs-api.html#ref) e retorna um *proxy* somente leitura para o original. Um *proxy* somente leitura é profundo: qualquer propriedade aninhada acessada também será somente leitura.

```js
const original = reactive({ count: 0 })

const copy = readonly(original)

watchEffect(() => {
  // realizando o rastreamento da reatividade
  console.log(copy.count)
})  

// a mutação do "original" executará observadores confiando na cópia
original.count++

// a mutação da cópia falhará e resultará em um aviso
copy.count++ // aviso!
```

Assim como acontece com [`reactive`](#reactive), se alguma propriedade usar `ref`, este será automaticamente desempacotado quando for acessado pelo proxy:

```js
const raw = {
  count: ref(123)
}

const copy = readonly(raw)

console.log(raw.count.value) // 123
console.log(copy.count) // 123
```

## `isProxy`

Verifica se um objeto é um *proxy* criado por [`reactive`](#reactive) ou [`readonly`](#readonly).

## `isReactive`

Verifica se um objeto é um *proxy* reativo criado por [`reactive`](#reactive).

```js
import { reactive, isReactive } from 'vue'
export default {
  setup() {
    const state = reactive({
      name: 'John'
    })
    console.log(isReactive(state)) // -> true
  }
}
```

Ele também retorna `true` se o *proxy* foi criado por [`readonly`](#readonly), mas está envolvendo outro *proxy* criado por [`reactive`](#reactive).

```js{7-15}
import { reactive, isReactive, readonly } from 'vue'
export default {
  setup() {
    const state = reactive({
      name: 'John'
    })
    // proxy "readonly" (somente leitura) criado de um objeto simples
    const plain = readonly({
      name: 'Mary'
    })
    console.log(isReactive(plain)) // -> false

    // proxy "readonly" (somente leitura) criado de um proxy "reactive"
    const stateCopy = readonly(state)
    console.log(isReactive(stateCopy)) // -> true
  }
}
```

## `isReadonly`

Verifica se um objeto é um *proxy* somente leitura criado por [`readonly`](#readonly).

## `toRaw`

Retorna o objeto bruto (*raw*) original de um *proxy* [`reactive`](#reactive) ou [`readonly`](#readonly). Esta é uma "válvula de escape" que pode ser usada para ler temporariamente sem incorrer em acesso de *proxy*/sobrecarga de rastreamento ou gravação sem acionar alterações. **Não** é recomendado manter uma referência persistente ao objeto original. Use com cuidado.

```js
const foo = {}
const reactiveFoo = reactive(foo)

console.log(toRaw(reactiveFoo) === foo) // true
```

## `markRaw`

Marca um objeto para que nunca seja convertido em *proxy*. Retorna o próprio objeto.

```js
const foo = markRaw({})
console.log(isReactive(reactive(foo))) // false

// também funciona quando aninhado dentro de outros objetos reativos
const bar = reactive({ foo })
console.log(isReactive(bar.foo)) // false
```

::: warning Aviso
`markRaw` e as APIs *shallowXXX* abaixo permitem que você desative seletivamente a conversão profunda reativa/somente leitura padrão e incorpore objetos brutos e não-*proxy* em seu gráfico de estado. Eles podem ser usados por vários motivos:

- Alguns valores simplesmente não devem se tornar reativos, por exemplo, uma instância complexa de classe de terceiros, ou um objeto de componente Vue.

- Ignorar a conversão de *proxy* pode fornecer melhorias de desempenho ao renderizar grandes listas com fontes de dados imutáveis.

Eles são considerados avançados porque a desativação bruta está apenas no nível raiz, portanto, se você definir um objeto bruto aninhado e não marcado em um objeto reativo e acessá-lo novamente, obterá a versão em *proxy* de volta. Isso pode levar a **riscos de identidade** - ou seja, realizar uma operação que depende da identidade do objeto, mas usando ambas a versão bruta e a versão *proxy* do mesmo objeto:

```js
const foo = markRaw({
  nested: {}
})

const bar = reactive({
  // embora `foo` esteja marcado como bruto, foo.nested não está.
  nested: foo.nested
})

console.log(foo.nested === bar.nested) // false
```

Riscos de identidade são geralmente raros. No entanto, para utilizar adequadamente essas APIs e ao mesmo tempo evitar riscos de identidade, é necessário um conhecimento sólido de como funciona o sistema de reatividade.
:::

## `shallowReactive`

Cria um *proxy* reativo que rastreia a reatividade de suas próprias propriedades, mas não executa conversão reativa profunda de objetos aninhados (expõe valores brutos).

```js
const state = shallowReactive({
  foo: 1,
  nested: {
    bar: 2
  }
})

// a mutação das próprias propriedades do "state" é reativa
state.foo++
// ...mas não converte objetos aninhados
isReactive(state.nested) // false
state.nested.bar++ // não reativo
```

Unlike [`reactive`](#reactive), any property that uses a [`ref`](/api/refs-api.html#ref) will **not** be automatically unwrapped by the proxy.

## `shallowReadonly`

Cria um *proxy* que torna suas próprias propriedades em somente leitura, mas não executa a conversão somente leitura profunda de objetos aninhados (expõe valores brutos).

```js
const state = shallowReadonly({
  foo: 1,
  nested: {
    bar: 2
  }
})

// a mutação das próprias propriedades do "state" irão falhar
state.foo++
// ...mas funciona em objetos aninhados
isReadonly(state.nested) // false
state.nested.bar++ // funciona
```

Unlike [`readonly`](#readonly), any property that uses a [`ref`](/api/refs-api.html#ref) will **not** be automatically unwrapped by the proxy.
