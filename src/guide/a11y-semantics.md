# Semântica

## Formulários

Quando se cria um formulário, você pode usar os seguintes elementos: `<form>`, `<label>`, `<input>`, `<textarea>`, e `<button>`

Labels são tipicamente posicionadas no topo ou à esquerda dos campos do formulário:

```html
<form action="/dataCollectionLocation" method="post" autocomplete="on">
  <div v-for="item in formItems" :key="item.id" class="form-item">
    <label :for="item.id">{{ item.label }}: </label>
    <input
      :type="item.type"
      :id="item.id"
      :name="item.id"
      v-model="item.value"
    />
  </div>
  <button type="submit">Enviar</button>
</form>
```

<common-codepen-snippet title="Simples formulário" slug="oNLxKxj" :height="368" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

Observe como você pode incluir o atributo `autocomplete='on'` no elemento formulário e ele irá aplicar a todos os _inputs_ do seu formulário. Você também pode definir diferentes [valores para o atributo _autocomplete_](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete) para cada _input_.

### Labels

Forneça labels para descrever o propósito de todos os controles do formulário; ligando `for` a `id`:

```html
<label for="name">Nome</label>
<input type="text" name="name" id="name" v-model="name" />
```

<common-codepen-snippet title="Form com Label" slug="rNLeXeR" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

Se você inspecionar este elemento em suas ferramentas de desenvolvedor do Chrome e abrir a guia Acessibilidade dentro da guia Elementos, verá como o _input_ obtém seu nome a partir da _label_:

![Ferramentas de Desenvolvedor do Chrome mostrando um nome acessível para o input usando a label](/images/AccessibleLabelChromeDevTools.png)

::: warning Aviso:
Embora você possa ter visto _labels_ envolvendo os campos _input_ como este:

```html
<label>
  Nome:
  <input type="text" name="name" id="name" v-model="name" />
</label>
```

Definir explicitamente as _labels_ com um _id_ correspondente é melhor suportado por tecnologias assistivas.
:::

#### aria-label

Você também pode dar ao _input_ um nome acessível com [`aria-label`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-label_attribute).

```html
<label for="name">Nome</label>
<input
  type="text"
  name="name"
  id="name"
  v-model="name"
  :aria-label="nameLabel"
/>
```

<common-codepen-snippet title="Form com ARIA label" slug="Vwjaoja" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

Sinta-se à vontade para inspecionar este elemento nas ferramentas de desenvolvedor do Chrome para ver como o nome acessível mudou:

![Ferramentas de Desenvolvedor do Chrome mostrando um nome acessível para o input usando aria-label](/images/AccessibleARIAlabelDevTools.png)

#### aria-labelledby

Usar [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute) é semelhante a `aria-label`, mas será usado se o texto da _label_ estiver visível na tela. Ele é emparelhado com outros elementos por seu `id` e você pode vincular vários `id`s:

```html
<form
  class="demo"
  action="/dataCollectionLocation"
  method="post"
  autocomplete="on"
>
  <h1 id="billing">Fatura</h1>
  <div class="form-item">
    <label for="name">Nome:</label>
    <input
      type="text"
      name="name"
      id="name"
      v-model="name"
      aria-labelledby="billing name"
    />
  </div>
  <button type="submit">Enviar</button>
</form>
```

<common-codepen-snippet title="Form com ARIA labelledby" slug="ZEOWgOV" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

![Ferramentas de Desenvolvedor do Chrome mostrando um nome acessível para o input usando aria-labelledby](/images/AccessibleARIAlabelledbyDevTools.png)

#### aria-describedby

[aria-describedby](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute) é usado da mesma maneira que `aria-labelledby`, exceto que fornece uma descrição com informações adicionais que o usuário pode precisar. Pode ser usado para descrever os critérios para qualquer _input_:

```html
<form
  class="demo"
  action="/dataCollectionLocation"
  method="post"
  autocomplete="on"
>
  <h1 id="billing">Fatura</h1>
  <div class="form-item">
    <label for="name">Nome completo:</label>
    <input
      type="text"
      name="name"
      id="name"
      v-model="name"
      aria-labelledby="billing name"
      aria-describedby="nameDescription"
    />
    <p id="nameDescription">Por favor, forneça seu nome e sobrenome.</p>
  </div>
  <button type="submit">Submit</button>
</form>
```

<common-codepen-snippet title="Form com ARIA describedby" slug="PozNMGj" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

Você pode ver a descrição inspecionando nas ferramentas de desenvolvedor do Chrome:

![Ferramentas de Desenvolvedor do Chrome mostrando um nome acessível para o input usando aria-labelledby e descrição com aria-describedby](/images/AccessibleARIAdescribedby.png)

### Placeholder

Evite usar _placeholders_, pois eles podem confundir muitos usuários.

