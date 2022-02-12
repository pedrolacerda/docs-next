---
badges:
  - breaking
---

# API de Eventos <MigrationBadges :badges="$frontmatter.badges" />

## Visão Geral

Os métodos `$on`, `$off` e `$once` foram retirados da instância. As instâncias de componentes não implementam mais a interface de emissor de eventos.

## Sintaxe v2.x

Na v2.x, uma instância Vue poderia ser usada para acionar manipuladores anexados imperativamente por meio da API do emissor de eventos (`$on`, `$off` e `$once`). Isso seria útil ao criar um _event bus_ para permitir a escuta de eventos globais usados em todo o aplicativo:

```js
// eventBus.js

const eventBus = new Vue()

export default eventBus
```

```js
// ChildComponent.vue
import eventBus from './eventBus'

export default {
  mounted() {
    // adicionando escutador eventBus
    eventBus.$on('custom-event', () => {
      console.log('Evento customizado disparado!')
    })
  },
  beforeDestroy() {
    // removendo o escutador eventBus
    eventBus.$off('custom-event')
  }
}
```

```js
// ParentComponent.vue
import eventBus from './eventBus'

export default {
  methods: {
    callGlobalCustomEvent() {
      eventBus.$emit('custom-event') // Quando o componente ChildComponent for montado, nós teremos uma mensagem no console
    }
  }
}
```

## Atualização v3.x

Removemos os métodos `$on`, `$off` e `$once` da instância completamente. `$emit` ainda é uma parte da API existente, pois é usado para acionar manipuladores de eventos declarativamente anexados por um componente pai.

## Estratégia de Migração

[Migration build flag: `INSTANCE_EVENT_EMITTER`](migration-build.html#compat-configuration)

In Vue 3, it is no longer possible to use these APIs to listen to a component's own emitted events from within a component. There is no migration path for that use case.

### Root Component Events

Static event listeners can be added to the root component by passing them as props to `createApp`:

```js
createApp(App, {
  // Listen for the 'expand' event
  onExpand() {
    console.log('expand')
  }
})
```

### Event Bus

Os padrão _event bus_ pode ser substituído usando uma biblioteca externa implementando a interface do emissor de eventos, por exemplo [mitt](https://github.com/developit/mitt) ou [tiny-emitter](https://github.com/scottcorgan/tiny-emitter).

Example:

```js
// eventBus.js
import emitter from 'tiny-emitter/instance'

export default {
  $on: (...args) => emitter.on(...args),
  $once: (...args) => emitter.once(...args),
  $off: (...args) => emitter.off(...args),
  $emit: (...args) => emitter.emit(...args)
}
```

This provides the same event emitter API as in Vue 2.

In most circumstances, using a global event bus for communicating between components is discouraged. While it is often the simplest solution in the short term, it almost invariably proves to be a maintenance headache in the long term. Depending on the circumstances, there are various alternatives to using an event bus:

* [Props](/guide/component-basics.html#passing-data-to-child-components-with-props) and [events](/guide/component-basics.html#listening-to-child-components-events) should be your first choice for parent-child communication. Siblings can communicate via their parent.
* [Provide and inject](/guide/component-provide-inject.html) allow a component to communicate with its slot contents. This is useful for tightly-coupled components that are always used together.
* `provide`/`inject` can also be used for long-distance communication between components. It can help to avoid 'prop drilling', where props need to be passed down through many levels of components that don't need those props themselves.
* Prop drilling can also be avoided by refactoring to use slots. If an interim component doesn't need the props then it might indicate a problem with separation of concerns. Introducing a slot in that component allows the parent to create the content directly, so that props can be passed without the interim component needing to get involved.
* [Global state management](/guide/state-management.html), such as [Vuex](https://next.vuex.vuejs.org/).
