# DOM

## `template`

- **Tipo:** `string`

- **Detalhes:**

  Um _template_ string a ser usado como marcação para a instância do componente. O _template_ **substituirá** o `innerHTML` do elemento montado. Qualquer marcação existente dentro do elemento montado será ignorada, a menos que os _slots_ de distribuição de conteúdo estejam presentes no _template_.

  Se a string começar com `#` ela será usada como um `querySelector` e usará o `innerHTML` do elemento selecionado como a _template_ string. Isso permite o uso do truque comum `<script type="x-template">` para incluir _templates_.

  :::tip Nota
  De uma perspectiva de segurança, você deve usar apenas _templates_ Vue nos quais possa confiar. Nunca use conteúdo gerado pelo usuário como seu _template_.
  :::

  :::tip Nota
  Se a função de renderização estiver presente na opção Vue, o _template_ será ignorado.
  :::

- **Ver também:**
   - [Diagrama do ciclo de vida](../guide/instance.html#diagrama-do-ciclo-de-vida)
   - [Distribuição de conteúdo com _slots_](../guide/component-basics.html#distribuicao-de-conteudo-com-slots)

## `render`

- **Tipo:** `Function`

- **Detalhes:**

  Uma alternativa aos _templates_ string, permitindo que você aproveite todo o poder programático do JavaScript.

- **Uso:**

  ```html
  <div id="app" class="demo">
    <my-title blog-title="Uma Vue (view) Perfeita"></my-title>
  </div>
  ```

  ```js
  const { createApp, h } = Vue
  const app = createApp({})

  app.component('my-title', {
    render() {
      return h(
        'h1', // nome da tag,
        this.blogTitle // conteúdo da tag
      )
    },
    props: {
      blogTitle: {
        type: String,
        required: true
      }
    }
  })

  app.mount('#app')
  ```

  :::tip Nota
  A função `render` tem prioridade sobre a função de renderização compilada da opção `template` ou o _template_ HTML no-DOM do elemento sendo montado
  :::

- **Ver também:** [Funções de Renderização](../guide/render-function.html)
