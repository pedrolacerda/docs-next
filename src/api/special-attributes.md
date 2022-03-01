# Atributos Especiais

## key

- **Espera:** `number | string | symbol`

  O atributo especial `key` é usado principalmente como uma dica para o algoritmo de DOM virtual do Vue identificar VNodes ao comparar a nova lista de nós com a anterior. Sem as chaves (`key`), o Vue usa um algoritmo que minimiza a movimentação de elementos e tenta corrigir/reusar elementos do mesmo tipo no local, tanto quanto possível. Com as chaves, ele reordenará os elementos com base na alteração da ordem das chaves, e os elementos com chaves que não estão mais presentes sempre serão removidos/destruídos.

  Filhos do mesmo pai comum devem ter **chaves únicas**. Chaves duplicadas causarão erros de renderização.

  O caso de uso mais comum é combinado com `v-for`:

  ```html
  <ul>
    <li v-for="item in items" :key="item.id">...</li>
  </ul>
  ```

  Também pode ser usado para forçar a substituição de um elemento/componente em vez de reutilizá-lo. Isso pode ser útil quando você deseja:

  - Acionar corretamente gatilhos de ciclo de vida de um componente
  - Acionar transições

  Por exemplo:

  ```html
  <transition>
    <span :key="text">{{ text }}</span>
  </transition>
  ```

  Quando `text` mudar, o `<span>` sempre será substituído ao invés de alterado, então uma transição será acionada.

## ref

- **Espera:** `string | Function`

  `ref` é usado para registrar uma referência a um elemento ou componente filho. A referência será registrada no objeto `$refs` do componente pai. Se usado em um elemento DOM simples, a referência será esse elemento; se usado em um componente filho, a referência será a instância do componente:

  ```html
  <!-- vm.$refs.p será o nó DOM -->
  <p ref="p">Olá</p>

  <!-- vm.$refs.child será a instância do componente filho -->
  <child-component ref="child"></child-component>

  <!-- Quando vinculado dinamicamente, podemos definir "ref" como uma função "callback", passando o elemento ou instância do componente explicitamente -->
  <child-component :ref="(el) => child = el"></child-component>
  ```

  Uma observação importante sobre o tempo de registro de uma "ref": como as próprias refs são criadas como resultado da função de renderização, você não pode acessá-las na renderização inicial - eles ainda não existem! `$refs` também não é reativo, portanto, você não deve tentar usá-lo em *templates* para vinculação de dados.

- **Veja também:** [Refs de Componente Filho](../guide/component-template-refs.html)

## is

- **Espera:** `string | Object (objeto de opções do componente)`

  Usado para [componentes dinâmicos](../guide/component-dynamic-async.html).

  Por exemplo:

  ```html
  <!-- componente muda quando currentView muda -->
  <component :is="currentView"></component>
  ```

- **Uso em elementos nativos** <Badge text="3.1+" />

  Quando o atributo `is` é usado em um elemento HTML nativo, ele será interpretado como um [Elemento integrado personalizado](https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements-customized-builtin-example), que é um recurso nativo da plataforma web.

  No entanto, há um caso de uso em que você pode precisar do Vue para substituir um elemento nativo por um componente Vue, conforme explicado em [Ressalvas na Análise do _template_ DOM](/guide/component-basics.html#ressalvas-na-analise-do-template-dom). Você pode prefixar o valor do atributo `is` com `vue:` para que o Vue renderize o elemento como um componente Vue:

  ```html
  <table>
    <tr is="vue:my-row-component"></tr>
  </table>
  ```

- **Veja também:**
  - [Componentes Dinâmicos](../guide/component-dynamic-async.html)
  - [RFC explicando a mudança desde o Vue 2](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0027-custom-elements-interop.md#customized-built-in-elements)
