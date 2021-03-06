# 03 HMR

In this demo we are going to configure Hot Module Replacement. This feature is great for productivity in development environment, allowing us to update the app code without having to redeploy the whole files or refresh our browser to get the changes updated.

We will start from sample _03 Environments/02 CSS Modules_.

Summary steps:
- Install `react-hot-loader`.
- Update `webpack.config.js` with HMR config.
- Update `babel`config.
- Update `students.jsx` file.
- Create `index.jsx` file.

# Steps to build it

## Prerequisites

Prerequisites, you will need to have nodejs installed in your computer. If you want to follow this step guides you will need to take as starting point sample _02 CSS Modules_.

## steps

- `npm install` to install previous sample packages:

```
npm install
```

- To work with HMR and `react`, we need to install `react-hot-loader`:

```bash
npm install react-hot-loader --save-dev
```

- Let' add react-hot-loader/babel plugin to the _[./babelrc](./babelrc)_:


### [./babelrc](./babelrc)

```diff
{
  "presets": [
    "@babel/preset-react",
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "entry",
        "corejs": "3"
      }
    ]
- ]
+ ],
+ "plugins": ["react-hot-loader/babel"]  
}
```

- Let's mark the root component as _hot-exported_:

_[./src/student.jsx](./src/student.jsx)_
```diff
+ import { hot } from 'react-hot-loader'
//(...)

+ const InnerApp = () =>
+  <div>
+    <h1>Hello from React DOM</h1>
+    <AverageComponent />
+    <TotalScoreComponent />
+  </div>
+
+const App = hot(module)(InnerApp);

ReactDOM.render(
+  <App/>,
-  <div>
-    <h1>Hello from React DOM</h1>
-    <AverageComponent />
-    <TotalScoreComponent />
-  </div>,
  document.getElementById('root')
);
```

- Let's add the flag 'hot' to our start command in package json

### [./package.json](./package.json)
```diff
  "scripts": {
-    "start": "webpack-dev-server --mode development --open",
+    "start": "webpack-dev-server --mode development --open --hot",
    "build": "rimraf dist && webpack  --mode development"
  }
```

> We could configure --hot in [./webpack.config.js](./webpack.config.js)

### [./webpack.config.js](./webpack.config.js)
```diff
...
+ devServer: {
+   hot: true,
+ },
  plugins: [
    //Generate index.html in /dist => https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html', //Name of file in ./dist/
      template: 'index.html', //Name of template in ./src
      hash: true,
    }),
//...
```

- Now you can launch `npm run start`, and try to .

- This won't work with CSS using miniCSSExtractTextPlugin, you will need to disable in by config when working on dev mode.

Guide 1: 
https://github.com/webpack-contrib/mini-css-extract-plugin#advanced-configuration-example

Guide 2:

https://github.com/webpack-contrib/mini-css-extract-plugin/issues/34

webpack config udpates

```
module.exports = async function(envCliArgs = {}, argv = {}) {
  const isHot = !!argv.hot || !!argv.hotOnly;
```

```
use: [
  {
    loader: isHot ? "style-loader" : MiniCssExtractPlugin.loader,
  },
```

```
if (!isHot) {
  config.plugins.push(
    new MiniCssExtractPlugin({
      filename: "[name].bundle.[chunkhash].css",
      chunkFilename: "[id].[chunkhash].css",
    }),
  );
}
```

- Meanwhile, we could do some changes:

### [./webpack.config.js](./webpack.config.js)

```diff
...
  output: {
-   filename: '[name].[chunkhash].js',
+   filename: '[name].js',
  },
...

  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
      },
      {
        test: /\.scss$/,
        use: [
-         MiniCssExtractPlugin.loader,
+         'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true,
              localIdentName: '[name]__[local]___[hash:base64:5]',
              camelCase: true,
            },
          },
          {
            loader: 'sass-loader',
            options: {
              implementation: require('sass'),
            },
          },
        ],
      },
      {
        test: /\.css$/,
        use: [
-         MiniCssExtractPlugin.loader,
+         'style-loader',
          'css-loader'
        ],
      },
    ],
  },
  devServer: {
    hot: true,
  },
  plugins: [
    //Generate index.html in /dist => https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      filename: 'index.html', //Name of file in ./dist/
      template: 'index.html', //Name of template in ./src
      hash: true,
    }),
-   new MiniCssExtractPlugin({
-     filename: '[name].css',
-     chunkFilename: '[id].css',
-   }),
  ],
};

```
