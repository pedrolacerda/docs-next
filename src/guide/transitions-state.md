# Transições de Estado

O sistema de transição do Vue oferece muitas maneiras simples de animar entradas, saídas e listas, mas que tal animar seus próprios dados? Por exemplo:

- números e cálculos
- cores exibidas
- as posições dos nós (*nodes*) SVG
- os tamanhos e outras propriedades dos elementos

Todos eles já estão armazenados como números brutos ou podem ser convertidos em números. Depois de fazer isso, podemos animar essas mudanças de estado usando bibliotecas de terceiros para interpolar o estado, em combinação com a reatividade e os sistemas de componentes do Vue.

## Animando Estado com Observadores

Os observadores nos permitem animar mudanças de qualquer propriedade numérica em outra propriedade. Isso pode parecer complicado no abstrato, então vamos mergulhar em um exemplo usando [GreenSock](https://greensock.com/):

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.2.4/gsap.min.js"></script>

<div id="animated-number-demo">
  <input v-model.number="number" type="number" step="20" />
  <p>{{ animatedNumber }}</p>
</div>
```

```js
const Demo = {
  data() {
    return {
      number: 0,
      tweenedNumber: 0
    }
  },
  computed: {
    animatedNumber() {
      return this.tweenedNumber.toFixed(0)
    }
  },
  watch: {
    number(newValue) {
      gsap.to(this.$data, { duration: 0.5, tweenedNumber: newValue })
    }
  }
}

Vue.createApp(Demo).mount('#animated-number-demo')
```

<common-codepen-snippet title="Transição de Estado 1" slug="22903bc3b53eb5b7817378ecb985ce96" tab="js,result" :editable="false" :preview="false" />

Quando você atualiza o número, a mudança é animada abaixo do `input`.

## Transições de Estado Dinâmicas

Tal como acontece com os componentes de transição do Vue, os dados nos quais as transições de estado se baseiam podem ser atualizados em tempo real, o que é especialmente útil para prototipagem! Mesmo usando um polígono SVG simples, você pode obter muitos efeitos que seriam difíceis de conceber até que você brincasse um pouco com as variáveis.

<common-codepen-snippet title="Atualizando SVG" slug="a8e00648d4df6baa1b19fb6c31c8d17e" :height="500" tab="js,result" :editable="false" />

## Organizando Transições em Componentes

Gerenciar muitas transições de estado pode aumentar rapidamente a complexidade de uma instância de componente. Felizmente, muitas animações podem ser extraídas em componentes filhos dedicados. Vamos fazer isso com o número inteiro animado de nosso exemplo anterior:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.2.4/gsap.min.js"></script>

<div id="app">
  <input v-model.number="firstNumber" type="number" step="20" /> +
  <input v-model.number="secondNumber" type="number" step="20" /> = {{ result }}
  <p>
    <animated-integer :value="firstNumber"></animated-integer> +
    <animated-integer :value="secondNumber"></animated-integer> =
    <animated-integer :value="result"></animated-integer>
  </p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      firstNumber: 20,
      secondNumber: 40
    }
  },
  computed: {
    result() {
      return this.firstNumber + this.secondNumber
    }
  }
})

app.component('animated-integer', {
  template: '<span>{{ fullValue }}</span>',
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      tweeningValue: 0
    }
  },
  computed: {
    fullValue() {
      return Math.floor(this.tweeningValue)
    }
  },
  methods: {
    tween(newValue, oldValue) {
      gsap.to(this.$data, {
        duration: 0.5,
        tweeningValue: newValue,
        ease: 'sine'
      })
    }
  },
  watch: {
    value(newValue, oldValue) {
      this.tween(newValue, oldValue)
    }
  },
  mounted() {
    this.tween(this.value, 0)
  }
})

app.mount('#app')
```

<common-codepen-snippet title="Componentes de Transição de Estado" slug="e9ef8ac7e32e0d0337e03d20949b4d17" tab="js,result" :editable="false" />

Agora podemos compor vários estados com esses componentes filhos. É empolgante - podemos usar qualquer combinação de estratégias de transição que foram abordadas nesta página, junto com as oferecidas pelo [sistema de transição integrado](transitions-enterleave.html) do Vue. Juntos, existem poucos limites para o que pode ser realizado.

Você pode ver como podemos usar isso para visualização de dados, efeitos de física, animações e interações de personagens, o céu é o limite!

## Trazendo Designs à Vida

Animar, por uma definição, significa trazer à vida. Infelizmente, quando os designers criam ícones, logotipos e mascotes, eles geralmente são entregues como imagens ou SVGs estáticos. Portanto, embora o polvo-gato do GitHub, o pássaro do Twitter e muitos outros logotipos se assemelhem à criaturas vivas, eles não parecem realmente vivos.

O Vue pode ajudar. Como os SVGs são apenas dados, precisamos apenas de exemplos de como essas criaturas se parecem quando empolgadas, pensando ou alarmadas. O Vue pode ajudar na transição entre esses estados, tornando suas páginas de boas-vindas, indicadores de carregamento e notificações mais emocionalmente atraentes.

Sarah Drasner demonstra isso abaixo, usando uma combinação de mudanças de estado cronometradas e orientadas à interatividade:

<common-codepen-snippet title="Wall-E controlado via Vue" slug="YZBGNp" :height="400" :team="false" user="sdras" name="Sarah Drasner" :editable="false" :preview="false" version="2" theme="light" />
