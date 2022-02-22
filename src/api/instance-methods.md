# Métodos de Instância

## $watch

- **Argumentos:**

  - `{string | Function} source`
  - `{Function | Object} callback`
  - `{Object} options (optional)`
    - `{boolean} deep`
    - `{boolean} immediate`
    - `{string} flush`

- **Retorno:** `{Function} unwatch`

- **Uso:**

  Observa uma propriedade reativa ou uma função computada na instância do componente esperando alterações. O _callback_ é chamado com o novo valor e o valor antigo para a propriedade fornecida. Só podemos passar o nome da propriedade `data`, `props` ou `computed` de nível superior como uma string. Para expressões mais complexas ou propriedades aninhadas, use uma função.

- **Exemplo:**

  ```js
  const app = createApp({
    data() {
      return {
        a: 1,
        b: 2,
        c: {
          d: 3,
          e: 4
        }
      }
    },
    created() {
      // nome da propriedade de nível superior
      this.$watch('a', (newVal, oldVal) => {
        // faça algo
      })

      // função para observar uma única propriedade aninhada
      this.$watch(
        () => this.c.d,
        (newVal, oldVal) => {
          // faça algo
        }
      )

      // função para observar uma expressão complexa
      this.$watch(
        // toda vez que a expressão `this.a + this.b` produz um resultado diferente,
        // o manipulador será chamado. É como se estivéssemos observando
        // um dado computado sem precisarmos definí-lo
        () => this.a + this.b,
        (newVal, oldVal) => {
          // faça algo
        }
      )
    }
  })
  ```

  Quando o valor observado é um objeto ou array, quaisquer alterações em suas propriedades ou elementos não acionarão o inspetor porque fazem referência ao mesmo objeto/array:

  ```js
  const app = createApp({
    data() {
      return {
        article: {
          text: 'Vue é incrível!'
        },
        comments: ['De fato!', 'Concordo']
      }
    },
    created() {
      this.$watch('article', () => {
        console.log('Artigo alterado!')
      })

      this.$watch('comments', () => {
        console.log('Comentários alterados!')
      })
    },
    methods: {
      // Esses métodos não acionarão um observador pois alteramos apenas uma propriedade do object/array,
      // não o objeto/array em si
      changeArticleText() {
        this.article.text = 'Vue 3 é incrível'
      },
      addComment() {
        this.comments.push('Novo comentário')
      },

      // Esses métodos acionarão um observador pois substituímos o objeto/array completamente
      changeWholeArticle() {
        this.article = { text: 'Vue 3 é incrível' }
      },
      clearComments() {
        this.comments = []
      }
    }
  })
  ```

  `$watch` retorna uma função `unwatch` que para de disparar o _callback_:

  ```js
  const app = createApp({
    data() {
      return {
        a: 1
      }
    }
  })

  const vm = app.mount('#app')

  const unwatch = vm.$watch('a', cb)
  // depois, desmonta o observador
  unwatch()
  ```

- **Opção: `deep`**

  Para também detectar mudanças de valores aninhados dentro de Objects, você precisa passar `deep: true` no argumento de opções. Esta opção também pode ser usada para observar mutações de array.

  > Nota: ao mudar (ao invés de substituir) um Object ou um Array e observar com a opção `deep`, o valor antigo será o mesmo que o novo valor porque eles fazem referência ao mesmo Object/Array. O Vue não mantém uma cópia do valor pré-modificação.

  ```js
  vm.$watch('someObject', callback, {
    deep: true
  })
  vm.someObject.nestedValue = 123
  // callback é acionado
  ```

- **Opção: `immediate`**

  Passar `immediate: true` na opção acionará o _callback_ imediatamente com o valor atual da expressão:

  ```js
  vm.$watch('a', callback, {
    immediate: true
  })
  // `callback` é acionado imediatamente com o valor atual de `a`
  ```

  Observe que com a opção `immediate` você não poderá dar _unwatch_ na propriedade fornecida na primeira chamada de _callback_.

  ```js
  // Isso causará um erro
  const unwatch = vm.$watch(
    'value',
    function() {
      doSomething()
      unwatch()
    },
    { immediate: true }
  )
  ```

  Se você ainda quiser chamar uma função _unwatch_ dentro do _callback_, verifique primeiro sua disponibilidade:

  ```js
  let unwatch = null

  unwatch = vm.$watch(
    'value',
    function() {
      doSomething()
      if (unwatch) {
        unwatch()
      }
    },
    { immediate: true }
  )
  ```

