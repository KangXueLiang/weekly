---
nav: zh-Hans
search: zh-Hans
---

# docute

docute 让你无需编译文件享受到无缝的文档撰写、发布体验。

<p class="danger">
  文档翻译已经严重滞后，请以 <a router-link="/home">英文文档</a> 为准。
</p>

## 简介

`docute` 让你直接写 markdown 文件作为文档来显示而不需要编译成 html 这一步，你的文档目录里只需要一个首页 `index.html` 和你的配置文件 `config.js`。`docute` 会直接渲染这些 markdown 文件为一个单页应用。

## 安装

使用 `docute-cli` 来方便初始化一个文档。

用 npm 安装:

```bash
npm i -g docute-cli
```

用 Yarn 安装

```bash
yarn global add docute-cli
```

## 快速入门

假设你想将 markdown 文档存放在 `./docs` 目录:

```bash
docute init ./docs
```

这样就成功初始化了该目录，目前为止你的文档目录中有:

- README.md: 作为文档首页的内容显示
- index.html: 访问网站时渲染的 html
- config.js: 配置文件
- .nojekyll: 表明这不是一个 jekyll 网站，如果你不是要发布到 github pages 那么可以忽略它

然后你可以在本地预览文档:

```bash
docute ./docs
```

打开 http://localhost:8080 就能看到你的网站了。

### 手动开始

你可能并不喜欢 node.js 或 npm，也可能觉得安装它们很麻烦，其实你并不需要 `docute-cli`，我们唯一需要的只是一个 `index.html` 文件:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0" />
  <title>My Awesome Doc</title>
  <!-- docute 的 CSS -->
  <link rel="stylesheet" href="https://unpkg.com/docute/dist/docute.css">
</head>
<body>
  <div id="app"></div>
  <!-- 你也可以把包含 $config 配置的脚本放到单独的文件里 -->
  <!-- 比如 config.js -->
  <script>
    self.$config = {
      // 配置...
    }
  </script>
  <!-- 加载 docute 的 js 库 -->
  <script src="https://unpkg.com/docute/dist/docute.js"></script>
</body>
</html>
```

## 配置文件

执行 `docute init` 后你的文档目录会有一个 `config.js` 配置文件:

```js
self.$config = {
  // 配置...
}
```

### 首页

文档目录里的 `README.md` 文件会渲染为文档首页，但有时你可能会想使用其它文件。比如你将 `./docs` 目录里的文件作为 github pages 使用的时候你可能想直接使用项目根目录里的 `README.md`，你可以通过配置指向该文件:

```js
self.$config = {
  // 项目根目录里的 README.md
  home: 'https://raw.githubusercontent.com/egoist/docute/master/README.md'
}
```

### Landing 页面

你可以开启 `landing` 选项来启用 Landing 页面:

```js
self.$config = {
  // true 将会默认使用 `landing.html`
  landing: true,
  // 也可以用自定义页面
  landing: '_my-landing.html',
  // 当然 markdown 文件也可以
  landing: 'landing.md'
}
```

如果你开启了这个功能，Landing 页面的路由将是 `/`，而文档的首页将改为用 `/home` 访问。

<p class="warning">
  你不能在动态添加的页面里使用 <code>script</code> 标签，因为动态添加的内容 JS 不会被浏览器执行。
</p>

### 多页面

当然你也可以添加更多页面来显示不同内容。比如添加一个 `chinese.md` 用作文档的中文翻译，然后就能通过 `/#/chinese` 访问到这个页面了！

多页面支持子目录，比如一个文件被放到 `language/chinese.md` 那么它的访问路径就是 `/#/lanaguage/chinese`。

注意: 像 `about/README.md` 这样的文件对应的路由是 `/#/about/`， 而对于路由 `/#/about` 它的源文件是 `about.md`。

### 侧边栏

想默认隐藏侧边栏，可以在 `config.js` 中将其设置为 `false`:

```js
self.$config = {
  sidebar: false
}
```

或者只隐藏某个页面的侧边栏:

```markdown
---
sidebar: false
---
用 front-matter 隐藏这个页面的侧边栏
```

