# Propriedades da Instância

## $data

- **Tipo:** `Object`

- **Detalhes:**

  O objeto de dados que a instância do componente está observando. A instância do componente faz _proxy_ de acesso às propriedades em seu objeto de dados.

- **Ver também:** [Opções - Dados](./options-data.html#data-2)

## $props

- **Tipo:** `Object`

- **Detalhes:**

  Um objeto representando as props atuais que o componente recebeu. A instância do componente faz _proxy_ de acesso às propriedades em seu objeto de props.

## $el

- **Tipo:** `any`

- **Somente leitura**

- **Detalhes:**

  O elemento raiz do DOM que a instância do componente está gerenciando.

  Para componentes usando [fragmentos](../guide/migration/fragments), `$el` será o nó DOM _placeholder_ que o Vue usa para acompanhar a posição do componente no DOM. Recomenda-se usar [_refs_ de _template_](../guide/component-template-refs.html) para acesso direto aos elementos DOM em vez de depender de `$el`.

## $options

- **Tipo:** `Object`

- **Somente leitura**

- **Detalhes:**

  As opções da instanciação usadas pela instância do componente atual. Isso é útil quando você deseja incluir propriedades personalizadas nas opções:

  ```js
  const app = createApp({
    customOption: 'foo',
    created() {
      console.log(this.$options.customOption) // => 'foo'
    }
  })
  ```

## $parent

- **Tipo:** `Component instance`

- **Somente leitura**

- **Detalhes:**

  A instância pai, se a instância atual tiver uma.

## $root

- **Tipo:** `Component instance`

- **Somente leitura**

- **Detalhes:**

  A instância do componente raiz da árvore de componentes atual. Caso a instância atual não tenha pais, este valor será ela mesmo.

## $slots

- **Tipo:** `{ [name: string]: (...args: any[]) => Array<VNode> | undefined }`

- **Somente Leitura**

- **Detalhes:**

  Usado para o acesso de conteúdo [distribuído com slots](../guide/component-basics.html#distribuicao-de-conteudo-com-slots) de forma programática. Cada [slot nomeado](../guide/component-slots.html#slots-nomeados) tem sua propriedade correspondente (e.g. o conteúdo de `v-slot:foo` será encontrado em `this.$slots.foo()`). A propriedade `default` contém os outros nós não incluídos nos slots nomeados ou conteúdos do `v-slot:default`.

  Acessar `this.$slots` é mais útil ao construir um componente com uma [função de renderização](../guide/render-function.html).

- **Exemplo:**

  ```html
  <blog-post>
    <template v-slot:header>
      <h1>Sobre</h1>
    </template>

    <template v-slot:default>
      <p>
        Aqui está o conteúdo da página, que será incluído em $slots.default.
      </p>
    </template>

    <template v-slot:footer>
      <p>Copyright 2020 Evan You</p>
    </template>
  </blog-post>
  ```

  ```js
  const { createApp, h } = Vue
  const app = createApp({})

  app.component('blog-post', {
    render() {
      return h('div', [
        h('header', this.$slots.header()),
        h('main', this.$slots.default()),
        h('footer', this.$slots.footer())
      ])
    }
  })
  ```

- **Ver também:**
  - [Componente `<slot>`](built-in-components.html#slot)
  - [Distribuição de Conteúdo com Slots](../guide/component-basics.html#distribuicao-de-conteudo-com-slots)
  - [Funções de Renderização - Slots](../guide/render-function.html#slots)

## $refs

- **Tipo:** `Object`

- **Somente leitura**

- **Detalhes:**

Um objeto dos elementos do DOM e as instâncias de componente, registrados com [atributos `ref`](../guide/component-template-refs.html)

- **Ver também:**
  - [_refs_ de _Template_](../guide/component-template-refs.html)
  - [Atributos especiais - `ref`](./special-attributes.md#ref)

## $attrs

- **Tipo:** `Object`

- **Somente leitura**

- **Detalhes:**

Contém atributos vinculados do escopo-pai e eventos que não são reconhecidos (e extraídos) como [props](./options-data.html#props) de componentes ou [eventos personalizados](./options-data.html#emits). Quando um componente não possui nenhuma propriedade declarada ou eventos customizados, esse essencialmente contém todos os vínculos do escopo-pai, e pode ser passado abaixo para um componente interno via `v-bind="$attrs"` - útil ao criar componentes de ordem superior.

- **Veja também:**
  - [Atributos Não-Propriedades](../guide/component-attrs.html)
  - [Opções / Diversos - inheritAttrs](./options-misc.html#inheritattrs)
