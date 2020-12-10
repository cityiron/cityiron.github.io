# hugo 白话文 | 添加友情链接


本文介绍如何在 hugo 中添加友情链接

<!--more-->

## 一、前言

使用 [Hugo](https://gohugo.io/) 搭建博客的你是否还在为没有友情链接而烦恼，阅读本文将为你制作属于自己的友情链接提供解决方案。

## 二、友情链接

### 2.1 效果

先看咋们的效果图

真实效果：

{{< friend name="Z字骚年" url="https://zouyx.github.io/" logo="https://avatars.githubusercontent.com/u/3828072?v=4&s=160" word="Joe Zou's Blog" >}}

首页：

![yqlj](https://raw.githubusercontent.com/cityiron/tuchuang/master/blog/yqlj.jpg)

点击后：

![htl.yqlj.02](https://raw.githubusercontent.com/cityiron/tuchuang/master/blog/htl.yqlj.02.jpg)

手机端：

![htl.yqlj.03](https://raw.githubusercontent.com/cityiron/tuchuang/master/blog/htl.yqlj.03.jpg)

### 2.2 制作过程

#### 2.2.1 增加菜单栏

```toml
[languages]
  [languages.en]
    ......
    [languages.en.menu]
      ......
      [[languages.en.menu.main]]
        identifier = "friend"
        pre = ""
        post = ""
        name = "Friend"
        url = "/friend/"
        title = ""
        weight = 5
      ......
    [languages.en.params]
      ......
```

首先在 `menu` 增加一个菜单项为 `friend` 表示友情链接，上面是多语言的版本，不是多语言也是一样的逻辑。

#### 2.2.2 制作 shortcodes

##### 2.2.2.1 创建 html

在你博客的路径 `/xxx/blog/layouts/shortcodes/` 下创建 `friend.html` 文件。

```html
{{ if .IsNamedParams }}
    {{- $src := .Get "logo" -}}
    {{- $small := .Get "logo_small" | default $src -}}
    {{- $large := .Get "logo_large" | default $src -}}
    <div class="flink" id="article-container">
      <div class="friend-list-div" >
        <div class="friend-div">
            <a target="_blank" href={{ .Get "url"  | safeURL }} title={{ .Get "name" }} >
                <img class="lazyload"
                     src="/svg/loading.min.svg"
                     data-src={{ $src | safeURL }}
                     alt={{ .Get "name" }}
                     data-sizes="auto"
                     data-srcset="{{ $small | safeURL }}, {{ $src | safeURL }} 1.5x, {{ $large | safeURL }} 2x" />
                <span class="friend-name">{{ .Get "name" }}</span>
                <span class="friend-info">{{ .Get "word" }}</span>
            </a>
        </div>
      </div>
    </div>
{{ end }}
```

- .Get 是 hugo shortcodes 的用法，可以获取到属性

##### 2.2.2.2 创建 scss

> 样式基本上就是拿来用了，没改过

- `assets/css/_partial/_single/` 下面新建 `_friend.scss` 文件

```scss
#article-container {
 word-wrap: break-word;
 overflow-wrap: break-word
}

#article-container a {
 color: #49b1f5
}

#article-container a:hover {
 text-decoration: underline
}

#article-container img {
 margin: 0 auto .8rem
}

.flink#article-container .friend-list-div > .friend-div a .friend-info,
.flink#article-container .friend-list-div > .friend-div a .friend-name {
 overflow: hidden;
 -o-text-overflow: ellipsis;
 text-overflow: ellipsis;
 white-space: nowrap
}

.flink#article-container .friend-list-div {
 overflow: auto;
 padding: 10px 10px 0;
 text-align: center;
}
.flink#article-container .friend-list-div > .friend-div {
 position: relative;
 float: left;
 overflow: hidden;
 margin: 15px 7px;
 width: calc(100% / 3 - 15px);
 height: 90px;
 border-radius: 8px;
 line-height: 17px;
 -webkit-transform: translateZ(0)
}

@media screen and (max-width: 1100px) {
 .flink#article-container .friend-list-div > .friend-div {
  width: calc(50% - 15px) !important
 }

@media screen and (max-width: 600px) {
 .flink#article-container .friend-list-div > .friend-div {
  width: calc(100% - 15px) !important
 }
}
}

.flink#article-container .friend-list-div > .friend-div:hover {
 background: rgba(87, 142, 224, 0.15);
}

.flink#article-container .friend-list-div > .friend-div:hover img {
 -webkit-transform: rotate(360deg);
 -moz-transform: rotate(360deg);
 -o-transform: rotate(360deg);
 -ms-transform: rotate(360deg);
 transform: rotate(360deg)
}

.flink#article-container .friend-list-div > .friend-div:before {
 position: absolute;
 top: 0;
 right: 0;
 bottom: 0;
 left: 0;
 z-index: -1;
 background: var(--text-bg-hover);
 content: '';
 -webkit-transition: -webkit-transform .3s ease-out;
 -moz-transition: -moz-transform .3s ease-out;
 -o-transition: -o-transform .3s ease-out;
 -ms-transition: -ms-transform .3s ease-out;
 transition: transform .3s ease-out;
 -webkit-transform: scale(0);
 -moz-transform: scale(0);
 -o-transform: scale(0);
 -ms-transform: scale(0);
 transform: scale(0)
}
.flink#article-container .friend-list-div > .friend-div:hover:before,
.flink#article-container .friend-list-div > .friend-div:focus:before,
.flink#article-container .friend-list-div > .friend-div:active:before {
 -webkit-transform: scale(1);
 -moz-transform: scale(1);
 -o-transform: scale(1);
 -ms-transform: scale(1);
 transform: scale(1)
}
.flink#article-container .friend-list-div > .friend-div a {
 color: var(--font-color);
 text-decoration: none
}

.flink#article-container .friend-list-div > .friend-div a img{
 float: left;
 margin: 15px 10px;
 width: 60px;
 height: 60px;
 border-radius: 35px;
 -webkit-transition: all .3s;
 -moz-transition: all .3s;
 -o-transition: all .3s;
 -ms-transition: all .3s;
 transition: all .3s
}

.flink#article-container .friend-list-div > .friend-div a .friend-name {
 display: block;
 padding: 16px 10px 0 0;
 height: 40px;
 font-weight: 700;
 font-size: 20px
}

.flink#article-container .friend-list-div > .friend-div a .friend-info {
 display: block;
 padding: 1px 10px 1px 0;
 height: 50px;
 font-size: 13px
}
```

- `assets/css/_page/` 下面新建 `_single.scss` 文件

> 把主题  `/themes/loveIt/assets/css/_page/_single.scss` 的文件拷贝到博客项目指定目录，关键在于添加一行 `@import "../_partial/_single/_friend";`

```scss
@import "../_partial/_single/toc";
@import "../_partial/_single/_friend";

.single {
  .single-title {
    margin: 1rem 0 .5rem;
    font-size: 1.6rem;
    font-weight: bold;
    line-height: 140%;
  }

  .single-subtitle {
    margin: .4rem 0;
    font-size: 1.2rem;
    font-weight: normal;
    font-style: italic;
    line-height: 100%;
  }

  .post-meta {
    font-size: .875rem;
    color: $global-font-secondary-color;

    span {
      display: inline-block;
    }

    [theme=dark] & {
      color: $global-font-secondary-color-dark;
    }

    @include link(false, true);

    .author {
      font-size: 1.05rem;
    }
  }

  .featured-image {
    margin: .5rem 0 1rem 0;

    img {
      display: block;
      max-width: 100%;
      height: auto;
      margin: 0 auto;
      overflow: hidden;
    }

    img.lazyloaded {
      width: 100%;
    }
  }

  .content {
    > h2 {
      font-size: 1.5rem;

      & code {
        font-size: 1.25rem;
      }
    }

    > h3 {
      font-size: 1.375rem;

      & code {
        font-size: 1.125rem;
      }
    }

    > h4 {
      font-size: 1.25rem;

      & code {
        font-size: 1rem;
      }
    }

    > h5 {
      font-size: 1.125rem;
    }

    > h6 {
      font-size: 1rem;
    }

    h2,
    h3,
    h4,
    h5,
    h6 {
      font-weight: bold;
      margin: 1.2rem 0;

      [theme=dark] & {
        font-weight: bolder;
      }
    }

    > h2,
    > h3,
    > h4,
    > h5,
    > h6 {
      > .header-mark::before {
        content: "|";
        margin-right: .3125rem;
        color: $single-link-color;

        [theme=dark] & {
          color: $single-link-color-dark;
        }
      }
    }

    > h2 > .header-mark::before {
      content: "#";
    }

    p {
      margin: .5rem 0;
    }

    b, strong {
      font-weight: bold;

      [theme=dark] & {
        color: #ddd;
      }
    }

    @include link(false, false);

    a {
      @include overflow-wrap(break-word);

      [theme=dark] & b, [theme=dark] & strong {
        color: $single-link-color-dark;
      }
    }

    [theme=dark] a:hover b, [theme=dark] a:hover strong {
      color: $single-link-hover-color-dark;
    }

    ul, ol {
      margin: .5rem 0;
      padding-left: 2.5rem;
    }

    ul {
      list-style-type: disc;
    }

    ruby {
      background: $code-background-color;

      rt {
        color: $global-font-secondary-color;
      }

      [theme=dark] & {
        background: $code-background-color-dark;

        rt {
          color: $global-font-secondary-color-dark;
        }
      }
    }

    .table-wrapper {
      overflow-x: auto;

      &::-webkit-scrollbar {
        background-color: $table-background-color;

        [theme=dark] & {
          background-color: $table-background-color-dark;
        }
      }

      > table {
        width: 100%;
        max-width: 100%;
        margin: .625rem 0;
        border-spacing: 0;
        background: $table-background-color;
        border-collapse: collapse;

        [theme=dark] & {
          background: $table-background-color-dark;
        }

        thead {
          background: $table-thead-color;

          [theme=dark] & {
            background-color: $table-thead-color-dark;
          }
        }

        th, td {
          padding: .3rem 1rem;
          border: 1px solid darken($table-thead-color, 2%);

          [theme=dark] & {
            border-color: darken($table-thead-color-dark, 2%);
          }
        }
      }
    }

    img {
      max-width: 100%;
      min-height: 1em;
    }

    figure {
      margin: .5rem;
      text-align: center;

      .image-caption:not(:empty) {
        min-width: 20%;
        max-width: 80%;
        display: inline-block;
        padding: .5rem;
        margin: 0 auto;
        font-size: .875rem;
        color: #969696;
      }

      img {
        display: block;
        height: auto;
        margin: 0 auto;
        overflow: hidden;
      }
    }

    .lazyloading {
      @include object-fit(none);
    }

    blockquote {
      display: block;
      border-left: .5rem solid $blockquote-color;
      background-color: rgba($blockquote-color, .2);
      padding: .25rem .75rem;
      margin: 1rem 0;

      [theme=dark] & {
        border-left-color: $blockquote-color-dark;
        background-color: rgba($blockquote-color-dark, .2);
      }
    }

    .footnotes {
      color: $global-font-secondary-color;

      [theme=dark] & {
        color: $global-font-secondary-color-dark;
      }

      p {
        margin: .25rem 0;
      }
    }

    @import "../_partial/_single/code";
    @import "../_partial/_single/instagram";
    @import "../_partial/_single/admonition";
    @import "../_partial/_single/echarts";
    @import "../_partial/_single/mapbox";
    @import "../_partial/_single/music";
    @import "../_partial/_single/bilibili";

    hr {
      margin: 1rem 0;
      position: relative;
      border-top: 1px dashed $global-border-color;
      border-bottom: none;

      [theme=dark] & {
        border-top: 1px dashed $global-border-color-dark;
      }
    }

    kbd {
      display: inline-block;
      padding: .25rem;
      background-color: $global-background-color;
      border: 1px solid $global-border-color;
      border-bottom-color: $global-border-color;
      @include border-radius(3px);
      @include box-shadow(inset 0 -1px 0 $global-border-color);
      font-size: .8rem;
      font-family: $code-font-family;
      color: $code-color;

      [theme=dark] & {
        background-color: $global-background-color-dark;
        border: 1px solid $global-border-color-dark;
        border-bottom-color: $global-border-color-dark;
        @include box-shadow(inset 0 -1px 0 $global-border-color-dark);
        color: $code-color-dark;
      }
    }

    .typeit {
      .code {
        padding: .375rem;
        font-size: .875rem;
        font-family: $code-font-family;
        font-weight: bold;
        word-break: break-all;
      }
    }

    .version {
      height: 1.25em;
      vertical-align: text-bottom;
    }
  }

  @import "../_partial/_single/footer";
  @import "../_partial/_single/comment";
}

.lg-toolbar .lg-icon::after {
  color: #999;
}
```

### 2.3 添加内容

在 `content/friend` 下新建 `index.语言.md`

```md
---
hiddenFromSearch: true
---
{{</* friend name="Z字骚年" url="https://zouyx.github.io/" logo="https://avatars.githubusercontent.com/u/3828072?v=4&s=160" word="Joe Zou's Blog" */>}}
{{</* friend name="张其" url="https://blog.csdn.net/u010927340" logo="https://avatars3.githubusercontent.com/u/12484497?v=4&s=160" word="张其的博客" */>}}
```

- 头部 `---`   是必须的，如果直接加内容会出现如下错误：

```bash
ERROR 2020/12/08 19:16:14 Rebuild failed:                                                     │@1154] - Unable to reconnect to ZooKeeper service, session 0x10001c32b1e001e has expired, clos                                                                                              │ing socket connection
ERROR 2020/12/08 19:16:14 "/Users/tc/Documents/workspace_2020/blog/content/friend/index.tc.md:│2020-12-08 01:13:40,558 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventT
1:1": unmarshal failed: invalid character '{' looking for beginning of object key string
```

[解决方案](https://github.com/gohugoio/hugo/issues/4066)

- friend 介绍

参数都是在 html 中通过 .Get 获取的，比较简单直观，直接看 创建 html 部分即可。

## 三、参考

https://gohugo.io/templates/shortcode-templates/

https://github.com/kkkgo/hugo-friendlinks/

https://blog.233so.com/2020/04/friend-link-shortcodes-for-hugo-loveit/

