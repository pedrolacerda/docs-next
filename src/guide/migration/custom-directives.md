---
badges:
  - breaking
---

# Diretivas Personalizadas <MigrationBadges :badges="$frontmatter.badges" />

## Visão Geral

Aqui está um rápido resumo do que mudou:

- API foi renomeada para se alinhar melhor com o ciclo de vida do componente
- As diretivas personalizadas serão controladas pelo componente filho via `v-bind="$attrs"`

Para mais informações, continue lendo!

## Sintaxe v2.x

No Vue 2, as diretivas personalizadas foram criadas usando os gatilhos listados abaixo para atingir um ciclo de vida de um elemento, todos opcionais:

- **bind** - Ocorre quando a diretiva é vinculada ao elemento. Ocorre apenas uma vez.
- **inserted** - Ocorre quando o elemento é inserido no DOM pai.
- **update** - Este gatilho é chamado quando o elemento é atualizado, mas os filhos ainda não foram atualizados.
- **componentUpdated** - Este gatilho é chamado assim que o componente e os filhos forem atualizados.
- **unbind** - Este gatilho é chamado assim que a diretiva é removida. Também chamado apenas uma vez.

Aqui está um exemplo disso:

```html
<p v-highlight="yellow">Destaque este texto em amarelo claro</p>
```

```js
Vue.directive('highlight', {
  bind(el, binding, vnode) {
    el.style.background = binding.value
  }
})
```

Aqui, na configuração inicial desse elemento, a diretiva vincula um estilo passando um valor, que pode ser atualizado para diferentes valores por meio da aplicação.

## Sintaxe v3.x

No Vue 3, no entanto, criamos uma API mais coesa para diretivas personalizadas. Como você pode ver, eles diferem muito de nossos métodos de ciclo de vida de componentes, embora estejamos nos conectando a eventos semelhantes. Agora, nós os unificamos assim:

- bind → **beforeMount**
- inserted → **mounted**
- **beforeUpdate**: novo! É chamado antes que o próprio elemento seja atualizado, muito semelhante aos gatilhos de ciclo de vida do componente.
- update → removido! Havia muitas semelhanças com "updated", então era redundante. Em vez disso, use "updated".
- componentUpdated → **updated**
- **beforeUnmount**: novo! Semelhante aos gatilhos de ciclo de vida do componente, será chamado logo antes de um elemento ser desmontado.
- unbind -> **unmounted**

A API final é a seguinte:

```js
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeUnmount() {}, // novo
  unmounted() {}
}
```

A API resultante poderia ser usada assim, espelhando o exemplo anterior:

```html
<p v-highlight="yellow">Destaque este texto em amarelo claro</p>
```

```js
const app = Vue.createApp({})

app.directive('highlight', {
  beforeMount(el, binding, vnode) {
    el.style.background = binding.value
  }
})
```

Agora que os gatilhos de ciclo de vida da diretiva personalizada espelham os dos próprios componentes, eles se tornam mais fáceis de raciocinar e lembrar!

### Caso Extremo: Acessando a instância do componente

Geralmente, é recomendado manter as diretivas independentes da instância do componente em que são usadas. Acessar a instância a partir de uma diretiva personalizada costuma ser um sinal de que a diretiva deve ser um componente em si. No entanto, existem situações em que isso realmente faz sentido.

No Vue 2, a instância do componente tinha que ser acessada por meio do argumento `vnode`:

```javascript
bind(el, binding, vnode) {
  const vm = vnode.context
}
```

No Vue 3, a instância agora faz parte do `binding`:

```javascript
mounted(el, binding, vnode) {
  const vm = binding.instance
}
```

## Detalhes de Implementação

No Vue 3, agora oferecemos suporte a fragmentos, o que nos permite retornar mais de um nó DOM por componente. Você pode imaginar como isso é útil para algo como um componente com vários elementos `<li>` ou os elementos filhos de uma tabela:

```html
<template>
  <li>Olá</li>
  <li>Desenvolvedores</li>
  <li>Vue!</li>
</template>
```

Tão maravilhosamente flexível quanto isso é, podemos potencialmente encontrar um problema com uma diretiva personalizada que pode ter vários nós raiz.

Como resultado, as diretivas personalizadas agora são incluídas como parte dos dados de um nó virtual do DOM. Quando uma diretiva personalizada é usada em um componente, gatilhos são passados para o componente como "props" alheios e acabam em `this.$attrs`.

Isso também significa que é possível conectar-se diretamente ao ciclo de vida de um elemento como neste *template*, o que pode ser útil quando uma diretiva personalizada está muito envolvida:

```html
<div @vnodeMounted="myHook" />
```

Isso é consistente com o comportamento de "fallthrough" de atributos, portanto, quando um componente filho usa `v-bind="$attrs"` em um elemento interno, ele também aplicará todas as diretivas personalizadas usadas nele.