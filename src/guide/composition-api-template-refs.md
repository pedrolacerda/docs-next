# _refs_ de Template

> Esta seção usa a sintaxe de [componente single-file](single-file-component.html) para exemplos de código

> Este guia pressupõe que você já leu a [Introdução à API de Composição](composition-api-introduction.html) e os [Fundamentos da Reatividade](reactivity-fundamentals.html). Leia lá primeiro se você for novo na API de Composição.

Ao usar a API de Composição, os conceitos de [_refs_ reativos](reactivity-fundamentals.html#criacao-de-valores-reativos-avulsos-como-refs) e [_refs_ de _template_](component-template-refs.html) são unificados. Para obter uma referência a um elemento do _template_ ou instância de componente, podemos declarar uma referência como de costume e retorná-la de [setup()](composition-api-setup.html):

```html
<template>
  <div ref="root">Este é um elemento raiz</div>
</template>

<script>
  import { ref, onMounted } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      onMounted(() => {
        // o elemento DOM será atribuído ao ref após a renderização inicial
        console.log(root.value) // <div>Este é um elemento raiz</div>
      })

      return {
        root
      }
    }
  }
</script>
```

Aqui estamos expondo `root` no contexto de renderização e vinculando-o a div como sua referência via `ref="root"`. No algoritmo de correção (_patch_) do DOM Virtual, se a chave `ref` de um VNode corresponder a uma referência no contexto de renderização, o elemento ou instância de componente correspondente do VNode será atribuído ao valor dessa referência. Isso é realizado durante o processo de montagem / _patch_ do Virtual DOM, portanto, as referências do _template_ só receberão valores atribuídos após a renderização inicial.

Referências usadas como _refs_ de _template_ se comportam como quaisquer outras _refs_: elas são reativas e podem ser passadas (ou retornadas) a funções de composição.

## Uso com JSX

```js
export default {
  setup() {
    const root = ref(null)

    return () =>
      h('div', {
        ref: root
      })

    // com JSX
    return () => <div ref={root} />
  }
}
```

## Uso dentro de `v-for`

As _refs_ de _template_ da API de Composição não têm tratamento especial quando usadas dentro de `v-for`. Em vez disso, use a sintaxe de função nas _refs_ para realizar um tratamento personalizado:

```html
<template>
  <div v-for="(item, i) in list" :ref="el => { if (el) divs[i] = el }">
    {{ item }}
  </div>
</template>

<script>
  import { ref, reactive, onBeforeUpdate } from 'vue'

  export default {
    setup() {
      const list = reactive([1, 2, 3])
      const divs = ref([])

      // certifique-se de limpar as refs antes de cada atualização
      onBeforeUpdate(() => {
        divs.value = []
      })

      return {
        list,
        divs
      }
    }
  }
</script>
```

## Observando _refs_ de Templates

Observar uma referência de _template_ por alterações pode ser uma alternativa ao uso de gatilhos de ciclo de vida que foi demonstrado nos exemplos anteriores.

Mas uma diferença chave para os gatilhos do ciclo de vida é que os efeitos `watch()` e `watchEffect()` são executados *antes* do DOM ser montado ou atualizado, de modo que o _ref_ de _template_ ainda não foi atualizado quando o observador executa o efeito:

```vue
<template>
  <div ref="root">Este é um elemento raiz</div>
</template>

<script>
  import { ref, watchEffect } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      watchEffect(() => {
        // Este efeito é executado antes que o DOM seja atualizado e, consequentemente,
        // o ref de template ainda não contém uma referência ao elemento.
        console.log(root.value) // => null
      })

      return {
        root
      }
    }
  }
</script>
```

Portanto, observadores que usam _refs_ de _template_ devem ser definidos com a opção `flush: 'post'`. Isso executará o efeito *depois* do DOM ser atualizado e garantirá que o _ref_ de _template_ permaneça em sincronia com o DOM e faça referência ao elemento correto.

```vue
<template>
  <div ref="root">Este é um elemento raiz</div>
</template>

<script>
  import { ref, watchEffect } from 'vue'

  export default {
    setup() {
      const root = ref(null)

      watchEffect(() => {
        console.log(root.value) // => <div>Este é um elemento raiz</div>
      }, 
      {
        flush: 'post'
      })

      return {
        root
      }
    }
  }
</script>
```

* Ver também: [Dados Computados e Observadores](./reactivity-computed-watchers.html#effect-flush-timing)
