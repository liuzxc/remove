---
layout: post
title:  Rails 应用开发笔记（七）
excerpt: 支持 Markdown 语法和代码高亮
comments: true
share: true
categories: articles
---

每当创建文章内容的时候，文本是以纯文本的形式被保存，不支持任何排版格式和语法，连换行符都被忽略了，
这显然是无法接受的，尤其是对于撰写技术类的文章而言，支持 Markdown 语法和代码高亮是非常有必要的。如果
是自己来开发这些功能的话显然工作量太大，幸运的是我们有强大的gem包可以使用，本次将介绍两个 gem 包的使用，
可以帮助我们快速实现该功能。

#### Redcarpet 和 coderay

> [Redcarpet for GitHub](https://github.com/vmg/redcarpet)

> [coderay for GitHub](https://github.com/rubychan/coderay)

Redcarpet 是一个 Markdown 的解析器，功能强大，但是官方的文档太简略而且不好懂，也许是我的英文太差了😓！

[Coderay](http://coderay.rubychan.de) 是用 ruby 写的语法高亮处理工具，支持多种编程语言。

在 Gemfile 中添加 redcarpet 和 coderay 包，然后运行 bundle

{% highlight ruby %}
gem 'redcarpet' #markdown 解析器
gem 'coderay'   #代码高亮
{% endhighlight %}

由于没有看懂原文的使用指南，本着万事不会问 Google 的指导思想，果断搜出了两篇指南（感谢美帝人民的分享精神😄）

> http://crabonature.pl/posts/16-markdown-in-ruby-on-rails
> http://allfuzzy.tumblr.com/post/27314404412/markdown-and-code-syntax-highlighting-in-ruby-on

以上两篇文章都是将文本的处理封装成了 helper 方法，目的是方便使用，不用在每一个显示的地方做改动，只需要调用固定的方法即可。

{% highlight ruby %}
#app/helper/application_helper.rb
module ApplicationHelper

  class CodeRayify < Redcarpet::Render::HTML
    def block_code(code, language)
      #language ||= :plaintext
      CodeRay.scan(code, language).div
    end
  end

  def markdown(text)
    coderayified = CodeRayify.new(:filter_html => true,
                                  :hard_wrap => true)
    render_options = {
      # will remove from the output HTML tags inputted by user
      filter_html:     true,
      # will insert <br /> tags in paragraphs where are newlines
      # (ignored by default)
      hard_wrap:       true,
      # hash for extra link options, for example 'nofollow'
      link_attributes: { rel: 'nofollow' }
      # more
      # will remove <img> tags from output
      # no_images: true
      # will remove <a> tags from output
      # no_links: true
      # will remove <style> tags from output
      # no_styles: true
      # generate links for only safe protocols
      # safe_links_only: true
      # and more ... (prettify, with_toc_data, xhtml)
    }
    renderer = Redcarpet::Render::HTML.new(render_options)

    extensions = {
      #will parse links without need of enclosing them
      autolink:           true,
      # blocks delimited with 3 ` or ~ will be considered as code block.
      # No need to indent.  You can provide language name too.
      # ```ruby
      # block of code
      # ```
      fenced_code_blocks: true,
      # will ignore standard require for empty lines surrounding HTML blocks
      lax_spacing:        true,
      # will not generate emphasis inside of words, for example no_emph_no
      no_intra_emphasis:  true,
      # will parse strikethrough from ~~, for example: ~~bad~~
      strikethrough:      true,
      # will parse superscript after ^, you can wrap superscript in ()
      superscript:        true
      # will require a space after # in defining headers
      # space_after_headers: true
    }
    Redcarpet::Markdown.new(coderayified, extensions).render(text).html_safe
  end
end
{% endhighlight %}

然后在相应的视图代码中添加 markdown 方法：

{% highlight erb %}
...
<div class="panel-body">
  <p><%= markdown(@article.content) %></p>
</div>
...
{% endhighlight %}

现在创建文章后的效果是不是超赞呢😄！

<figure>
    <img src="/images/20150825-01.png">
</figure>

原作者中的代码存在一个问题，当代码块中没有制定语言的时候就会出错，导致文章创建失败，例如文本中出现这样的代码：

{% highlight ruby %}
```
def test
end
```
{% endhighlight %}

保存的时候就会出错：

<figure>
    <img src="/images/20150825-02.png">
</figure>

解决的方案就是当没有制定语言的时候，设置一个默认的语言：

{% highlight ruby %}
class CodeRayify < Redcarpet::Render::HTML
  def block_code(code, language)
    language ||= :plaintext #设置默认语言防止出错
    CodeRay.scan(code, language).div
  end
end
{% endhighlight %}

