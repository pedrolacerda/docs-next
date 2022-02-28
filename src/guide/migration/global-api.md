---
badges:
  - breaking
---

# API Global <MigrationBadges :badges="$frontmatter.badges" />

O Vue 2.x possui várias APIs e configurações globais que alteram globalmente o comportamento do Vue. Por exemplo, para registrar um componente global, você usaria a API `Vue.component` assim:

```js
Vue.component('button-counter', {
  data: () => ({
    count: 0
  }),
  template: '<button @click="count++">Clicado {{ count }} vezes.</button>'
})
```

Da mesma forma, é assim que uma diretiva global é declarada:

```js
Vue.directive('focus', {
  inserted: el => el.focus()
})
```

Embora essa abordagem seja conveniente, ela leva a alguns problemas. Tecnicamente, o Vue 2 não tem um conceito de "aplicativo". O que definimos como um aplicativo é simplesmente uma instância raiz do Vue criada via `new Vue()`. Cada instância raiz criada a partir do mesmo construtor Vue **compartilha a mesma configuração global**. Como resultado:

- A configuração global facilita a poluição acidental de outros casos de teste durante o teste. Os usuários precisam armazenar cuidadosamente a configuração global original e restaurá-la após cada teste (ex.: redefinindo `Vue.config.errorHandler`). Algumas APIs como `Vue.use` e `Vue.mixin` nem têm como reverter seus efeitos. Isso torna os testes envolvendo plugins particularmente complicados. Na verdade, vue-test-utils tem que implementar uma API especial `createLocalVue` para lidar com isso:

  ```js
  import { createLocalVue, mount } from '@vue/test-utils'

  // cria um construtor `Vue` estendido
  const localVue = createLocalVue()

  // instala um plugin “globalmente” no construtor “local” do Vue
  localVue.use(MyPlugin)

  // passa o `localVue` para as opções de montagem
  mount(Component, { localVue })
  ```

- A configuração global dificulta o compartilhamento da mesma cópia do Vue entre vários "apps" na mesma página, mas com configurações globais diferentes.

  ```js
  // isso afeta ambas as instâncias raiz
  Vue.mixin({
    /* ... */
  })

  const app1 = new Vue({ el: '#app-1' })
  const app2 = new Vue({ el: '#app-2' })
  ```

Para evitar esses problemas, no Vue 3 apresentamos…

## Uma Nova API Global: `createApp`

Chamar `createApp` retorna uma _instância de aplicativo_, um novo conceito no Vue 3.

```js
import { createApp } from 'vue'

const app = createApp({})
```

