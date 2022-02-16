# Propriedades de Dados e Métodos

<VideoLesson href="https://vueschool.io/lessons/methods-in-vue-3?friend=vuejs" title="Aprenda a usar métodos na Vue School">Aprenda a trabalhar com dados e métodos com uma aula gratuita da Vue School</VideoLesson>

## Propriedades de Dados

A opção `data` para um componente é uma função. Vue chama essa função como parte da criação de uma nova instância de componente. Ela deve retornar um objeto, que o Vue irá envolver em seu sistema de reatividade e armazenar na instância do componente como `$data`. Por conveniência, quaisquer propriedades de nível superior desse objeto também são expostas diretamente por meio da instância do componente:

```js
const app = Vue.createApp({
  data() {
    return { count: 4 }
  }
})

const vm = app.mount('#app')

console.log(vm.$data.count) // => 4
console.log(vm.count)       // => 4

// Atribuir um valor a vm.count também atualizará $data.count
vm.count = 5
console.log(vm.$data.count) // => 5

// ... e vice-versa
vm.$data.count = 6
console.log(vm.count) // => 6
```

Essas propriedades de instância são adicionadas apenas quando a instância é criada pela primeira vez, então você precisa garantir que todas estejam presentes no objeto retornado pela função `data`. Onde necessário, use `null`, `undefined` ou algum outro valor que reserve o lugar para propriedades onde o valor desejado ainda não está disponível.

É possível adicionar uma nova propriedade diretamente à instância do componente sem incluí-la em `data`. No entanto, como essa propriedade não é suportada pelo objeto reativo `$data`, ela não será rastreada automaticamente pelo [sistema de reatividade do Vue](reactivity.html).

O Vue usa o prefixo `$` ao expor suas próprias APIs integradas por meio da instância do componente. Ele também reserva o prefixo `_` para propriedades internas. Você deve evitar usar nomes para propriedades `data` de nível superior que começam com qualquer um desses caracteres.

## Métodos

Para adicionar métodos a uma instância do componente, usamos a opção `methods`. Isto deve ser um objeto contendo os métodos desejados:

```js
const app = Vue.createApp({
  data() {
    return { count: 4 }
  },
  methods: {
    increment() {
      // `this` irá se referir à instância do componente
      this.count++
    }
  }
})

const vm = app.mount('#app')

console.log(vm.count) // => 4

vm.increment()

console.log(vm.count) // => 5
```

O Vue vincula automaticamente o valor de `this` nos `methods` para que sempre se refira à instância do componente. Isso garante que um método retenha o valor de `this` correto se for usado como um escutador de evento ou _callback_. Você deve evitar o uso de _arrow functions_ ao definir `methods`, pois isso impede que o Vue vincule o valor de `this` apropriado.

Assim como todas as outras propriedades da instância do componente, os métodos em `methods` são acessíveis a partir do _template_ do componente. Dentro de um _template_, eles são mais comumente usados como escutadores de eventos:

```html
<button @click="increment">Voto positivo</button>
```

No exemplo acima, o método `increment` será chamado quando o `<button>` for clicado.

Também é possível chamar um método diretamente de um _template_. Como veremos em breve, geralmente é melhor usar um [dado computado](computed.html) em vez disso. No entanto, usar um método pode ser útil em cenários onde os dados computados não são uma opção viável. Você pode chamar um método em qualquer lugar em que um _template_ suporte expressões JavaScript:

```html
<span :title="toTitleDate(date)">
  {{ formatDate(date) }}
</span>
```

Se os métodos `toTitleDate` ou `formatDate` acessarem qualquer dado reativo, ele será rastreado como uma dependência de renderização, como se tivesse sido usado diretamente no template.

Os métodos chamados a partir de um _template_ não devem ter efeitos colaterais, como alteração de dados ou acionamento de processos assíncronos. Se você se sentir tentado a fazer isso, provavelmente deve usar um [gatilho de ciclo de vida](instance.html#gatilhos-de-ciclo-de-vida).

### _Debouncing_ e _Throttling_

O Vue não inclui suporte integrado para _debouncing_ ou _throttling_, mas pode ser implementado usando bibliotecas como [Lodash](https://lodash.com/).

Nos casos em que um componente é usado apenas uma vez, o _debouncing_ pode ser aplicado diretamente em `methods`:

```html
<script src="https://unpkg.com/lodash@4.17.20/lodash.min.js"></script>
<script>
  Vue.createApp({
    methods: {
      // Debouncing com Lodash
      click: _.debounce(function() {
        // ... responder ao clique ...
      }, 500)
    }
  }).mount('#app')
</script>
```

No entanto, essa abordagem é potencialmente problemática para componentes que são reutilizados, porque todos eles compartilharão a mesma função _debounced_. Para manter as instâncias do componente independentes umas das outras, podemos adicionar a função _debounced_ no gatilho do ciclo de vida `created`:

```js
app.component('save-button', {
  created() {
    // Debouncing com Lodash
    this.debouncedClick = _.debounce(this.click, 500)
  },
  unmounted() {
    // Cancela o cronômetro quando o componente é removido
    this.debouncedClick.cancel()
  },
  methods: {
    click() {
      // ... responder ao clique ...
    }
  },
  template: `
    <button @click="debouncedClick">
      Save
    </button>
  `
})
```
