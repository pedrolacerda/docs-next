# Eventos Customizados

> Esta página assume que você já leu o [Básico sobre Componentes](component-basics.md). Leia lá antes se você está iniciando com componentes.

## Nomes de Eventos

Como componentes e props, nomes de eventos fornecem uma transformação automática de maiúsculas e minúsculas. Se você emitir um evento do componente filho em *camelCase*, poderá adicionar um escutador em *kebab-case* no pai:

```js
this.$emit('myEvent')
```

```html
<my-component @my-event="doSomething"></my-component>
```

Assim como com a [escrita de props](/guide/component-props.html#prop-casing-camelcase-vs-kebab-case), recomendamos o uso de escutadores de evento em *kebab-case* ao usar *templates* no DOM.

## Definindo Eventos Customizados

<VideoLesson href="https://vueschool.io/lessons/defining-custom-events-emits?friend=vuejs" title="Aprenda à definir quais eventos um componente pode emitir com a Vue School (em inglês)">Assista um vídeo grátis sobre como definir Eventos Customizados na Vue School (em inglês)</VideoLesson>

Os eventos emitidos podem ser definidos no componente através da opção `emits`.

```js
app.component('custom-form', {
  emits: ['inFocus', 'submit']
})
```

Quando um evento nativo (por exemplo, `click`) for definido na opção `emits`, o evento do componente será usado **ao invés** de um escutador de evento nativo.

::: tip DICA
Recomenda-se definir todos os eventos emitidos para documentar melhor como o componente deve funcionar.
:::

### Validar Eventos Emitidos

Semelhante à validação do tipo de propriedades, um evento emitido pode ser validado se for definido com a sintaxe object em vez da sintaxe array.

Para adicionar validação, o evento recebe uma função que recebe os argumentos passados ​​para a chamada `$emit` e retorna um booleano para indicar se o evento é válido ou não.

```js
app.component('custom-form', {
  emits: {
    // Sem validação
    click: null,

    // Validar evento submit
    submit: ({ email, password }) => {
      if (email && password) {
        return true
      } else {
        console.warn('Argumentos inválidos para o evento submit!')
        return false
      }
    }
  },
  methods: {
    submitForm(email, password) {
      this.$emit('submit', { email, password })
    }
  }
})
```

## Argumentos do `v-model`

Por padrão, em um componente o `v-model` usa `modelValue` como propriedade e `update:modelValue` como evento. Podemos modificar esses nomes passando um argumento para `v-model`:

```html
<my-component v-model:title="bookTitle"></my-component>
```

Nesse caso, o componente filho espera a propriedade `title` e emite o evento `update:title` para sincronizar:

```js
app.component('my-component', {
  props: {
    title: String
  },
  emits: ['update:title'],
  template: `
    <input
      type="text"
      :value="title"
      @input="$emit('update:title', $event.target.value)">
  `
})
```


## Múltiplos Vínculos `v-model`

Aproveitando a capacidade de direcionar uma determinada propriedade e evento como aprendemos antes em [argumentos do `v-model`](#argumentos-do-v-model), agora podemos criar múltiplos vínculos de `v-model` em uma única instância do componente.

Cada `v-model` será sincronizado com uma propriedade diferente, sem a necessidade de opções extras no componente:

```html
<user-name
  v-model:first-name="firstName"
  v-model:last-name="lastName"
></user-name>
```

```js
app.component('user-name', {
  props: {
    firstName: String,
    lastName: String
  },
  emits: ['update:firstName', 'update:lastName'],
  template: `
    <input
      type="text"
      :value="firstName"
      @input="$emit('update:firstName', $event.target.value)">

    <input
      type="text"
      :value="lastName"
      @input="$emit('update:lastName', $event.target.value)">
  `
})
```

<common-codepen-snippet title="Múltiplos v-models" slug="bGpZrLy" tab="html,result" user="lucianotonet" name="L. Tonet" />

## Manipulando Modificadores do `v-model`

Quando estávamos aprendendo sobre interligações em elementos _input_, vimos que `v-model` tem [modificadores embutidos](/guide/forms.html#modifiers) - `.trim`, `.number` e `.lazy`. Em alguns casos, entretanto, você também pode querer adicionar seus próprios modificadores personalizados.

Vamos criar um modificador personalizado de exemplo, `capitalize`, que coloca em maiúscula a primeira letra da string fornecida pela vinculação `v-model`.

Modificadores adicionados ao `v-model` de um componente serão fornecidos ao componente por meio da propriedade `modelModifiers`. No exemplo abaixo, criamos um componente que contém uma propriedade `modelModifiers` cujo padrão é um objeto vazio.

Observe que quando o gatilho de ciclo de vida `created` do componente é acionado, a propriedade `modelModifiers` contém `capitalize` e seu valor é `true` - devido ao fato de ser definido na vinculação `v-model.capitalize="myText"`.

```html
<my-component v-model.capitalize="myText"></my-component>
```

```js
app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  template: `
    <input type="text"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)">
  `,
  created() {
    console.log(this.modelModifiers) // { capitalize: true }
  }
})
```

Agora que temos nossa propriedade configurada, podemos verificar as chaves do objeto `modelModifiers` e escrever um manipulador para alterar o valor emitido. No código a seguir, colocaremos a string em maiúscula sempre que o elemento `<input />` disparar um evento `input`.

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      myText: ''
    }
  }
})

app.component('my-component', {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  },
  emits: ['update:modelValue'],
  methods: {
    emitValue(e) {
      let value = e.target.value
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
      }
      this.$emit('update:modelValue', value)
    }
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`
})

app.mount('#app')
```

Para os vínculos `v-model` com argumentos, o nome da propriedade gerada será `arg + "Modifiers"`:

```html
<my-component v-model:description.capitalize="myText"></my-component>
```

```js
app.component('my-component', {
  props: ['description', 'descriptionModifiers'],
  emits: ['update:description'],
  template: `
    <input type="text"
      :value="description"
      @input="$emit('update:description', $event.target.value)">
  `,
  created() {
    console.log(this.descriptionModifiers) // { capitalize: true }
  }
})
```
