---
tags: [pmr3304]
title: "Ruby on Rails e React: Convertendo a Livraria Virtual em uma SPA (Single Page Application)"
created: "2020-12-03T15:45:56.175Z"
modified: "2021-01-14T22:12:29.982Z"
---

# Ruby on Rails e React: Convertendo a Livraria Virtual em uma SPA (Single Page Application)

## Introdução

Como mencionado na apostila de Javascript/React, os frameworks de Javascript foram criados principalmente para o desenvolvimento de Aplicações de Página Única (_Single Page Applications_ ou SPA). Nesta apostila, demonstraremos o desenvolvimento de uma SPA a partir da adaptação da aplicação de Livraria Virtual desenvolvida anteriormente. Continuaremos com basicamente a mesma aplicação Ruby on Rails no Backend, enquanto que, para o Front-end, será utilizado o framework React.

Em SPAs, a renderização é feita quase completamente no browser (cliente), através da manipulação do DOM com código Javascript (ver apostila do Laboratório 3 para relembrar o conceito de renderização no cliente). A atualização do conteúdo da página é feita através de requisições HTTP assíncronas, cujo conteúdo é geralmente formatado em JSON. Assim, o Backend-end deve gerar apenas uma página HTML, que deve fazer o link com os arquivos de Javascript que compõem a aplicação do Front-end.

A principal vantagem de SPAs é a drástica redução do número de recarregamento (_reloading_) completo de páginas. Como as requisições são assíncronas, não é necessário ficar esperando a resposta do servidor a cada requisição HTTP. Além disso, uma vez obtida a resposta, ou seja, os dados requisitados, é possível alterar apenas a parte do DOM (estrutura da página) impactada. Como principal desvantagem das SPAs, é o seu pior desempenho em Otimização para Mecanismos de Buscas (_ Search Engine Optimization_ ou SEO), uma vez que o conteúdo dos arquivos HTMLs que geralmente são analisados nas buscas. Por fim, para páginas estáticas do site, o tempo de resposta é bem mais elevado quando comparado com páginas HTML puras.

## Capítulo 1 - Instalação

Caso você já tenha pronta e funcionando a aplicação da Livraria Virtual em RoR, não é necessário instalar mais nenhum software. Caso contrário, é necessário instalar o Ruby on Rails (veja o procedimento em apostilas anteriores).

O código inicial da aplicação Livraria Virtual pode ser obtido no repositório no seguinte link:
<https://gitlab.uspdigital.usp.br/andre.kubagawa/livra-rails>.

É recomendada a cópia do repositório para sua própria conta do Github ou Gitlab da USP. Assim, você pode gerar os seus próprios commits e armazenar no repositório remoto. Para tal, primeiramente crie o seu repositório no GitHub ou Gitlab; por exemplo, em <https://github.com/usuario/livra-rails-react>. Depois, vamos fazer uma clonagem "crua" (sem os arquivos do diretório de trabalho) do repositório da livraria com o comando:

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

Com o código da nossa aplicação inicial pronta, devemos instalar as bibliotecas e inicializar o banco de dados. Para isso, podemos executar os comandos (rails e bundle devem já estar instalados):

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
    <%= csrf_meta_tags %> <%= csp_meta_tag %>

    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link
      href="https://fonts.googleapis.com/css?family=Inter&display=swap"
      rel="stylesheet"
    />
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

![Hello World SPA com backend RoR](images/hello_world_spa.png)

## Capítulo 3 - Recriando o leiaute base utilizando componentes

No React, o desenvolvimento da aplicação pode ser compartimentalizada em componentes. Isso permite uma maior legibilidade e manutenção do código e também permite a reutilização de componentes. Na linguagem de templates do RoR, isso também era possível, só que com uma flexibilidade bem menor.

Com isso em mente, vamos reescrever o leiaute base em forma de componentes React. Edite o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";

const NavBar = () => <div className="w-full h-32 bg-green-500"></div>; // NavBar temporário
const Footer = () => <div className="w-full h-16 bg-green-500"></div>; // Footer temporário

