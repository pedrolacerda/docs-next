# Testando

## Introdução

Quando falamos na construção de aplicações confiáveis, testes podem ter um papel crítico na capacidade de uma pessoa ou time construir novas funcionalidades, refatorar código, corrigir bugs, etc. Embora existam muitas linhas de pensamento sobre testes, existem três categorias frequentemente discutidas no contexto de aplicações web:

- Testes Unitários
- Testes de Componentes
- Testes Ponta-a-Ponta (E2E)

O objetivo desta seção é servir como um guia para te ajudar a navegar pelo ecossistema de testes e te auxiliar na escolha da ferramenta certa para testar sua aplicação Vue ou biblioteca de componentes.

## Testes Unitários

### Introdução

Testes unitários permitem que você teste unidades do código de forma isolada. O propósito dos testes unitários é fornecer para os desenvolvedores a confiança em seu código. Escrevendo testes minuciosos e significativos você obtém a confiança de que, à medida que novos recursos são criados ou seu código é refatorado, seu aplicativo permanecerá funcional e estável.

Testes unitários em um aplicativo Vue não difere significativamente dos testes de outros tipos de aplicativos.

### Escolhendo Seu Framework

Uma vez que conceitos de testes unitários são geralmente agnósticos de frameworks, aqui estão alguns pontos para se ter em mente quando estiver avaliando qual ferramenta de testes unitários será melhor para sua aplicação.

#### Relatórios de erros de primeira classe

Quando os testes falham, é muito importante que seu framework de testes unitários forneça erros úteis. Este é o trabalho da biblioteca de asserções. Uma asserção com mensagens de erro de alta qualidade ajuda a diminuir o tempo necessário para a depuração do problema. Além de simplesmente te mostrar quais testes estão falhando, bibliotecas de asserções fornecem um contexto do porquê os testes estão falhando, como o que é esperado vs. o que foi recebido.

Alguns frameworks de testes unitários, como o Jest, já possuem essa biblioteca de asserções. Outras, como o Mocha, exigem que bibliotecas de asserções sejam instaladas separadamente (normalmente se utiliza o Chai).

#### Comunidade ativa e times

Uma vez que a maior parte dos frameworks de testes unitários são de código aberto, ter uma comunidade ativa pode ser crítico para times que vão manter seus testes por um longo período de tempo e precisam que o projeto seja ativamente mantido. Além do mais, ter uma comunidade ativa tem o benefício de fornecer mais suporte sempre que você tiver problemas.

### Frameworks

Embora existam muitas ferramentas no ecossistema, aqui estão algumas ferramentas de testes unitários que são comumente utilizadas no ecossistema Vue.js.

#### Jest

Jest é um framework de testes JavaScript focado na simplicidade. Um de seus recursos exclusivos é a capacidade de tirar instantâneos (_snapshots_) de testes para fornecer um meio alternativo de verificação de unidades de sua aplicação.

**Recursos:**

