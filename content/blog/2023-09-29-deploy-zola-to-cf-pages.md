+++
title="将 Zola 部署到 Cloudflare Pages"
[extra]
author = "改改"
[taxonomies]
tags = ["Cloudflare", "Zola"]
+++
测试一下workflow

测试一下deploy hook
买了一个域名之后想认真建个blog，于是看了看选了zola。

结果部署到cf pages的时候掉坑了，改了，用了cloudflare自己的 ~~pages-action~~ wrangler-action

~~他妈的cloudflare自带的zola版本太低了，就只能用 github actions 去build，然后用pages的deploy hook通知cf去部署，这时候记得把自动部署关了。然后deploy URL写到secret里面，别用明文。~~

具体代码在下面的文件里。


<https://github.com/HeilAsuka/Blog/blob/main/.github/workflows/build%20and%20deploy.yaml>

```YAML
name: Build Zola Website

on:
  push:

jobs:
  build-webiste:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4.1.0
        with:
          fetch-depth: 2
          submodules: true
      - name: Get changes
        run: git diff --name-only -r HEAD^1 HEAD
      - name: "Install Zola"
        id: install-zola
        uses: taiki-e/install-action@v2
        with:
          #tool: zola@0.14.0 旧版本会报错
          tool:zola #我就改用新版本了
      - name: "Build Website"
        id: build-website
        run: "zola build"
      - name: Deploy
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          packageManager: npm
          command: pages deploy ./public --project-name=blog
```

今天又把Waline加上了，用的是 `shortcodes`,文件链接在下面
<https://github.com/HeilAsuka/Blog/blob/main/templates/shortcodes/comment.html>
具体用法是这样的`{{/*comment(text='想说啥就说，憋着不好。')*/}}`

```HTML
<div>{% if text %}{{text}}{% endif %}</div>
<link rel="stylesheet" href="https://unpkg.com/@waline/client@v2/dist/waline.css" />
<div id="waline"></div>
<script type="module">
    import { init } from 'https://unpkg.com/@waline/client@v2/dist/waline.mjs';

    init({
        el: '#waline',
        serverURL: 'https://comment.iamonyou.top',
        dark: 'html.dark',
        imageUploader: false,
    });
</script>
```

{{ comment(text='想说啥就说，憋着不好。') }}
