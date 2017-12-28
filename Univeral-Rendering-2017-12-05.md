# What is Isomorphic JavaScript(Server side rendering)?
Isomorphic JavaScript apps are JavaScript applications that can run both client-side and server-side.

The backend and frontend share the same code.


# How to do server side rendering?
1. [next.js](https://github.com/zeit/next.js/)
2. [webpack-isomorphic-tools](https://github.com/catamphetamine/webpack-isomorphic-tools)
3. our own naive way

The reasone why we don't use next.js is because of the following major reason.

next.js has its own way to route/ink called `next/link`, rather than using `react-router`. Hence, if we adopt next.js, we will need to change lots of code and corresponding packages such as `@twreporter/react-components`, `@twreporter/react-redux-registration` ... etc.

However, `webpack-isomorphic-tools` looks too complicated to follow up. In its doc, the author recommend to use next.js instead.

Because of the above situations, we decide to do the server side rendering by our own selves.


## Do server side rendering by naive way.

The following list is the obstacles to do server side rendering.

1. Use `css-module` and `scss` in the source codes. Those codes need to be preprocessed by either babel(for server side) or webpack(for client side) before starting the server or running webpack js bundles on the browser. 
 

2. Javascript codes is written in ECMAS2016(es6), those codes should be transpiled into es5 by babel before starting the server. Otherwise, the server will encouter errors.


**For point 1**, see the following example.

```
// filename: Common.js
// we use scss and css module in the source
import styles from './Common.scss'

...

render() {
	return (
    	<div className={styles.title}>{$title}</div>
    )
}
```

For server side, in the preporcess steps, we need to preprocess the scss files into css files, and use postcss to require css files into `Common.js`. 

Those preprocesses could be done by babel. I use `babel-plugin-css-modules-transform` to transpile the `Common.js`.

You can set the plugin in `.babelrc`
```
{
  "presets": [
    "es2015",
    "react",
    "stage-0"
  ],
  "env": {
    "ssr": {
      "plugins": [
        [
          "css-modules-transform",
          {
            "extensions": [
              ".css",
              ".scss"
            ],
            "generateScopedName": "[name]__[local]___[hash:base64:5]"
          }
        ]
      ]
    }
  },
  "plugins": [
    [
      "system-import-transformer",
      {
        "commonJS": {
          "useRequireEnsure": true
        }
      }
    ],
    "inline-import-data-uri"
  ]
}
```
And run `./node_modules/.bin/babel BABEL_ENV=ssr Common.js -o Common-es5.js`


**IMPORTANT**: Remember to set `BABEL_ENV=ssr`(use `babel-plugins-css-modules-transform`) when you want to transpile javascript codes for server side use.


The transpiled `Common-es5.js` will be like the following code
```
// import styles from './Common.scss' ====>

var styles = {
	{
    	"title":"Common__title___2cOrF"
    }
}
```

Hence, you can import the scss file in your javscript source code.

For client side, these preprocess steps could be done by webpack. And you can easily bundle css into js by css-loader, postcss-loader, sass-loader.

```
// webpack.conf.js
rules: [
	{
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: 'babel-loader'
    },
 	{
        test: /\.(scss|sass)$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
              modules: true,
              sourceMap: true,
              localIdentName: '[name]__[local]___[hash:base64:5]'
            }
          },
          {
            loader: 'postcss-loader',
            options: {
              plugins: function (loader) {
                return [ autoprefixer({ browsers: [ '> 1%' ] }) ]
              }
            }
          },
          'sass-loader'
        ]
      }
   ]
```


**For point 2**, we can use `babel-node` or `babel` cli to transpile es6 files runtime or offline.

**IMPORTANT**: Do not set `BABEL_ENV=ssr`(which means using `babel-plugins-css-modules-transform`) to transpile es6 while using babel-loader. Because if you transpile the javascript codes with `babel-plugins-css-modules-transform`, the following style, css or sass loader won't have any effects.


After transpiling and bundling, we only need to add these assets path into html like
```
<link href="/dist/main.bundle.css" type="text/css" rel="stylesheet" charSet="utf-8"/>
<script src="/dist/main.bundle.js" charSet="utf-8"/>
```

**BUT, HOW DO WE KNOW THE FILENAMES OF THESE ASSETS?**

Simply write the webpack plugin(`BundleListPlugin` in the following code) to record the compiled files and record css filenames in the `ExtractTextPlugin` into the file, `webpack-assets.json` we named.

```
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const autoprefixer = require('autoprefixer');
const config = require('./config')
const fs = require('fs')
const path = require('path');
const webpack = require('webpack');

const isProduction = process.env.NODE_ENV === 'production'
const webpackDevServerHost = config.webpackDevServerHost
const webpackDevServerPort = config.webpackDevServerPort
const webpackPublicPath = config.webpackPublicPath
const webpackOutputFilename = config.webpackOutputFilename

const defaulAssets = {
  javascripts: {
    chunks: []
  },
  stylesheets: []
}

function BundleListPlugin(options) {}

// BundleListPlugin is used to write the paths of files compiled by webpack
// such as javascript files transpiled by babel,
// and scss files handled by sass, postcss and css loader,
// into webpack-assets.json
BundleListPlugin.prototype.apply = function(compiler) {
  compiler.plugin('emit', function(compilation, callback) {
    let assets
    try {
      assets = require('./webpack-assets.json')
    } catch (e) {
      assets = defaulAssets
    }
    for (const filename in compilation.assets) {
      if (filename.startsWith('main')) {
        assets.javascripts.main = `/dist/${filename}`
      } else if (filename.indexOf('-chunk-') > -1) {
        assets.javascripts.chunks.push(`/dist/${filename}`)
      }
    }

    fs.writeFileSync('./webpack-assets.json', JSON.stringify(assets))
    callback();
  });
};

const webpackConfig = {
  context: path.resolve(__dirname),
  entry: {
    main: './src/client.js',
  },
  output: {
    chunkFilename: '[id]-chunk-[chunkhash].js',
    filename: webpackOutputFilename,
    path: path.join(__dirname, 'dist'),
    publicPath: webpackPublicPath
  },
  devtool: 'inline-source-map',
  devServer: {
    hot: true,
    host: webpackDevServerHost,
    port: webpackDevServerPort
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      },
      {
        test: /node_modules\/react-flex-carousel\/index.js/,
        use: 'babel-loader'
      },
      {
        test: /\.(scss|sass)$/,
        use: isProduction ? ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: [
            {
              loader: 'css-loader',
              options: {
                importLoaders: 2,
                modules: true,
                sourceMap: true,
                // Make sure this setting is equal to settings in .bablerc
                localIdentName: '[name]__[local]___[hash:base64:5]'
              }
            }, {
              loader: 'postcss-loader',
              options: {
                plugins: function (loader) {
                  return [ autoprefixer({ browsers: [ '> 1%' ] }) ]
                }
              }}, 'sass-loader']
        }) : [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
              modules: true,
              sourceMap: true,
              localIdentName: '[name]__[local]___[hash:base64:5]'
            }
          },
          {
            loader: 'postcss-loader',
            options: {
              plugins: function (loader) {
                return [ autoprefixer({ browsers: [ '> 1%' ] }) ]
              }
            }
          },
          'sass-loader'
        ]
      },
      {
        test: /\.css$/,
        use: isProduction ? ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: [{
            loader: 'css-loader',
            options: {
              importLoaders: 1
            }
          }, {
            loader: 'postcss-loader',
            options: {
              plugins: function (loader) {
                return [ autoprefixer({ browsers: [ '> 1%' ] }) ]
              }
            }
          }]
        }) : [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              plugins: function (loader) {
                return [ autoprefixer({ browsers: [ '> 1%' ] }) ]
              }
            }
          }
        ]
      },
      {
        test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
        use: [{
          loader: 'url-loader',
          options: {
            limit: 10000,
            mimetype: 'image/svg+xml'
          }
        }]
      }
    ]
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': {
        BROWSER: true,
        NODE_ENV: isProduction ? '"production"' : '"development"'
      },
      __CLIENT__: true,
      __DEVELOPMENT__: !isProduction,
      __DEVTOOLS__: true,  // <-------- DISABLE redux-devtools HERE
      __SERVER__: false
    })
  ]
}


if (isProduction) {
  webpackConfig.plugins.push(
    new BundleListPlugin(),
    // css files from the extract-text-plugin loader
    new ExtractTextPlugin({
      // write css filenames into webpack-assets.json
      filename: function(getPath) {
        const filename = getPath('[chunkhash].[name].css')
        let assets
        try {
          assets = require('./webpack-assets.json')
        } catch (e) {
          assets = defaulAssets
        }
        assets.stylesheets.push(webpackConfig.output.publicPath + filename)
        fs.writeFileSync('./webpack-assets.json', JSON.stringify(assets))
        return filename
      },
      disable: false,
      allChunks: true
    }),
    new webpack.optimize.UglifyJsPlugin()
  )
} else {
  webpackConfig.plugins.push(
    new webpack.HotModuleReplacementPlugin()
  )
}

module.exports = webpackConfig;
```

### Improvements
Just write `styled-components`. 
And we don't have to do such preprocess and write so many config file.