function App() {
  return (
    <div className="flex flex-col h-screen justify-between overflow-y-scroll">
      <NavBar />
      <main className="w-full max-w-screen-xl mx-auto flex-grow">
        <h1 className="text-3xl p-4">Hello world react</h1>
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

Neste código, o componente App é composto da barra de navegação `<NavBar />`, do conteúdo principal `<main>...</main>` e do rodapé `<Footer />`. A implementação dos dois componentes novos é temporária. Ao abrir a aplicação no browser (em <http://localhost:3000/>), devemos ver a seguinte página:

![Leiaute simples no SPA com React](images/bare_layout_spa.png)

onde as faixas superior e inferior são os componentes Navbar e Footer, respectivamente.

### 3.1 Implementando o componente Footer

Vamos agora substituir as implementações temporárias dos componentes pela versão real. Começaremos pelo Footer, que é mais simples. Para utilizar os ícones do Fontawesome de modo mais conveniente, podemos instalar algumas bibliotecas do React com os comandos:

```bash
yarn add @fortawesome/fontawesome-svg-core
yarn add @fortawesome/free-solid-svg-icons
yarn add @fortawesome/free-brands-svg-icons
yarn add @fortawesome/react-fontawesome
```

Vamos organizar os arquivos dos nossos componentes desenvolvidos da seguinte forma: primeiro criaremos a pasta `app/javascript/packs/components`; depois, criaremos uma pasta própria para cada novo componente (iniciando com letra maiúscula), sendo o `index.js` o ponto de entrada.

Assim, para o Footer, vamos criar o arquivo `app/javascript/packs/components/Footer/index.js` com o conteúdo

```js
// app/javascript/packs/components/Footer/index.js
import React from "react";
import "../../../stylesheets/application.css";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faEnvelope } from "@fortawesome/free-solid-svg-icons";
import {
  faFacebook,
  faTwitter,
  faInstagram,
  faYoutube,
} from "@fortawesome/free-brands-svg-icons";

const socialMediaIcons = [
  faFacebook,
  faTwitter,
  faInstagram,
  faEnvelope,
  faYoutube,
];

const Footer = () => (
  <footer className="w-full bg-green-500 text-white p-4 flex flex-col justify-center items-center">
    <ul>
      {socialMediaIcons.map((icon, id) => (
        <li key={id} className="inline-block px-2 text-lg">
          <FontAwesomeIcon icon={icon} className="hover:text-gray-300" />
        </li>
      ))}
    </ul>
    <p className="text-sm pt-4">Copyright ©Livra</p>
  </footer>
);

export default Footer;
```

Vamos tecer alguns poucos comentários sobre o código mostrado, que deve ser de relativa fácil compreensão. A principal novidade é o uso da biblioteca `react-fontawesome`, que é importada no início do arquivo e utilizada a partir do componente `FontAwesomeIcon`. Este recebe no seu _props_ o nome do ícone, que deve ser importado de uma dos pacotes de ícones grátis ou pro (ver todas as opções em <https://fontawesome.com/how-to-use/on-the-web/using-with/react>).

Uma vez que o componente está pronto, vamos substituí-lo no App. Para tal, edite o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";
import Footer from "./components/Footer";

const NavBar = () => <div className="w-full h-32 bg-green-500"></div>; // NavBar temporário

function App() {
  return (
    <div className="flex flex-col h-screen justify-between overflow-y-scroll">
      <NavBar />
      <main className="w-full max-w-screen-xl mx-auto flex-grow">
        <h1 className="text-3xl p-4">Hello world react</h1>
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

Ao acessar <http://localhost:3000/>), devemos obter o resultado abaixo:

![Leiaute com footer no SPA com React](images/footer_layout_spa.png)

### 3.2 Implementando o componente NavBar

Agora vamos criar o component Navbar, que é um pouco mais complexo e deve ser responsivo. Crie e pasta `app/javascript/packs/components/NavBar` e o arquivo `index.js` dentro dela. O conteúdo do novo arquivo `app/javascript/packs/components/NavBar/index.js` deve ser

```js
// app/javascript/packs/components/NavBar/index.js
import React from "react";
import "../../../stylesheets/application.css";
import logo from "../../../images/logo.svg";
import Menu from "./Menu";
import MenuButton from "./MenuButton";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faShoppingCart } from "@fortawesome/free-solid-svg-icons";

const Logo = () => <img src={logo} className="h-6" alt="A Livraria" />;
const CartIcon = () => (
  <FontAwesomeIcon
    icon={faShoppingCart}
    className="text-white text-xl hover:text-gray-300 hover:bg-transparent md:order-last"
  />
);

class NavBar extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      menuOpen: false,
    };
    this.handleMenuClick = this.handleMenuClick.bind(this);
  }

  handleMenuClick() {
    this.setState((prevState) => ({ menuOpen: !prevState.menuOpen }));
  }

  render() {
    const { menuOpen } = this.state;
    return (
      <header className="w-full bg-green-500 text-white">
        <div className="max-w-screen-xl mx-auto p-4 flex items-center justify-between flex-wrap">
          <MenuButton onClick={this.handleMenuClick} />
          <Logo />
          <CartIcon />
          <Menu open={menuOpen} />
        </div>
      </header>
    );
  }
}

export default NavBar;
```

Desta vez criamos um componente React de classe, pois ele possui um estado, que armazena se o menu "hamburger" está aberto ou fechado. Na versão com renderização no servidor (usando apenas o RoR), era inserido um código Javascript com a biblioteca alpinejs para essa função. Como estamos usando o React, isso é incorporada mais naturalmente com o uso da variável de estado como veremos.

Tanto o componente `<Logo />` quanto o `<CartIcon />` já estão definidos no mesmo arquivo. Só falta implementar o `<Menu />` e o `<MenuButton />`. Já nos adiantamos e passamos a função `this.handleMenuClick` para o _props_ do MenuButton a fim de possibilitar que ele modifique o estado (propriedade `menuOpen`). Além disso, o componente Menu deve receber o `menuOpen` para possibilitar que o menu possa ser escondido ou exibido.

Para implementar o MenuButton, crie o arquivo `app/javascript/packs/components/NavBar/MenuButton.js` com o conteúdo

```js
// app/javascript/packs/components/NavBar/MenuButton.js
import React from "react";
import "../../../stylesheets/application.css";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faBars } from "@fortawesome/free-solid-svg-icons";

const MenuButton = (props) => {
  const { onClick } = props;
  return (
    <button className="md:hidden" onClick={onClick}>
      <FontAwesomeIcon
        icon={faBars}
        className="text-white text-xl hover:text-gray-300"
      />
    </button>
  );
};

export default MenuButton;
```

Observe que, ao clicar no botão de "hambúrguer", a variável `menuOpen` deverá ser modificada através da função `onClick` recebida no _props_. Já para implementar o Menu, crie o arquivo `app/javascript/packs/components/NavBar/Menu.js` com o conteúdo

```js
// app/javascript/packs/components/NavBar/Menu.js
import React from "react";
import "../../../stylesheets/application.css";

const menuItems = ["Blog", "Perguntas", "Notícias", "Contato"];

const Menu = (props) => {
  const { open } = props;
  return (
    <nav className="w-full md:w-auto md:block">
      <ul>
        {menuItems.map((item, id) => (
          <li
            key={id}
            className={"pt-4 md:inline md:px-4 " + (!open && "hidden")}
          >
            {item}
          </li>
        ))}
      </ul>
    </nav>
  );
};

export default Menu;
```

Neste código, veja que a variável `open` determina a visibilidade do componente. Isto é feito adicionando a classe `hidden` do tailwincss nos elementos `li` de forma dinâmica.

Uma vez criado o NavBar, vamos reescrever o componente principal no arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";
import NavBar from "./components/NavBar";
import Footer from "./components/Footer";

function App() {
  return (
    <div className="flex flex-col h-screen justify-between overflow-y-scroll">
      <NavBar />
      <main className="w-full max-w-screen-xl mx-auto flex-grow">
        <h1 className="text-3xl p-4">Hello world react</h1>
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

Se tudo estiver correto, é esperado que a aplicação fique como a da figura abaixo:

![Leiaute completo no SPA com React](images/full_layout_spa.png)

## Capítulo 4 - Utilizando a API Fetch para fazer requisições assíncronas

O próximo passo é montar a página inicial da nossa loja, exibindo os livros cadastrados no banco de dados. Para tal, devemos fazer a requisição HTTP assíncrona da rota `/products.json`, que é configurada automaticamente quando utilizamos o comando `rails scaffold` para gerar o modelo Products no RoR.

Vamos antes personalizar a resposta da rota `/products.json`, que retorna a lista de todos os livros em formato JSON. Desejamos adicionar informações a respeito da foto do livro. Para tal, edite o arquivo `app/views/products/_product.json.jbuilder`

```rb
# app/views/products/_product.json.jbuilder
json.extract! product, :id, :title, :description, :extension, :price, :created_at, :updated_at
json.photo_path product.photo_path # ADICIONE ESTA LINHA
json.has_photo product.has_photo? # ADICIONE ESTA LINHA
json.url product_url(product, format: :json)
```

Para conferir a mudança, podemos acessar a rota <http://localhost:3000/products.json> pelo browser mesmo, que deve exibir algo do tipo:

![Resposta em JSON da lista de produtos](images/product_json_resp.png)

Para exibir nossa vitrine virtual, vamos criar um novo componente chamado Store. Crie o arquivo `app/javascript/packs/components/Store/index.js` com o conteúdo

```js
// app/javascript/packs/components/Store/index.js
import React from "react";
import "../../../stylesheets/application.css";
import Card from "./Card";

class Store extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      books: [],
    };
  }

  componentDidMount() {
    fetch("/products.json")
      .then((response) => response.json())
      .then((result) => {
        this.setState({ books: result });
      });
  }

  render() {
    const { books } = this.state;
    return (
      <section className="flex flex-col sm:flex-row sm:flex-wrap">
        {books.map((book) => (
          <Card book={book} key={book.id} />
        ))}
      </section>
    );
  }
}

export default Store;
```

Nesta listagem, a API fetch foi utilizada para fazer a requisição assíncrona para o backend, na rota que configuramos anteriormente. Para entender este código, é necessário relembrar o conhecimento de funções _lifecycle_ do React, que foi discutido na apostila de React. A aplicação final desta apostila é bem parecida com a exibida na listagem acima.

Cada livro é renderizado segundo o componente Card, que deve ser gerado. Para tal, crie o arquivo `app/javascript/packs/components/Store/Card.js` com o conteúdo

```js
// app/javascript/packs/components/Store/Card.js
import React from "react";
import "../../../stylesheets/application.css";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faShoppingBasket } from "@fortawesome/free-solid-svg-icons";
import noImage from "../../../images/no_image.svg";

const Card = (props) => {
  const { book } = props;
  const formattedPrice = parseFloat(book.price).toLocaleString("en-US", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
  return (
    <div className="w-full sm:self-stretch sm:w-1/2 lg:w-1/3 xl:w-1/4 flex flex-col items-center py-8 px-4">
      <img
        src={
          book.has_photo
            ? require("../../../images/" + book.photo_path.slice(6))
            : noImage
        }
        alt={book.title}
        className="h-64"
      />
      <a
        href="#"
        className="text-base text-center py-4 sm:flex-grow hover:text-gray-500 hover:bg-transparent"
      >
        {book.title}
      </a>
      <p className="text-xl">R${formattedPrice}</p>
      <button className="w-full btn cursor-pointer">
        <FontAwesomeIcon icon={faShoppingBasket} className="text-2xl pr-3" />
        <span className="text-sm">Comprar</span>
      </button>
    </div>
  );
};

export default Card;
```

A única pequena novidade aqui é o uso da função `require`, geralmente adotada para importar pacotes, para importar dinamicamente uma imagem com Webpack.

Por fim, basta incluir o novo componente Store na nossa aplicação principal, editando o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";
import NavBar from "./components/NavBar";
import Footer from "./components/Footer";
import Store from "./components/Store"; // ADICIONE ESTA LINHA

function App() {
  return (
    <div className="flex flex-col h-screen justify-between overflow-y-scroll">
      <NavBar />
      <main className="w-full max-w-screen-xl mx-auto flex-grow">
        <Store /> {/* MODIFIQUE ESTA LINHA */}
      </main>****
      <Footer />
    </div>
  );
}

export default App;
```

Navegue para <http://localhost:3000> para conferir a nova página inicial com os livros:

![Vitrine da nossa livraria SPA](images/store_front_spa.png)

### 4.1 Implementando requisição POST para compra de livros

Na aplicação original, quando o usuário clica no botão comprar de algum produto, é realiza uma requisição do tipo POST na rota `/line_items`. Na nossa SPA, o mesmo pode ser realizado assincronamente com a API fetch adicionando um argumento adicional a chamada. Para entender melhor, vamos implementar esta modificação; edite o arquivo `app/javascript/packs/components/Store/Card.js`

```js
// app/javascript/packs/components/Store/Card.js
import React from "react";
import "../../../stylesheets/application.css";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faShoppingBasket } from "@fortawesome/free-solid-svg-icons";
import noImage from "../../../images/no_image.svg";

const Card = (props) => {
  function postLineItem(id) { /* ADICIONE ESTA FUNÇÃO */ }
    const csrf = document
      .querySelector("meta[name='csrf-token']")
      .getAttribute("content");
    const requestOptions = {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "X-CSRF-Token": csrf,
      },
      body: JSON.stringify({ product_id: id }),
    };
    fetch("/line_items.json", requestOptions)
      .then((response) => response.json())
      .then((data) => console.log(data));
  }

  ...

      <p className="text-xl">R${formattedPrice}</p>
      <button onClick={() => postLineItem(book.id)} className="w-full btn cursor-pointer"> {/* MODIFIQUE ESTA LINHA */}
        <FontAwesomeIcon icon={faShoppingBasket} className="text-2xl pr-3" />
        <span className="text-sm">Comprar</span>
      </button>
    </div>
  );
};

export default Card;
```

O método (POST), assim como campos de cabeçalhos HTTP, todos armazenados na variável `requestOptions`, devem ser passados no segundo argumento da função `fetch`, como mostrado no código. O campo `X-CSRF-Token` é necessário para aplicações RoR, pois esta proteção é inserida por padrão. Na prática, as rotas POSTS são apenas acessíveis para usuários autenticados e, portanto, podemos remover essa proteção e, consequentemente, não necessitaríamos desse cabeçalho.

Note que estamos fazendo o _print_ da resposta do servidor por motivos de depuração apenas. Assim, podemos verificar se está funcionando clicando o botão e checando a saída no console, como mostrado na figura:

![Comprando um produto da nossa livraria SPA](images/buy_store_spa.png)

Lembre que a resposta do POST na rota `/line_items` retorna dados do carrinho. Assim, podemos deduzir que o item foi inserido no carrinho com o _id_ 4 e, checando em <http://localhost:3000/carts/4>, verificamos que o livro realmente está no carrinho:

![Carrinho com o novo produto inserido](images/cart_example.png)

## Capítulo 5 - Configurando Rotas com o React Router

Como a SPA é contida em apenas uma página HTML, que é gerada no servidor e enviada para o browser do cliente, não é possível utilizar as configurações de rota do Backend.

Sendo assim, uma forma de simular a navegação de links é utilizar a renderização condicional (discutida na apostila de React). No entanto, existe uma biblioteca bastante popular para essa função: o React Router <https://reactrouter.com/>, com vários recursos adicionais.

Vamos utilizar esta biblioteca para implementar as rotas, como as páginas adicionais e o link para o carrinho e checkout. A primeira coisa a se fazer é instalar a biblioteca com o yarn:

```bash
yarn add react-router-dom
```

Para utilizar o React Router, devemos seguir um procedimento padrão. Inicialmente, devemos envolver nossa aplicação com no componente `BrowserRouter`. Ou seja, para aplicar no nosso componente App, devemos fazer algo do tipo:

```js
import { BrowserRouter as Router } from "react-router-dom";

function App() {
  return <Router>...</Router>;
}

export default App;
```

Dentro da região definida pela abertura/fechamento da tag Router, podemos incluir o componente `Switch`, que define a região com o conteúdo a ser chaveado. Dentro dele, podemos associar diferentes rotas para renderizar componentes distintos. Vamos modificar nosso código para testar isso, edite o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";
import NavBar from "./components/NavBar";
import Footer from "./components/Footer";
import Store from "./components/Store";
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";

const GenericPage = (props) => (
  <h1 className="text-center p-4 text-lg">{props.title}</h1>
);
const Blog = () => <GenericPage title="Blog Page" />;
const Perguntas = () => <GenericPage title="Perguntas Page" />;
const Noticias = () => <GenericPage title="Noticias Page" />;
const Contato = () => <GenericPage title="Contato Page" />;

function App() {
  return (
    <Router>
      <div className="flex flex-col h-screen justify-between overflow-y-scroll">
        <NavBar />
        <main className="w-full max-w-screen-xl mx-auto flex-grow">
          <Switch>
            <Route path="/" exact component={Store} />
            <Route path="/blog" component={Blog} />
            <Route path="/perguntas" component={Perguntas} />
            <Route path="/noticias" component={Noticias} />
            <Route path="/contato" component={Contato} />
          </Switch>
        </main>
        <Footer />
      </div>
    </Router>
  );
}

export default App;
```

Antes de testar no browser, temos que alterar nosso Backend para responder as novas rotas (com o mesmo arquivo HTML que a raiz). Ou seja, edite o arquivo `config/routes.rb`

```rb
Rails.application.routes.draw do
  resources :orders
  resources :line_items, only: :create
  resources :carts, only: :show
  root to: "store#index"
  resources :products

  get 'blog', to: 'store#index' # ADICIONAR ESTA LINHA
  get 'perguntas', to: 'store#index' # ADICIONAR ESTA LINHA
  get 'noticias', to: 'store#index' # ADICIONAR ESTA LINHA
  get 'contato', to: 'store#index' # ADICIONAR ESTA LINHA
end
```

Agora podemos testar o funcionamento do roteador React! Abra o seu browser em <http://localhost:3000/blog> e confira se o resultado é o seguinte

![Roteamento manual com o React Router](images/router_blog_spa.png)

Teste as demais rotas também: <http://localhost:3000/perguntas>, <http://localhost:3000/noticias> e <http://localhost:3000/contato>.

Falta apenas aprendermos como criar links no nosso código React. Para tal, vamos adicionar os links na barra de navegação. Isto é, edite o arquivo `app/javascript/packs/components/NavBar/Menu.js`

```js
// app/javascript/packs/components/NavBar/Menu.js
import React from "react";
import "../../../stylesheets/application.css";
import { Link } from "react-router-dom";

const menuItems = [
  { title: "Blog", link: "/blog" },
  { title: "Perguntas", link: "/perguntas" },
  { title: "Notícias", link: "/noticias" },
  { title: "Contato", link: "/contato" },
];

const Menu = (props) => {
  const { open } = props;
  return (
    <nav className="w-full md:w-auto md:block">
      <ul>
        {menuItems.map((item) => (
          <Link key={item.link} to={item.link}>
            <li
              className={
                "pt-4 md:inline md:px-4 text-white hover:text-gray-300" +
                (!open && "hidden")
              }
            >
              {item.title}
            </li>
          </Link>
        ))}
      </ul>
    </nav>
  );
};

export default Menu;
```

Como é possível notar, o componente Link recebe a rota na propriedade `to` do _props_. Vamos também adicionar um link para a _Home_ no logo; edite o arquivo `app/javascript/packs/components/NavBar/index.js`

```js
// app/javascript/packs/components/NavBar/index.js
...
import { Link } from "react-router-dom";

const Logo = () => (
  <Link to="/">
    <img src={logo} className="h-6" alt="A Livraria" />
  </Link>
);

...
```

### 5.1 Criando uma rota com parâmetros

Em frameworks de Backend, como o RoR, é comum configurar rotas com parâmetros. Por exemplo, podemos configurar a aplicação para responder no endereço <http://localhost:3000/books/1> com uma página de detalhes do livro com id igual a 1; o endereço <http://localhost:3000/books/2> com uma página do livro #2, e assim por diante.

A biblioteca React Router permite replicar este comportamento direto no Frontend, ou seja, no código Javascript (+ React). Para demonstrar isso, vamos gerar um novo componente para exibir os detalhes do livro: crie o arquivo `app/javascript/packs/components/BookView/index.js` com o conteúdo

```js
// app/javascript/packs/components/BookView/index.js
import React from "react";
import "../../../stylesheets/application.css";
import { withRouter } from "react-router-dom";
import noImage from "../../../images/no_image.svg";

class BookView extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      details: {},
    };
  }

  componentDidMount() {
    const { bookId } = this.props.match.params;
    fetch(`/products/${bookId}.json`)
      .then((response) => response.json())
      .then((result) => {
        this.setState({ details: result });
      });
  }

  render() {
    const {
      id,
      title,
      description,
      price,
      has_photo,
      photo_path,
    } = this.state.details;
    const bookImage = has_photo
      ? require("../../../images/" + photo_path.slice(6))
      : noImage;
    return (
      <section className="flex flex-col sm:flex-row p-4">
        <img
          src={bookImage}
          alt={title}
          className="w-9/12 mx-auto sm:h-64 sm:w-auto sm:flex-shrink-0"
        />
        <div className="flex flex-col p-4">
          <h3 className="text-2xl">{title}</h3>
          <h3 className="text-lg pt-4">Preço: R${price}</h3>
          <p className="pt-4">Descrição: {description} </p>
        </div>
      </section>
    );
  }
}

export default withRouter(BookView);
```

A principal diferença neste componente é a utilização do componente de alta ordem `withRouter` para "envelopar" o BookView, exportado na última linha. Este processo permiti acessar o parâmetro id da URL na variável `this.props.match.params` do _props_.

Para implementar o roteamento, são necessário dois passos: 1) adicionar o `Route` dentro da região do `Switch` e 2) configurar o Backend para responder na rota `/books/:id`.

Para o primeiro passo, basta editar o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React from "react";
import "../stylesheets/application.css";
import NavBar from "./components/NavBar";
import Footer from "./components/Footer";
import Store from "./components/Store";
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";
import BookView from "./components/BookView"; // ADICIONE ESTA LINHA

...

            <Route path="/contato" component={Contato} />
            <Route path="/books/:bookId" component={BookView} /> // ADICIONE ESTA LINHA
          </Switch>
...
```

Note que o nome do parâmetro `bookId` é passado no `props.match.params` do componente `withRouter(BookView)` da última listagem.

O segundo e último passo é alterar as configurações da rota, ou seja, editar o arquivo `config/routes.rb`

```rb
Rails.application.routes.draw do
  resources :orders
  resources :line_items, only: :create
  resources :carts, only: :show
  root to: "store#index"
  resources :products

  get 'blog', to: 'store#index'
  get 'perguntas', to: 'store#index'
  get 'noticias', to: 'store#index'
  get 'contato', to: 'store#index'
  get 'books/:id', to: 'store#index' # ADICIONAR ESTA LINHA
end
```

Agora, ao acessar a URL <http://localhost:3000/books/1>, você deve ver uma tela parecida com a seguinte

![Roteamento manual com parâmetros com o React Router](images/router_book_spa.png)

Para finalizar, vamos adicionar um link para esta página na vitrine da loja. Para tal, edite o arquivo `app/javascript/packs/components/Store/Card.js`

```js
// app/javascript/packs/components/Store/Card.js
import React from "react";
import "../../../stylesheets/application.css";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faShoppingBasket } from "@fortawesome/free-solid-svg-icons";
import noImage from "../../../images/no_image.svg";
import { Link } from "react-router-dom"; // ADICIONAR ESTA LINHA

...

  return (
    <div className="w-full sm:self-stretch sm:w-1/2 lg:w-1/3 xl:w-1/4 flex flex-col items-center py-8 px-4">
      <Link to={`/books/${book.id}`}> {/* ADICIONAR ESTA LINHA */}
        <img
          src={
            book.has_photo
              ? require("../../../images/" + book.photo_path.slice(6))
              : noImage
          }
          alt={book.title}
          className="h-64"
        />
      </Link> {/* ADICIONAR ESTA LINHA */}
      <Link to={`/books/${book.id}`} {/* MODIFICAR ESTA LINHA */}
        className="text-base text-center py-4 sm:flex-grow hover:text-gray-500 hover:bg-transparent"
      >
        {book.title}
      </Link> {/* MODIFICAR ESTA LINHA */}
...
```

Acesse <http://localhost:3000> e teste os novos links para ver se está funcionando.

## Capítulo 6 - Criando componentes funcionais com estado utilizando Hooks (opcional)

Quando introduzimos o conceito de estado no React em apostilas anteriores, sempre destacamos que componentes com estado necessariamente deveriam ser implementados como classes. Isso não é mais verdade, desde a introdução dos _Hooks_, que permitem adicionar estado em componentes funcionais.

Na realidade, os _Hooks_ possuem uso mais amplo, como demonstraremos mais adiante. A sua principal finalidade é possibilitar a reutilização de lógica de estado entre componentes. Por motivos não discutidos nesta apostila, isso era complexo de ser feito apenas com componentes de classe. Para mais detalhes, consulte <https://reactjs.org/docs/hooks-intro.html>.

Sem mais delongas, vamos aplicar os _Hooks_ na nossa aplicação. O primeiro componente a ser analisado é o `Navbar`, que utiliza o estado para determinar se o menu está aberto ou não. Edite o arquivo `app/javascript/packs/components/NavBar/index.js`

```js
// app/javascript/packs/components/NavBar/index.js
import React, { useState } from "react";

...

function NavBar() {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <header className="w-full bg-green-500 text-white">
      <div className="max-w-screen-xl mx-auto p-4 flex items-center justify-between flex-wrap">
        <MenuButton
          onClick={() => {
            setMenuOpen(!menuOpen);
          }}
        />
        <Logo />
        <CartIcon />
        <Menu open={menuOpen} />
      </div>
    </header>
  );
}

export default NavBar;
```

Agora o componente NavBar é funcional, então foi substituído por uma função. O `useState` retorna uma variável com o valor da propriedade do estado e uma função para modificá-la. O argumento é o valor inicial. Sendo assim, foi criado uma variável de estado `menuOpen` com o valor inicial `false`.

### 6.2 Hooks e métodos lifecycle

Você deve se lembrar que outra funcionalidade que só podia ser implementada em componentes de classe eram os métodos _lifecyle_. Vamos rever o código do componente `Store` para relembrar

```js
class Store extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      books: [],
    };
  }

  componentDidMount() {
    fetch("/products.json")
      .then((response) => response.json())
      .then((result) => {
        this.setState({ books: result });
      });
  }

  render() {
    ...
  }
}