- [Site Oficial do Jest](https://jestjs.io)
- [Plugin Oficial da Vue 2 CLI - Jest](https://cli.vuejs.org/core-plugins/unit-jest.html)

#### Mocha

Mocha é um framework de testes JavaScript focado na flexibilidade. Por causa dessa flexibilidade, o framework permite que você use outras bibliotecas para funcionalidades comuns como _spying_ (ex.: Sinon) e asserções (ex.: Chai). Outro recurso único do Mocha é a capacidade de também poder executar testes no navegador além de no Node.js.

**Recursos:**

- [Site Oficial do Mocha](https://mochajs.org)
- [Plugin Oficial da Vue CLI - Mocha](https://cli.vuejs.org/core-plugins/unit-mocha.html)

## Testes de Componente

### Introdução

Para testar a maioria dos componentes Vue, eles devem ser montados no DOM (virtual ou real) para garantir totalmente que estão funcionando. Esse é outro conceito agnóstico de framework. Como resultado, frameworks de testes de componentes foram criados para dar aos usuários a capacidade de criar testes confiáveis e ao mesmo tempo entregar conveniências específicas do Vue como integrações para Vuex, Vue Router e outros plugins Vue.

### Escolhendo Seu Framework

A seção seguinte fornece um guia de coisas para se ter em mente quando estiver avaliando qual framework de testes de componente é melhor para sua aplicação.

#### Compatibilidade ideal com o ecossistema Vue

Não deveria ser surpresa que um dos primeiros critérios seria que a biblioteca de testes de componentes deve ser o máximo compatível com o ecossistema Vue.js. Embora isso possa parecer compreensível, algumas integrações chave para se ter em mente incluem componentes single-file (SFC), Vuex, Vue Router, e quaisquer outros plugins específicos do Vue que seu aplicativo dependa.

#### Relatórios de erros de primeira classe

Quando os testes falham, é crucial que sua biblioteca de testes de componentes forneça erros úteis que ajudem a diminuir a quantidade de tempo necessário para depurar o problema. Além de mostrar quais testes falharam, o framework deve te dar um contexto do porque os testes falharam, como o que era esperado e o que foi recebido.

### Recomendações

#### Vue Testing Library (@testing-library/vue)

Vue Testing Library é um conjunto de ferramentas focado no teste de componentes sem se preocupar com os detalhes da implementação. Construído com acessibilidade em mente, sua abordagem também facilita a refatoração.

Seu princípio guia é que quanto mais os testes se parecerem com a forma como o software é usado, mais confiança eles fornecem.

**Recursos:**

- [Site Oficial do Vue Testing Library](https://testing-library.com/docs/vue-testing-library/intro)

#### Vue Test Utils

Vue Test Utils é a biblioteca de baixo nível oficial para testes de componentes que foi escrita para fornecer aos usuários acesso a APIs específicas do Vue. Se você é novo em testes de aplicações Vue, nós fortemente recomendamos o uso da Vue Testing Library, qual é uma abstração do Vue Test Utils.

**Recursos:**

- [Documentação Oficial do Vue Test Utils](https://vue-test-utils.vuejs.org)
- [Manual de Testes do Vue](https://lmiller1990.github.io/vue-testing-handbook/v3/#what-is-this-guide) escrito por Lachlan Miller

## Testes Ponta-a-Ponta (E2E)

### Introdução

Embora testes unitários forneçam aos desenvolvedores algum grau de confiança, testes unitários e de componentes são limitados quando se trata de fornecer uma cobertura holística da aplicação quando enviada para a produção. Como resultado, testes ponta-a-ponta (E2E) fornecem uma cobertura ao que é discutivelmente o aspecto mais importante de uma aplicação: o que acontece quando usuários utilizam sua aplicação.

Em outras palavras, testes E2E validam todas as camadas da sua aplicação. Isso não inclui apenas o código frontend, mas todos os serviços e infraestrutura backend associados ao ambiente que seus usuários vão estar. Ao testar como as ações dos usuários impactam sua aplicação, testes E2E são geralmente a chave para a alta confiança de que sua aplicação está funcionando apropriadamente ou não.

### Escolhendo Seu Framework

Embora testes E2E tenham ganhado uma reputação negativa na web como testes não confiáveis ​​(esquisitos) e atrasar o processo de desenvolvimento, ferramentas modernas de testes E2E deram passos a frente na criação de testes confiáveis, interativos e úteis. Quando estiver escolhendo uma ferramenta de testes E2E, a seção seguinte fornecerá um guia de coisas para se ter em mente na escolha do framework de testes para sua aplicação.

#### Testes entre navegadores

Um dos principais benefícios conhecidos dos testes ponta-a-ponta (E2E) é a capacidade de testar sua aplicação em diversos navegadores. Embora possa parecer desejável ter 100% de cobertura em todos os navegadores, é importante notar que os testes em múltiplos navegadores podem ter retornos negativos para uma equipe devido ao tempo e poder computacional adicionais necessários para a execução consistente dos testes. Sendo assim, é importante estar consciente dessa troca quando estiver escolhendo a quantidade de testes entre navegadores que sua aplicação precisa.

::: tip Dica
Uma estratégia recente para captura de erros em browsers específicos é o uso de ferramentas de monitoramento e relatórios de erros em aplicações (ex.: Sentry, LogRocket, etc.) para navegadores que não são comumente utilizados (ex.: < IE11, versões antigas do Safari, etc.).
:::

#### Ciclos de _feedback_ mais rápidos

Um dos principais problemas com os testes ponta-a-ponta (E2E) é que executar a suíte inteira de testes pode levar bastante tempo. Normalmente, isso é feito apenas nos _pipelines_ de integrações e _deploy_ contínuos (CI/CD). Frameworks modernos de testes E2E ajudaram a resolver esse problema com recursos como paralelização, que permite que os _pipelines_ do CI/CD rodem muito mais rápidos do que antes. Além do mais, quando desenvolvido localmente, a capacidade de selecionar e executar um teste para a página em que você está trabalhando e ao mesmo tempo fornecer _hot reloading_ dos testes pode ajudar a melhorar significativamente o fluxo de trabalho e produtividade dos desenvolvedores.

#### Experiência de depuração de primeira classe

Embora desenvolvedores tradicionalmente escaneiam mensagens em telas de terminais para determinar o que deu errado em um teste, frameworks modernos de testes ponta-a-ponta (E2E) permitem que os desenvolvedores tirem proveito de ferramentas que eles já estão familiarizados, como as ferramentas de desenvolvedor dos navegadores.

#### Visibilidade no modo _headless_

Quando os testes ponta-a-ponta estão rodando nas _pipelines_ de integração / _deployment_ contínuas, eles estão rodando em navegadores _headless_ (ou seja, nenhum navegador visível é aberto para o usuário ver). Como resultado, quando erros acontecem, uma funcionalidade crucial de frameworks modernos de testes E2E é a capacidade de fornecer _prints_ e/ou vídeos dos testes da sua aplicação durante vários momentos afim de fornecer _insights_ sobre o porque os erros estão acontecendo. Historicamente, era tedioso manter essas integrações.

### Recomendações

Embora existam muitas ferramentas no ecossistema, aqui estão alguns frameworks de testes ponta-a-ponta (E2E) comumente utilizados no ecossistema Vue.js.

#### Cypress.io

Cypress.io é um framework de testes focado em melhorar a produtividade dos desenvolvedores ao permitir que testem suas aplicações confiavelmente e ao mesmo tempo fornecer uma experiência de desenvolvimento de primeira classe.

**Recursos:**

- [Site Oficial do Cypress](https://www.cypress.io)
- [Plugin Oficial do Cypress para Vue CLI](https://cli.vuejs.org/core-plugins/e2e-cypress.html)
- [Cypress Testing Library](https://github.com/testing-library/cypress-testing-library)

#### Nightwatch.js

Nightwatch.js é um _framework_ de testes ponta-a-ponta que pode ser utilizado para testar aplicações web e sites, assim como testes unitários e de integração no Node.js.

**Recursos:**

- [Site Oficial do Nightwatch](https://nightwatchjs.org)
- [Plugin Oficial do Nightwatch para Vue CLI](https://cli.vuejs.org/core-plugins/e2e-nightwatch.html)

#### Puppeteer

Puppeteer é uma biblioteca Node que fornece uma API de alto nível para controle do navegador e pode trabalhar junto com outras ferramentas de teste (ex.: o Jest) para testar sua aplicação.

**Recursos:**

- [Site oficial do Puppeteer](https://pptr.dev)

#### TestCafe

TestCafe é uma framework de testes ponta-a-ponta baseado no Node.js que foca em fornecer uma fácil configuração para que os desenvolvedores foquem na construção de testes confiáveis e de fácil escrita.

**Recursos:**

- [Site Oficial do TestCafe](https://devexpress.github.io/testcafe/)
