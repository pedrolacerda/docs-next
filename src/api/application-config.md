# Configuração da Aplicação

Todo aplicativo Vue expõe um objeto `config` que contém as configurações para esse aplicativo:

```js
const app = createApp({})

console.log(app.config)
```

Você pode modificar suas propriedades, listadas abaixo, antes de montar a sua aplicação.

## errorHandler

- **Tipo:** `Function`

- **Padrão:** `undefined`

- **Uso:**

```js
app.config.errorHandler = (err, vm, info) => {
  // manipula erro
  // `info` traz informações de erros específicos do Vue,
  // ex.: em qual gatilho do ciclo de vida o erro foi encontrado
}
```

Atribua um manipulador para erros não capturados durante a função de renderização do componente e os observadores. O manipulador é chamado com o erro e a instância da aplicação correspondente.

> Os serviços de rastreamento de erros [Sentry](https://sentry.io/for/vue/) e [Bugsnag](https://docs.bugsnag.com/platforms/browsers/vue/) fornecem integrações oficiais usando essa opção.

## warnHandler

- **Tipo:** `Function`

- **Padrão:** `undefined`

- **Uso:**

```js
app.config.warnHandler = function(msg, vm, trace) {
  // `trace` é o rastro da hierarquia do componente
}
```

Atribua um manipulador customizado para avisos do Vue em tempo de execução. Note que isso funciona apenas durante o desenvolvimento e é ignorado em produção.

## globalProperties

- **Tipo:** `[key: string]: any`

- **Padrão:** `undefined`

- **Uso:**

```js
app.config.globalProperties.foo = 'bar'

app.component('child-component', {
  mounted() {
    console.log(this.foo) // 'bar'
  }
})
```

Adiciona uma propriedade global que pode ser acessada em qualquer instância de componente dentro da aplicação. A propriedade do componente terá prioridade quando houver chaves conflitantes.

Isto pode substituir o ato de estender o `Vue.prototype` no Vue 2.x:

```js
// Before
Vue.prototype.$http = () => {}

// After
const app = createApp({})
app.config.globalProperties.$http = () => {}
```

## optionMergeStrategies

- **Tipo:** `{ [key: string]: Function }`

- **Padrão:** `{}`

- **Uso:**

```js
const app = createApp({
  mounted() {
    console.log(this.$options.hello)
  }
})

app.config.optionMergeStrategies.hello = (parent, child) => {
  return `Olá, ${child}`
}

app.mixin({
  hello: 'Vue'
})

// 'Olá, Vue'
```

Defina estratégias de mesclagem para opções customizadas.

A estratégia de mesclagem recebe o valor daquela opção definida nas instâncias de pai e filho como primeiro e segundo argumentos, respectivamente.

- **Ver também:** [Estratégias de Mesclagem de Opções Personalizadas](../guide/mixins.html#estrategias-de-mesclagem-de-opcoes-personalizadas)

## performance

- **Tipo:** `boolean`

- **Padrão:** `false`

- **Uso**:

Defina isto como `true` para habilitar a inicialização, compilação, renderização e rastreamento do desempenho de correções do componente no painel de desempenho ou linha do tempo no devtool do navegador. Funciona apenas em modo de desenvolvimento e em navegadores que suportam a API [performance.mark](https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark).

## compilerOptions <Badge text="3.1+" />

- **Tipo:** `Object`

Configure as opções do compilador de tempo de execução. Os valores definidos neste objeto serão passados para o compilador de _template_ no navegador e afetarão todos os componentes no app configurado. Note que você também pode sobrepor essas opções por componente usando a [opção `compilerOptions`](/api/options-misc.html#compileroptions).

::: tip Importante
Esta opção de configuração só é respeitada ao usar a compilação completa (ou seja, o `vue.js` independente que pode compilar _templates_ no navegador). Se você utilizar a compilação de tempo de execução (_runtime_) em um ambiente de construção, então as opções do compilador devem ser passadas para `@vue/compiler-dom` através das configurações da ferramenta de construção.

- Para `vue-loader`: [passe pela opção `compilerOptions` do loader](https://vue-loader.vuejs.org/options.html#compileroptions). Veja também [como configurá-lo no `vue-cli`](https://cli.vuejs.org/guide/webpack.html#modifying-options-of-a-loader).

- Para `vite`: [passe pelas opções do `@vitejs/plugin-vue`](https://github.com/vitejs/vite/tree/main/packages/plugin-vue#example-for-passing-options-to-vuecompiler-dom).
:::

### compilerOptions.isCustomElement

- **Tipo:** `(tag: string) => boolean`

- **Padrão:** `undefined`

- **Uso:**

```js
// qualquer elemento começando com 'ion-' será reconhecido como personalizado
app.config.compilerOptions.isCustomElement = tag => tag.startsWith('ion-')
```

Especifica um método para reconhecer elementos personalizados definidos fora do Vue (ex.: usando APIs de _Web Components_). Caso um componente corresponda à essa condição, ele não precisará do registro local ou global e o Vue não lançará avisos sobre um elemento personalizado desconhecido (`Unknown custom element`).

> Note que todas as tags de HTML nativo e SVG não precisam corresponder a esta função. O _parser_ do Vue desempenha esta conferência automaticamente.

### compilerOptions.whitespace

- **Tipo:** `'condense' | 'preserve'`

- **Padrão:** `'condense'`

- **Uso:**

```js
app.config.compilerOptions.whitespace = 'preserve'
```

Por padrão, o Vue remove/condensa espaços em branco entre elementos do _template_ para produzir resultados compilados mais eficientes:

1. Espaços em branco no começo ou no fim de um elemento são condensados em um espaço único
2. Espaços em branco entre elementos que contenham novas linhas são removidos
3. Espaços em branco consecutivos em nós de texto são condensados em um único espaço.

Definir o valor como `'preserve'` desabilitará ambos (2) e (3).

### compilerOptions.delimiters

- **Tipo:** `Array<string>`

- **Padrão:** `{{ "['\u007b\u007b', '\u007d\u007d']" }}`

- **Uso:**

```js
// Delimitadores modificados para o estilo de template de string do ES6
app.config.compilerOptions.delimiters = ['${', '}']    
```

Define os delimitadores usados para a interpolação de texto dentro do _template_.

Tipicamente isso é usado para evitar conflitos com frameworks do lado do servidor que também usam sintaxe _mustache_.

### compilerOptions.comments

- **Tipo:** `boolean`

- **Padrão:** `false`

- **Uso:**

```js
app.config.compilerOptions.comments = true
```

Por padrão, o Vue removerá comentários HTML de dentro de _templates_ em produção. Definir esta opção como `true` forçará o Vue a preservar os comentários mesmo em produção. Comentários são sempre preservados em desenvolvimento.

Esta opção é tipicamente usada quando o Vue é usado com outras bibliotecas que dependem de comentários HTML.

## isCustomElement <Badge text="deprecated" type="warning"/>

Descontinuado na versão 3.1.0. Use [`compilerOptions.isCustomElement`](#compileroptions-iscustomelement).
