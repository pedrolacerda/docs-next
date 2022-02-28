---
badges:
  - breaking
---

# Interop. de Elementos Personalizados <MigrationBadges :badges="$frontmatter.badges" />

## Visão Geral

- **QUEBRA:** As verificações para determinar se as tags devem ser tratadas como elementos personalizados agora são executadas durante a compilação do _template_ e devem ser configuradas por meio de opções do compilador em vez de configuração em tempo de execução.
- **QUEBRA:** O uso do atributo especial `is` é restrito apenas à tag reservada `<component>`.
- **NOVO:** Para suportar casos de uso da v2.x onde `is` foi usado em elementos nativos para contornar restrições da análise de HTML nativo, prefixe o valor com `vue:` para resolvê-lo como um componente Vue.

## Elementos Personalizados Autônomos

Se quisermos adicionar um elemento personalizado definido fora do Vue (ex.:, usando a API de _Web Components_), precisamos 'instruir' o Vue a tratá-lo como um elemento personalizado. Vamos usar o _template_ a seguir como exemplo.

```html
<plastic-button></plastic-button>
```

### Sintaxe v2.x

No Vue 2.x, a configuração de tags como elementos personalizados era feita via `Vue.config.ignoredElements`:

```js
// Isso fará com que o Vue ignore o elemento personalizado definido fora do Vue
// (ex.: usando as APIs de Web Components)

Vue.config.ignoredElements = ['plastic-button']
```

### Sintaxe v3.x

**No Vue 3.0, esta verificação é realizada durante a compilação do _template_.** Para instruir o compilador a tratar `<plastic-button>` como um elemento personalizado:

- Se estiver usando uma etapa de compilação: passe a opção `isCustomElement` para o compilador de _templates_ do Vue. Se estiver usando `vue-loader`, isso deve ser passado através da opção `compilerOptions` do `vue-loader`:

  ```js
  // na configuração do webpack
  rules: [
    {
      test: /\.vue$/,
      use: 'vue-loader',
      options: {
        compilerOptions: {
          isCustomElement: tag => tag === 'plastic-button'
        }
      }
    }
    // ...
  ]
  ```

- Se estiver usando a compilação de _template_ _on-the-fly_, passe-o via `app.config.isCustomElement`:

  ```js
  const app = Vue.createApp({})
  app.config.isCustomElement = tag => tag === 'plastic-button'
  ```

  É importante observar que a configuração em tempo de execução afeta apenas a compilação do _template_ em tempo de execução - não afetará os _templates_ pré-compilados.

## Elementos Integrados Personalizados

A especificação de Elementos Personalizados fornece uma maneira de usar elementos personalizados como um [Elemento Integrado Personalizado](https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements-customized-builtin-example) adicionando o atributo `is` a um elemento interno:

```html
<button is="plastic-button">Clique em mim!</button>
```

O uso do atributo especial `is` pelo Vue estava simulando o que o atributo nativo faz antes de ser disponibilizado universalmente nos navegadores. No entanto, na v2.x foi interpretado como renderizando um componente Vue com o nome `plastic-button`. Isso bloqueia o uso nativo do Elemento Integrado Personalizado mencionado acima.

Na v3.0, estamos limitando o tratamento especial do Vue ao atributo `is` apenas para a tag `<component>`.

- Quando usado na tag reservada `<component>`, se comportará exatamente como na v2.x;
- Quando usado em componentes normais, ele se comportará como um atributo normal:

  ```html
  <foo is="bar" />
  ```

  - Comportamento 2.x: renderiza o componente `bar`.
  - Comportamento 3.x: renderiza o componente `foo` e passa o atributo `is`.

- Quando usado em elementos simples, será passado para a chamada `createElement` como o prop `is`, e também renderizado como um atributo nativo. Isso suporta o uso de elementos internos personalizados.

  ```html
  <button is="plastic-button">Clique em mim!</button>
  ```

  - Comportamento v2.x: renderiza o componente `plastic-button`.
  - Comportamento v3.x: renderiza um botão nativo chamando

    ```js
    document.createElement('button', { is: 'plastic-button' })
    ```

[Sinalizador na compilação de migração: `COMPILER_IS_ON_ELEMENT`](migration-build.html#configuracao-de-compatibilidade)

## Prefixo `vue:` para _Workarounds_ na Análise do _template_ no DOM

> Nota: esta seção afeta apenas os casos em que os _templates_ Vue são escritos diretamente no HTML da página.
> Ao usar _templates_ no DOM, o _template_ está sujeito a regras de análise de HTML nativo. Alguns elementos HTML, como `<ul>`, `<ol>`, `<table>` e `<select>` têm restrições sobre quais elementos podem aparecer dentro deles, e alguns elementos como `<li>`, `<tr>` e `<option>` só podem aparecer dentro de alguns outros elementos.

### Sintaxe v2.x

No Vue 2, recomendamos trabalhar com essas restrições usando o atributo `is` em uma tag nativa:

```html
<table>
  <tr is="blog-post-row"></tr>
</table>
```

### Sintaxe v3.x

Com a mudança de comportamento de `is`, um prefixo `vue:` agora é necessário para resolver o elemento como um componente Vue:

```html
<table>
  <tr is="vue:blog-post-row"></tr>
</table>
```

## Estratégia de Migração

- Substitua `config.ignoredElements` por `vue-loader`'s `compilerOptions` (com a etapa de compilação) ou `app.config.isCustomElement` (com compilação de _template_ _on-the-fly_)

- Altere todas as tags não-`<component>` com uso de `is` para `<component is="...">` (para _templates_ SFC) ou prefixe-as com `vue:` (para _templates_ no DOM).
