# Diretivas Customizadas

## Introdução

Em adição ao conjunto de diretivas existente por padrão em seu núcleo (como `v-model` ou `v-show`), o Vue te possibilita registrar diretivas customizadas. Note que no Vue, a forma primária de reuso e abstração de código são os componentes - no entanto, pode haver casos em que você precise um acesso de nível mais baixo aos elementos no DOM, e é aqui que as diretivas customizadas são úteis. Um exemplo seria acionar o foco em um elemento `input`, como esse:

<common-codepen-snippet title="Diretivas customizadas: exemplo básico" slug="JjdxaJW" :preview="false" />

Quando a página carrega, o elemento ganha o foco (nota: o atributo nativo autofocus não funciona no Safari para dispositivos móveis). Na verdade, se você não clicou em nada desde que visitou essa página, o input acima deve estar focado agora. Além disso, você pode clicar no botão `Rerun` e o input vai ganhar foco.

Agora vamos construir a diretiva que faz isso:

```js
const app = Vue.createApp({})
// Registra uma diretiva customizada global chamada `v-focus`
app.directive('focus', {
  // Quando o elemento vinculado é inserido no DOM
  mounted(el) {
    // Foca o elemento
    el.focus()
  }
})
```

Se você deseja registrar uma diretiva local em vez disso, os componentes também aceitam a opção `directives`:

```js
directives: {
  focus: {
    // definição da diretiva
    mounted(el) {
      el.focus()
    }
  }
}
```

Então no *template*, você pode usar o novo atributo `v-focus` em qualquer elemento, por exemplo:

```html
<input v-focus />
```

## Funções de Gatilhos (Hook Functions)

As funções de hook são as funções que são executadas conforme o estado da diretiva e há uma variedade disponível para uso (todas opcionais):

- `created`: Executa antes que os atributos do elemento vinculado ou ouvintes de evento sejam aplicados. Isso é útil nos casos em que a diretiva precisa anexar ouvintes de eventos que devem ser chamados antes dos ouvintes de eventos `v-on` normais.

- `beforeMount`: Executa quando a diretiva é ligada pela primeira vez ao elemento e antes que o componente pai seja montado. Aí é onde você pode fazer o trabalho de configuração inicial.

- `mounted`: Executa quando o componente pai do elemento ligado é montado.

- `beforeUpdate`: Executa antes que o VNode contido no componente seja atualizado.

:::tip Nota
Vamos cobrir VNodes com mais detalhes [depois](render-function.html#the-virtual-dom-tree), quando discutirmos as funções de renderização (render functions).
:::

- `updated`: Executado após o VNode contido no componente **e os VNodes de seus filhos** terem sido atualizados.

- `beforeUnmount`: Executado antes do pai dos elementos ligados sejam desmontados.

- `unmounted`: Executado apenas uma vez, quando a diretiva é desligada do elemento e o componente pai é desmontado

Você pode checar os argumentos que foram passados para esses hooks (ex: `el`, `binding`, `vnode` e `prevNode`) em [API de Diretivas Customizadas](../api/application-api.html#directive)

### Argumentos Dinâmicos de Diretiva

Os argumentos da diretiva podem ser dinâmicos. Por exemplo, em `v-mydirective:[argument]="value"`, o `argument` pode ser atualizado baseada nas propriedades de dados na nossa instância do componente! Isso faz nossas diretivas customizadas flexíveis para utilizar na aplicação.

Digamos que você queira fazer uma diretiva customizada que permite "pregar" elementos na sua página utilizando posicionamento fixo. Nós poderiamos criar uma diretiva customizada onde o valor atualiza a posição vertical em pixels, desse jeito:

```vue-html
<div id="dynamic-arguments-example" class="demo">
  <p>Role para baixo</p>
  <p v-pin="200">Me pregue 200px do topo da página</p>
</div>
```

```js
const app = Vue.createApp({})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.value é o valor que vamos passar para a diretiva - nesse caso, é 200.
    el.style.top = binding.value + 'px'
  }
})

app.mount('#dynamic-arguments-example')
```

Isso iria pregar o elemento 200px do topo da página. Mas o que acontece quando encontramos um cenário que precisamos pregar um elemento da esquerda, ao invés do topo? É aqui que os argumentos dinâmicos que podem ser atualizados por instância de componente são úteis:

```vue-html
<div id="dynamicexample">
  <h3>Role para baixo nessa seção ↓</h3>
  <p v-pin:[direction]="200">Estou pregado na página 200px à esquerda.</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      direction: 'right'
    }
  }
})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.arg é um argumento que passamos para a diretiva
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})

app.mount('#exemplo-argumentos-dinamicos')
```

Resultado:

<common-codepen-snippet title="Diretivas customizadas: argumentos dinâmicos" slug="YzXgGmv" :preview="false" />

A nossa diretiva customizada agora está flexível o suficiente para atender a vários casos diferentes. Para deixá-lo mais dinâmico, podemos possibilitar alterar um valor vinculado. Vamos criar uma propriedade adicional `pinPadding` e vincular ao `<input type="range">`

```vue-html{4}
<div id="dynamicexample">
  <h2>Role a página para baixo</h2>
  <input type="range" min="0" max="500" v-model="pinPadding">
  <p v-pin:[direction]="pinPadding">Me pregue {{ pinPadding + 'px' }} da {{ direction || 'top' }} da página</p>
</div>
```

```js{5}
const app = Vue.createApp({
  data() {
    return {
      direction: 'right',
      pinPadding: 200
    }
  }
})
```

Agora vamos incrementar a lógica da diretiva para recalcular a distância dos elementos pregados quando atualizar o componente:

```js{7-10}
app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  },
  updated(el, binding) {
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})
```

Resultado:

<common-codepen-snippet title="Diretivas customizadas: argumentos dinâmicos + vínculo dinâmico" slug="rNOaZpj" :preview="false" />

## Atalho da Função (Function Shorthand)

No exemplo anterior, você pode desejar o mesmo comportamento no `mounted` e `updated`, mas não liga para os outros gatilhos. Você pode fazer isso passando a *callback* para a diretiva:

```js
app.directive('pin', (el, binding) => {
  el.style.position = 'fixed'
  const s = binding.arg || 'top'
  el.style[s] = binding.value + 'px'
})
```

## Objetos Literais (Object Literals)

Se a sua diretiva precisa de múltiplos valores, você também pode passar um objeto literal do javascript. Lembre-se que as diretivas pode receber qualquer expressão Javascript válida.

```vue-html
<div v-demo="{ color: 'white', text: 'olá!!' }"></div>
```

```js
app.directive('demo', (el, binding) => {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text) // => "olá!"
})
```

## Uso em Componentes

Quando usada em componentes, a diretiva customizada sempre se aplicará ao nó raiz do componente, de forma semelhante a [atributos não-prop](component-attrs.html).

```vue-html
<my-component v-demo="test"></my-component>
```

```js
app.component('my-component', {
  template: `
    <div> // a diretiva v-demo será aplicada aqui
      <span>Conteúdo do meu componente</span>
    </div>
  `
})
```

Ao contrário dos atributos, as diretivas não podem ser passadas para um elemento diferente com `v-bind="$attrs"`.

Com o suporte de [fragmentos](/guide/migration/fragments.html#visao-geral), os componentes podem ter potencialmente mais de um nó raiz. Quando aplicada a um componente de múltiplas raízes, a diretiva será ignorada e o aviso será lançado.