export default Store;
```

O método de _lifecycle_ `componentDidMount` é chamado assim que o componente é montado. Como reproduzir este comportamento em um componente funcional com _Hooks_? Basta utilizar o _hook_ `useEffect`. Edite o arquivo `app/javascript/packs/components/Store/index.js`

```js
// app/javascript/packs/components/Store/index.js
import React, { useState, useEffect } from "react";
import "../../../stylesheets/application.css";
import Card from "./Card";

function Store() {
  const [books, setBooks] = useState([]);

  useEffect(() => {
    fetch("/products.json")
      .then((response) => response.json())
      .then((result) => {
        setBooks(result);
      });
  }, []);

  return (
    <section className="flex flex-col sm:flex-row sm:flex-wrap">
      {books.map((book) => (
        <Card book={book} key={book.id} />
      ))}
    </section>
  );
}

export default Store;
```

A função passada como argumento do _hook_ `useEffect` é chamada na montagem do componente e é geralmente executada a cada atualização do estado. Ao fornecer um _array_ vazio como segundo argumento, estamos indicando que só é necessário executar uma vez. Também podemos fornecer funções para serem executadas durante a desmontagem, para mais detalhes confira <https://reactjs.org/docs/hooks-effect.html>.

O componente BookView é muito semelhante ao Store, então fica como exercício a sua conversão para componente funcional.

## Capítulo 7 - Implementando o carrinho (opcional)

Para finalizar, vamos implementar o carrinho de compras. Nenhum novo conceito será introduzido nessa seção, então a sua leitura é opcional (mas recomendada).

### 7.1 Configurando a API do Backend para fornecer dados do carrinho

A primeira atividades que faremos é configurar na API Backend uma rota para responder com as informações do carrinho. No RoR, isto deve ser feito em três passos. Primeiro, precisamos adicionar uma nova ação no controlador: edite o arquivo `app/controllers/carts_controller.rb`:

```rb
# app/controllers/carts_controller.rb
class CartsController < ApplicationController
  def show
    @cart = Cart.find(params[:id])
  end

  def current # ADICIONAR ESTA LINHA
  end # ADICIONAR ESTA LINHA
