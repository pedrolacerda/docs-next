---
badges:
  - breaking
---

# Diretivas Personalizadas <MigrationBadges :badges="$frontmatter.badges" />

## Visão Geral

As funções de gatilho para diretivas foram renomeadas para se alinhar melhor com o ciclo de vida do componente.

Além disso, a string `expression` não é mais passada como parte do objeto `binding`.

## Sintaxe v2.x

No Vue 2, as diretivas personalizadas foram criadas usando os gatilhos listados abaixo para atingir um ciclo de vida de um elemento, todos opcionais:

- **bind** - Ocorre quando a diretiva é vinculada ao elemento. Ocorre apenas uma vez.
- **inserted** - Ocorre quando o elemento é inserido no DOM pai.
- **update** - Este gatilho é chamado quando o elemento é atualizado, mas os filhos ainda não foram atualizados.
- **componentUpdated** - Este gatilho é chamado assim que o componente e os filhos forem atualizados.
- **unbind** - Este gatilho é chamado assim que a diretiva é removida. Também chamado apenas uma vez.

Aqui está um exemplo disso:

```html
<p v-highlight="'yellow'">Destaque este texto em amarelo claro</p>
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

- **created** - novo! É chamado antes que os atributos do elemento ou escutas de evento sejam aplicados.
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
  created(el, binding, vnode, prevVnode) {}, // new
  beforeMount() {},
  mounted() {},
  beforeUpdate() {}, // new
  updated() {},
  beforeUnmount() {}, // novo
  unmounted() {}
}
```

A API resultante poderia ser usada assim, espelhando o exemplo anterior:

```html
<p v-highlight="'yellow'">Destaque este texto em amarelo claro</p>
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

```js
bind(el, binding, vnode) {
  const vm = vnode.context
}
```

No Vue 3, a instância agora faz parte do `binding`:

```js
mounted(el, binding, vnode) {
  const vm = binding.instance
}
```

:::warning Aviso
Com o suporte à [fragmentos](/guide/migration/fragments.html#overview), os componentes podem ter mais de um nó raiz. Quando aplicada a um componente de múltiplas raízes, uma diretiva personalizada será ignorada e um aviso será lançado.
:::

## Migration Strategy

[Migration build flag: `CUSTOM_DIR`](migration-build.html#compat-configuration)
