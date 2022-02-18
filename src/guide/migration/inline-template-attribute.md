---
badges:
  - breaking
---

# Atributo Inline Template <MigrationBadges :badges="$frontmatter.badges" />

## Visão geral

O suporte a [funcionalidade inline-template](https://vuejs.org/v2/guide/components-edge-cases.html#Inline-Templates) foi removido.

## Sintaxe 2.x

Na versão 2.x, o Vue disponibilizava o atributo `inline-template` em componentes filhos para usar seu conteúdo interno como *template* ao invés de o tratar como conteúdo distribuído.

```html
<my-component inline-template>
  <div>
    <p>Estes são compilados como template do próprio componente.</p>
    <p>Não como conteúdo da transclusão dos pais.</p>
  </div>
</my-component>
```

## Sintaxe 3.x

Esta funcionalidade não receberá mais suporte.

## Estratégia de Migração

A maioria dos casos de uso de `inline-template` assume uma configuração sem ferramentas de compilação, onde todos os templates são escritos diretamente dentro da página HTML.

[Sinalizador da compilação de migração: `COMPILER_INLINE_TEMPLATE`](migration-build.html#compat-configuration)

### Opção #1: Utilize a tag `<script>`

A solução mais simples nestes casos é utilizar a tag `<script>` com um tipo alternativo:

```html
<script type="text/html" id="my-comp-template">
  <div>{{ hello }}</div>
</script>
```

E no componente, aponte para o *template* utilizando um seletor:

```js
const MyComp = {
  template: '#my-comp-template'
  // ...
}
```

Isto não requer nenhuma configuração de compilação, funciona em todos os navegadores, não está sujeito a quaisquer ressalvas de análise HTML em DOM (por exemplo, você pode usar nomes de propriedades em camelCase), e fornece destaque de sintaxe na maioria das IDEs. Em *frameworks* tradicionais do lado do servidor, estes *templates* podem ser divididos em partes de *templates* do servidor (incluídas no template HTML principal) para uma melhor manutenção.

### Opção #2: Slot Padrão

Um componente que anteriormente utilizava `inline-template` também pode ser refatorado utilizando o *slot* padrão - o que torna a definição do escopo de dados mais explícita enquanto preserva a conveniência de escrever o conteúdo filho em linha:

```html
<!-- Sintaxe 2.x -->
<my-comp inline-template :msg="parentMsg">
  {{ msg }} {{ childState }}
</my-comp>

<!-- Versão com Slot Padrão -->
<my-comp v-slot="{ childState }">
  {{ parentMsg }} {{ childState }}
</my-comp>
```

O componente filho, ao invés de não fornecer nenhum template, agora deve renderizar o *slot* padrão\*:

```html
<!--
  no template filho, renderiza o slot padrão enquanto passa
  o estado privado necessário do filho.
-->
<template>
  <slot :childState="childState" />
</template>
```

> - Nota: na versão 3.x, *slots* podem ser renderizados como raiz com suporte nativo a [fragmentos](/guide/migration/fragments)!
