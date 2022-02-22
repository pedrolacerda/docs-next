---
badges:
  - new
---

# Componentes Assíncronos <MigrationBadges :badges="$frontmatter.badges" />

## Visão Geral

Sobre o que mudou, em visão geral:

- Novo método helper `defineAsyncComponent` que define explicitamente componentes assíncronos
- Opção `component` renomeada para `loader`
- Função Loader não recebe inerentemente os argumentos `resolve` e `reject` e deve retornar uma Promise

Para uma explicação mais aprofundada, continue lendo!

## Introdução

Anteriormente, componentes assíncronos eram criados simplesmente definindo um componente como uma função que retornava uma Promise, tal como:

```js
const asyncModal = () => import('./Modal.vue')
```

Ou, para os componentes com sintaxe mais avançada com opções:

```js
const asyncModal = {
  component: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}
```

## Sintaxe v3.x

Agora, no Vue 3, uma vez que os componentes funcionais são definidos como funções puras, as definições de componentes assíncronos precisam ser definidas explicitamente, envolvendo-o em um novo helper `defineAsyncComponent`:

```js
import { defineAsyncComponent } from 'vue'
import ErrorComponent from './components/ErrorComponent.vue'
import LoadingComponent from './components/LoadingComponent.vue'

// Componente Assíncrono sem opções
const asyncModal = defineAsyncComponent(() => import('./Modal.vue'))

// Componente Assíncrono com opções
const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

::: tip NOTA
O Vue Router suporta um mecanismo similar para carregar componentes de rota de maneira assíncrona, conhecido como *lazy loading*. Apesar das similaridades, este recurso é distinto do suporte do Vue para componentes assíncronos. Você **não** deve usar `defineAsyncComponent` ao configurar componentes de rota com o Vue Router. Você pode ler mais sobre isso na seção [_Lazy Loading Routes_](https://next.router.vuejs.org/guide/advanced/lazy-loading.html) da documentação do Vue Router.
:::

Outra mudança em relação a v2.x é a de que a opção `component` foi renomeada para `loader` a fim de comunicar com precisão que uma definição de componente não pode ser fornecida diretamente.

```js{4}
import { defineAsyncComponent } from 'vue'

const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})
```

Além disso, ao contrário da versão 2.x, a função loader não recebe mais os argumentos `resolve` e `reject`, e deve sempre retornar uma Promise.

```js
// Versão 2.x
const oldAsyncComponent = (resolve, reject) => {
  /* ... */
}

// Versão 3.x
const asyncComponent = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      /* ... */
    })
)
```

Para mais informações sobre o uso de componentes assincronos, leia:

- [Guia: Componentes Dinâmicos & Assíncronos](/guide/component-dynamic-async.html#componentes-dinamicos-com-keep-alive)
- [Migration build flag: `COMPONENT_ASYNC`](migration-build.html#compat-configuration)
