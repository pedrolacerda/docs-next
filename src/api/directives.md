# Diretivas

## v-text

- **Espera:** `string`

- **Detalhes:**

  Atualiza o [textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent) do elemento. Se você precisa atualizar a parte de `textContent`, você deve usar [interpolações mustache](/guide/template-syntax.html#texto).

- **Exemplo:**

  ```html
  <span v-text="msg"></span>
  <!-- o mesmo que -->
  <span>{{msg}}</span>
  ```

- **Ver também:** [Sintaxe de Templates - Interpolações](../guide/template-syntax.html#texto)

## v-html

- **Espera:** `string`

- **Detalhes:**

  Atualiza o [innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) do elemento. **Perceba que os conteúdos são inseridos como HTML puro - eles não serão compilados como templates Vue**. Se você se encontra tentando compor *templates* usando `v-html`, tente repensar a solução usando componentes em vez disso.

  ::: warning AVISO
  Renderizar dinamicamente HTML arbitrário em seu website pode ser muito perigoso porque isso pode facilmente levar a [ataques XSS](https://en.wikipedia.org/wiki/Cross-site_scripting). Apenas use `v-html` em conteúdo confiável e **nunca** em conteúdo fornecido pelo usuário.
  :::

  Em [Componentes Single-File](../guide/single-file-component.html), estilos `scoped` não serão aplicados a conteúdos `v-html`, pois tal HTML não é processado pelo compilador de *templates* do Vue. Se você quer atingir o conteúdo de `v-html` com CSS `scoped`, é possível utilizar [CSS Modules](https://vue-loader.vuejs.org/en/features/css-modules.html) ou um adicional elemento `<style>` global, com uma estratégia manual de escopo como BEM.

- **Exemplo:**

  ```html
  <div v-html="html"></div>
  ```

- **Ver também:** [Sintaxe de Templates - Interpolações](../guide/template-syntax.html#raw-html)

## v-show

- **Espera:** `any`

- **Uso:**

  Alterna a propriedade CSS `display` do elemento baseado na condição de verdade do valor da expressão.

  Esta diretiva dispara transições quando sua condição muda.

- **Ver também:** [Renderização Condicional - v-show](../guide/conditional.html#v-show)

## v-if

- **Espera:** `any`

- **Uso:**

  Renderiza condicionalmente o elemento baseado na condição de verdade do valor da expressão. O elemento e suas diretivas/componentes contidos são destruídos e reconstruídos durante alternâncias (*toggles*). Se o elemento é um `<template>`, o seu conteúdo será extraído como o bloco condicional.

  Esta diretiva dispara transições quando sua condição muda.

  Quando usados juntos, `v-if` tem uma prioridade maior que `v-for`. Não recomendamos o uso dessas duas diretivas juntas em um elemento - consulte o [guia de renderização de listas](../guide/list.html#utilizando-v-if-com-v-for) para obter detalhes.

- **Ver também:** [Renderização Condicional - v-if](../guide/conditional.html#v-if)

## v-else

- **Não espera expressão**

- **Restriction:** o elemento-irmão anterior deve ter `v-if` ou `v-else-if`.

- **Uso:**

  Denota o "bloco senão" de um `v-if` ou cadeia de `v-if`/`v-else-if`.

  ```html
  <div v-if="Math.random() > 0.5">
    Agora você me vê
  </div>
  <div v-else>
    Agora você não me vê
  </div>
  ```

- **Ver também:** [Renderização Condicional - v-else](../guide/conditional.html#v-else)

## v-else-if

- **Espera:** `any`

- **Restriction:** o elemento-irmão anterior deve ter `v-if` ou `v-else-if`.

- **Uso:**

  Denota o "bloco senão se" para `v-if`. Pode ser encadeado.

  ```html
  <div v-if="type === 'A'">
    A
  </div>
  <div v-else-if="type === 'B'">
    B
  </div>
  <div v-else-if="type === 'C'">
    C
  </div>
  <div v-else>
    Não é A/B/C
  </div>
  ```

- **Ver também:** [Renderização Condicional - v-else-if](../guide/conditional.html#v-else-if)

## v-for

- **Espera:** `Array | Object | number | string | Iterable`

- **Uso:**

  Renderiza o elemento ou bloco de *template* múltiplas vezes baseado nos dados de origem (*source data*). O valor da diretiva deve usar a sintaxe especial `alias in expression` para fornecer um apelido para o elemento atual sendo iterado:

  ```html
  <div v-for="item in items">
    {{ item.text }}
  </div>
  ```

  Alternativamente, você também pode especificar um apelido para o índice (ou a chave, se usada em um Objeto):

  ```html
  <div v-for="(item, index) in items"></div>
  <div v-for="(value, key) in object"></div>
  <div v-for="(value, name, index) in object"></div>
  ```

  O comportamento padrão de `v-for` tentará corrigir os elementos no local sem os mover. Para forçá-lo a reordenar elementos, você precisa fornecer uma sugestão de ordenação com o atributo especial `key`:

  ```html
  <div v-for="item in items" :key="item.id">
    {{ item.text }}
  </div>
  ```

  `v-for` também pode funcionar em valores que implementam o [Protocolo Iterável](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterable_protocol), incluindo `Map` e `Set` nativos.

  O uso detalhado de `v-for` é explicado na seção do guia com *link* abaixo.

- **Ver também:**
  - [Renderização de Listas](../guide/list.html)

## v-on

- **Forma abreviada:** `@`

- **Espera:** `Function | Inline Statement | Object`

- **Argumento:** `event`

- **Modificadores:**

  - `.stop` - chama `event.stopPropagation()`.
  - `.prevent` - chama `event.preventDefault()`.
  - `.capture` - adiciona escuta de eventos em modo de captura.
  - `.self` - aciona o manipulador somente se o evento foi disparado a partir deste elemento.
  - `.{keyAlias}` - aciona o manipulador apenas em certas teclas.
  - `.once` - aciona o manipulador somente uma vez.
  - `.left` - aciona o manipulador somente para eventos do botão esquerdo do mouse.
  - `.right` - aciona o manipulador somente para eventos do botão direito do mouse.
  - `.middle` - aciona o manipulador somente para eventos do botão do meio do mouse.
  - `.passive` - atribui um evento ao DOM com `{ passive: true }`.

- **Uso:**

  Atribui uma escuta de evento ao elemento. O tipo de evento é indicado pelo argumento. A expressão pode ser um nome de método, uma declaração *inline*, ou omitida quando há modificadores presentes.

  Quando usado em um elemento normal, escuta somente [**eventos nativos do DOM**](https://developer.mozilla.org/en-US/docs/Web/Events). Quando usado em um componente de elemento personalizado, escuta **eventos personalizados** emitidos naquele componente-filho.

  Quando escutando a eventos nativos do DOM, o método recebe o evento nativo como argumento único. Quando usada declaração *inline*, ela tem acesso à propriedade especial `$event`: `v-on:click="handle('ok', $event)"`.

  `v-on` também oferece suporte à vinculação a um objeto de pares evento/ouvinte sem um argumento. Observe que ao usar a sintaxe de objeto, ela não oferece suporte a nenhum modificador.

- **Exemplo:**

  ```html
  <!-- método manipulador -->
  <button v-on:click="doThis"></button>

  <!-- evento dinâmico -->
  <button v-on:[event]="doThis"></button>

  <!-- declaração inline -->
  <button v-on:click="doThat('hello', $event)"></button>

  <!-- forma abreviada -->
  <button @click="doThis"></button>

  <!-- forma abreviada para o evento dinâmico -->
  <button @[event]="doThis"></button>

  <!-- para a propagação -->
  <button @click.stop="doThis"></button>

  <!-- previne o padrão -->
  <button @click.prevent="doThis"></button>

  <!-- previne o padrão sem expressão -->
  <form @submit.prevent></form>

  <!-- modificadores encadeados -->
  <button @click.stop.prevent="doThis"></button>

  <!-- modificador de tecla usando keyAlias -->
  <input @keyup.enter="onEnter" />

  <!-- o evento de clique será acionado somente uma vez -->
  <button v-on:click.once="doThis"></button>

  <!-- sintaxe de objeto -->
  <button v-on="{ mousedown: doThis, mouseup: doThat }"></button>
  ```

  Escutando eventos personalizados em um componente-filho (o manipulador é chamado quando “my-event” é emitido no filho):

  ```html
  <my-component @my-event="handleThis"></my-component>

  <!-- declaração inline -->
  <my-component @my-event="handleThis(123, $event)"></my-component>
  ```

- **Ver também:**
  - [Manipulação de Eventos](../guide/events.html)
  - [Componentes - Eventos Personalizados](../guide/component-basics.html#escutando-eventos-dos-filhos)

## v-bind

- **Forma abreviada:** `:`

- **Espera:** `any (com argumento) | Object (sem argumento)`

- **Argumento:** `attrOrProp (opcional)`

- **Modificadores:**

  - `.camel` - transforma o nome do atributo de *kebab-case* para *camelCase*.

- **Uso:**

  Dinamicamente vincula um ou mais atributos ou propriedades de um componente a uma expressão.

  Quando usado para o vínculo de atributos `class` ou `style`, suporta tipos de valores adicionais como Array ou Objects. Veja na seção do guia com *link* abaixo para mais detalhes.

  Quando usado para vincular uma propriedade, a propriedade deve estar devidamente declarada no elemento-filho.

  Quando usado sem argumento, pode ser utilizado para vincular em um objeto contendo pares nome-valor. Perceba que, neste modo,`class` e `style` não suportam Array ou Objects.

- **Exemplo:**

  ```html
  <!-- vinculando a um atributo -->
  <img v-bind:src="imageSrc" />

  <!-- nome de atributo dinâmico -->
  <button v-bind:[key]="value"></button>

  <!-- forma abreviada -->
  <img :src="imageSrc" />

  <!-- forma abreviada do nome de atributo dinâmico -->
  <button :[key]="value"></button>

  <!-- com concatenação de string inline -->
  <img :src="'/path/to/images/' + fileName" />

  <!-- vinculando classes -->
  <div :class="{ red: isRed }"></div>
  <div :class="[classA, classB]"></div>
  <div :class="[classA, { classB: isB, classC: isC }]">
    <!-- vinculando estilos -->
    <div :style="{ fontSize: size + 'px' }"></div>
    <div :style="[styleObjectA, styleObjectB]"></div>

    <!-- vinculando um objeto com atributos -->
    <div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

    <!-- vinculando propriedade. "prop" deve estar declarado em my-component -->
    <my-component :prop="someThing"></my-component>

    <!-- transmite todas as props do pai em comum com o componente-filho -->
    <child-component v-bind="$props"></child-component>

    <!-- XLink -->
    <svg><a :xlink:special="foo"></a></svg>
  </div>
  ```

  O modificador `.camel` permite colocar na notação *camelCase* (*camelizing*) um nome do atributo `v-bind` quando usado em *templates* no DOM, por exemplo, o atributo `viewBox` de um SVG:

  ```html
  <svg :view-box.camel="viewBox"></svg>
  ```

  `.camel` não é necessário se você está usando *templates* baseados em String ou compilando com `vue-loader`/`vueify`.

- **Ver também:**
  - [Interligações em Classes e Estilos](../guide/class-and-style.html)
  - [Componentes - Propriedades](../guide/component-basics.html#passando-dados-aos-filhos-com-propriedades)

## v-model

- **Espera:** varia baseado no valor dos elementos de input de formulário ou *output* de componentes

- **Limitado a:**

  - `<input>`
  - `<select>`
  - `<textarea>`
  - componentes

- **Modificadores:**

  - [`.lazy`](../guide/forms.html#lazy) - escuta por eventos `change` ao invés de `input`
  - [`.number`](../guide/forms.html#number) - converte a String do input válida para números
  - [`.trim`](../guide/forms.html#trim) - faz *trim* dos dados do input

- **Uso:**

  Cria um vínculo de mão dupla (*two-way binding*) em um elemento input de formulário ou componente. Para uso detalhado e outras observações, veja o *link* abaixo para a seção do Guia.

- **Ver também:**
  - [Interligações em Formulários](../guide/forms.html)
  - [Componentes - Componentes de Formulários usando Eventos Personalizados](../guide/component-custom-events.html#argumentos-do-v-model)

## v-slot

- **Forma abreviada:** `#`

- **Espera:** expressão JavaScript que seja válida na posição de um argumento de função (aceita desestruturação em [ambientes suportados](../guide/component-slots.html#desestruturando-props-do-slot)). Opcional - somente necessário se estiver esperando que propriedades sejam passadas ao *slot*.

- **Argumento:** nome do *slot* (opcional, o valor padrão é `default`)

- **Limitado a:**

  - `<template>`
  - [componentes](../guide/component-slots.html#sintaxe-abreviada-para-slot-unico-e-default) (para um único *slot* padrão com propriedades)

- **Uso:**

  Denote *slots* nomeados ou *slots* que esperam receber propriedades.

- **Exemplo:**

  ```html
  <!-- Slots nomeados -->
  <base-layout>
    <template v-slot:header>
      Conteúdo do Header
    </template>

    <template v-slot:default>
      Conteúdo do slot Default
    </template>

    <template v-slot:footer>
      Conteúdo do Footer
    </template>
  </base-layout>

  <!-- Slot nomeado que recebe propriedades -->
  <infinite-scroll>
    <template v-slot:item="slotProps">
      <div class="item">
        {{ slotProps.item.text }}
      </div>
    </template>
  </infinite-scroll>

  <!-- Slot padrão que recebe propriedades, com desestruturação -->
  <mouse-position v-slot="{ x, y }">
    Mouse position: {{ x }}, {{ y }}
  </mouse-position>
  ```

  Para mais detalhes, veja os *links* abaixo.

- **Ver também:**
  - [Components - Slots](../guide/component-slots.html)

## v-pre

- **Não espera expressão**

- **Uso:**

  Pula a compilação para esse elemento e todos seus filhos. Você pode usar isso para mostrar tags mustache sem conversão (*raw*). Pular uma grande quantidade de nós sem diretivas também pode acelerar a compilação.

- **Exemplo:**

  ```html
  <span v-pre>{{ isso não será compilado }}</span>
  ```

## v-cloak

- **Não espera expressão**

- **Uso:**

  Essa diretiva permanecerá no elemento até que a instância de componente associada termine de compilar. Ao ser combinada com regras CSS como `[v-cloak] { display: none }`, essa diretiva pode ser usada para esconder interligações mustache não-compiladas até que a instância do componente esteja pronta.

- **Exemplo:**

  ```css
  [v-cloak] {
    display: none;
  }
  ```

  ```html
  <div v-cloak>
    {{ message }}
  </div>
  ```

  A `<div>` não ficará visível até que a compilação tenha terminado.

## v-once

- **Não espera expressão**

- **Detalhes:**

  Renderiza o elemento e componente **apenas uma vez**. Em re-renderizações subsequentes, o elemento/componente e todos seus filhos serão tratados como conteúdo estático e pulados. Isso pode ser usado para otimizar o desempenho da atualização.

  ```html
  <!-- elemento único -->
  <span v-once>Isso nunca vai mudar: {{msg}}</span>
  <!-- o elemento tem filhos -->
  <div v-once>
    <h1>comentário</h1>
    <p>{{msg}}</p>
  </div>
  <!-- componente -->
  <my-component v-once :comment="msg"></my-component>
  <!-- diretiva `v-for` -->
  <ul>
    <li v-for="i in list" v-once>{{i}}</li>
  </ul>
  ```

- **Ver também:**
  - [Sintaxe de Templates - Interpolações](../guide/template-syntax.html#texto)

## v-is

> Note: esta seção afeta apenas os casos em que os *templates* Vue são escritos diretamente no HTML da página.

- **Espera:** string literal

- **Limitado a:** elementos HTML nativos

- **Uso:** Ao usar *templates* no DOM, o *template* está sujeito às regras de análise nativas de HTML. Alguns elementos HTML, como `<ul>`, `<ol>`, `<table>` e `<select>` têm restrições sobre quais elementos podem aparecer dentro deles, e alguns elementos como `<li>`, `<tr>`, e `<option>` só podem aparecer dentro de alguns determinados elementos. Como solução alternativa, podemos usar a diretiva `v-is` nestes elementos:

```html
<table>
  <tr v-is="'blog-post-row'"></tr>
</table>
```

:::warning AVISO
`v-is` funciona como o vínculo dinâmico `:is` da v2.x - então, para renderizar um componente por seu nome registrado, seu valor deve ser um string literal de JavaScript:

```html
<!-- Incorreto, nada será renderizado -->
<tr v-is="blog-post-row"></tr>

<!-- Correto -->
<tr v-is="'blog-post-row'"></tr>
```

:::
