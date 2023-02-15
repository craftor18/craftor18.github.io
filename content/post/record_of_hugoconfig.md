---
title: "小记hugo中的一些配置问题"
date: 2023-02-06T19:46:12+08:00
categories : [
  "学习",
]
tags : [
  "hugo",
]
typora-root-url: ./
---
# 小记hugo中的一些配置问题
## 1.如何将首页的logo从Even改成自己的名字
在`config.toml`配置文件中找到`logoTitle = "yourname"`这一行，将其修改成你想要的名字。当时我还以为那个东西要改icon之类的，没想到哇就在配置文件里改，我真是个小可爱捏。
## 2.添加`utterances`评论系统
先在github自己的仓库里面安装`utterances`应用，再在`config.toml`配置文件里找到`[params.utterances]`这几行，改成：
```go
  [params.utterances]
    owner = "yourid"        
    repo = "yourrepo"        
```
改完push到github之后发现评论系统一点击要登陆到github就错误，结果发现是`config.toml`配置文件里`baseURL = ""`这里还是个本地url所以跳转会错误，改成自己的`github.io`那个就行了比如`baseURL = "https://craftor18.github.io/"`。
## 3.将原本花里胡哨各种social关掉并改一下footer
原本不是下面一堆社交链接吗，如何改成下图这样捏？
![footer](/img/footer.png)
其实就是将`themes/even/layouts/partials/post/footer.html`文件中
```html
<!-- <div class="social-links">
  {{- range $name, $path := .Site.Params.social }}
    {{- if $path }}
      {{- $realName := slicestr $name 2 }}
      <a href="{{ $path | safeURL }}" class="iconfont icon-{{ $realName }}" title="{{ $realName }}"></a>
    {{- end }}
  {{- end }}
  {{ if .Site.LanguagePrefix -}}
    <a href="{{ .Site.LanguagePrefix | absURL }}/index.xml" type="application/rss+xml" class="iconfont icon-rss" title="rss"></a>
  {{- else -}}
    <a href="{{ .Site.RSSLink }}" type="application/rss+xml" class="iconfont icon-rss" title="rss"></a>
  {{- end }}
</div> -->
```
上面这一堆代码块注释掉，就行了（虽然我已经注释了，嘿嘿），然后在最后一个`</div>`的上一行添加
```html
  <span class="copyright-and-others">Copyright &copy; 2023 <a href="https://github.com/craftor18" style="color:#c05b4d">Craftor18</a></span>. All Rights Reserved
```
ps：可以改一下年份和链接哟

总之，以上是我在配置自己的even主题中遇到的一些问题，还有许多不足的辣，也参考了很多大佬的教程，附上链接：

[从零开始搭建个人博客网站系列 一、使用hugo创建个人站点 - 码农在新加坡的个人博客 (leftpocket.cn)](https://www.leftpocket.cn/post/hugo/hugo_creation/)

[Hugo 静态网站图片插入问题 - Don't Judge (dontjudge.cn)](https://blog.dontjudge.cn/post/hugo-静态网站图片插入/)

[Hugo + Github Pages 搭建Blog - Don't Judge (dontjudge.cn)](https://blog.dontjudge.cn/post/hugo-+-github-pages-搭建blog/)

