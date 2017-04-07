# block-content-to-html-js

Converts the flat Sanity block content structure into HTML.

## Installation

``npm install --save @sanity/block-content-to-html``

## Quick example

```js

// The flat block content structure
const data = {
  "_type": "block",
  "style": "normal",
  "spans": [
    {
      "_type": "span",
      "text": "String with an ",
      "marks": []
    },
    {
      "_type": "span",
      "text": "italicized",
      "marks": [
        "em"
      ]
    },
    {
      "_type": "span",
      "text": " word.",
      "marks": []
    }
  ]
}


// Now convert it with block-content-to-html
const BlockContentToHtml = require('@sanity/block-content-to-html')
const toHtml = new BlockContentToHtml()

const html = toHtml.convert(data)
```

This will result in ``html`` being:

```html
<p>String with an <em>italicized</em> word.</p>
```


## Interface

The constructor will take an object for options:

```js
const toHtml = new BlockContentToHtml(options: Object)
```

Converting the block content is done by calling ``.convert``:

```js
toHtml.convert(data: Array|Object)
```

### Constructor options

Options is an object with any of the following keys:

```js
{
  customTypeHandlers: {
    ...
  },
  blockTypeHandlers: {
    textBlock: {
      ...
    },
    listBlock: {
      ...
    },
    span: {
      ...
    }
    marks: {
      ...
    }
  }
}

```

Read below for a closer description of each option.


#### ``customTypeHandlers: Object``

An object with keys for your custom block types (which is not of type ``block``).
Each key is mapped to a type, and their value is a function which will get the node as input.
It returns HTML.

```js
customTypeHandlers: {
  author: node => {
    return `<div>${node.attributes.name}</div>`
  }
}
```

The ``node`` in this example has the following structure:

```js
{ type: 'author', attributes: { name: 'Test Person' } }
```


#### ``blockTypeHandlers: Object``

Handlers for manipulating the output of the default, built in, block types.
The default block type holds either a block of text or a list.
A text block is built up of spans (with marks), where a list block is built up of list items,
which can contain a text block.

The ``blockTypeHandlers`` object can contain the follow keys:

* ##### ``textBlock: Object``
  Each text block has a ``style``. With this option you can manipulate how each style is rendered.
  By default the style ``normal`` is wrapped in a ``<p>``,
  where other default styles are mapped 1:1 (style ``h2`` produces ``<h2>``).

  The option works in a similar way as ``customTypeHandlers`` described above,
  with the distinction that is has a ``children`` property, and the key is the style name:

  ```js
  textBlock: {
    normal: node => {
      return `<p class="funky-paragraph">${node.children}</p>`
    },
    h2: node => {
      return `<div class="big-heading">${node.children}</div>`
    }
  }
  ```

* ##### ``listBlock: Object``
  By default lists are rendered with plain ``<ol>``, ``<ul>``and ``<li>`` tags.
  With this option you can tweak them into your own liking.

  The object takes the following keys:

  ```js
  listBlock: {
    number: node => {
      return `<ol class="article-list">${node.children}</ol>`
    },
    bullet: node => {
      return `<ul class="article-list">${node.children}</ul>`
    },
    listItem: node => {
      return `<li class="article-list-item">${node.children}</li>`
    }
  }
  ```

* ##### ``span: Object``
  Let you tweak how spans within blocks are rendered. By default the spans are
  just text and marks. As spans may have attributes with data, you can
  make your own render which manipulates the output based on their attributes:

  ```js
  span: node => {
    let result = ''
    if (node.attributes.author) {
      result = `
        <div class="author-bio">
          <img src="${node.attributes.author.image.url}" />
          ${node.attributes.author.name}
        </div>`
    }
    if (node.attributes.link) {
      result += `<a href="${node.attributes.link.href}">${node.children}</a>`
    }
    if (Object.keys(node.attributes).length === 0) {
      result = node.children
    }
    return result
  }
  ```

* ##### ``marks: Object``
  Marks are by default mapped 1:1. If the mark is 'em' the output will be ``<em>``.
  With this option you can map marks to other tags, or just ignore them:

  ```js
  marks: {
    em: null // Just igonore 'em' marks.
    code: 'pre' // Render 'code' marks as 'pre' tags
  }
  ```

## Custom data on the block content

``block-content-to-html`` supports setting custom data on the original block content
through the ``.extra`` property on a block. This is handy if you for example want
to generate and keep track of a HTML id attribute or other arbitrary data related to rendering.
The value can be anything you like.

Example:

```js
const blockContent = {
  "_type": "block",
  "style": "h2",
  "extra": "header_1234", // Our extra property added to the original content.
  "spans": [
    {
      "_type": "span",
      "text": "Such h2 header, much amaze",
      "marks": []
    }
  ]
}

const toHtml = new BlockContentToHtml({
  blockTypeHandlers: {
    textBlock: {
      h2: node => {
        const htmlId = node.extra // Here it is available for us
        return `<h2 id="${htmlId}">${node.children}</h2>`
      }
    }
  }
})

const html = toHtml.convert(blockContentdata)

```

Resulting in ``html`` being:

```html
<h2 id="header_1234">Such h2 header, much amaze</h2>
```

## Utility function
```js
BlockContentToHtml.escapeHtml(unsafe: string)
```
Escape unsafe text to HTML safe text. To be used with your own handler functions.

## More information / examples

Please see the tests.

## License

MIT-licensed
