# html-react-parser

[![NPM](https://nodei.co/npm/html-react-parser.png)](https://nodei.co/npm/html-react-parser/)

[![NPM version](https://img.shields.io/npm/v/html-react-parser.svg)](https://www.npmjs.com/package/html-react-parser)
[![Build Status](https://github.com/remarkablemark/html-react-parser/workflows/build/badge.svg?branch=master)](https://github.com/remarkablemark/html-react-parser/actions?query=workflow%3Abuild)
[![Coverage Status](https://coveralls.io/repos/github/remarkablemark/html-react-parser/badge.svg?branch=master)](https://coveralls.io/github/remarkablemark/html-react-parser?branch=master)
[![Dependency status](https://david-dm.org/remarkablemark/html-react-parser.svg)](https://david-dm.org/remarkablemark/html-react-parser)
[![NPM downloads](https://img.shields.io/npm/dm/html-react-parser.svg?style=flat-square)](https://www.npmjs.com/package/html-react-parser)
[![Financial Contributors on Open Collective](https://opencollective.com/html-react-parser/all/badge.svg?label=financial+contributors)](https://opencollective.com/html-react-parser)

HTML to React parser that works on both the server (Node.js) and the client (browser):

```
HTMLReactParser(string[, options])
```

The parser converts an HTML string to one or more [React elements](https://reactjs.org/docs/react-api.html#creating-react-elements):

```js
const parse = require('html-react-parser');
parse('<div>text</div>'); // equivalent to `React.createElement('div', {}, 'text')`
```

To replace an element with a custom element, check out the [replace option](#replacedomnode).

Demos:

[CodeSandbox](https://codesandbox.io/s/940pov1l4w) | [Repl.it](https://repl.it/@remarkablemark/html-react-parser) | [JSFiddle](https://jsfiddle.net/remarkablemark/7v86d800/) | [Examples](https://github.com/remarkablemark/html-react-parser/tree/master/examples)

<details>
<summary>Table of Contents</summary>

- [Install](#install)
- [Usage](#usage)
  - [Options](#options)
    - [replace(domNode)](#replacedomnode)
  - [library](#library)
  - [htmlparser2](#htmlparser2)
  - [trim](#trim)
- [FAQ](#faq)
  - [Is this XSS safe?](#is-this-xss-safe)
  - [Does invalid HTML get sanitized?](#does-invalid-html-get-sanitized)
  - [Are `<script>` tags parsed?](#are-script-tags-parsed)
  - [Attributes aren't getting called](#attributes-arent-getting-called)
  - [Parser throws an error](#parser-throws-an-error)
  - [Is SSR supported?](#is-ssr-supported)
  - [Elements aren't nested correctly](#elements-arent-nested-correctly)
  - [Warning: validateDOMNesting(...): Whitespace text nodes cannot appear as a child of table](#warning-validatedomnesting-whitespace-text-nodes-cannot-appear-as-a-child-of-table)
  - [Don't change case of tags](#dont-change-case-of-tags)
- [Benchmarks](#benchmarks)
- [Contributors](#contributors)
  - [Code Contributors](#code-contributors)
  - [Financial Contributors](#financial-contributors)
    - [Individuals](#individuals)
    - [Organizations](#organizations)
- [Support](#support)
- [License](#license)

</details>

## Install

[NPM](https://www.npmjs.com/package/html-react-parser):

```sh
$ npm install html-react-parser --save
```

[Yarn](https://yarnpkg.com/package/html-react-parser):

```sh
$ yarn add html-react-parser
```

[CDN](https://unpkg.com/html-react-parser/):

```html
<!-- HTMLReactParser depends on React -->
<script src="https://unpkg.com/react@17/umd/react.production.min.js"></script>
<script src="https://unpkg.com/html-react-parser@latest/dist/html-react-parser.min.js"></script>
<script>
  window.HTMLReactParser(/* string */);
</script>
```

## Usage

Import or require the module:

```js
// ES Modules
import parse from 'html-react-parser';

// CommonJS
const parse = require('html-react-parser');
```

Parse single element:

```js
parse('<h1>single</h1>');
```

Parse multiple elements:

```js
parse('<li>Item 1</li><li>Item 2</li>');
```

Make sure to render parsed adjacent elements under a parent element:

```jsx
<ul>
  {parse(`
    <li>Item 1</li>
    <li>Item 2</li>
  `)}
</ul>
```

Parse nested elements:

```js
parse('<body><p>Lorem ipsum</p></body>');
```

Parse element with attributes:

```js
parse(
  '<hr id="foo" class="bar" data-attr="baz" custom="qux" style="top:42px;">'
);
```

### Options

#### replace(domNode)

The `replace` option allows you to replace an element with another React element.

The `replace` callback's 1st argument is a [domhandler](https://github.com/fb55/domhandler#example) node:

```js
parse('<br>', {
  replace: domNode => {
    console.dir(domNode, { depth: null });
  }
});
```

Console output:

```js
Element {
  parent: null,
  prev: null,
  next: null,
  startIndex: null,
  endIndex: null,
  children: [],
  name: 'br',
  attribs: {}
}
```

The element is replaced if a **valid** React element is returned:

```jsx
parse('<p id="replace">text</p>', {
  replace: domNode => {
    if (domNode.attribs && domNode.attribs.id === 'replace') {
      return <span>replaced</span>;
    }
  }
});
```

The following [example](https://repl.it/@remarkablemark/html-react-parser-replace-example) modifies the element along with its children:

```jsx
import parse, { domToReact } from 'html-react-parser';

const html = `
  <p id="main">
    <span class="prettify">
      keep me and make me pretty!
    </span>
  </p>
`;

const options = {
  replace: ({ attribs, children }) => {
    if (!attribs) {
      return;
    }

    if (attribs.id === 'main') {
      return <h1 style={{ fontSize: 42 }}>{domToReact(children, options)}</h1>;
    }

    if (attribs.class === 'prettify') {
      return (
        <span style={{ color: 'hotpink' }}>
          {domToReact(children, options)}
        </span>
      );
    }
  }
};

parse(html, options);
```

HTML output:

<!-- prettier-ignore-start -->

```html
<h1 style="font-size:42px">
  <span style="color:hotpink">
    keep me and make me pretty!
  </span>
</h1>
```

<!-- prettier-ignore-end -->

Convert DOM attributes to React props with `attributesToProps`:

```jsx
import parse, { attributesToProps } from 'html-react-parser';

const html = `
  <main class="prettify" style="background: #fff; text-align: center;" />
`;

const options = {
  replace: domNode => {
    if (domNode.attribs && domNode.name === 'main') {
      const props = attributesToProps(domNode.attribs);
      return <div {...props} />;
    }
  }
};

parse(html, options);
```

HTML output:

```html
<div class="prettify" style="background:#fff;text-align:center"></div>
```

[Exclude](https://repl.it/@remarkablemark/html-react-parser-56) an element from rendering by replacing it with `<React.Fragment>`:

```jsx
parse('<p><br id="remove"></p>', {
  replace: ({ attribs }) => attribs && attribs.id === 'remove' && <></>
});
```

HTML output:

```html
<p></p>
```

### library

This option specifies the library that creates elements. The default library is **React**.

To use Preact:

```js
parse('<br>', {
  library: require('preact')
});
```

Or a custom library:

```js
parse('<br>', {
  library: {
    cloneElement: () => {
      /* ... */
    },
    createElement: () => {
      /* ... */
    },
    isValidElement: () => {
      /* ... */
    }
  }
});
```

### htmlparser2

The default [htmlparser2 options](https://github.com/fb55/htmlparser2/wiki/Parser-options) are:

```js
{
  decodeEntities: true,
  lowerCaseAttributeNames: false
}
```

Since [v0.12.0](https://github.com/remarkablemark/html-react-parser/tree/v0.12.0), the htmlparser2 options can be overridden.

The following example enables [`decodeEntities`](https://github.com/fb55/htmlparser2/wiki/Parser-options#option-decodeentities) and [`xmlMode`](https://github.com/fb55/htmlparser2/wiki/Parser-options#option-xmlmode):

```js
parse('<p /><p />', {
  htmlparser2: {
    decodeEntities: true,
    xmlMode: true
  }
});
```

> **WARNING**: `htmlparser2` options do not apply on the _client-side_ (browser). The options only apply on the _server-side_ (Node.js). By overriding `htmlparser2` options, universal rendering can break. Do this at your own risk.

### trim

Normally, whitespace is preserved:

```js
parse('<br>\n'); // [React.createElement('br'), '\n']
```

Enable the `trim` option to remove whitespace:

```js
parse('<br>\n', { trim: true }); // React.createElement('br')
```

This fixes the warning:

```
Warning: validateDOMNesting(...): Whitespace text nodes cannot appear as a child of <table>. Make sure you don't have any extra whitespace between tags on each line of your source code.
```

However, intentional whitespace may be stripped out:

```js
parse('<p> </p>', { trim: true }); // React.createElement('p')
```

## FAQ

#### Is this XSS safe?

No, this library is _**not**_ [XSS (cross-site scripting)](https://wikipedia.org/wiki/Cross-site_scripting) safe. See [#94](https://github.com/remarkablemark/html-react-parser/issues/94).

#### Does invalid HTML get sanitized?

No, this library does _**not**_ sanitize HTML. See [#124](https://github.com/remarkablemark/html-react-parser/issues/124), [#125](https://github.com/remarkablemark/html-react-parser/issues/125), and [#141](https://github.com/remarkablemark/html-react-parser/issues/141).

#### Are `<script>` tags parsed?

Although `<script>` tags and their contents are rendered on the server-side, they're not evaluated on the client-side. See [#98](https://github.com/remarkablemark/html-react-parser/issues/98).

#### Attributes aren't getting called

The reason why your HTML attributes aren't getting called is because [inline event handlers](https://developer.mozilla.org/docs/Web/Guide/Events/Event_handlers) (e.g., `onclick`) are parsed as a _string_ rather than a _function_. See [#73](https://github.com/remarkablemark/html-react-parser/issues/73).

#### Parser throws an error

If the parser throws an erorr, check if your arguments are valid. See ["Does invalid HTML get sanitized?"](#does-invalid-html-get-sanitized).

#### Is SSR supported?

Yes, server-side rendering on Node.js is supported by this library. See [demo](https://repl.it/@remarkablemark/html-react-parser-SSR).

#### Elements aren't nested correctly

If your elements are nested incorrectly, check to make sure your [HTML markup is valid](https://validator.w3.org/). The HTML to DOM parsing will be affected if you're using self-closing syntax (`/>`) on non-void elements:

```js
parse('<div /><div />'); // returns single element instead of array of elements
```

See [#158](https://github.com/remarkablemark/html-react-parser/issues/158).

#### Warning: validateDOMNesting(...): Whitespace text nodes cannot appear as a child of table

Enable the [trim](#trim) option. See [#155](https://github.com/remarkablemark/html-react-parser/issues/155).

#### Don't change case of tags

Tags are lowercased by default. To prevent that from happening, pass the [htmlparser2 option](#htmlparser2):

```js
const options = {
  htmlparser2: {
    lowerCaseTags: false
  }
};
parse('<CustomElement>', options); // React.createElement('CustomElement')
```

> **Warning**: By preserving case-sensitivity of the tags, you may get rendering warnings like:
>
> ```
> Warning: <CustomElement> is using incorrect casing. Use PascalCase for React components, or lowercase for HTML elements.
> ```

See [#62](https://github.com/remarkablemark/html-react-parser/issues/62) and [example](https://repl.it/@remarkablemark/html-react-parser-62).

## Benchmarks

```sh
$ npm run test:benchmark
```

Here's an example output of the benchmarks run on a MacBook Pro 2017:

```
html-to-react - Single x 415,186 ops/sec ±0.92% (85 runs sampled)
html-to-react - Multiple x 139,780 ops/sec ±2.32% (87 runs sampled)
html-to-react - Complex x 8,118 ops/sec ±2.99% (82 runs sampled)
```

## Contributors

### Code Contributors

This project exists thanks to all the people who contribute. [[Contribute](https://github.com/remarkablemark/html-react-parser/blob/master/CONTRIBUTING.md)].

[![Code Contributors](https://opencollective.com/html-react-parser/contributors.svg?width=890&button=false)](https://github.com/remarkablemark/html-react-parser/graphs/contributors)

### Financial Contributors

Become a financial contributor and help us sustain our community. [[Contribute](https://opencollective.com/html-react-parser/contribute)]

#### Individuals

[![Financial Contributors - Individuals](https://opencollective.com/html-react-parser/individuals.svg?width=890)](https://opencollective.com/html-react-parser)

#### Organizations

Support this project with your organization. Your logo will show up here with a link to your website. [[Contribute](https://opencollective.com/html-react-parser/contribute)]

[![Financial Contributors - Organization 0](https://opencollective.com/html-react-parser/organization/0/avatar.svg)](https://opencollective.com/html-react-parser/organization/0/website)
[![Financial Contributors - Organization 1](https://opencollective.com/html-react-parser/organization/1/avatar.svg)](https://opencollective.com/html-react-parser/organization/1/website)
[![Financial Contributors - Organization 2](https://opencollective.com/html-react-parser/organization/2/avatar.svg)](https://opencollective.com/html-react-parser/organization/2/website)
[![Financial Contributors - Organization 3](https://opencollective.com/html-react-parser/organization/3/avatar.svg)](https://opencollective.com/html-react-parser/organization/3/website)
[![Financial Contributors - Organization 4](https://opencollective.com/html-react-parser/organization/4/avatar.svg)](https://opencollective.com/html-react-parser/organization/4/website)
[![Financial Contributors - Organization 5](https://opencollective.com/html-react-parser/organization/5/avatar.svg)](https://opencollective.com/html-react-parser/organization/5/website)
[![Financial Contributors - Organization 6](https://opencollective.com/html-react-parser/organization/6/avatar.svg)](https://opencollective.com/html-react-parser/organization/6/website)
[![Financial Contributors - Organization 7](https://opencollective.com/html-react-parser/organization/7/avatar.svg)](https://opencollective.com/html-react-parser/organization/7/website)
[![Financial Contributors - Organization 8](https://opencollective.com/html-react-parser/organization/8/avatar.svg)](https://opencollective.com/html-react-parser/organization/8/website)
[![Financial Contributors - Organization 9](https://opencollective.com/html-react-parser/organization/9/avatar.svg)](https://opencollective.com/html-react-parser/organization/9/website)

## Support

- [GitHub Sponsors](https://b.remarkabl.org/github-sponsors)
- [Open Collective](https://b.remarkabl.org/open-collective-html-react-parser)
- [Tidelift](https://b.remarkabl.org/tidelift-html-react-parser)
- [Patreon](https://b.remarkabl.org/patreon)
- [Ko-fi](https://b.remarkabl.org/ko-fi)
- [Liberapay](https://b.remarkabl.org/liberapay)
- [Teepsring](https://b.remarkabl.org/teespring)

## License

[MIT](https://github.com/remarkablemark/html-react-parser/blob/master/LICENSE)
