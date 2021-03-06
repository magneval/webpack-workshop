# Coding with style :nail_care:

> To start this exercise, be sure to be in `./packages/intermediate/style` folder.
> Be sure you have [installed this repository first](../README.md#install)

## Introduction

We often use a CSS preprocessor in our project to help us handle CSS mixins, dependencies, palette, generating classes.
By chance those preprocessor have a their own specific loader to help webpack build their dependencies tree.
In this workshop, we will setup `sass` loader to help use reduce the quantity of CSS generated by Webpack.

::: tip
We are using [Bulma](https://bulma.io/) css library here.
:::

## Sass loader to take only the CSS we need from Bulma

Try to load the scss file of Bulma first and edit your configuration to include the sass loader.
The needed file is `bulma/bulma.sass`.

<details>
<summary>Solution</summary>

```js{25-28}
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./src/main.js", // The source module of our dependency graph
  output: {
    // Configuration of what we tell webpack to generate (here, a ./dist/main.js file)
    filename: "main.bundle.js",
    path: path.resolve(__dirname, "dist")
  },
  module: {
    rules: [
      {
        test: /\.jpg$/,
        use: [
          {
            loader: "file-loader",
            options: {
              outputPath: "assets",
              publicPath: "dist/assets"
            }
          }
        ]
      },
      {
        test: /\.sass$/,
        use: ["style-loader", "css-loader", "sass-loader"]
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html"
    })
  ]
};
```

```js{1}
import "bulma/bulma.sass";
import PokemonComponent from "./pokemon.component";
import { getPokemons } from "./pokemon.service";

const pokemonList = document.querySelector("#pokemons");

getPokemons().then(response => {
  response.results.map(({ name }, index) => {
    pokemonList.appendChild(PokemonComponent(name, index + 1));
  });
});
```

</details>

::: warning
Did you see the `node-sass` dev dependency is mandatory for "sass-loader". You can see it in the `package.json`.
:::

## Extracting your CSS in specific bundles

Having your CSS in your bundle is a quick win, but it may transform your bundle into huge files containing all your CSS stuff you don't need to have to render your website.
You will probably want to split your style out of the JS bundle and extract it in `css` bundles.

Try to configure [Mini CSS Extract Plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)

<details>
<summary>Solution</summary>

```js{3}{37-40}
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  entry: "./src/main.js", // The source module of our dependency graph
  output: {
    // Configuration of what we tell webpack to generate (here, a ./dist/main.js file)
    filename: "main.bundle.js",
    path: path.resolve(__dirname, "dist")
  },
  module: {
    rules: [
      {
        test: /\.jpg$/,
        use: [
          {
            loader: "file-loader",
            options: {
              outputPath: "assets",
              publicPath: "dist/assets"
            }
          }
        ]
      },
      {
        test: /\.sass$/,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          "css-loader",
          "sass-loader"
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].css",
      chunkFilename: "[id].css"
    }),
    new HtmlWebpackPlugin({
      template: "./src/index.html"
    })
  ]
};
```

</details>

## Use a CSS post processor

If you want to target many browsers with your CSS, you will need a post processor. Most commonly used is `PostCSS`.
Try to change the configuration to use postCSS loader in your app to generate a cross browser compatible CSS.

[PostCSS Loader](https://github.com/postcss/postcss-loader) is already installed. :wink:
You will need to configure only the [autoprefixer plugin](https://www.npmjs.com/package/autoprefixer).

<details>
<summary>Solution</summary>

```js{30-40}
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const autoprefixer = require("autoprefixer");

module.exports = {
  entry: "./src/main.js", // The source module of our dependency graph
  output: {
    // Configuration of what we tell webpack to generate (here, a ./dist/main.js file)
    filename: "main.bundle.js",
    path: path.resolve(__dirname, "dist")
  },
  module: {
    rules: [
      {
        test: /\.jpg$/,
        use: [
          {
            loader: "file-loader",
            options: {
              outputPath: "assets",
              publicPath: "dist/assets"
            }
          }
        ]
      },
      {
        test: /\.sass$/,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          "css-loader",
          {
            loader: "postcss-loader",
            options: {
              plugins: [
                autoprefixer({
                  browsers: ["IE >= 10", "last 2 versions", "chrome >= 28"]
                })
              ]
            }
          },
          "sass-loader"
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].css",
      chunkFilename: "[id].css"
    }),
    new HtmlWebpackPlugin({
      template: "./src/index.html"
    })
  ]
};
```

</details>