end
```

Conjuntamente, devemos configurar a rota para esta ação: edite o arquivo `config/routes.rb`:

```rb
# config/routes.rb
Rails.application.routes.draw do
  resources :orders
  resources :line_items, only: :create
  get 'carts/current' # ADD THIS LINE
  resources :carts, only: :show
  ...
```

Por fim, a terceira e última etapa consiste em configurar uma _view_ JSON com as informações do carrinho e de seus itens. Para tal, crie o arquivo `app/views/carts/current.json.jbuilder` com o conteúdo

```rb
# app/views/carts/current.json.jbuilder
json.extract! @cart, :id, :created_at, :updated_at
json.cart_total @cart.total_items
json.url cart_url(@cart, format: :json)
json.line_items @cart.line_items do |line_item|
    json.extract! line_item, :id, :product_id, :cart_id, :quantity, :created_at, :updated_at
    json.product_title line_item.product.title
    json.product_price line_item.product.price
    json.product_photo_path line_item.product.photo_path
    json.product_has_photo line_item.product.has_photo?
end
```

Para testar a nossa nova funcionalidade, adicione alguns itens no carrinho e depois acesse <http://localhost:3000/carts/current.json>. Um exemplo de resposta é mostrada na imagem abaixo

![Resposta em JSON do carrinho atual](images/cart_json_resp.png)

### 7.2 Criando os componentes React e o roteamento no Frontend

O procedimento para implementar o carrinho no Frontend vai seguir os mesmos passos para a criação de componentes com _data fetching_ do capítulo 4 e do roteamento do capítulo 5. Ademais, utilizaremos componentes funcionais com estado, que foram introduzidos no capítulo 6.

Inicialmente, vamos criar o componente React do carrinho. Crie um novo arquivo `app/javascript/packs/components/Cart/index.js` com o conteúdo

```js
// app/javascript/packs/components/Cart/index.js
import React, { useState, useEffect } from "react";
import "../../../stylesheets/application.css";
import noImage from "../../../images/no_image.svg";

