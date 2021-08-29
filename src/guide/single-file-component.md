# Componentes Single-File

## Introdução

Componentes Single-File do Vue (também conhecido como arquivos `*.vue`, abreviado como **SFC**) é um formato de arquivo especial que nos permite encapsular o *template*, lógica **e** estilização de um componente Vue em um único arquivo. Aqui está um exemplo de SFC:

```vue
<script>
export default {
  data() {
    return {
      greeting: 'Olá Mundo!'
    }
  }
}
</script>

<template>
  <p class="greeting">{{ greeting }}</p>
</template>

<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
```

Como podemos ver, SFC no Vue é uma extensão natural do trio clássico de HTML, CSS e JavaScript. Cada arquivo `*.vue` consiste em três tipos de blocos de linguagem de nível superior: `<template>`, `<script>` e `<style>`:

- A seção `<script>` é um módulo JavaScript padrão. Ele deve exportar uma definição de componente Vue como sua exportação padrão.
- A seção `<template>` define o *template* (árvore HTML) do componente.
- A seção `<style>` define o CSS associado ao componente.

Verifique mais detalhes na [Especificação da sintaxe SFC](/api/sfc-spec).

## Como funciona

Vue SFC is a framework-specific file format and must be pre-compiled by [@vue/compiler-sfc](https://github.com/vuejs/vue-next/tree/master/packages/compiler-sfc) into standard JavaScript and CSS. A compiled SFC is a standard JavaScript (ES) module - which means with proper build setup you can import an SFC like a module:

```js
import MyComponent from './MyComponent.vue'

export default {
  components: {
    MyComponent
  }
}
```

`<style>` tags inside SFCs are typically injected as native `<style>` tags during development to support hot updates. For production they can be extracted and merged into a single CSS file.

You can play with SFCs and explore how they are compiled in the [Vue SFC Playground](https://sfc.vuejs.org/).

In actual projects, we typically integrate the SFC compiler with a build tool such as [Vite](https://vitejs.dev/) or [Vue CLI](http://cli.vuejs.org/) (which is based on [webpack](https://webpack.js.org/)), and Vue provides official scaffolding tools to get you started with SFCs as fast as possible. Check out more details in the [SFC Tooling](/api/sfc-tooling) section.

## Why SFC

While SFCs require a build step, there are numerous benefits in return:

- Author modularized components using familiar HTML, CSS and JavaScript syntax
- Pre-compiled templates
- [Component-scoped CSS](/api/sfc-style)
- [More ergonomic syntax when working with Composition API](/api/sfc-script-setup)
- More compile-time optimizations by cross-analyzing template and script
- [IDE support](/api/sfc-tooling.html#ide-support) with auto-completion and type-checking for template expressions
- Out-of-the-box Hot-Module Replacement (HMR) support

SFC is a defining feature of Vue as a framework, and is the recommended approach for using Vue in the following scenarios:

- Single-Page Applications (SPA)
- Static Site Generation (SSG)
- Any non-trivial frontends where a build step can be justified for better development experience (DX).

That said, we do realize there are scenarios where SFCs can feel like overkill. This is why Vue can still be used via plain JavaScript without a build step. If you are just looking for enhancing largely static HTML with light interactions, you can also check out [petite-vue](https://github.com/vuejs/petite-vue), a 5kb subset of Vue optimized for progressive enhancement.

## What About Separation of Concerns?

Some users coming from a traditional web development background may have the concern that SFCs are mixing different concerns in the same place - which HTML/CSS/JS were supposed to separate!

To answer this question, it is important for us to agree that **separation of concerns is not equal to separation of file types.** The ultimate goal of engineering principles is to improve maintainability of codebases. Separation of concerns, when applied dogmatically as separation of file types, does not help us reach that goal in the context of increasingly complex frontend applications.

In modern UI development, we have found that instead of dividing the codebase into three huge layers that interweave with one another, it makes much more sense to divide them into loosely-coupled components and compose them. Inside a component, its template, logic, and styles are inherently coupled, and collocating them actually makes the component more cohesive and maintainable.

Note even if you don't like the idea of Single-File Components, you can still leverage its hot-reloading and pre-compilation features by separating your JavaScript and CSS into separate files using [Src Imports](/api/sfc-spec.html#src-imports).
