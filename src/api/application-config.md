# Configuração da aplicação

Cada aplicação Vue expõe um objeto `config` que contém as definições de configuração para aquela aplicação:

```js
const app = createApp({})

console.log(app.config)
```

Você pode modificar as propriedades listadas abaixo antes de montar a sua aplicação.

## errorHandler

- **Tipo:** `Function`

- **Padrão:** `undefined`

- **Uso:**

```js
app.config.errorHandler = (err, vm, info) => {
  // gerenciar erro
  // `info` é uma informação de erro específica do Vue, e.g. em qual gatilho
  // do ciclo de vida o erro foi encontrado
}
```

Atribua um gerenciador para erros não capturados durante a função render do componente e os observadores. O gerenciador é chamado com o erro e a instância da aplicação.

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

Atribua um gerenciador customizado para avisos do Vue em tempo de execução. Note que isso funciona apenas durante o desenvolvimento e é ignorado em produção.

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

Adiciona uma propriedade global que pode ser acessada em qualquer instância de componente dentro da aplicação. A propriedade do componente irá tomar prioridade quando houver chaves conflitantes.

Isto pode subsstituir o `Vue.prototype` do Vue 2.x estendendo:

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

app.config.optionMergeStrategies.hello = (pai, filho) => {
  return `Olá, ${filho}`
}

app.mixin({
  hello: 'Vue'
})

// 'Olá, Vue'
```

Defina estratégias de mesclagem para opções customizadas.

A estratégia de mesclagem recebe o valor daquela opção definida nas instâncias de pai e filho como primeiro e segundo argumentos, respectivamente.

- **Veja também:** [Custom Option Merging Strategies](../guide/mixins.html#custom-option-merge-strategies)

## performance

- **Tipo:** `boolean`

- **Padrão:** `false`

- **Uso**:

Defina isto como `true` para habilitar a inicialização, compilação, renderização e correção de desempenho do componente rastreando no painel de desempenho ou linha do tempo devtool do navegador. Funciona apenas em modo de desenvolvimento e em navegadores que suportam a API [performance.mark](https://developer.mozilla.org/en-US/docs/Web/API/Performance/mark).

## compilerOptions <Badge text="3.1+" />

- **Tipo:** `Object`

Configure as opções do compilador no tempo de execução. Os valores definidos neste objeto serão pasados para o modelo no navegador e afetarão todos os componentes no app configurado. Note que você também pode sobrepor essas opções de uma maneira por componente usando a [opção `compilerOptions`](/api/options-misc.html#compileroptions).

::: tip Importante
Esta opção de configuração só é respeitada ao usar a _build_ completa (e.g. o `vue.js` independente que pode compilar templates no navegador). Se você utilizar a _build_ apenas do momento de execução com uma configuração de instalação, as opções do compilador devem então ser passadas para `@vue/compiler-dom` pela configuração de opções da _build_.

- Para `vue-loader`: [passe pela opção `compilerOptions` loader](https://vue-loader.vuejs.org/options.html#compileroptions). Veja também [como configurá-lo no `vue-cli`](https://cli.vuejs.org/guide/webpack.html#modifying-options-of-a-loader).

- Para `vite`: [passe pelas opções `@vitejs/plugin-vue`](https://github.com/vitejs/vite/tree/main/packages/plugin-vue#example-for-passing-options-to-vuecompiler-dom).
:::

### compilerOptions.isCustomElement

- **Tipo:** `(tag: string) => boolean`

- **Padrão:** `undefined`

- **Uso:**

```js
// qualquer elemento começando com 'ion-' será reconhecido como um elemento personalizado
app.config.compilerOptions.isCustomElement = tag => tag.startsWith('ion-')
```

Especifica um método para reconhecer elementos personalizados definidos fora do Vue (e.g. usando APIs de _Web Components_). Caso um componente corresponda à essa condição, ele não precisará do registro local ou global e o Vue não lançará avisos sobre um `Elemento personalizado desconhecido`.

> Note que todas as tags de HTML nativo e SVG não precisam corresponder a esta função. O Vue parser desempenha esta conferência automaticamente.

### compilerOptions.whitespace

- **Tipo:** `'condense' | 'preserve'`

- **Padrão:** `'condense'`

- **Uso:**

```js
app.config.compilerOptions.whitespace = 'preserve'
```

Por padrão, o Vue remove/condensa espaços em branco entre elementos template para produzir resultados compilados mais eficientes:

1. Espaços em branco no começo ou no fim de um elemento são condensados em um espaço único
2. Espaços em branco entre elementos que contenham novas linhas são removidos
3. Espaços em branco consecutivos em _nodes_ de texto são condensados em um único espaço.

Definir o valor como `'preserve'` irá desabilitar o (2) e (3).

### compilerOptions.delimiters

- **Tipo:** `Array<string>`

- **Padrão:** `{{ "['\u007b\u007b', '\u007d\u007d']" }}`

- **Uso:**

```js
// Delimitadores modificados para o estilo de template de string ES6
app.config.compilerOptions.delimiters = ['${', '}']    
```

Define os delimitadores usados para a interpolação de texto dentro do template.

Tipicamente isso é usado para evitar conflitos com _frameworks server-side_ que também usam sintaxe _mustache_.

### compilerOptions.comments

- **Tipo:** `boolean`

- **Padrão:** `false`

- **Uso:**

```js
app.config.compilerOptions.comments = true
```

Por padrão, o Vue irá remover comentários HTML dentro de templates em produção. Definir esta opção como `true` irá forçar o Vue a preservar os comentários mesmo em produção. Comentários são sempre preservados em desenvolvimento.

Esta opção é tipicamente usada quando o Vue é usado com outras bibliotecas que dependem de comentários HTML.

## isCustomElement <Badge text="deprecated" type="warning"/>

Descontinuado na versão 3.1.0. Use [`compilerOptions.isCustomElement`](#compileroptions-iscustomelement).
