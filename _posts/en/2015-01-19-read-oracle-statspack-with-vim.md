---
layout: post
title:  "read Oracle statspack with vim"
date:   2015-01-19 23:51:27
name: vim-oracle-statpack
---

Vim syntax to highlight Oracle statpack files.

/usr/share/vim/vim72/syntax/statspack.vim:

{% highlight vim %}
" Quit when a (custom) syntax file was already loaded
if exists("b:current_syntax")
  finish
endif

syn match       title           "^
                                  .*"
syn match       line            "---*"
"syn match      line2           "\~*"

hi def link title Comment
hi def link line Comment
"hi def link line2 Comment

let b:current_syntax = "statspack"
{% endhighlight %}

Result:

![vim statpack syntax screenshot](/pub/vim-statpack.png)
