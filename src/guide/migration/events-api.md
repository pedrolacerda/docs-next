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

[Sinalizador na compilação de migração: `INSTANCE_EVENT_EMITTER`](migration-build.html#compat-configuration)

No Vue 3, não é mais possível usar essas APIs para ouvir os próprios eventos emitidos de um componente de dentro de um componente. Não há caminho de migração para esse caso de uso.

### Eventos do Componente Raiz

Os escutadores de evento estáticos podem ser adicionados ao componente raiz passando-os como props para `createApp`:

```js
createApp(App, {
  // Escuta o evento 'expand'
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

Isso fornece a mesma API de emissor de eventos que no Vue 2.

Na maioria das circunstâncias, o uso de um _event bus_ global para comunicação entre componentes é desencorajado. Embora muitas vezes seja a solução mais simples a curto prazo, quase invariavelmente prova ser uma dor de cabeça de manutenção a longo prazo. Dependendo das circunstâncias, existem várias alternativas ao uso de um _event bus_:

* [Props](/guide/component-basics.html#passando-dados-aos-filhos-com-propriedades) e [eventos](/guide/component-basics.html#escutando-eventos-dos-filhos) deve ser sua primeira escolha para comunicação entre pais e filhos. Irmãos podem se comunicar através de seus pais.
* [Prover e injetar](/guide/component-provide-inject.html) permite que um componente se comunique com seu conteúdo de slot. Isso é útil para componentes fortemente acoplados que são sempre usados ​​juntos.
* `provide`/`inject` também pode ser usado para comunicação de longa distância entre componentes. Isso pode ajudar a evitar a 'perfuração com props', onde as props precisam ser passadas abaixo por muitos níveis de componentes que não precisam delas.
* A perfuração com props também pode ser evitada refatorando para usar slots. Se um componente provisório não precisar das props, isso pode indicar um problema com a separação de preocupações. A introdução de um slot nesse componente permite que o pai crie o conteúdo diretamente, para que as props possam ser passadas sem que o componente provisório precise se envolver.
* [Gerenciamento de estado global](/guide/state-management.html), como [Vuex](https://next.vuex.vuejs.org/).
