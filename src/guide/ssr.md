# Renderizando no Lado do Servidor

## Guia Completo de SSR

Criamos um guia independente para a criação de aplicativos Vue renderizados a partir do servidor. Este é um guia bem aprofundado para aqueles que já estão familiarizados com o desenvolvimento *client-side* com Vue, desenvolvimento *server-side* com Node.js e utilização de webpack. Confira [aqui](/guide/ssr/introduction.html).

## Nuxt.js

Configurar corretamente todos os aspectos de uma aplicação pronta para produção utilizando renderização pelo servidor pode ser uma tarefa árdua. Felizmente, existe um excelente projeto da comunidade que visa tornar tudo isso mais fácil: [Nuxt.js](https://nuxtjs.org/). Nuxt.js é um *framework* de alto nível criado por cima do ecossistema Vue que fornece uma experiência extremamente simplificada para escrever aplicações universais. Melhor ainda, você pode utilizá-lo como um gerador de páginas estáticas (geradas a partir de Componentes *Single-File*)! Nós recomendamos fortemente experimentá-lo.

## Quasar Framework SSR + PWA

[Quasar Framework](https://quasar.dev) gerará um aplicativo SSR (com opcional entrega de PWA) que aproveita seu sistema de *build* de ponta, configuração sensata e extensibilidade pelo desenvolvedor para facilitar o desenho e construção de sua ideia. Com mais de cem componentes específicos compatíveis com “Material Design 2.0”, você pode decidir quais executar no servidor, quais estarão disponíveis no navegador, e até gerenciar as tags `<meta>` do seu *site*. Quasar é um ambiente de desenvolvimento baseado em Node.js e webpack que incrementa e agiliza o rápido desenvolvimento de aplicativos SPA, PWA, SSR, Electron, Capacitor e Cordova — tudo a partir de uma única base de código.

## Vite SSR

[Vite](https://vitejs.dev/) é uma nova geração de ferramentas de construção de front-end que melhora significativamente a experiência de desenvolvimento de front-end. Ele consiste em duas partes principais:

- Um servidor de desenvolvimento que atende seus arquivos de origem por meio de módulos ES nativos, com recursos integrados e surpreendentemente rápido (Substituição de Módulo a Quente), Hot Module Replacement (HMR).

- Um comando de construção que agrupa seu código com [Rollup](https://rollupjs.org/), pré-configurado para gerar ativos estáticos altamente otimizados para produção.

Vite também fornece [suporte integrado para renderização do lado do servidor](https://vitejs.dev/guide/ssr.html). Você pode encontrar um projeto de exemplo com Vue [aqui](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-vue)