左下角也会有个按钮用于切换侧边栏，你也可以隐藏它:

```js
self.$config = {
  disableSidebarToggle: true
}
```

#### 目录表

侧边栏的目录表 (table of contents) 是从你的 markdown 文件解析来的，我们获取 markdown 里的 h2 ~ h5 标题来构成这个目录。

默认只会显示 h2 到 h4 的标题，其它标题只有在你页面滚动到相应区域才会显示，不过你可以更改最深显示的标题层级:

```js
self.$config = {
  // 显示 h2 到 h4 的标题
  tocVisibleDepth: 3
}
```

要彻底隐藏目录表把 `toc` 设置为 `false` 即可。

### 导航栏

当你有多个页面的时候你很可能需要一个导航栏方便用户浏览，你可以通过配置文件添加导航栏:

```js
self.$config = {
  nav: [
    // 首页
    {title: 'Home', path: '/'},
    // 中文文档
    {title: '中文文档', path: '/language/chinese'}
  ]
}
```

<p class="tip">
  `path` 也可以指向一个外部链接，那样它就会像一个普通的超链接一样工作。<br><br>
  除此之外你也可以用 `source` 来指定一个自定义的 markdown 文件而不是默认的 `路径加.md` 这种形式，比如 `source: 'https://foo.com/bar.md'`
</p>

#### Custom source

A path like `/language/chinese` will make docute fetch `/language/chinese.md`, you can use `source` option to fetch another file:

```js
{
  title: 'Chinese',
  path: '/language/chinese',
  source: '/language/chinese-foo.md'
  // or even external file
  source: 'https://raw.githubusercontent.com/user/repo/master/file.md'
}
```

<p class="tip">
  You may wonder why there's `$config.home` option when we already have `source` option, that's because `source` option is only available for nav item, while `$config.home` is always available no matter if you add `/` to nav.
</p>

#### 图标

##### Icon short-hand

```js
self.$config = {
  // slug for your github repo
  repo: 'tj/co',
  // twitter username
  twitter: 'realDonaldTrump',
  // the link to source file of current page
  'edit-link': 'https://github.com/egoist/docute/blob/master/docs'
}
```

To fully customize the `label`, `link` or use custom svg icons, see below.

##### Custom Icons

The built-in svg icons: `github` `twitter` `edit` `menu` `link` `search` `close`, which you can use in the `icon` attribute in `icons` option:

```js
self.$config = {
  icons: [{
    icon: 'github',
    label: 'Contribute on GitHub',
    link: 'https://github.com/owner/repo'
  }]
}
```

This example will have the same effect as using `repo: 'owner/repo'` but with custom label text here.

