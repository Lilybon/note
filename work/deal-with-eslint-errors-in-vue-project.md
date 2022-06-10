# Deal with ESLint errors in Vue project

![eslint logo](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/ESLint_logo.svg/1200px-ESLint_logo.svg.png =200x)

### Reasons to fix

ESLint is a linter for javascript used for ensuring better smell of code, if we develop our application instead of fixing errors and warings already existed, **bad practices** will pass on, and one day we find there is an extreme error in our porject but it's already too late to fix.

### SOP

1. Collect all the rules which are not passed.
    * Keep a note when running the application to check what types of error occur.
    ![collect error rules](https://i.imgur.com/2DCsrjo.png)

2. Checkout each rule's documentations.

    * For example above, we can check the detail for both [component-definition-name-casing](https://eslint.vuejs.org/rules/component-definition-name-casing.html) and [no-mutating props](https://eslint.vuejs.org/rules/no-mutating-props.html) via `eslint-plugin-vue` documentation, it indicates how the code be defined as good or bad, and how to ignore the rule by configuration.

3. Checkout scope for change.
    * More details in solution section.

4. Fix or ignore problems.
    * If the scope for change is too huge to be measured, you'd better **ignore it by configuration** or **remain the same**. It depends on how it costs to fix each rules.
    * If the application scale is huge, **automation testing** is nice to have to verify whether the change doesn't break any other test case.

5. Repeat iterating steps 2 ~ 4 until your console is neat for debugging.

### Solutions

Focus on errors and warning occurs by `eslint-plugin-vue`.

#### `vue/no-mutating-props`

Remember one thing for all modern frontend framworks: always keep data flow **top down**.

![water flow](https://cdn.pixabay.com/photo/2015/09/03/21/10/water-921325_960_720.jpg =350x)

1. Find out the component which executes the invalid mutation. 

    Replace direct mutation with emitter.
    ```javascript
    // Vue 2 example for emitter

    // <your-component :foo.sync="foo" />
    this.$emit('update:foo', newFoo)

    // <your-component v-model="foo" />
    this.$emit('input', value)
    ```

    For a deeper prop, a computed variable or vuex state variable is recommended.

    ```javascript
    // Vue 2 example for computed getter/setter
    export default {
      // ...
      props: {
        foo: {
          type: Number,
          required: true
        }
      },
      computed: {
        currentFoo: {
          get () {
            return this.foo
          },
          set (newFoo) {
            this.$emit('update:foo', newFoo)
          }
        }
      } 
    }
    ```

2. Check data flows in both parent and children work normally.
    * No shallow copy object props bind in data attribute.
    * Once any bad practice exists, refactor first.

#### `vue/multi-word-component-names`

The reason to fix this problem is to prevent conflicted same name with native DOM element name.<br/>

For components, I add component name or rename file names to fix it.
```javascript
// Bad
export default {
  name: 'Helloworld'
}
```

```javascript
// Good
export default {
  name: 'HelloWorld'
}
```
For routes, I can't cahnge the route name with restriction of specification, I ignore it by configuration.
```javascript
// .eslintrc.js
module.exports = {
  'vue/multi-word-component-names': 0
}
```

#### `vue/attribute-hyphenation`

```html
<!--Bad-->
<your-component :isActive="isActive" />
```

```html
<!--Good-->
<your-component :is-active="isActive" />
```

#### `vue/no-lone-template`

```html
<!--Bad-->
<template>
  <!--Simple DOM element inside-->
  <div>Hello World!</div>
</template>
```

```html
<!--Good-->
<div>Hello World!</div>
```

#### `vue/valid-v-slot`
 
```html
<b-table>
  <template #cell(winner.price)="row">
    {{ formatPrice(row.value) }}
  </template>
</b-table>
```

For the usage of `bootstrap-vue` table key mapping, I ignore it by configuration.

```javascript
// .eslintrc.js
module.exports = {
  'vue/valid-v-slot': ['error', {
    'allowModifiers': true
  }]
}
```

### Result

![throw computer](https://media0.giphy.com/media/n9ewEcw0oyHEYEuH1c/giphy.gif =350x)

Errors and warning are reduced by 87.5%.<br/>
the terminal is neater. Developers can get feedback by ESLint immediately 🎉.

Before `v6.3.0`

![before v6.3.0](https://i.imgur.com/YRy6PaO.png)

After `v6.7.0`

![after v6.7.0](https://i.imgur.com/9jTpAhs.png)

###### tags: `Work` `JavaScript` `Vue`