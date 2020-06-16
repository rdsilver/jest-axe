# jest-axe

[![npm version](https://img.shields.io/npm/v/jest-axe.svg)](http://npm.im/jest-axe)
![node](https://img.shields.io/node/v/jest-axe)
[![Build Status](https://travis-ci.org/nickcolley/jest-axe.svg?branch=main)](https://travis-ci.org/nickcolley/jest-axe)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)

Custom [Jest][Jest] matcher for [aXe](https://github.com/dequelabs/axe-core) for testing accessibility

## ⚠️✋ This project does not guarantee what you build is accessible.
The GDS Accessibility team found that only [~30% of issues are found by automated testing](https://accessibility.blog.gov.uk/2017/02/24/what-we-found-when-we-tested-tools-on-the-worlds-least-accessible-webpage).

Tools like aXe are similar to [code linters](https://en.wikipedia.org/wiki/Lint_%28software%29) such as [eslint](https://eslint.org/) or [stylelint](https://stylelint.io/): they can find common issues but cannot guarantee what you build works for users.

You'll also need to:
- test your interface with the [assistive technologies that real users use](https://www.gov.uk/service-manual/technology/testing-with-assistive-technologies#when-to-test) (see also [WebAIM's survey results](https://webaim.org/projects/screenreadersurvey8/#primary)).
- include disabled people in user research.

## Installation:
```bash
npm install --save-dev jest-axe
```

[TypeScript](https://www.typescriptlang.org/) users can install the community maintained types package:

```bash
npm install --save-dev @types/jest-axe
```

## Usage:

```javascript
const { axe, toHaveNoViolations } = require('jest-axe')

expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage', async () => {
  const render = () => '<img src="#"/>'

  // pass anything that outputs html to axe
  const html = render()

  expect(await axe(html)).toHaveNoViolations()
})
```

![Screenshot of the resulting output from the usage example](example-cli.png)

> Note, you can also require `'jest-axe/extend-expect'` which will call `expect.extend` for you.
> This is especially helpful when using the jest `setupFilesAfterEnv` configuration.

### Testing React

```javascript
const React = require('react')
const { render } =  require('react-dom')
const App = require('./app')

const { axe, toHaveNoViolations } = require('jest-axe')
expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with react', async () => {
  render(<App/>, document.body)
  const results = await axe(document.body)
  expect(results).toHaveNoViolations()
})
```

### Testing React with [Enzyme](https://enzymejs.github.io/enzyme/)

```javascript
const React = require('react')
const App = require('./app')

const { mount } = require('enzyme')
const { axe, toHaveNoViolations } = require('jest-axe')
expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with enzyme', async () => {
  const wrapper = mount(<App/>)
  const results = await axe(wrapper.getDOMNode())
  
  expect(results).toHaveNoViolations()
})
```

### Testing React with [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)

```javascript
const React = require('react')
const App = require('./app')

const { render } = require('@testing-library/react')
const { axe, toHaveNoViolations } = require('jest-axe')
expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with react testing library', async () => {
  const { container } = render(<App/>)
  const results = await axe(container)
  
  expect(results).toHaveNoViolations()
})
```

> Note: If you're using `react testing library` <9.0.0 you should be using the
> [`cleanup`](https://testing-library.com/docs/react-testing-library/api#cleanup) method. This method removes the rendered application from the DOM and ensures a clean HTML Document for further testing.

### Testing Vue with [Vue Test Utils](https://vue-test-utils.vuejs.org/)

```javascript
const App = require('./App.vue')

const { mount } = require('@vue/test-utils')
const { axe, toHaveNoViolations } = require('jest-axe')
expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with vue test utils', async () => {
  const wrapper = mount(Image)
  const results = await axe(wrapper.element)

  expect(results).toHaveNoViolations()
})
```

### Testing Vue with [Vue Testing Library](https://testing-library.com/docs/vue-testing-library/intro)

```javascript
const React = require('react')
const App = require('./app')

const { render } = require('@testing-library/vue')
const { axe, toHaveNoViolations } = require('jest-axe')
expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with react testing library', async () => {
  const { container } = render(<App/>)
  const results = await axe(container)
  
  expect(results).toHaveNoViolations()
})
```
> Note: If you're using `vue testing library` <3.0.0 you should be using the
> [`cleanup`](https://testing-library.com/docs/vue-testing-library/api#cleanup) method. This method removes the rendered application from the DOM and ensures a clean HTML Document for further testing.

### Axe configuration

The `axe` function allows options to be set with the [same options as documented in axe-core](https://github.com/dequelabs/axe-core/blob/master/doc/API.md#options-parameter):

```javascript
const { axe, toHaveNoViolations } = require('jest-axe')

expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with a custom config', async () => {
  const render = () => `
    <div>
      <img src="#"/>
    </div>
  `

  // pass anything that outputs html to axe
  const html = render()

  const results = await axe(html, {
    rules: {
      // for demonstration only, don't disable rules that need fixing.
      'image-alt': { enabled: false }
    }
  })

  expect(results).toHaveNoViolations()
})
```

## Setting global configuration

If you find yourself repeating the same options multiple times, you can export a version of the `axe` function with defaults set.

Note: You can still pass additional options to this new instance; they will be merged with the defaults.

This could be done in [Jest's setup step](https://jestjs.io/docs/en/setup-teardown)

```javascript
// Global helper file (axe-helper.js)
const { configureAxe } = require('jest-axe')

const axe = configureAxe({
  rules: {
    // for demonstration only, don't disable rules that need fixing.
    'image-alt': { enabled: false }
  }
})

module.exports = axe
```

```javascript
// Individual test file (test.js)
const { toHaveNoViolations } = require('jest-axe')
const axe = require('./axe-helper.js')

expect.extend(toHaveNoViolations)

it('should demonstrate this matcher`s usage with a default config', async () => {
  const render = () => `
    <div>
      <img src="#"/>
    </div>
  `

  // pass anything that outputs html to axe
  const html = render()

  expect(await axe(html)).toHaveNoViolations()
})
```

## Thanks
- [Jest][Jest] for the great test runner that allows extending matchers.
- [aXe](https://www.deque.com/axe/) for the wonderful axe-core that makes it so easy to do this.
- Government Digital Service for making coding in the open the default.
  - GOV.UK Publishing Frontend team who published the [basis of the aXe reporter](https://github.com/alphagov/govuk_publishing_components/blob/581c22c9d35d85d5d985571d007f6397a4399f4c/spec/javascripts/govuk_publishing_components/AccessibilityTestSpec.js)
- [jest-image-snapshot](https://github.com/americanexpress/jest-image-snapshot) for inspiration on README and repo setup

[Jest]: https://jestjs.io/
