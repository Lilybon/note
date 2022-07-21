# Lit Translate

### Reason to use

If you're developing web component ui library with Lit framework and some widgets have static string which requires localization in your project, it's recommend to apply official library `@lit/localize` which is minified and nested dom structure supported for developer.

But for me, `@lit/localize` isn't matched my requirement.<br/>
There are some reasons:
1. The translation file is javascript or typescript generated from XLIFF file. If translation is executed by someone who is not a developer, It's hard for the user to edit XLIFF and build static locale translation files, and he might even have no permission for PR.
2. XLIFF file isn't supported for our CMS which supports translators to create auto tranlation PR. We can only use specific file format such as JSON.

Finally, I found a third party library called `lit-translate` and solve my problem on my work. 

### Steps 

You can follow repo documents to setup code.<br/>
Here are my steps to add localization for our legacy library.

#### 1. Planning

```
└── packages
    ├── widgets                  # bundle entry for all widgets
    └── localization
        ├── locales              # translation files
        │   ├── locales-en.json
        │   ├── locales-ja.json
        │   ├── locales-th.json
        │   └── locales-zh.json
        ├── localization.ts      # localization config
        └── index.ts             # localization common entry
```

#### 2. Install dependency

```bash
yarn add lit-translate@1.2.1
```

If you are using `lit` instead of `lit-element`, use `2.x.x`.

#### 3. Fill in locales

```json
{
  "bundle.hdr-mission": "任務總數 {{ totalCount }}, 已完成 {{ completedCount }}",
  "bundle.lbl-notification": "活動期間每天提醒我",
}
```

A nested JSON is also fine.

#### 4. Set config

Set up initial locale and customized look up for the project.

```typescript
// localization.ts
import { registerTranslateConfig, use, translate } from 'lit-translate'

const DEFAULT_LOCALE = 'en'

const getDefaultLang = () => {
  let lang = navigator.language || DEFAULT_LOCALE
  // Remove language code postfix, for example, zh => zh, zh-TW => zh, zh-CN => zh, en => en, en-US => en
  lang = lang.match(/[a-z]{2}(?=(?:-[A-Z]{2})?)/)![0]
  return lang
}

registerTranslateConfig({
  loader: (lang) =>
    import(`./assets/locales-${lang}.json`).then((module) => module.default),
  // Default "a.b" will looks up multi-layer JSON, but we want to customize the behavior to find translation value in one-layer JSON.
  lookup: (key, config) =>
    config.strings ? config.strings[key].toString() : null,
})

const defaultLang = getDefaultLang()

use(defaultLang)

export { use, translate }
```

```typescript
// index.ts
export * from './localization'
```

Because we are using JSON in typescript, make sure to add additional rule for tsconfig

```json
{
  "resolveJsonModule": true
}
```

#### 5. Implement with web component

```typescript
import {
  customElement,
  LitElement,
  html,
  state,
} from 'lit-element'
import { translate } from '@/localization'


@customElement('bundle')
export class Bundle extends LitElement {
  @state()
  private totalCount: number = 3

  @state()
  private completedCount: number = 1
  
  render () {
    html`
      <div class="bundle__header">
        <p class="bundle__summary">
          ${translate('bundle.hdr-mission', {
            totalCount: this.totalCount,
            completedCount: this.completedCount,
          })}
        </p>
      </div>
    `
  }
}
```

### Result

#### ZH
![ZH](https://i.imgur.com/RLM8SrH.png)

#### EN
![EN](https://i.imgur.com/dGLvl8U.png)

###### tags: `Work` `TypeScript` `Lit`