- **Opção: flush**

  A opção `flush` permite maior controle sobre o momento de acionamento do _callback_. Pode ser definido como `'pre'`, `'post'` ou `'sync'`.

  O valor padrão é `'pre'`, que especifica que o _callback_ deve ser invocado antes da renderização. Isso permite que o _callback_ atualize outros valores antes que o _template_ seja executado.

  O valor `'post'` pode ser usado para adiar o _callback_ até depois da renderização. Isso deve ser usado se o _callback_ precisar acessar o DOM atualizado ou componentes filho por meio de `$refs`.

  Se `flush` for definido como `'sync'`, o _callback_ será chamado de forma síncrona, assim que o valor mudar.

  Para ambos `'pre'` e `'post'`, o _callback_ é armazenado em _buffer_ usando uma fila. O _callback_ será adicionado à fila apenas uma vez, mesmo que o valor observado seja alterado várias vezes. Os valores provisórios serão ignorados e não serão passados ​​para o _callback_.

  O armazenamento em _buffer_ do _callback_ não apenas melhora o desempenho, mas também ajuda a garantir a consistência dos dados. Os observadores não serão acionados até que o código que executa as atualizações de dados seja concluído.

  Observadores `'sync'` devem ser usados ​​com moderação, pois eles não têm esses benefícios.

  Para obter mais informações sobre `flush`, consulte [Momento de Limpeza do Efeito](../guide/reactivity-computed-watchers.html#momento-de-limpeza-do-efeito).

- **Ver também:** [Observadores](../guide/computed.html#observadores)

## `$emit`

- **Argumentos:**

  - `{string} eventName`
  - `...args (opcional)`

  Acionar um evento na instância atual. Quaisquer argumentos adicionais serão passados ​​para a função _callback_ do escutador.

- **Exemplos:**

  Usando `$emit` com apenas um nome de evento:

  ```html
  <div id="emit-example-simple">
    <welcome-button v-on:welcome="sayHi"></welcome-button>
  </div>
  ```

  ```js
  const app = createApp({
    methods: {
      sayHi() {
        console.log('Oi!')
      }
    }
  })

  app.component('welcome-button', {
    emits: ['welcome'],
    template: `
      <button v-on:click="$emit('welcome')">
        Clique-me para as boas vindas
      </button>
    `
  })

  app.mount('#emit-example-simple')
  ```

  Usando `$emit` com argumentos adicionais:

  ```html
  <div id="emit-example-argument">
    <advice-component v-on:advise="showAdvice"></advice-component>
  </div>
  ```

  ```js
  const app = createApp({
    methods: {
      showAdvice(advice) {
        alert(advice)
      }
    }
  })

  app.component('advice-component', {
    emits: ['advise'],
    data() {
      return {
        adviceText: 'Alguns conselhos'
      }
    },
    template: `
      <div>
        <input type="text" v-model="adviceText">
        <button v-on:click="$emit('advise', adviceText)">
          Clique em mim para enviar conselhos
        </button>
      </div>
    `
  })

  app.mount('#emit-example-argument')
  ```

- **Ver também:**
  - [Opção `emits`](./options-data.html#emits)
  - [Emitindo um Valor Com um Evento](../guide/component-basics.html#emitindo-um-valor-com-um-evento)

## `$forceUpdate`

- **Uso:**

  Força a instância do componente a renderizar novamente. Observe que isso não afeta todos os componentes filho, apenas a própria instância e os componentes filho com conteúdo de _slot_ inserido.

## `$nextTick`

- **Argumentos:**

  - `{Function} callback (opcional)`

- **Uso:**

  Adie o _callback_ para ser executado após o próximo ciclo de atualização do DOM. Use-o imediatamente após alterar alguns dados para aguardar a atualização do DOM. Isso é o mesmo que o `nextTick` global, exceto que o contexto `this` do _callback_ é automaticamente vinculado à instância que chama esse método.

- **Exemplo:**

  ```js
  createApp({
    // ...
    methods: {
      // ...
      example() {
        // modifica os dados
        this.message = 'changed'
        // DOM ainda não foi atualizado
        this.$nextTick(function() {
          // DOM agora está atualizado
          // `this` está vinculado à instância atual
          this.doSomethingElse()
        })
      }
    }
  })
  ```

- **Ver também:** [nextTick](global-api.html#nexttick)