Um dos problemas com os _placeholders_ é que eles não atendem aos [critérios de contraste de cor](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html) por padrão; corrigir o contraste da cor faz com que o _placeholder_ se pareça com dados pré-preenchidos nos _inputs_. Olhando para o exemplo a seguir, você pode ver que o _placeholder_ para Sobrenome que atende aos critérios de contraste de cor, parece um dado pré-preenchido:

<common-codepen-snippet title="Form com placeholder" slug="MWeyNjZ" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

É melhor fornecer todas as informações que o usuário precisa para preencher formulários fora de qualquer _input_.

### Instruções

Ao adicionar instruções para seus _inputs_, certifique-se de vinculá-los corretamente.
Você pode fornecer instruções adicionais e vincular vários ids dentro de um [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-labelledby_attribute). Isso permite um design mais flexível.

```html
<fieldset>
  <legend>Usando aria-labelledby</legend>
  <label id="date-label" for="date">Data atual:</label>
  <input
    type="date"
    name="date"
    id="date"
    aria-labelledby="date-label date-instructions"
  />
  <p id="date-instructions">MM/DD/YYYY</p>
</fieldset>
```

Alternativamente, você pode anexar intruções ao _input_ com [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute):

```html
<fieldset>
  <legend>Usando aria-describedby</legend>
  <label id="dob" for="dob">Data de nascimento:</label>
  <input type="date" name="dob" id="dob" aria-describedby="dob-instructions" />
  <p id="dob-instructions">MM/DD/YYYY</p>
</fieldset>
```

<common-codepen-snippet title="Form com instruções" slug="qBNZeqd" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

### Escondendo Conteúdo

Normalmente não é recomendado ocultar visualmente as _labels_, mesmo se o _input_ tiver um nome acessível. No entanto, se a funcionalidade do _input_ pode ser compreendida com o conteúdo ao redor, podemos ocultar a _label_ visual.

Veja este campo de pesquisa:

```html
<form role="search">
  <label for="search" class="hidden-visually">Pesquisar: </label>
  <input type="text" name="search" id="search" v-model="search" />
  <button type="submit">Pesquisar</button>
</form>
```

Podemos fazer isso porque o botão de pesquisa ajudará os usuários visuais a identificar a finalidade do _input_.

Podemos usar CSS para ocultar elementos visualmente, mas mantê-los disponíveis para tecnologias assistivas:

```css
.hidden-visually {
  position: absolute;
  overflow: hidden;
  white-space: nowrap;
  margin: 0;
  padding: 0;
  height: 1px;
  width: 1px;
  clip: rect(0 0 0 0);
  clip-path: inset(100%);
}
```

<common-codepen-snippet title="Form de pesquisa" slug="gOMrVLR" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

#### aria-hidden="true"

Adicionar `aria-hidden="true"` ocultará o elemento de tecnologias assistivas, mas o deixará visualmente disponível para outros usuários. Não use em elementos focáveis, puramente em conteúdo decorativo, duplicados ou fora da tela.

```html
<p>Este não está escondido de leitores de tela.</p>
<p aria-hidden="true">Este está escondido de leitores de tela.</p>
```

### Botões

Ao usar botões dentro de um formulário, você deve definir o tipo para evitar o envio do formulário.
Você também pode usar _inputs_ para criar botões:

```html
<form action="/dataCollectionLocation" method="post" autocomplete="on">
  <!-- Botões -->
  <button type="button">Cancelar</button>
  <button type="submit">Enviar</button>

  <!-- Botões usando input -->
  <input type="button" value="Cancelar" />
  <input type="submit" value="Enviar" />
</form>
```

<common-codepen-snippet title="Form com botões" slug="pobyMNx" :height="467" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />

#### Imagens Funcionais

Você pode usar esta técnica para criar imagens funcionais.

- Campo de _input_

  - Estas imagens irão agir como um botão de enviar em formulários

  ```html
  <form role="search">
    <label for="search" class="hidden-visually">Pesquisar: </label>
    <input type="text" name="search" id="search" v-model="search" />
    <input
      type="image"
      class="btnImg"
      src="https://img.icons8.com/search"
      alt="Pesquisar"
    />
  </form>
  ```

- Ícones

```html
<form role="search">
  <label for="searchIcon" class="hidden-visually">Pesquisar: </label>
  <input type="text" name="searchIcon" id="searchIcon" v-model="searchIcon" />
  <button type="submit">
    <i class="fas fa-search" aria-hidden="true"></i>
    <span class="hidden-visually">Pesquisar</span>
  </button>
</form>
```

<common-codepen-snippet title="Imagens funcionais" slug="qBNZeRd" :height="265" tab="js,result" theme="light" :preview="false" :editable="false" user="emanuelgsouza" name="Emanuel Gonçalves" />
