---
layout: post
title:  Rails 应用开发笔记（十五）
excerpt: 使用 jQuery 实现自动补全功能
comments: true
share: true
categories: articles
---

最近一段时间在忙着找工作，很久没有更新了，之前做了一个自动补全的功能，花费了我很多时间，因为涉及到很多前端开发相关的知识，包括DOM, js, coffeescript, jquery等等，现在打算详细的描述一下了。很多网站都有用户名或者输入内容自动补全的功能，我实现的是比较简单的一种，即评论区用户名自动补全，在 @ 其他评论人的时候，文本框会自动筛选出所有评论着的名字。

很显然，该功能主要依靠前端相关技术，即 javascript，而我本身并不了解这门语言，所以我选用了语法更简单的 coffeescript。coffeescript 相当于一种简化版的 js，它可以完全被编译成 js 代码，它的语法很简单，有点类似 ruby，如果要尽快上手，coffeescript 是个不错的选择，另外我们还要依赖一些现成的库来帮助我们减少工作量，比如 jQuery。jQuery 是 js 的一个库，它实现了很多很酷的功能和插件，很方便的供大家调用，而我此次主要使用的就是 jQuery 的 Autocomplete 插件。

> [coffeescript 官网](http://coffeescript.org/)

#### 需求分析

开始做之前，需要做好需求分析，我们实现的功能是做什么的，要怎么去实现，用什么技术，都要考虑好。如下图所示，我希望实现一个类似于 stackoverflow 评论栏里的@功能。

<figure>
  <img src="http://i.stack.imgur.com/75plH.png">
</figure>

用户每次在评论栏输入 @ 的时候，自动出现一个已经评论过当前问题的用户名列表，用户可以选择想要 @ 的人，而不必手动输入名称。如果我要在我的网站实现这个功能，那么现在存在几个问题：

1. 如何获取用户名列表？

有两种方法可以做到，一是到服务器去查询当前文章下所有评论过的用户名列表，这个是很容易做到的，但是效率低，影响性能；二是在 HTML 页面中筛选出用户名，这样做是比较科学的，不需要和服务器交互。我暂时采用第一种，因为第二种需要一些js的知识，目前我还不熟悉。

2. 如何判断用户输入了 @ 符号？

只有在用户输入了 @ 符号的前提下，系统才会显示用户列表，所以需要随时判断用户的输入内容，显然这个必须依赖于 js。


#### Autocomplete

Autocomplete 可以帮助我减少工作量，但是我需要了解它是如何工作的，因此必须好好看下文档。

> [jqueryui Autocomplete](http://jqueryui.com/autocomplete/)

以下是取自官方的示例代码：

{% highlight coffeescript %}
<script>
  $(function() {
    var availableTags = [
      "ActionScript",
      "AppleScript",
      "Asp",
      "BASIC",
      "C",
      "C++",
      .....
    ];
    $( "#tags" ).autocomplete({
      source: availableTags
    });
  });
</script>
{% endhighlight %}

此段代码的意思是：在网页页面加载完毕的时候，jQuery 告诉 DOM 替他做一些事情，即 function() 里面需要做的。
DOM 会去找一个叫 tags 的ID选择器，然后去执行自动补全，source 即是需要显示的列表内容。

然后对应到我的需求，我需要给评论栏加上 tags 标签，source 对应的值是评论过的用户名列表，但是这个列表是未知的，我需要去服务器动态获取，幸运的是，autocomplete 的 source属性支持多种类型，一种是 Array，就像上面的 availableTags；一种是 String，说是 string，其实是一个 url 地址，jQuery 会根据这个地址发送一个 get 请求道服务器，然后返回一个 json  格式的数据，显然这种方式可以很好的满足我的需求。

{% highlight erb %}
#app/view/comments/_form.html.erb
<div class="form-group ui-widget">
  <div class="col-sm-5">
    <%= f.text_area :content, rows: 5, placeholder: '说点什么...', class: 'form-control', id: "tags", 'data-article-id': @article.id.to_s %>
  </div>
</div>
{% endhighlight %}

为评论栏添加 ID选择器 tags，以便 js 找到它，data-article-id 的作用是为了让 js 获取文章的 id，便于后面构造 url。

{% highlight coffeescript %}
#app/assets/javascripts/comments.coffee
# input: it's an event that triggers whenever the input changes
$ ->
  $(document).on("input", "#tags", ->
    id = $("#tags").data("article-id")
    $("#tags").autocomplete
      source:  '/articles/' + id + '/autocomplete.json')
{% endhighlight %}

此段 coffee 代码的意思是：在页面加载完成后，js 解释器会监听ID选择器为 tags 上的输入事件，一旦有输入，则获取文章id，启动自动补全功能，根据 source 中的 url 地址，发送一个 ajax 请求到服务器去获取用户名列表。

{% highlight ruby %}
#config/routes.rb
resources :articles, only: [] do
  resources :comments
  get :autocomplete
end
{% endhighlight %}

添加路由

{% highlight ruby %}
class ArticlesController < ApplicationController
  ....
  def autocomplete
    @articles = Article.find_by(id: params[:article_id])
    term = params[:term].split(/@(\w+)$/).last
    @commentor_names = @articles.comments.map{|e| e.name if e.name.match(/^#{term}/i) }.compact.uniq
    respond_to do |format|
      format.html
      format.json {
        render json: @commentor_names
      }
    end
  end
  ....
end
{% endhighlight %}

autocomplete 方法用于在服务器查找匹配的用户名，jQuery 发送 ajax 消息的时候，会附带一个 term 的参数，内容即为当前输入框的文本，由于我希望用户在输入 @ 符号的时候进行匹配，所以在此处我需要对参数进行处理。

{% highlight ruby %}
Started GET "/articles/55fc103ebd172df3f600000c/autocomplete.json?term=I+love+%40liu" for ::1 at 2015-10-25 00:38:06 +0800
Processing by ArticlesController#autocomplete as JSON
  Parameters: {"term"=>"I love @liu", "article_id"=>"55fc103ebd172df3f600000c"}
{% endhighlight %}

现在可以看一下效果如何：

<figure>
  <img src="http://zippy.gfycat.com/PepperyReasonableBunting.gif">
</figure>

看起来还不错，但是存在两个问题：

1. 虽然成功的匹配了输入内容，但是从列表中选取了用户名之后，文本框中的内容被替换了，而不是追加，这显然是错误的。

2. 在没有输入 @ 符号的情况下，也出现了选择列表。

要解决第一个问题，我们需要为 autocomplete 添加几个新的属性：

{% highlight coffeescript %}
$ ->
  $(document).on("input", "#tags", ->
    id = $("#tags").data("article-id")
    $("#tags").autocomplete
      source:  '/articles/' + id + '/autocomplete.json'
      minLength: 1  #在输入的字符串长度为1的情况下才触发请求
      focus: (event) -> # event在焦点被移动到条目中时被触发，默认的行为是用列表栏中聚焦项目的值取代文本框中的值
        event.preventDefault() #阻止默认的行为被触发
      select: (event, ui) ->   #event在列表中的条目被选中时触发，默认的行为是用列表栏中选中项目的值取代文本框中的值
        event.preventDefault()
        #将列表中的内容追加在@符号末尾，避免覆盖文本框中的内容
        this.value = this.value.replace(/@(\w*)$/, "@" + ui.item.value))
{% endhighlight %}

对于第二个问题，我们需要判断文本框中的内容，如果没有输入 @ 符号，就不触发自动补全的功能：

{% highlight coffeescript %}
$ ->
  $(document).on("input", "#tags", ->
    content = $("#tags").val() #获取当前输入框中的文本
    if content[content.length - 1] == "@" #判断文本的最后一个字符串是否是@，只有是@的情况下才触发自动补全
      id = $("#tags").data("article-id")
      $("#tags").autocomplete
        source:  '/articles/' + id + '/autocomplete.json'
        minLength: 1
        focus: (event) -> # event在焦点被移动到条目中时被触发，默认的行为是用列表栏中聚焦项目的值取代文本框中的值
          event.preventDefault() #阻止默认的行为被触发
        select: (event, ui) ->   #event在列表中的条目被选中时触发，默认的行为是用列表栏中选中项目的值取代文本框中的值
          event.preventDefault()
          this.value = this.value.replace(/@(\w*)$/, "@" + ui.item.value))
{% endhighlight %}

在修复了这两个问题以后，再来看看效果：

<figure>
  <img src="http://zippy.gfycat.com/ForthrightCriminalBushbaby.gif">
</figure>

看起来还不错，基本上已经实现了该功能，但是每次输入的时候都会触发一次服务器请求，不仅效率差，还可能导致性能问题，因此在这里需要做改进。正如我开始讲到的，如果从页面筛选出用户名，就不需要和服务器进行交互了，会减轻服务器的负担。这一块的改进我还没有完成，等优化之后会再进行讲解。