---
badges:
  - breaking
---

# Atributo _Inline Template_ <MigrationBadges :badges="$frontmatter.badges" />

## Visão Geral

O suporte ao [recurso _inline-template_](https://br.vuejs.org/v2/guide/components-edge-cases.html#Templates-Inline) foi removido.

## Sintaxe v2.x

Na versão 2.x, o Vue disponibilizava o atributo `inline-template` em componentes filhos para usar seu conteúdo interno como _template_ ao invés de o tratar como conteúdo distribuído.

```html
<my-component inline-template>
  <div>
    <p>Estes são compilados como template do próprio componente.</p>
    <p>Não como conteúdo da transclusão do componente pai.</p>
  </div>
</my-component>
```

## Sintaxe v3.x

Esta funcionalidade não receberá mais suporte.

## Estratégia de Migração

A maioria dos casos de uso de `inline-template` assume um ambiente sem ferramentas de compilação, onde todos os _templates_ são escritos diretamente dentro da página HTML.

[Sinalizador na compilação de migração: `COMPILER_INLINE_TEMPLATE`](migration-build.html#configuracao-de-compatibilidade)

### Opção #1: Utilize a tag `<script>`

A solução mais simples nestes casos é utilizar a tag `<script>` com um tipo alternativo:

```html
<script type="text/html" id="my-comp-template">
  <div>{{ hello }}</div>
</script>
```

E no componente, aponte para o _template_ utilizando um seletor:

```js
const MyComp = {
  template: '#my-comp-template'
  // ...
}
```

Isto não requer nenhuma configuração de compilação, funciona em todos os navegadores, não está sujeito a quaisquer limitações da análise de HTML no-DOM (ex.: você pode usar nomes de propriedades em camelCase) e fornece destaque de sintaxe na maioria das IDEs. Em frameworks tradicionais que trabalham no lado do servidor, estes _templates_ podem ser divididos em partes de _templates_ do servidor (incluídas no _template_ HTML principal) para uma melhor manutenção.

### Opção #2: _Slot_ _Default_

Um componente que anteriormente utilizava `inline-template` também pode ser refatorado utilizando o _slot_ _default_ - o que torna a definição do escopo de dados mais explícita enquanto preserva a conveniência de escrever o conteúdo filho _inline_:

```html
<!-- Sintaxe v2.x -->
<my-comp inline-template :msg="parentMsg">
  {{ msg }} {{ childState }}
</my-comp>

<!-- Versão com Slot Default -->
<my-comp v-slot="{ childState }">
  {{ parentMsg }} {{ childState }}
</my-comp>
```

O componente filho, ao invés de não fornecer nenhum _template_, agora deve renderizar o _slot_ _default_\*:

```html
<!--
  no template filho, renderiza o slot default enquanto passa
  o estado privado necessário do filho.
-->
<template>
  <slot :childState="childState" />
</template>
```

> - Nota: na versão 3.x, _slots_ podem ser renderizados como raiz com suporte nativo a [fragmentos](/guide/migration/fragments)!
