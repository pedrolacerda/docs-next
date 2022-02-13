# Diversos

## name

- **Tipo:** `string`

- **Detalhes:**

  Permite que o componente invoque si mesmo recursivamente no *template*. Observe que quando um componente é registrado globalmente com [`app.component`](/api/application-api.html#component), o ID global é automaticamente definido como seu nome.

  Outro benefício de especificar a opção `name` é a depuração. Componentes nomeados resultam em mensagens de aviso mais úteis. Também, ao inspecionar um *app* no [vue-devtools](https://github.com/vuejs/vue-devtools), componentes sem nome serão exibidos como `<AnonymousComponent>`, o que não é muito informativo. Provendo a opção `name`, você terá uma árvore de componentes muito mais informativa.

## inheritAttrs

- **Tipo:** `boolean`

- **Predefinição:** `true`

- **Detalhes:**

  Por padrão, vínculos de atributos do escopo pai irreconhecíveis como props vão "cair" (*fallthrough*). Isso signifca que quando nós temos um componente de raiz única, esses vínculos serão aplicados ao elemento raiz do componente filho como atributos normais do HTML. Ao criar um componente que envolve um elemento alvo ou outro componente, nem sempre este será o comportamento desejado. Configurando `inheritAttrs` para `false`, esse comportamento padrão pode ser desabilitado. Os atributos estarão disponíveis via a propriedade de instância `$attrs` que pode ser explicitamente vinculada a um elemento que não esteja na raiz usando `v-bind`.

- **Uso:**

  ```js
  app.component('base-input', {
    inheritAttrs: false,
    props: ['label', 'value'],
    emits: ['input'],
    template: `
      <label>
        {{ label }}
        <input
          v-bind="$attrs"
          v-bind:value="value"
          v-on:input="$emit('input', $event.target.value)"
        >
      </label>
    `
  })
  ```

- **Veja também:** [Desativando a Herança de Atributos](../guide/component-attrs.html#desativando-a-heranca-de-atributos)

## compilerOptions <Badge text="3.1+" />

- **Tipo:** `Object`

- **Detalhes:**

  Este é o equivalente em nível de componente da [configuração em nível de aplicativo `compilerOptions`](/api/application-config.html#compileroptions).

- **Uso:**

  ```js
  const Foo = {
    // ...
    compilerOptions: {
      delimiters: ['${', '}'],
      comments: true
    }
  }
  ```

  ::: tip Importante
  Semelhante à configuração `compilerOptions` em nível de aplicativo, esta opção só é respeitada ao usar a distribuição completa com a compilação de *template* no navegador.
  :::

## delimiters <Badge text="deprecated" type="warning" />

Obsoleto na v3.1.0. Use `compilerOptions.delimiters` em vez disso.