Se você estiver usando uma versão [CDN](/guide/installation.html#cdn) do Vue, o `createApp` será exposto por meio do objeto global `Vue`:

```js
const { createApp } = Vue

const app = createApp({})
```

Uma instância de aplicativo expõe um subconjunto das APIs globais do Vue 2. A regra geral é _qualquer API que altere globalmente o comportamento do Vue agora foi movida para a instância do aplicativo_. Aqui está uma tabela das APIs globais do Vue 2 e suas APIs de instância correspondentes:

| 2.x API Global             | 3.x API de Instância (`app`)                                                                                                      |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Vue.config                 | app.config                                                                                                                        |
| Vue.config.productionTip   | _removido_ ([veja abaixo](#config-productiontip-removido))                                                                         |
| Vue.config.ignoredElements | app.config.compilerOptions.isCustomElement ([veja abaixo](#config-ignoredelements-agora-e-config-compileroptions-iscustomelement)) |
| Vue.component              | app.component                                                                                                                     |
| Vue.directive              | app.directiva                                                                                                                     |
| Vue.mixin                  | app.mixin                                                                                                                         |
| Vue.use                    | app.use ([veja abaixo](#uma-nota-para-autores-de-plugins))                                                                               |
| Vue.prototype              | app.config.globalProperties ([veja abaixo](#vue-prototype-substituido-por-config-globalproperties))                                   |
| Vue.extend                 | _removido_ ([veja abaixo](#vue-extend-removido))                                                                                   |

Todas as outras APIs globais que não alteram globalmente o comportamento agora são exportações nomeadas, conforme documentado em [_Treeshaking_ da API Global](./global-api-treeshaking.html).

### `config.productionTip` Removido

No Vue 3.x, a dica _"use production build"_ só aparecerá ao usar o _"dev + full build"_ (a versão que inclui o compilador em tempo de execução e tem avisos).

Para construções de módulos ES, como são usados ​​com empacotadores e, na maioria dos casos, uma CLI ou _boilerplate_ configuraria o ambiente de produção corretamente, essa dica não aparecerá mais.

[Sinalizador na compilação de migração: `CONFIG_PRODUCTION_TIP`](migration-build.html#configuracao-de-compatibilidade)

### `config.ignoredElements` Agora É `config.compilerOptions.isCustomElement`

Essa opção de configuração foi introduzida com a intenção de oferecer suporte a elementos personalizados nativos, então a renomeação transmite melhor o que ela faz. A nova opção também espera uma função, o que fornece mais flexibilidade do que a antiga abordagem de string / RegExp:

```js
// antes
Vue.config.ignoredElements = ['my-el', /^ion-/]

// depois
const app = createApp({})
app.config.compilerOptions.isCustomElement = tag => tag.startsWith('ion-')
```

::: tip Importante

No Vue 3, a verificação de se um elemento é um componente foi movida para a fase de compilação do _template_, portanto esta opção de configuração só é respeitada quando se utiliza a compilação em tempo de execução. Se você estiver usando a compilação somente em tempo de execução, `isCustomElement` deve ser passado para `@vue/compiler-dom` na configuração da compilação - por exemplo, através da opção [`compilerOptions` no vue-loader](https://vue-loader.vuejs.org/options.html#compileroptions).

- Se `config.compilerOptions.isCustomElement` for atribuído ao usar uma compilação somente em tempo de execução, um aviso será emitido instruindo o usuário a passar a opção na configuração da compilação;
- Esta será uma nova opção de nível superior na configuração do Vue CLI.
:::

[Sinalizador na compilação de migração: `CONFIG_IGNORED_ELEMENTS`](migration-build.html#configuracao-de-compatibilidade)

### `Vue.prototype` Substituído por `config.globalProperties`

No Vue 2, `Vue.prototype` era comumente usado para adicionar propriedades que seriam acessíveis em todos os componentes.

O equivalente no Vue 3 é [`config.globalProperties`](/api/application-config.html#globalproperties). Essas propriedades serão copiadas como parte da instanciação de um componente dentro do aplicativo:

```js
// antes - Vue 2
Vue.prototype.$http = () => {}
```

```js
// depois - Vue 3
const app = createApp({})
app.config.globalProperties.$http = () => {}
```

Usar `provide` (discutido [abaixo](#prover-injetar)) também deve ser considerado como uma alternativa para `globalProperties`.

[Sinalizador na compilação de migração: `GLOBAL_PROTOTYPE`](migration-build.html#configuracao-de-compatibilidade)

### `Vue.extend` Removido

No Vue 2.x, `Vue.extend` era usado para criar uma "subclasse" do construtor Vue base com o argumento que deveria ser um objeto contendo opções de componente. No Vue 3.x, não temos mais o conceito de construtores de componente. A montagem de um componente deve sempre usar a API global `createApp`:

```js
// antes - Vue 2

// cria construtor
const Profile = Vue.extend({
  template: '<p>{{firstName}} {{lastName}} também conhecido como {{alias}}</p>',
  data() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
})
// cria uma instância de Profile e monta em um elemento
new Profile().$mount('#mount-point')
```

```js
// depois - Vue 3
const Profile = {
  template: '<p>{{firstName}} {{lastName}} também conhecido como {{alias}}</p>',
  data() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
}

Vue.createApp(Profile).mount('#mount-point')
```

#### Inferência de Tipos

No Vue 2, `Vue.extend` também era usado para fornecer a inferência de tipo do TypeScript para as opções do componente. No Vue 3, a API global `defineComponent` pode ser usada no lugar do `Vue.extend` para o mesmo propósito.

Observe que, embora o tipo de retorno de `defineComponent` seja um tipo estilo construtor, ele é usado apenas para inferência do TSX. Em tempo de execução, o `defineComponent` é em grande parte um _noop_ e retornará o objeto de opções como está.

#### Herança de Componentes

No Vue 3, é altamente recomendável favorecer a composição via [API de Composição](/api/composition-api.html) sobre herança e mixins. Se por algum motivo você ainda precisar de herança de componentes, você pode usar a [opção `extends`](/api/options-composition.html#extends) em vez de `Vue.extend`.

[Sinalizador na compilação de migração: `GLOBAL_EXTEND`](migration-build.html#configuracao-de-compatibilidade)

### Uma Nota para Autores de Plugins

É uma prática comum para os autores de plugins instalarem os plugins automaticamente em suas compilações UMD usando `Vue.use`. Por exemplo, é assim que o plugin oficial `vue-router` se instala em um ambiente de navegador:

```js
var inBrowser = typeof window !== 'undefined'
/* … */
if (inBrowser && window.Vue) {
  window.Vue.use(VueRouter)
}
```

Como a API global `use` não está mais disponível no Vue 3, este método deixará de funcionar e chamar `Vue.use()` agora acionará um aviso. Em vez disso, o usuário final terá que especificar explicitamente o uso do plugin na instância do aplicativo:

```js
const app = createApp(MyApp)
app.use(VueRouter)
```

## Montando a Instância do Aplicativo

Após ser inicializado com `createApp(/* options */)`, a instância do aplicativo `app` pode ser usada para montar uma instância de componente raiz com `app.mount(domTarget)`:

```js
import { createApp } from 'vue'
import MyApp from './MyApp.vue'

const app = createApp(MyApp)
app.mount('#app')
```

Com todas essas mudanças, o componente e a diretiva que temos no início do guia serão reescritos em algo assim:

```js
const app = createApp(MyApp)

app.component('button-counter', {
  data: () => ({
    count: 0
  }),
  template: '<button @click="count++">Clicado {{ count }} vezes.</button>'
})

app.directive('focus', {
  mounted: el => el.focus()
})

// agora cada instância do aplicativo montada com app.mount(), junto com sua
// árvore de componentes, terá o mesmo componente “button-counter”
// e diretiva “focus” sem poluir o ambiente global
app.mount('#app')
```

[Sinalizador na compilação de migração: `GLOBAL_MOUNT`](migration-build.html#configuracao-de-compatibilidade)

## Prover / Injetar

Semelhante ao uso da opção `provide` em uma instância raiz 2.x, uma instância de aplicativo Vue 3 também pode fornecer dependências que podem ser injetadas por qualquer componente dentro do aplicativo:

```js
// na entrada
app.provide('guide', 'Guia do Vue 3')

// em um componente filho
export default {
  inject: {
    book: {
      from: 'guide'
    }
  },
  template: `<div>{{ book }}</div>`
}
```

Usar `provide` é especialmente útil ao escrever um plugin, como uma alternativa para `globalProperties`.

## Compartilhe Configurações Entre Aplicativos

Uma maneira de compartilhar configurações, como componentes ou diretivas entre apps é criar uma função fabricadora, assim:

```js
import { createApp } from 'vue'
import Foo from './Foo.vue'
import Bar from './Bar.vue'

const createMyApp = options => {
  const app = createApp(options)
  app.directive('focus' /* ... */)

  return app
}

createMyApp(Foo).mount('#foo')
createMyApp(Bar).mount('#bar')
```

Agora a diretiva `focus` estará disponível em ambas as instâncias `Foo` e `Bar` e seus descendentes.
