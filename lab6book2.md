---
tags: [pmr3304]
title: 'Ruby on Rails e React: Convertendo a Livraria Virtual em uma SPA (Single Page Application)'
created: '2020-12-03T15:45:56.175Z'
modified: '2021-01-14T22:12:29.982Z'
---

Ruby on Rails e React: Convertendo a Livraria Virtual em uma SPA (Single Page Application)
============================================================================================

## Introdução

Como mencionado na apostila de Javascript/React, os frameworks de Javascript foram criados principalmente para o desenvolvimento de Aplicações de Página Única (_Single Page Applications_ ou SPA). Nesta apostila, demonstraremos o desenvolvimento de uma SPA a partir da adaptação da aplicação de Livraria Virtual desenvolvida anteriormente. Continuaremos com basicamente a mesma aplicação Ruby on Rails no Backend, enquanto que, para o Front-end, será utilizado o framework React.

Em SPAs, a renderização é feita quase complemetamente no browser (cliente), através da manipulação do DOM com código Javascript (ver apostila do Laboratório 3 para relembrar o conceito de renderização no cliente). A atualização do conteúdo da página é feita através de requisições HTTP assíncronas, cujo conteúdo é geralmente formatado em JSON. Assim, o Backend-end deve gerar apenas uma página HTML, que deve fazer o link com os arquivos de Javascript que compõem a aplicação do Front-end.

A principal vantagem de SPAs é a drástica redução do número de recarregamento (_reloading_) completo de páginas. Como as requisições são assíncronas, não é necessário ficar esperando a resposta do servidor a cada requisição HTTP. Além disso, uma vez obtida a resposta, ou seja, os dados requisitados, é possível alterar apenas a parte do DOM (estrutura da página) impactada. Como principal desvantagem das SPAs, é o seu pior desempenho em Otimização para Mecanismos de Buscas (_ Search Engine Optimization_ ou SEO), uma vez que o conteúdo dos arquivos HTMLs que geralmente são analisados nas buscas. Por fim, para páginas estáticas do site, o tempo de resposta é bem mais elevado quando comparado com páginas HTML puras.

## Capítulo 1 - Instalação

Caso você já tenha pronta e funcionando a aplicação da Livraria Virtual em RoR, não é necessário instalar mais nenhum software. Caso contrário, é necessário instalar o Ruby on Rails (veja o procedimento em apostilas anteriores). 

O código inicial da aplicação Livraria Virtual pode ser obtido no repositório no seguinte link:
<https://gitlab.uspdigital.usp.br/andre.kubagawa/livra-rails>.

É recomendada a cópia do repositório para sua própria conta do Github ou Gitlab da USP. Assim, você pode gerar os seus própios commits e armazenar no repositório remoto. Para tal, primeiramente crie o seu repositório no GitHub ou Gitlab; por exemplo, em <https://github.com/usuario/livra-rails-react>. Depois, vamos fazer uma clonagem "crua" (sem os arquivos do diretório de trabalho) do repositório da livraria com o comando:

```bash
git clone --bare https://gitlab.uspdigital.usp.br/andre.kubagawa/livra-rails.git
```

Agora vamos fazer o push com "espelhamento" para o seu repositório remoto:

```bash
cd livra-rails.git
git push --mirror https://github.com/usuario/livra-rails-react
```

Pronto, temos o nosso repositório configurado. Podemos, então, apagar a pasta `livra-rails.git` e clonar o novo repositório em outra pasta com:

```bash
cd ..
git clone https://github.com/usuario/livra-rails-react
cd livra-rails-react
```

Com o código da nossa aplicação inicial pronta, devemos instalar as bibliotecas e inicializar o banco de dados. Para isso, podemos executar os comnandos (rails e bundle devem já estar instalados):

```bash
bundle install
yarn install
rails db:migrate
rails db:seed
```

Por fim, para testar se está tudo correto, vamos iniciar a aplicação RoR com:

```bash
rails server
```

Confira em <http://localhost:3000/> se a livraria virtual está operando corretamente. Caso esteja funcionado, podemos prosseguir finalmente com o desenvolvimento da nossa SPA.

## Capítulo 2 - Configurando o React com o Webpacker

Relembrando, uma aplicação SPA utiliza apenas uma página HTML praticamente vazia e grosso o código do Frontend é declarado nos arquivos JS (Javascript). Vamos aplicar essa estrutura e criar a aplicação mais básica possível, ou seja, vamos primeiramente montar uma SPA "_Hello World_". 

Para tal, a primeira ação é instalar as bibliotecas React e integrar com o Webpacker. Lembre que o Webpacker é responsável por gerenciar todos os módulos em JS, inclusive os componentes React. Como saída, o Webpacker gera arquivos JS otimizados para serem executados pelo browser; o Webpacker também garante que a página HTML referencie os arquivos JS corretos.

Sendo assim, instale e configure o React no Webpacker com o comando:

```bash
rails webpacker:install:react
```

Agora, vamos editar a página inicial da nossa aplicação, removendo tudo e só retornando um link para o arquivo JS de entrada da nossa SPA React. Para tal, edite o arquivo `pp/views/store/index.html.erb`

```html
<!-- app/views/store/index.html.erb -->
<noscript>You need to enable JavaScript to run this app.</noscript>
<div id="root"></div>
<%= javascript_pack_tag 'index' %>
```

Note que o arquivo `index.js` será linkado no HTML da nossa página solitária. Também precisamos fazer a limpa no leiaute base - edite o arquivo `app/views/layouts/application.html.erb`

```html
<!-- app/views/layouts/application.html.erb -->
<!DOCTYPE html>
<html>

<head>
  <title>Livra</title>
  <%= csrf_meta_tags %>
  <%= csp_meta_tag %>

  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link href="https://fonts.googleapis.com/css?family=Inter&display=swap" rel="stylesheet">
</head>

<body>
  <%= yield %>
</body>
</html>
```

Finalmente, podemos criar nossa aplicação React básica. Lembrando que o ponto de entrada da aplicação é o arquivo `index.js`. Por padrão, o helper `javascript_pack_tag` chamado em `app/views/store/index.html.erb` busca o arquivo JS na pasta `app/javascript/packs`. Por isso, devemos criar o arquivo `app/javascript/packs/index.js` com o conteúdo

```js
// app/javascript/packs/index.js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById("root")
);
```

Esse arquivo foi inspirado no criado pelo comando `npx create-react-app` utilizado na apostila de Javascript. Assim, nossa aplicação vai estar contida no componente React `App`. Para criá-lo, vamos criar o arquivo `app/javascript/packs/App.js` com o conteúdo

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";

function App() {
  return (
    <div className="w-full flex flex-col items-center justify-center h-32 bg-green-500 text-white">
      <h1 className="text-3xl">Hello world react</h1>
    </div>
  );
}

export default App;
```

Caso não esteja executando, inicie o servidor com `rails server`. Depois, acesse <http://localhost:3000/> e verifique que a nossa SPA _Hello World_ está executando corretamente (como mostrado na figura abaixo). 

![Hello World SPA with RoR backend](images/hello_world_spa.png)

## Capítulo 3 - Criando o leiaute base utilizando componentes

## Capítulo 4 - Criando componentes funcionais com estado utilizando Hooks

## Capítulo 5 - Utilizando a API Fetch ou Axios para fazer requisições assíncronas

## Capítulo 6 - Configurando Rotas com o React Router

## Capítulo 7 - Compartilhando o estado entre componentes com o Context

## Capítulo 8 - Adicionando autenticação com JWT

