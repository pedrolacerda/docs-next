# Limitações na Detecção de Mudanças na v2

> Esta página se aplica apenas ao Vue 2.x e inferiores, e assume que você já tenha lido a [Seção de Reatividade](reactivity.md). Por favor leia esta seção primeiro.

Devido a limitações no Javascript, existem alguns tipos de mudanças que o Vue **não consegue detectar**. No entanto, existem formas de contorná-las para preservar a reatividade.

### Para Objetos

Vue não consegue detectar a inserção ou remoção de propriedades. Como o Vue realiza o processo de conversão para *getter*/*setter* durante a inicialização da instância, uma propriedade precisa estar presente no objeto `data` para que o Vue possa convertê-lo e torná-lo reativo. Por exemplo:

```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` agora é reativo

vm.b = 2
// `vm.b` NÃO é reativo
```

Vue não permite adicionar dinamicamente novas propriedades ao nível raiz de uma instância já criada. Contudo, é possível adicionar propriedades reativas a um objeto aninhado usando o método `Vue.set(object, propertyName, value)`:

```js
Vue.set(vm.someObject, 'b', 2)
```

Você também pode utilizar o método de instância `vm.$set`, que é uma referência ao método global `Vue.set`:

```js
this.$set(this.someObject, 'b', 2)
```

Às vezes você pode querer atribuir mais de uma propriedade em um objeto existente, por exemplo usando `Object.assign()` ou `_.extend()`. No entanto, novas propriedades adicionadas ao objeto não irão acionar mudanças. Nestes casos, crie um objeto novo com propriedades de ambos o original e o alterado:

```js
// invés de `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```

### Para Arrays

Vue não consegue detectar as seguintes mudanças em um array:

1. Quando você atribui diretamente um item através do índice, e.g. `vm.items[indexOfItem] = newValue`
2. Quando você modifica o comprimento do array, e.g. `vm.items.length = newLength`

Por exemplo:

```js
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // NÃO é reativo
vm.items.length = 2 // NÃO é reativo
```

Para contornar a limitação 1, ambas as instruções seguintes farão o mesmo que `vm.items[indexOfItem] = newValue`, mas também acionarão as atualizações no estado do sistema reativo:

```js
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
```

```js
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

Você pode também usar o método [`vm.$set`](https://vuejs.org/v2/api/#vm-set) da instância que é uma referência ao método global `Vue.set`:

```js
vm.$set(vm.items, indexOfItem, newValue)
```

Para lidar com a limitação 2, você pode usar o método `splice`:

```js
vm.items.splice(newLength)
```

## Declarando Propriedades Reativas

Como o Vue não permite inserir propriedades ao nível raiz dinamicamente, você tem que inicializar a instância de um componente declarando todas as propriedades de nível raiz já inicializadas, mesmo que com valores vazios:

```js
var vm = new Vue({
  data: {
    // declara `message` com o valor vazio
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// atribui valor a `message` posteriormente
vm.message = 'Hello!'
```

Se você não declarar a propriedade `message` no objeto data, Vue te avisará que a função *render* está tentando acessar uma propriedade que não exite.

Existem razões técnicas por tráz desta restrição - elas eliminam um conjunto de casos extremos no sistema de rastreamento de dependências, além de fazer com que as instâncias dos componenntes funcionem melhor com sistemas de checagem de tipos. Porém há também uma consideração importante em termos de manutenção de código: o objeto `data` funciona como um *schema* para o estado do seu componente. Declarando todas as propriedades reativas inicialmente faz com que o código do componente seja mais fácil de entender quando revisitado posteriormente ou lido por outro desenvolvedor.

## Fila de Atualizações Assíncrona

No caso de você não ter notado ainda, Vue realiza atualizações no DOM **assincronamente**. Sempre que uma mudança de dados é observada, é aberta uma fila e armazenado em *buffer* todas as alterações que ocorreram no mesmo *loop* de eventos. Se o mesmo observador é acionado muitas vezes, ele será inserido na fila apenas uma vez. Essa eliminação de duplicatas em *buffer* é importante para evitar cálculos e manipulações do DOM desnecessários. Então, no próximo *"tick"* de *loop* de eventos, Vue esvazia a fila e executa o trabalho atual (já desduplicado). Internamente o Vue tenta utilizar nativamente `Promise.then`, `MutationObserver`, e `setImmediate` para a enfileiração e retorna para o `setTimeout(fn, 0)` em caso de pouco suporte.

Por exemplo, quando você atribui `vm.someData = 'new value'`, o componente não renderizará novamente de imediato. Ele irá atualizar no próximo *"tick"*, quando a fila for esvaziada. Na maioria dos casos nós não precisamos nos preocupar com isso, mas isto pode complicar quando você quer fazer algo que dependa do estado do DOM pós-atualização. Apesar de Vue.js geralmente encorajar os desenvolvedores a pensar de um modo *"data-driven"* e evitar tocar no DOM diretamente, em alguns casos pode ser necessário sujar as mãos. A fim de esperar até o Vue.js ter finalizado a atualização do DOM depois de uma mudança de dados, você pode usar `Vue.nextTick(callback)` imediatamente depois da mudança dos dados. O *callback* irá ser chamado depois que o DOM for atualizado. Por exemplo: 

```html
<div id="example">{{ message }}</div>
```

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'new message' // modifica dados
vm.$el.textContent === 'new message' // false
Vue.nextTick(function() {
  vm.$el.textContent === 'new message' // true
})
```

Existe também o método de instância `vm.$nextTick()`, que é especialmente conveniente dentro de componentes, pois não precisa da global `Vue` e seus *callback's* e o contexto da variável `this` será automaticamente vinculado ao componente atual:

```js
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function() {
    return {
      message: 'não atualizado'
    }
  },
  methods: {
    updateMessage: function() {
      this.message = 'atualizado'
      console.log(this.$el.textContent) // => 'não atualizado'
      this.$nextTick(function() {
        console.log(this.$el.textContent) // => 'atualizado'
      })
    }
  }
})
```

Como `$nextTick()` retorna uma *promise*, você pode ter o mesmo efeito que acima usando a nova sintaxe [ES2017 async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function): 

```js
  methods: {
    updateMessage: async function () {
      this.message = 'atualizado'
      console.log(this.$el.textContent) // => 'não atualizado'
      await this.$nextTick()
      console.log(this.$el.textContent) // => 'atualizado'
    }
  }
```