function Cart() {
  const [cart, setCart] = useState({});

  useEffect(() => {
    fetch("/carts/current.json")
      .then((response) => response.json())
      .then((result) => {
        setCart(result);
      });
  }, []);

  function formatPrice(price) {
    return parseFloat(price).toLocaleString("pt-BR", {
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
    });
  }

  const line_items = (cart && cart.line_items) || [];

  return (
    <section>
      <table className="table-fixed w-full m-4">
        <thead>
          <tr className="bg-gray-500 text-white text-base">
            <th className="py-2 w-1/2 ">Produto</th>
            <th className="py-2 w-2/12 text-left">Preço</th>
            <th className="py-2 w-2/12 text-left">Quantidade</th>
            <th className="py-2 w-2/12 text-left">Subtotal</th>
          </tr>
        </thead>
        <tbody>
          {line_items.map((item) => (
            <tr key={item.id}>
              <td className="py-4 flex items-center text-sm">
                <img
                  src={
                    item.product_has_photo
                      ? require("../../../images/" +
                          item.product_photo_path.slice(6))
                      : noImage
                  }
                  alt={item.product_title}
                  className="h-12"
                />
                <span className="pl-4">{item.product_title}</span>
              </td>
              <td className="py-4 text-sm">
                {formatPrice(item.product_price)}
              </td>
              <td className="py-4 text-sm">{item.quantity}</td>
              <td className="py-4 text-sm">
                {formatPrice(item.product_price * item.quantity)}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
      <div className="w-full flex justify-end">
        <div className="w-full md:max-w-md">
          <h2 className="text-3xl pb-2 mt-8">Total no carrinho</h2>
          <p className="text-sm py-4">
            <span className="font-bold mr-4 ">Total</span>
            {formatPrice(
              line_items.reduce(
                (a, b) => a + b["product_price"] * b["quantity"],
                0
              )
            )}
          </p>
        </div>
      </div>
    </section>
  );
}

export default Cart;
```

Note que seria desejável subdividir este componente em outros para melhorar a legibilidade e a manutenção do código. Isto fica como exercício.

Em seguida, vamos adicionar este componente no roteamento da aplicação. Edite o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import Cart from "./Cart";
...

          <Switch>
            <Route path="/" exact component={Store} />
            <Route path="/blog" component={Blog} />
            <Route path="/perguntas" component={Perguntas} />
            <Route path="/noticias" component={Noticias} />
            <Route path="/contato" component={Contato} />
            <Route path="/books/:bookId" component={BookView} />
            <Route path="/cart" component={Cart} /> // ADICIONE ESTA LINHA
          </Switch>

...
```

Como adicionamos uma nova rota no Frontend, é necessário criá-la também no Backend. Para tal, edite o arquivo `config/routes.rb`:

```rb
# config/routes.rb
Rails.application.routes.draw do
  ...

  get 'books/:id', to: 'store#index'
  get 'cart', to: 'store#index' # ADICIONE ESTA LINHA
end
```

Por fim, vamos adicionar o link para que o usuário possa ser redirecionado quando clicar no ícone do carrinho. Edite o arquivo `app/javascript/packs/components/NavBar/index.js`

```js
// app/javascript/packs/components/NavBar/index.js
...

const CartIcon = () => (
  <Link className="group md:order-last flex" to="/cart"> { /* ADICIONAR ESTA LINHA */ }
    <FontAwesomeIcon
      icon={faShoppingCart}
      className="text-white text-xl group-hover:text-gray-300 hover:bg-transparent"
    />
  </Link> { /* ADICIONAR ESTA LINHA */ }
);

...
```

A nossa implementação está completa - teste acessando <http://localhost:3000>, adicionando alguns itens no carrinho e clicando no ícone do canto superior esquerdo. Uma tela similar a seguinte deve ser exibida

![Carrinho implementado na SPA React](images/cart_view_spa.png)

### 7.3 Adicionando o indicador no ícone carrinho

Para implementar o total do carrinho, vamos primeiro criar uma variável de estado na nossa aplicação raiz. Esta nova variável `total` vai armazenar o total do carrinho. Edite o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
import React, { useEffect, useState } from "react";

...

function App() {
  const [total, setTotal] = useState(0);

  useEffect(() => {
    fetch("/carts/current.json")
      .then((response) => response.json())
      .then((result) => {
        setTotal(result.cart_total);
      });
  }, []);

  return (
    <Router>
      <div className="flex flex-col h-screen justify-between overflow-y-scroll">
        <NavBar total={total} />

  ...
```

É possível ver pelo código que estamos fazendo uma requisição na montagem do componente para obter o valor atual do total do carrinho. Isso foi feito utilizando _hooks_, que é mais recomendado, mas poderia ser facilmente implementada com componente de classe (como no capítulo 4).

Além disso, a propriedade `total` do estado é passado para o componente `NavBar`. Sendo assim, vamos editar o arquivo `app/javascript/packs/components/NavBar/index.js`

```js
// app/javascript/packs/components/NavBar/index.js
...

const CartIcon = ({ total }) => ( // MODIFICAR ESTA LINHA
  <Link className="group md:order-last flex" to="/cart">
    <FontAwesomeIcon
      icon={faShoppingCart}
      className="text-white text-xl group-hover:text-gray-300 hover:bg-transparent"
    />
    <span
      className={`rounded-full -ml-1 -mt-2 self-start flex items-center justify-center bg-red-500 group-hover:bg-red-700 text-white group-hover:text-gray-300 w-${
        total < 10 ? 4 : 5
      } h-4`}
    > { /* ADICIONAR ESTE ELEMENTO */ }
      <p>{total}</p>
    </span>
  </Link>
);

function NavBar(props) {
  const [menuOpen, setMenuOpen] = useState(false);
  const { total } = props; // ADICIONAR ESTA LINHA

  return (
    <header className="w-full bg-green-500 text-white">
      <div className="max-w-screen-xl mx-auto p-4 flex items-center justify-between flex-wrap">
        <MenuButton
          onClick={() => {
            setMenuOpen(!menuOpen);
          }}
        />
        <Logo />
        <CartIcon total={total} /> { /* MODIFICAR ESTA LINHA */ }
        <Menu open={menuOpen} />
      </div>
    </header>
  );
}

export default NavBar;
```

O NavBar recebe o `total` no _props_ e passa adiante para o componente `CartIcon`, que faz a renderização do ícone do carrinho com o indicador de total.

Ao abrir a nossa aplicação em <http://localhost:3000> agora, deve aparecer o indicador em vermelho no ícone do carrinho, como mostra a imagem

![Ícone do carrinho com indicador implementado na SPA React](images/total_cart_spa.png)

### 7.4 Atualizando o total após uma compra

Toda vez que o usuário realizar uma compra bem sucedida, devemos atualizar o total do carrinho. Isso pode ser feito com uma nova requisição HTTP GET como feita na subseção anterior.

No entanto, para simplificar nossa aplicação, vamos modificar a resposta da requisição POST de compra para devolver o total do carrinho. Para tal, edite o arquivo `app/controllers/line_items_controller.rb`:

```rb
# app/controllers/line_items_controller.rb
class LineItemsController < ApplicationController
    def create
      product = Product.find(params[:product_id])
      @line_item = @cart.add_product(product)

      respond_to do |format|
        if @line_item.save
          format.html { redirect_to @line_item.cart, notice: 'Item inserido com sucesso ao carrinho.' }
          format.json { render json: {"cart_total"=>@line_item.cart.total_items}, status: :created  } # MODIFICAR ESTA LINHA
        else
          format.html { render :new }
          format.json { render json: @line_item.errors, status: :unprocessable_entity }
        end
      end
    end
  end
```

O botão de compra e, consequentemente, a requisição POST estão codificadas no componente `Card`. Sendo assim, precisamos passar a função `setTotal` para o componente `Card` para que este possa modificar o valor total. Como o estado `total` origina no App, vamos editar este componente. Edite o arquivo `app/javascript/packs/App.js`

```js
// app/javascript/packs/App.js
...

          <Switch>
            <Route
              path="/"
              exact
              render={(props) => <Store {...props} setTotal={setTotal} />}
            />
            <Route path="/blog" component={Blog} />

...
```

O `App` enviou o estado para o componente inferior `Store`, que agora deve enviar mais para baixo para o `Card`. Edite o arquivo `app/javascript/packs/components/Store/index.js`

```js
// app/javascript/packs/components/Store/index.js
...

function Store(props) {

  ...

  const { setTotal } = props; // ADICIONAR ESTA LINHA
  return (
    <section className="flex flex-col sm:flex-row sm:flex-wrap">
      {books.map((book) => (
        <Card book={book} key={book.id} setTotal={setTotal} /> { /* MODIFICAR ESTA LINHA */ }
      ))}
    </section>
  );
}

export default Store;
```

Finalmente, podemos receber o estado `total` no componente `Card` e implementar a atualização após compra. Para tal, edite o arquivo `app/javascript/packs/components/Store/Card.js`

```js
// app/javascript/packs/components/Store/Card.js
...

const Card = (props) => {
  const { book, setTotal } = props;

  ...

  function postLineItem(id) {
    ...
    fetch("/line_items.json", requestOptions)
      .then((response) => response.json())
      .then((data) => {
        setTotal(data.cart_total);
      });
  }

...
```

Para testar, basta acessar a loja em <http://localhost:3000> e clicar no botão COMPRAR de algum livro. O indicador deve ser atualizado automaticamente.

## Capítulo 8 - Considerações Finais

Seguem algumas observações para aprimorar a aplicação e expandir o seu conhecimento de desenvolvimento de SPAs com React:

- Se você realizou a atividade do último capítulo, você deve ter percebido que gerenciar o estado é relativamente complexo, mesmo para uma aplicação simples como a nossa. Por este motivo, existem bibliotecas específicas para facilitar este gerenciamento. A mais popular é a biblioteca **Redux** (<https://react-redux.js.org/>). No nosso caso, como não necessitamos de todas as funcionalidades do Redux, poderíamos utilizar apenas o **React Context** (<https://reactjs.org/docs/context.html>).

- Nesta aplicação utilizamos o tailwindcss, que facilita bastante a estilização da nossa aplicação. Se você for utilizar o CSS puro, é bastante recomendado a utilização de uma biblioteca de estilo, como o **styled components** (https://styled-components.com/). Esta biblioteca isola o estilo de cada componente, facilitando o desenvolvimento. Além disso, cria nome de classes automaticamente, entre outras vantagens.

- Se, ao invés de utilizar CSS, preferir frameworks mais "prontas para uso", você pode utilizar o Bootstrap diretamente como componentes com a biblioteca React Bootstrap (<https://react-bootstrap.github.io/>). No entanto, existem outros frameworks de design mais populares e mais adaptados ao React como o **Material-UI** (<https://material-ui.com/>).

- As queries com a Fecth API que criamos tem uma deficiência na sua aplicação. Não especificamos o que exibir quando os dados estão em carregamento ou quando ocorre algum erro na requisição. Isso pode ser implementado criando variáveis de estado para cada _fetch_. Outra possibilidade é utilizar a biblioteca **React Query** (https://react-query.tanstack.com/) que, além de fornecer o estado da query ainda automatiza e isola o processo de sincronização do estado "remoto". Esta biblioteca é implementado com _hooks_; sendo assim, é uma boa oportunidade para verificar a vantagem dos _hooks_ em relação a componentes de classe.

- Por fim, nada foi dito em relação a autenticação em SPAs React. Com o RoR de Backend, uma solução natural é utilizar o Devise, com uma página de login gerada pelo Rails e o resto da aplicação como SPA. Outra opção é utilizar a autenticação com tokens JWT (<https://jwt.io/>). A sua implementação depende do framework de Backend. De qualquer modo, o Frontend deve receber e armazenar o token (que tem o formato JSON) fornecido pelo Backend e deve enviá-lo em cada requisição.

Uma última observação, um pouco fora do escopo desta apostila, é que também é possível gerar aplicações com renderização no servidor (_server-side rendering_ ou SSR) e até páginas estáticas com React! Para tal, é recomendado utilizar um outro Backend (não baseado em RoR). Dois frameworks bastante populares para realizar isso são: **Next.js** (<https://nextjs.org/>) para SSR e páginas estáticas e o gerador de sites estáticos **Gatsby** (<https://www.gatsbyjs.com/>).

<!-- ## Capítulo 7 - Compartilhando o estado entre componentes com o Context -->

<!-- ## Capítulo 8 - Adicionando autenticação com JWT -->
