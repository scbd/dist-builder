# dist-builder

Is a vite based too to assist in developing, testing and deploying self embedding vue components.  The latest version is based on [js import maps](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap).  This greatly reduces build complexity elimination the need for a separate  browser build and server/tooling build.

The caveat is that browsers only permit one import map loaded.  This posses a problem for multiple independent components.  Thus dist-build is dependant on the package [es-module-shims](https://www.npmjs.com/package/es-module-shims).

dist-builder utilizes [es-module-shims](https://www.npmjs.com/package/es-module-shims) in shim mode.  This permits the import of external and multiple import maps currently not supported by any browser.  Further, will polyfill what is needed for the 5% of browsers that do not support ES Modules presently.  As a result 72% us users are passed through utilizing the browsers native functions. Usage results in  ~5ms extra initialization time over native for ES Module Shims fetching, execution and initialization with an 10kb additional download.

[es-module-shims](https://www.npmjs.com/package/es-module-shims)

# CLI Commands

## build

```${ yarn | npx} dist-builder build```

Will build a self embedding vue component utilizing vite, building from 2 entry points widget.js and the default.  Build will transpiles to the dist folder.  Do to complexity all other build formats were removed and now only builds in es format.  A dev folder inside dist also builds an es version not minified for preview and preview widget testing.

## releaseDev [-f]

```${ yarn | npx} dist-builder releaseDev```

Will release a package to scbd's dev component host ```https://scbd-components.s3.amazonaws.com/${packageName}@${packageVersion}/dist/widget/index.js```.

-f options will overwrite an existing version if it exists.


Test the widget released on dev:


```
<script 
      type = "module"
      src  = "https://scbd-components.s3.amazonaws.com/%40scbd/idb-views%400.0.2-alpha/dist/widget/index.js"
      options = "{ 
                    debug      : true,
                    baseApiUrl : 'https://api.cbddev.xyz/api',
                    accountsUrl: 'https://accounts.cbddev.xyz',
                    year       : 2023,
                    view       : 'IdbActionsAdmin',
                    editUrl    : 'https://rjh.bioland.cbddev.xyz/idb-message'
                  }"> 
</script>
```

# dist-builder.config.js

```
import  DistBuilder from '@scbd/dist-builder';

const { viteConfig,  external:deps, getPackageVersion } = DistBuilder;

export const debug   =  true; // see errors when building
export const minify  =  true; // minify production version
export const globals = { vue: 'Vue', 'vue-i18n': 'VueI18n'}; // global within the module scope

export const cdnUrl  = 'https://cdn.jsdelivr.net/npm';  //cdn url where replaced excluding import maps

export const widget           = true; // will this component have a widget entry?
export const testWidget       = true; // build a test widget to for preview 
export const includeInBuild   = [  ]; //  all deps that should be bushed into build and not retrieved from cdn


// point to a different location in import map other than esm.sh
export const importMapOverRide = {
    'pinia'                                         : `https://cdn.jsdelivr.net/npm/pinia@${getPackageVersion('pinia')}/dist/pinia.esm-browser.js`
}

// include extra deps in the import map not seen in package
export const importExtras = {
    'https://esm.sh/stable/vue@3.2.47/es2022/vue.js': `https://esm.sh/stable/vue@3.3.0-alpha.4/es2022/vue.js`,
    '@vue/devtools-api'                             : `https://esm.sh/@vue/devtools-api@^6.5.0`,
    'json5'                                         : `https://esm.sh/json5@2.2.3`
}

const external = deps.filter((item)=>!includeInBuild.includes(item)); // everything is external except specified

// main config export used by the dist builder, cannot be in vit config ... calling it  self
export const distBuilderConfig = {
    widget, testWidget, cdnUrl, external, debug, minify, globals, importMapOverRide, importExtras
}

// format it for vite yet called multiple times
export const makeViteConfig = () => viteConfig(distBuilderConfig)

export default makeViteConfig
```