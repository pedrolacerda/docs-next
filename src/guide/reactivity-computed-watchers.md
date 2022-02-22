# Dados Computados e Observadores

> Esta seção usa a sintaxe de [componente de single-file](single-file-component.html) para exemplos de código

## Valores computados

Às vezes precisamos de um estado que depende de outro estado - no Vue isso é tratado com [dados computados](computed.html#computed-properties) do componente. Para criar diretamente um valor computado, podemos usar a função `computed`: ela pega uma função _getter_ e retorna um objeto reativo imutável [ref](reactivity-fundamentals.html#criacao-de-valores-reativos-avulsos-como-refs) para o valor retornado do _getter_.

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // erro
```

Alternativamente, ele pode receber um objeto com as funções `get` e `set` para criar um objeto _ref_ modificável.

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

### Depuração de Computados <Badge text="3.2+" />

`computed` aceita um segundo argumento com as opções `onTrack` e `onTrigger`:

- `onTrack` será chamado quando uma propriedade reativa ou _ref_ for rastreada como uma dependência.
- `onTrigger` será chamado quando o _callback_ do observador for acionado pela mutação de uma dependência.

Ambos os _callbacks_ receberão um evento depurador que contém informações sobre a dependência em questão. Recomenda-se colocar uma instrução `debugger` nesses _callbacks_ para inspecionar interativamente a dependência:

```js
const plusOne = computed(() => count.value + 1, {
  onTrack(e) {
    // acionado quando count.value é rastreado como uma dependência
    debugger
  },
  onTrigger(e) {
    // acionado quando count.value é modificado
    debugger
  }
})

// acessa plusOne, deve acionar onTrack
console.log(plusOne.value)

// muda count.value, deve acionar onTrigger
count.value++
```

`onTrack` e `onTrigger` só funcionam no modo de desenvolvimento.

## `watchEffect`

Para aplicar e _automaticamente reaplicar_ um efeito colateral baseado no estado reativo, podemos usar a função `watchEffect`. Ela executa uma função imediatamente enquanto rastreia de forma reativa suas dependências e a executa novamente sempre que as dependências são alteradas.

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> loga 0

setTimeout(() => {
  count.value++
  // -> loga 1
}, 100)
```

### Parando o Observador

Quando `watchEffect` é chamado durante a função [setup()](composition-api-setup.html) de um componente ou [gatilhos do ciclo de vida](composition-api-lifecycle-hooks.html), o observador é vinculado ao ciclo de vida do componente e será interrompido automaticamente quando o componente for desmontado.

Em outros casos, ele retorna um manipulador de parada que pode ser chamado para parar explicitamente o observador:

```js
const stop = watchEffect(() => {
  /* ... */
})

// depois
stop()
```

### Invalidação de Efeito Colateral

Às vezes, a função do efeito observado executará efeitos colaterais assíncronos que precisam ser limpos quando forem invalidados (ou seja, estado alterado antes que os efeitos possam ser concluídos). A função de efeito recebe uma função `onInvalidate` que pode ser usada para registrar um _callback_ de invalidação. Esse _callback_ de invalidação é chamado quando:

- o efeito está prestes a ser executado novamente
- o observador é parado (ou seja, quando o componente é desmontado se `watchEffect` for usado dentro de `setup()` ou gatilhos de ciclo de vida)

```js
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id foi alterado ou o observador está parado.
    // invalida a operação assíncrona pendente anteriormente
    token.cancel()
  })
})
```

Estamos registrando o _callback_ de invalidação por meio de uma função passada em vez de retorná-lo do _callback_ porque o valor retornado é importante para o tratamento de erros assíncronos. É muito comum que a função de efeito seja uma função assíncrona ao realizar a requisição de dados:

```js
const data = ref(null)
watchEffect(async onInvalidate => {
  onInvalidate(() => {
    /* ... */
  }) // registramos a função de limpeza antes que a Promise seja resolvida
  data.value = await fetchData(props.id)
})
```

Uma função assíncrona retorna implicitamente uma Promise, mas a função de limpeza precisa ser registrada imediatamente antes que a Promise seja resolvida. Além disso, o Vue depende da Promise retornada para lidar automaticamente com possíveis erros na cadeia de Promises.

### Momento de Limpeza do Efeito

O sistema de reatividade do Vue armazena em _buffer_ os efeitos invalidados e os libera de forma assíncrona para evitar invocação duplicada desnecessária quando há muitas mutações de estado acontecendo no mesmo "tick". Internamente, a função `update` de um componente também é um efeito observado. Quando um efeito de usuário é enfileirado, ele é invocado por padrão **antes** de todos os efeitos de `update` do componente:

```vue
<template>
  <div>{{ count }}</div>
</template>

<script>
export default {
  setup() {
    const count = ref(0)

    watchEffect(() => {
      console.log(count.value)
    })

    return {
      count
    }
  }
}
</script>
```

Neste exemplo:

- A contagem será registrada de forma síncrona na execução inicial.
- Quando `count` for modificado, o callback será chamado **antes** de o componente ser atualizado.

Nos casos em que um efeito de observador precisa ser executado novamente **após** atualizações de componentes (ou seja, ao trabalhar com [_Refs_ de _Template_](./composition-api-template-refs.md#observando-refs-de-template)), nós podemos passar um objeto `options` adicional com a opção `flush` (o padrão é `'pre'`):

```js
// dispara após atualizações de componentes para que você possa acessar o DOM atualizado
// Nota: isso também adiará a execução inicial do efeito até
// a primeira renderização do componente ser finalizada.
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'post'
  }
)
```

A opção `flush` também aceita `'sync'`, o que força o efeito a sempre disparar de forma síncrona. No entanto, isso é ineficiente e raramente deve ser necessário.

No Vue >= 3.2.0, os apelidos `watchPostEffect` e `watchSyncEffect` também podem ser usados ​​para tornar a intenção do código mais óbvia.

### Depuração do Observador

As opções `onTrack` e `onTrigger` podem ser usadas para depurar o comportamento de um observador.

- `onTrack` será chamado quando uma propriedade reativa ou _ref_ for rastreada como uma dependência.
- `onTrigger` será chamado quando o _callback_ do observador for acionado pela mutação de uma dependência.

Ambos os _callbacks_ receberão um evento depurador que contém informações sobre a dependência em questão. Recomenda-se colocar uma instrução `debugger` nesses _callbacks_ para inspecionar interativamente a dependência:

```js
watchEffect(
  () => {
    /* efeito colateral */
  },
  {
    onTrigger(e) {
      debugger
    }
  }
)
```

`onTrack` e `onTrigger` só funcionam no modo de desenvolvimento.

## `watch`

A API `watch` é o equivalente exato da propriedade [watch](computed.html#observadores) do componente. `watch` requer a observação de uma fonte de dados específica e aplica efeitos colaterais em um _callback_ separado. Também é preguiçoso por padrão - ou seja, o _callback_ só é chamado quando a fonte observada foi alterada.

- Comparado com [watchEffect](#watcheffect), `watch` nos permite:

  - Executar o efeito colateral preguiçosamente;
  - Ser mais específico sobre qual estado deve acionar o observador para executar novamente;
  - Acessar o valor anterior e atual do estado observado.

### Observando uma Única Fonte

Uma fonte de dados do observador pode ser uma função _getter_ que retorna um valor ou diretamente um `ref`:

``` js
// observando um getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// observando diretamente um ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

### Observando Várias Fontes

Um observador também pode observar várias fontes ao mesmo tempo usando um array:

```js
const firstName = ref('')
const lastName = ref('')

watch([firstName, lastName], (newValues, prevValues) => {
  console.log(newValues, prevValues)
})

firstName.value = 'John' // logs: ["John", ""] ["", ""]
lastName.value = 'Smith' // logs: ["John", "Smith"] ["John", ""]
```

No entanto, se você estiver alterando as duas fontes observadas simultaneamente na mesma função, o observador será executado apenas uma vez:

```js{9-13}
setup() {
  const firstName = ref('')
  const lastName = ref('')

  watch([firstName, lastName], (newValues, prevValues) => {
    console.log(newValues, prevValues)
  })

  const changeValues = () => {
    firstName.value = 'John'
    lastName.value = 'Smith'
    // logs: ["John", "Smith"] ["", ""]
  }

  return { changeValues }
}
```

Observe que várias alterações síncronas acionarão o observador apenas uma vez.

É possível forçar o observador a disparar após cada mudança usando a configuração `flush: 'sync'`, embora isso geralmente não seja recomendado. Como alternativa, [nextTick](/api/global-api.html#nexttick) pode ser usado para aguardar a execução do observador antes de fazer outras alterações. por exemplo.:

```js
const changeValues = async () => {
  firstName.value = 'John' // logs: ["John", ""] ["", ""]
  await nextTick()
  lastName.value = 'Smith' // logs: ["John", "Smith"] ["John", ""]
}
```

### Observando Objetos Reativos

Usar um observador para comparar valores de um array ou objeto que são reativos requer que ele tenha uma cópia feita apenas dos valores.

```js
const numbers = reactive([1, 2, 3, 4])

watch(
  () => [...numbers],
  (numbers, prevNumbers) => {
    console.log(numbers, prevNumbers)
  }
)

numbers.push(5) // logs: [1,2,3,4,5] [1,2,3,4]
```

A tentativa de verificar alterações de propriedades em um objeto ou array profundamente aninhado ainda exigirá que a opção `deep` seja verdadeira:

```js
const state = reactive({
  id: 1,
  attributes: {
    name: ''
  }
})

watch(
  () => state,
  (state, prevState) => {
    console.log('not deep', state.attributes.name, prevState.attributes.name)
  }
)

watch(
  () => state,
  (state, prevState) => {
    console.log('deep', state.attributes.name, prevState.attributes.name)
  },
  { deep: true }
)

state.attributes.name = 'Alex' // Loga: "deep" "Alex" "Alex"
```

No entanto, observar um objeto ou array reativo sempre retornará uma referência ao valor atual desse objeto para ambos o valor atual e anterior do estado. Para observar completamente objetos e arrays profundamente aninhados, pode ser necessária uma cópia profunda dos valores. Isso pode ser feito com um utilitário como [lodash.cloneDeep](https://lodash.com/docs/4.17.15#cloneDeep)

```js
import _ from 'lodash'

const state = reactive({
  id: 1,
  attributes: {
    name: ''
  }
})

watch(
  () => _.cloneDeep(state),
  (state, prevState) => {
    console.log(state.attributes.name, prevState.attributes.name)
  }
)

state.attributes.name = 'Alex' // Loga: "Alex" ""
```

### Comportamento Compartilhado com `watchEffect`

`watch` compartilha comportamento com [`watchEffect`](#watcheffect) em termos de [parada manual](#parando-o-observador), [invalidação de efeito colateral](#invalidacao-de-efeito-colateral) (com `onInvalidate` passado ao _callback_ como o terceiro argumento), [momento de limpeza](#momento-de-limpeza-do-efeito) e [depuração](#depuracao-do-observador).
