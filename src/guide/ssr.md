# Renderizando no Lado do Servidor

## Guia Completo de SSR

Criamos um guia independente para a criação de aplicativos Vue renderizados a partir do servidor. Este é um guia bem aprofundado para aqueles que já estão familiarizados com o desenvolvimento *client-side* com Vue, desenvolvimento *server-side* com Node.js e utilização de webpack. Confira [aqui](/guide/ssr/introduction.html).

## Nuxt.js

Configurar corretamente todos os aspectos de uma aplicação pronta para produção utilizando renderização pelo servidor pode ser uma tarefa árdua. Felizmente, existe um excelente projeto da comunidade que visa tornar tudo isso mais fácil: [Nuxt.js](https://nuxtjs.org/). Nuxt.js é um *framework* de alto nível criado por cima do ecossistema Vue que fornece uma experiência extremamente simplificada para escrever aplicações universais. Melhor ainda, você pode utilizá-lo como um gerador de páginas estáticas (geradas a partir de Componentes *Single-File*)! Nós recomendamos fortemente experimentá-lo.

## Quasar Framework SSR + PWA

[Quasar Framework](https://quasar.dev) gerará um aplicativo SSR (com opcional entrega de PWA) que aproveita seu sistema de *build* de ponta, configuração sensata e extensibilidade pelo desenvolvedor para facilitar o desenho e construção de sua ideia. Com mais de cem componentes específicos compatíveis com “Material Design 2.0”, você pode decidir quais executar no servidor, quais estarão disponíveis no navegador, e até gerenciar as tags `<meta>` do seu *site*. Quasar é um ambiente de desenvolvimento baseado em Node.js e webpack que incrementa e agiliza o rápido desenvolvimento de aplicativos SPA, PWA, SSR, Electron, Capacitor e Cordova — tudo a partir de uma única base de código.

## Vite SSR

[Vite](https://vitejs.dev/) is a new breed of frontend build tool that significantly improves the frontend development experience. It consists of two major parts:

- A dev server that serves your source files over native ES modules, with rich built-in features and astonishingly fast Hot Module Replacement (HMR).

- A build command that bundles your code with [Rollup](https://rollupjs.org/), pre-configured to output highly optimized static assets for production.

Vite also provides built-in [support for server-side rendering](https://vitejs.dev/guide/ssr.html). You can find an example project with Vue [here](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-vue)
