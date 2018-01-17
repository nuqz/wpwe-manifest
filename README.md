# wpwe-manifest

[Webpack](https://webpack.js.org/) plugin which generates [WebExtension](https://developer.mozilla.org/en-US/Add-ons/WebExtensions) manifest file(s).

## Add plugin to your project

Use one of your favourite package managers

`$ npm i --save-dev wpwe-manifest`

or

`$ yarn add --dev wpwe-manifest`

## Basics

Following example shows minimal plugin setup:

```js
// Require plugin package
const wpwe = require('wpwe-manifest')

// Represents WebExtension manifest file
const manifestOptions = {
    name: 'WebExtension name',
    version: '0.0.1-beta'
}

const myWebpackConfig = {
    plugins: [
        new wpwe.Plugin({
            // (required)
            manifest: new wpwe.Manifest(manifestOptions),

            // (optional) default "manifest.json"
            filename: 'browser_manifest.json',

            // (optional) these are passed to JSON.stringify(...)
            replacer: myReplacer,
            space: mySpacer
        }),
        
        ...
    ],
    
    ...
}
```

`manifestOptions` represents [`manifest.json`](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/manifest.json) file using following rules:

> [`manifest_version`](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/manifest.json/manifest_version) option with value 2 will be added automatically and will not be mutable.

> `name` and `version` keys are required by WebExtensions and plugin will throw an exception.

> Chunks names may be used instead of real file paths where possible (e.g. in `background.scripts`, `content_scripts[i].css`, etc). When webpack finishes compilation chunk names will be substituted by real file paths in the output manifest file.

> Chunks, which are required by HTML pages (e.g. `popup.html`, `options.html`), should be injected into that pages by webpack and be accessible when you open page as file in browser.

## Extended example

You may want to prepare `manifest.json` template file for your WebExtension. The following snippet is how that file may look like:

```json
// src/webExtension/manifest.json

{
    "name": "New awesome WebExtension",
    "background": {
        "persistent": false,
        "scripts": ["manifest", "vendor", "background"]
    },
    "browser_action": {
        "default_title": "NAWE Popup Window",
        "default_popup": "popup.html"
    },
    "content_scripts": [
        {
            "matches": ["*://*/*"],
            "css": ["contentScript"],
            "js": ["manifest", "vendor", "contentScript"]
        }
    ],
    "permissions": [
        "activeTab",
        "storage",
        "http://*/",
        "https://*/"
    ]
}
```

The next step is to configure webpack:

```js
// webpack.config.js

// Will be used to build "popup.html" for browser action
const HtmlWebpackPlugin = require('html-webpack-plugin')

// Require plugin package
const wpwe = require('wpwe-manifest')

// (Optional)
const pkg = require('./package.json')

// Require manifest template from file and add values from package.json
const manifestOptions = Object.assign({}, require('./src/webExtension/manifest.json'), {
        version: pkg.version,
        description: pkg.description,
        author: pkg.author
    })

// Webpack configuration
module.exports = {
    context: './src/webExtension',
    entry: {
        background: 'background.js',
        contentScript: 'content_script.js',
        popup: 'popup.js',
    },
    output: {
        filename: '[name].[chunkhash].js',
        ...
    },
    plugins: [
        new HtmlWebpackPlugin({
            filename: 'popup.html',
            template: 'popup.html',

            // Take care about chunks injection
            chunks: ['manifest', 'vendor', 'popup'],
            inject: true,

            ...
        }),
        new wpwe.Plugin({ manifest: new wpwe.Manifest(manifestOptions) }),
        
        ...
    ]
}
```

Finally, webpack builds will produce minified version of something like following:

```json
// ./dist/manifest.json

{
    "manifest_version": 2,
    "name": "New awesome WebExtension",
    "background": {
        "persistent": false,
        "scripts": [
            "/assets/js/manifest.hashab123.js",
            "/assets/js/vendor.hashbc456.js",
            "/assets/js/background.hashcd789.js"
        ]
    },
    "browser_action": {
        "default_title": "NAWE Popup Window",
        "default_popup": "popup.html"
    },
    "content_scripts": [
        {
            "matches": ["*://*/*"],
            "css": ["/assets/css/content_script.hashfg654.css"],
            "js": [
                "/assets/js/manifest.hashab123.js",
                "/assets/js/vendor.hashbc456.js",
                "/assets/js/background.hashde987.js"
            ]
        }
    ],
    "permissions": [
        "activeTab",
        "storage",
        "http://*/",
        "https://*/"
    ],

    "version": "Version from package.json",
    "description": "Description from package.json",
    "author": "Author from package.json"
}
```

## Extend

Feel free to fork, extend and send pull requests with new features.

To run test use:

`$ npm run test`

or

`$ yarn run test`
