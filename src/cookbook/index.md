# Introdução

## O livro de receitas x o guia

Como o livro de receitas é diferente do guia? Por que é necessário?

- **Maior Foco**: Neste guia, estamos essencialmente contando uma história. Cada seção se desenvolve e assume o conhecimento de cada seção anterior. No livro de receitas, cada receita pode e deve se manter por si própria. Isto significa que receitas podem focar em um aspecto específico do Vue, ao invés de precisar fornecer uma visão geral.

- **Maior Profundidade**: Para evitar tornar este guia muito longo, decidimos incluir apenas os exemplos mais simples possíveis para ajudar você a entender cada funcionalidade. E aí, seguimos. No livro de receitas podemos incluir exemplos mais complexos, combinando funcionalidades de maneiras interessantes. Cada receita pode também ser tão longa e detalhada quanto necessário, a fim de explorar totalmente o seu nicho.

- **Ensinando JavaScript**: Neste guia, assumimos pelo menos familiaridade intermediária com JavaScript ES5. Por exemplo, não explicaremos como o `Array.prototype.filter` funciona em uma propriedade computada que filtra uma lista. No livro de receitas, entretanto, funcionalidades essenciais do JavaScript (incluindo ES6/2015+) podem ser exploradas e explicadas no contexto de como elas nos ajudam a construir melhores aplicações Vue.

- **Explorando o Ecossistema**: Para funcionalidades avançadas, assumimos algum conhecimento do ecossistema. Por exemplo, se você quer utilizar Componentes _Single-File_ no Webpack, não explicaremos como configurar as partes que não do Vue nas definições do Webpack. No livro de receitas, temos espaço para explorar o ecossistema de bibliotecas mais a fundo - ao menos até o ponto onde seja universalmente útil para desenvolvedores Vue.

::: tip
Com todas essas diferenças, por favor, observe que o livro de receitas ainda _não_ é um manual passo-a-passo. É esperado que você tenha conhecimento básico de habilidades como HTML, CSS, JavaScript, npm/yarn pela maior parte do seu conteúdo.
:::

## Contribuições para o Livro de receitas

### O que estamos buscando

O Livro de receitas dá exemplos aos desenvolvedores que cobrem desde casos comuns até casos de uso interessantes, e que também explicam detalhes mais complexos progressivamente. Nosso objetivo é ir além de um simples exemplo introdutório, e demonstrar conceitos que podem ser aplicáveis de maneira ampla, assim como ressalvas para essas abordagens.

Se você está interessado em contribuir, por favor, inicie uma colaboração preenchendo uma _issue_ com a _tag_ **cookbook idea** com o seu conceito para que possamos ajudá-lo a realizar o _pull request_ com sucesso.  Depois que sua ideia for aprovada, por favor, siga o modelo abaixo conforme possível. Algumas seções são exigidas, e algumas são opcionais. Seguir a ordem numérica é recomendado fortemente, mas não obrigatório.

Receitas geralmente devem:

- Resolver um problema comum e específico
- Começar com o exemplo mais simples possível
- Introduzir uma complexidade por vez
- Ligar outros documentos, ao invés de reexplicar conceitos
- Descrever o problema, ao invés de assumir familiaridade
- Explicar o processo, ao invés apenas o resultado final
- Explicar os prós e contras de sua estratégia, incluindo quando ela é e não é adequada
- Mencionar soluções alternativas, se relevante, mas deixar explorações a fundo para outra receita

Pedimos que você siga o modelo abaixo. Queremos entender, entretanto, que pode haver situações em que você deve necessariamente desviar pela claridade ou pelo fluxo. De Qualquer maneira, todas as receitas devem em algum ponto discutir a nuance da sua decisão feita usando esse padrão, preferencialmente no formulário da seção de padrões alternativos.

### Exemplo base <Badge text="required" type="error" />

1.  Articule o problema em uma frase ou duas.
2.  Explica a solução mais simples possível em uma frase ou duas.
3.  Mostre um pequeno exemplo de código.
4.  Explique o que ele realiza em uma frase.

### Detalhes sobre o Valor <Badge text="required" type="error" />

1.  Aborde perguntas comuns que podem surgir enquanto se olha o exemplo. (Blockquotes são ótimos para isso)
2.  Dê exemplos de erros comuns e como eles podem ser evitados.
3.  Mostre exemplos de código muito simples sobre bons e maus padrões.
4.  Debata sobre o porquê este possa ser um padrão contundente. Links para referência não são exigidos, mas são encorajados.

### Exemplo de caso real <Badge text="required" type="error" />

Demonstre o código que empoderaria um caso de uso comum ou interessante, seja por:

1.  Num guia passo-a-passo por alguns exemplos curtos, ou
2.  Embutindo um exemplo no codepen/jsfiddle

Se você escolher fazer o último, você ainda deve falar sobre ele é e o que faz.

### Contexto Adicional <Badge text="optional" />

É extremamente útil escrever um pouco sobre tal padrão, ou onde mais se aplicaria, porque funciona bem, e passar um pouco de código conforme você explica, ou forneça materiais de leitura para as pessoas.

### Quando evitar este padrão <Badge text="optional" />

Esta seção não é obrigatória, mas altamente recomendada. Não fará sentido escrevê-la por algo tão simples como acionar classes baseadas em mudança de estado, mas para padrões mais avançados como mixins é vital. A resposta para a maioria das questões de desenvolvimento é ["Depende!"] (https://codepen.io/rachsmith/pen/YweZbG), esta seção aborda isso. Aqui, faremos uma análise sincera sobre quando o padrão é útil e sobre quando ele deve ser evitado, ou quando outra coisa fizer mais sentido.

### Padrões alternativos <Badge text="required with avoidance section" type="warning" />

Esta seção é exigida quando você forneceu a seção acima sobre prevenção. É importante explorar outros métodos para que as pessoas que ouviram que algo é um antipadrão em certas situações não fiquem só imaginando. Ao fazer isso, considere que a web é uma grande tenda e que muitas pessoas possuem estruturas diferentes de código-base e que estão resolvendo problemas diferentes. O aplicativo é grande ou pequeno? Eles integrarão o Vue em um projeto existente, ou irão construir do zero? Seus usuários estão tentando alcançar uma finalidade ou muitas? Há muitos dados assíncronos? Todas estas considerações irão impactar as implementações alternativas. Um bom livro de receitas dá aos desenvolvedores esse conceito.

## Obrigado

Tempo é necessário para contribuir com a documentação, e se você passou tempo elaborando um PR para esta seção de documentos, você fez isso com a nossa gratidão.