If you want to use your own SVG, you can set custom icon by providing the id of SVG symbol, [SVG symbols](https://css-tricks.com/svg-symbol-good-choice-icons/) is just a simple way to use inline SVG.

First, add it to your HTML file, which is `index.html` here:

```html
<body>
  <div id="app"></div>
  <!-- you can add it everywhere outside #app -->
  <svg xmlns="http://www.w3.org/2000/svg" style="display:none">
    <symbol id="my-icon"  viewBox="0 0 22 22">
      <!-- all shapes like `<path>` go here -->
    </symbol>
    <!-- ... other symbols -->
  </svg>
</body>
```

Then use it in `config.js`:

```js
self.$config = {
  icons: [
    {
      label: 'Hovered!', // the text for tooltip
      svgId: 'my-icon', // the id of symbol
      link: 'http://blah.blah'
    }
  ]
}
```

You can also add `svgClass` property to use CSS to control the style of your icon

```js
self.$config = {
  icons: [
    {
      label: 'Hovered!',
      svgId: 'my-icon',
      svgClass: 'my-icon-class',
      link: 'http://blah.blah'
    }
  ]
}
```

```css
/*
  To make consistent with default icon hover effect
  You can:
*/
.my-icon-class {
  fill: #ccc;
}
.my-icon-class:hover {
  fill: #333;
}
```

Check out [index.html](https://github.com/egoist/docute/blob/8aa85c42251aaa7298bd19562f961e59e781717c/docs/index.html#L20-L22) and [config.js](https://github.com/egoist/docute/blob/ab83d3a769b97e77887ccaf124168e88dc4c1c85/docs/config.js#L75-L80) of this doc to see how we add a custom icon for weibo.com (the last icon in the header).

There're many resources for good free SVG icons, for example: [bytesize-icons](https://github.com/danklammer/bytesize-icons) and [simple icons](https://simpleicons.org/).

##### 具名图标

你可以设置多套图标，然后给不同页面指定不同图标:

```js
self.$config = {
  icons: {
    default: [{label: 'hello'}],
    chinese: [{label: '你好'}]
  }
}
```

现在所有页面都会使用 `default` 这套图标，你可以通过 front-matter 指定页面使用其它图标，比如使用 `chinese`:

```markdown
---
icons: chinese
---
你好世界
```

#### 下拉菜单

显示一个下拉菜单以容纳多个页面:

```js
self.$config = {
  nav: [
    {title: '其他语言', type: 'dropdown', items: [
      {title: '中文', path: '/language/chinese'},
      {title: '日语', path: '/language/japanese'}
    ]}
  ]
}
```

##### matchPath

Type: `RegExp`

To make dropdown menu display the actual title of active page, for example, show `Chinese` instead of `Languages` as the dropdown title when user enters relevant page, use `matchPath`. The target of `matchPath` is `this.$route.path`,eg: in `https://example.com/en/get-started` the target is `/en/get-started`

```js
self.$config = {
  nav: [{
    title: 'Languages', type: 'dropdown', items: [{
      path: '/en',
      title: 'English',
      // show `English` instead of `Languages` as the dropdown title
      // only match `/en` and `/en/xxx` not `/enxxx`
      matchPath: /^\/en[\/$]/
    }]
  }]
}
```

If no macthed item was found, it uses the title of `dropdown` menu instead.

##### `label` and `sep`

To have such dropdown menu:

<img src="/assets/dropdown-label-sep.png" alt="label" width="300">

You will need the `label` and `sep` helper:

```js
self.$config = {
  nav: [
    {
      title: 'Ecosystem', type: 'dropdown', items: [
        {type: 'label', title: 'Help'},
        // ... items
        {type: 'sep'} // separator
        // ... other items
      ]
    }
  ]
}
```

#### 具名导航

你可能想要在不同的页面显示不同的导航栏，比如英文文档用默认导航单，然后中文文档用汉化了的导航栏。

当配置文件中的 `nav` 选项是一个数组的时候，它会被作为所有页面的默认导航栏，但是你也可以把它设置为一个纯对象 `{}` 来拥有多个导航栏。

```js
self.$config = {
  nav: {
    default: [{title: 'Home', path: '/'}],
    chinese: [{title: '首页', path: '/chinese'}]
  }
}
```

现在你拥有两个有各自名字的导航栏，目前为止所有页面还是只会使用 `default` 这个默认导航栏，你可以通过 markdown 文件里的 front-matter 来切换:

```markdown
---
nav: chinese
---
<!-- 现在这个页面的导航栏就是叫 chinese 的这个了 -->
你好世界！
```

### Markdown 配置

docute 使用 [marked](https://github.com/chjj/marked) 来解析 markdown 代码，你可以调整 marked 的默认参数:

```js
self.$config = {
  marked: {
    smartypants: true
    // ...
  }
}
```

完整的参数说明请参考[官方文档](https://github.com/chjj/marked#options-1)。

### debug

将 `debug` 设置为 `true` 来启用浏览器的 vue-devtools 插件:

```js
self.$config = {
  debug: true
}
```

## 指南

### 页面标题

每个页面在浏览器标签栏中显示的标题默认是配置文件中 `nav` 里你定义的 `title` 的值，这个 `title` 的值也会被用作导航栏中显示的文字。

不过对于浏览器标签栏，你也可以直接通过 markdown 文件里的 front-matter 来覆盖页面在标签栏中显示的标题:

```markdown
---
title: 首页
---