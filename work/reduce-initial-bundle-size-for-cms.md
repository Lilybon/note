# Reduce Initial Bundle Size for CMS

As sprints go on, our CMS project becomes a huge project. But this application responses very slow due to very large **app chunk** and **vendor chunk**, and it might irritate clients because of the long time waiting for static bundle responding.

There are two problems in our CMS:

1. Too many **unused pages** are included in **app chunk** - In fact, there are only few of them might be invited for the first time. **Lazy route** can split page routes into multiple async chunks to minimize app chunk, only when user actually needs the page, web server serve the page js bundle for him.
2. Too many **unused third party dependencies** included in **vendor chunk** - Not all the dependencies are important. For example: `vue-code-diff` is a large and rarely used dependency only for event change history page, then it's a worst strategy to add it in vendor chunk, since only user who invites this page requires this package.

There are few strategies to resolve the struggle below.

### 1. Dynamic import page component

Not all routes are required in the initial app chunk, we should minimize bundle size for client as possible.

```javascript
[
    // ...
    {
        path: '/page/foo/editor',
        name: 'Foo Editor',
        component: () => import('@/pages/Foo/FooEditor'),
     }
]
```

### 2. Replace `underscore` with `ramda` and `lodash-es`

Our project is based on `underscore` and `ramda` as util functions. But in fact, only little `underscore` functions are used. I tried monolitic import `underscore` to gain small bundle size but failed. Finally, I replace almost 95% of `unserscore` functions with `ramda` functions, as for some functions which are not pure-function such as`throttle` and `uniqueId`, I used `lodash-es`.

#### AS-IS
![underscore-npm](https://i.imgur.com/sObMF1b.png)

#### TO-BE
![lodash-es-npm](https://i.imgur.com/GVkq2qf.png)

### 3. Split vendor chunk into multiple chunks

To decrease vendor size, I drop large and rarely used third party dependencies, and then split vendor to ui and base chunk to make sure all chunks are as tiny as possible.

```javascript
const webpackConfig = {
    // ...
    optimization: {
        splitChunks: {
            cacheGroups: {
                ui: {
                    name: 'ui',
                    chunks: 'initial',
                    test:  /[\\/]node_modules[\\/](bootstrap|bootstrap-vue)[\\/]/,
                    priority: -5,
                    enforce: true,
                    reuseExistingChunk: true
                },
                base: {
                    name: 'base',
                    chunks: 'initial',
                    // exclude all large or rarely used third party dependencies
                    test: /[\\/]node_modules[\\/]((?!(highlights|vue-code-diff|vue-carousel|vue-datepicker)).+)[\\/]/,
                    priority: -10,
                    enforce: true,
                    reuseExistingChunk: true
                },
                default: {
                    chunks: 'initial',
                    minChunks: 2,
                    priority: -20,
                    enforce: true
                }
            }
        }
    }
}
```

### 4. Tree Shake BootstrapVue components

Add two cutom plugins `BoostrapVue` and `BootstrapVueIcons` to import only used components.

#### AS-IS

![bootstrap-vue-npm@before](https://i.imgur.com/JlzH59m.png)

#### TO-BE

```javascript
// BootstrapVue
import {
    AlertPlugin,
    BForm,
    BFormText,
    BFormInvalidFeedback,
} from 'bootstrap-vue'

export default {
    install (vm) {
        // Alert
        vm.use(AlertPlugin)
      
        // Form
        vm.component('BForm', BForm)
        vm.component('BFormText', BFormText)
        vm.component('BFormInvalidFeedback', BFormInvalidFeedback)
    }
}
```

```javascript
// BootstrapVueIcons
import {
    BIcon,
    BIconPersonFill
} from 'bootstrap-vue'

export default {
    install (vm) {
    // Icons
        vm.component('BIcon', BIcon)

        // Icon Reference: https://github.com/bootstrap-vue/bootstrap-vue/blob/dev/src/icons/icons.js
        vm.component('BIconPersonFill', BIconPersonFill)
        // ...
    }
}

```

![bootstrap-vue-npm@after](https://i.imgur.com/NzBVTNq.png)

### 5. Ignore MomentJS locale

Since we don't use any locale format date string for usage. Drop all locales in `momentjs`.

```javascript
const webpack = require('webpack')

const webpackConfig = {
    // ...
    plugins: [
        // exclude node_modules/moment/locale/*.js
        new webpack.IgnorePlugin({
          resourceRegExp: /^\.\/locale$/,
          contextRegExp: /moment$/,
        })
    ]
}
```

### 6. Final result

#### AS-IS

##### Bundle analyze

Max chunk size was 7.69 MB (uncompressed).
Many unused pages were involved in app chunk.
Many unused third party dependencies were involved in vendor chunk.

![bundle analyze before](https://i.imgur.com/Eiwcuqd.png)

##### Network

User paid about 7~11 seconds for the page response.

![network before 1](https://i.imgur.com/pU4Z2m5.png)



#### TO-BE

##### Bundle analyze

Max chunk size is 2.54 MB (uncompressed).
Only core business logic are invloved in app chunk, and not all pages included in the initial bundle.
Only core third party dependencies are involved in vendor chunk, as for user requires a rarely dependency, he needs an additional network request for it.
Besides, chunks seems more average after code splitting.

![bundle anaylze after](https://i.imgur.com/fuVShQr.jpg)

##### Network

User pays about 1~4 seconds for the page response.

![network after](https://i.imgur.com/XlPeQk2.png)



### 6. What's next for CMS?

Here are some tips for refactoring our CMS after all these changes:

1. Replace `momentjs` with `dayjs` for better bundle size.
2. Setup `gzip` for `js`, `css`, `html` resource for web server.
3. Load `ckeditor4` only for required page.
4. Stub `ec-lib` and `bootstrap-vue` for component unit test as possible to reduce unit test time.
5. Extract `/pages` from `/components` to improve readability of project.

###### tags: `Work` `JavaScript` `Vue` `Vue2`
