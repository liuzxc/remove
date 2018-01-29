---
layout: post
title: 10个最常见的 Rails 编程错误（译文）
excerpt: Rails 虽然易于使用，但也很容易被滥用。本文聚焦于10个最常见的 Rails 陷进，并指导你如何避免它们以及由它们引发的问题
comments: true
share: true
categories: articles
---

本文由 Jasonliu 翻译自 [BRIAN VANLOO](http://www.toptal.com/ruby-on-rails/top-10-mistakes-that-rails-programmers-make)。


Ruby on Rails是一个非常流行的开源web框架，它基于ruby语言，致力于精简web开发流程。

Rails基于“约定优于配置“（[convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration)）的原则，
简而言之，在默认情况下，Rails假设你遵循了“标准”的约定（像命名规则，代码结构等等)，如果你这样做了，很多事情会自动完成而不需要关心细节。
这样做虽然有优点，但同时也存在一些陷阱。最显著的问题就是：这种隐藏在框架之后的“魔法”通常会带来困惑，你经常会问：为什么会这样？同时，
它也带来了安全和性能问题。

因此，Rails虽然易于使用，但也很容易被滥用。本文聚焦于10个最常见的rails陷进，并指导你如何避免它们以及由它们引发的问题。

<figure>
  <img src="/images/rails_mistakes.png" alt="image">
</figure>

#### 常见错误1: 将太多的逻辑放在controller

Rails基于`MVC`架构。 在Rails社区里，一直以来我们谈论的都是`fat model,skinny controller`，然而在最近的几个rails项目中，我打破了这个原则，
因为将视图层逻辑或模型层逻辑放入控制器中实在是轻而易举。

问题是在于，一旦控制器对象违反单一责任原则，未来修改代码将变得困难且容易出错。总的来说，放在controller中的逻辑应该有：

* `Session` 和 `cookie` 处理。 这可能包括`authentication/authorization`或任何你需要的cookie操作；
* 选择model。 根据请求中所带的参数找到正确的model对象。理想情况下,调用用一个带有实例变量的find方法，然后响应；
* 请求参数管理。 收集请求参数，并且调用一个合适的model方法去持久化它们；
* 渲染结果(html、xml、json等)或重定向

虽然单一责任原则增加了局限性，但这是rails框架要求在控制器中所做的最低限度的事情。

> 个人认为，将逻辑放在controller最大的坏处是不利于测试。模型层的测试用例是最好写的，你只需要关心你构造的数据和逻辑代码产生的结果是否一致，为控制器写测试用例很容易受其他因素的干扰，比如路由规则，测试结果依赖控制层的方法和请求的参数等等。所以有很多的rails开源项目并没有为控制器逻辑写测试用例，而且控制器中的代码应该精简到不需要写测试用例。

#### 常见错误2: 将太多的逻辑放在view

Rails模版引擎－`ERB`是构建可变内容页面的良好方式。可是如果你不够细心的话，你将很快被一大堆难以管理和维护的，并且糅合了大量`html`和`ruby`代码的文件所
包围，这也是导致大量代码重复和违背`DRY`原则的地方。在视图层中过度使用条件逻辑就是违背该原则的其中之一，一个简单的例子:

`current_user`方法可以返回当前登录的用户，通常情况下，代码中会出现这样的条件逻辑结构：

{% highlight ruby linenos %}
<h3>
  Welcome,
  <% if current_user %>
    <%= current_user.name %>
  <% else %>
    Guest
  <% end %>
</h3>
{% endhighlight %}

其实有更好的方法来处理这样的问题，那就是无论用户是否登录，都确保`current_user`返回的对象总是存在。例如，你也许在`app/controllers/application_controller.rb`定义了一个`current_user`帮助函数：

{% highlight ruby linenos %}
require 'ostruct'

helper_method :current_user

def current_user
  @current_user ||= User.find session[:user_id] if session[:user_id]
  if @current_user
    @current_user
  else
    OpenStruct.new(name: 'Guest')
  end
end

{% endhighlight %}

你可以用一行代码代替之前的代码：

{% highlight ruby linenos %}
<h3>Welcome, <%= current_user.name -%></h3>
{% endhighlight %}

一些其他的建议：

* 适当的使用[view layouts and partials](http://guides.rubyonrails.org/layouts_and_rendering.html)去封装页面中重复的东西；
* 使用像[Draper gem](https://github.com/drapergem/draper)这样的装饰者，在一个ruby对象中去封装构建视图层的逻辑，然后将方法添加到这个对象中去执行逻辑操作。

#### 常见错误3: 将太多的逻辑放在model

鉴于将最少的逻辑放在view和controller的指导思想，在MVC的架构之下，只有把所有的逻辑放在model下面么？

其实，并不是这样。

许多的开发人员经常犯这样一个错误，他们坚持将一切东西都放在`ActiveRecord`的模型类下面，这不仅违反了单一职责原则，也带了了维护的噩梦。

像生成邮件通知，连接外部服务，数据格式转换这样的功能，它们并不像`ActiveRecord`这样肩负核心的责任，而`ActiveRecord`应该多做一些数据查找和持久化数据的事情。

因此，如果逻辑不应该放在view，不应该放在controller，也不应该放在model，那么它应该放在哪里呢？

像rails这样的综合框架，初级开发者通常不愿意在框架之外创建自己的类。然而，将逻辑从model移到普通ruby对象中正合我意，这可以避免过于复杂的模型。
通过ruby普通对象，你可以把邮件通知或API接口放在它们自己的类中，而不是放在`ActiveRecord`模型中。

总的来说，model中应该保留的逻辑有：

* `ActiveRecord configuration` (例如关联和验证)；
* 封装字段更新以及把其保存到数据库的方法；
* 访问隐藏内部模型信息的包装器（例如：一个`full_name`方法是数据库中`first_name`和`last_name`字段的结合）；
* 复杂的查询（比`find`更复杂的查询）。总的来说，除了model类本身之外，你不应该在其他地方使用`where`或其他类似的查询。

#### 常见错误4: 把通用的helper类当作垃圾场

这个错误是以上错误的一种推论。如前所述，Rails框架重视指定的`MVC`框架组件（例如`model`，`view`，`controller`），每一个组件的类都有定义好的东西，
但有些时候，我们需要的一些方法并不适合放在这三个组件中。

Rails生成器方便地构建了一个`helper`目录和一个新的`helper class`去对应我们创建的每个新资源。它太诱人了，可以把任何不适合放在`view`，`model`和`controller`的功能放在它们对应的helper类中。

Rails确实是`MVC-centric`，没有什么阻止你去创建自己类并添加适当的目录来保存这些类的代码。当你有额外的功能，考虑哪些方法需要组在一起，并为类取个好名字去保存这些方法的时候，
使用Rails这样全面的框架并不表示你要放弃良好的面向对象设计原则。

#### 常见错误5: 使用太多的gem包

Rails被一个丰富的gems生态系统所支持，并且它提供给所有的开发人员使用，对于快速构建复杂应用，这相当的棒！我也见过许多gem包数量庞大的应用程序，但其`Gemfile`中的gem包和它所提供的功能并不成正比。

这会导致几个问题：过度的使用gem包会使rails的体积过大，这降低了在生产环境下的性能。除了用户体验变差以外，也增加了内存消耗和操作成本。rails应用越大，相应的启动时间就会越长，这不仅降低了开发效率，也使自动化测试耗费更多时间。

请记住，每个带进应用程序的gem包都可能依赖于其他的gem包，反过来，这些gem包又可能依赖于别的，添加其他的gems会造成复合影响。例如，添加一个`rails_admin`包会连带安装其他的11个gem包，这会导致rails安装时间延长10%。

在撰写本文时，如果安装一个新的Rails4.1.0，在`gemfile.lock`文件中会包含43个gem包，这明显的要比Gemfile中定义的的要多，这表明，每个gem包都会有相关的依赖。

当你添加每一个gem包的时候，请仔细的考虑是否值得这样做。很多的开发者经常会添加`rails_admin`，只是因为它可以给model结构提供一个漂亮的web前端，但这真的不比一个数据库浏览工具更好。即使你的应用需要一个有额外权限的管理员用户，你可能并不想给他原始数据库的访问权限，开发更合理的管理功能通常比添加这些gem包要好。

#### 常见错误6: 忽略日志文件

虽然许多的开发者意识到rails在开发和生产环境下会提供默认的日志文件，但这些文件中的信息并没有引起足够的重视。即使许多的应用在生产环境下依赖于像[Honeybadger](https://www.honeybadger.io/)和[New Relic](http://newrelic.com/)这样的日志监测工具，但在开发和测试过程中，重视你的日志文件也很重要。

像之前提到的，rails框架为你提供了很多的“魔法”，特别是在model中，定义模型之间的关联变得非常容易，所有模块对象的SQL语句都自动为你生成，这很棒！但是你知道这些被生成的SQL语句的效率么？

经常会遇见的问题就是`N＋1`查询，虽然这个问题已经相当的明了，但是发现他们的有效途径就是在日志文件中观察SQL查询。

例如，在一个典型的博客应用中，查询要显示一些文章的评论：

{% highlight ruby linenos %}
def comments_for_top_three_posts
  posts = Post.limit(3)
  posts.flat_map do |post|
    post.comments.to_a
  end
end
{% endhighlight %}

当一个请求调用这个方法的时候，在日志文件中我们会看到：一个查询得到了三个post对象，为了得到每个对象的评论又进行了更多的查询。

{% highlight ruby linenos %}
Started GET "/posts/some_comments" for 127.0.0.1 at 2014-05-20 20:05:13 -0700
Processing by PostsController#some_comments as HTML
  Post Load (0.4ms)  SELECT "posts".* FROM "posts" LIMIT 3
  Comment Load (5.6ms)  ELECT "comments".* FROM "comments" WHERE "comments"."post_id" = ?  [["post_id", 1]]
  Comment Load (0.4ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = ?  [["post_id", 2]]
  Comment Load (1.5ms)  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = ?  [["post_id", 3]]
  Rendered posts/some_comments.html.erb within layouts/application (12.5ms)
Completed 200 OK in 581ms (Views: 225.8ms | ActiveRecord: 10.0ms)
{% endhighlight %}

Rails中，`ActiveRecord`的`eager loading`特性可以显著减少查询次数，它可以让相关联的对象提前加载，这是通过调用内建`Arel (ActiveRecord::Relation) `对象的`includes`（或`preload`）方法实现的。通过`includes`，`ActiveRecord`确保所有制定的关联都被加载，尽可能使用最小的查询次数：

{% highlight ruby linenos %}
def comments_for_top_three_posts
  posts = Post.includes(:comments).limit(3)
  posts.flat_map do |post|
    post.comments.to_a
  end
end
{% endhighlight %}

当以上代码被执行时，在log文件中我们看到，所有的评论都在单独的一个查询中被找到：

{% highlight ruby linenos %}
Started GET "/posts/some_comments" for 127.0.0.1 at 2014-05-20 20:05:18 -0700
Processing by PostsController#some_comments as HTML
  Post Load (0.5ms)  SELECT "posts".* FROM "posts" LIMIT 3
  Comment Load (4.4ms)  SELECT "comments".* FROM "comments" WHERE"comments "."post_id" IN (1, 2, 3)
  Rendered posts/some_comments.html.erb within layouts/application (12.2ms)
Completed 200 OK in 560ms (Views: 219.3ms | ActiveRecord: 5.0ms)
{% endhighlight %}

`N+1`问题只是低效率一个例子，如果你没有引起足够的重视，类似的问题在你的应用中可能还会存在，问题在于你应该检查开发和测试日志文件来定位低效的代码。

浏览日志文件是在应用进入生产环境之前找到低效率代码和改正它们的一个非常好的手段。否则，只要程序还在运行，你不会意识到性能问题，因为在开发和测试
环境下，数据量比生产环境要小的多。如果你维护一个新的app，在生产环境的数据集很小，它一样会工作良好，然而，随着数据量的增大，你的应用会变得越来越慢。

#### 常见错误7: 缺乏自动化测试

Rails默认提供强大的自动测试功能。许多Rails开发人员使用TDD和BDD风格编写非常复杂的测试，并且善于利用强大的测试框架（如[rspec](https://www.relishapp.com/rspec/)和[cucumber](http://cukes.info/)）。虽然在应用中添加自动化测试是如此的容易，但是我失望的是，在我接手的很多项目中，前任开发团队并没有（或很少）写测试用例。虽然很多人讨论应该如何进行测试，但很显然的是，至少应该有一些自动化测试存在每一个应用程序中。

根据一般的经验，在控制器的每一个action中，至少有一个高级的集成测试。在未来的某个时候，开发人员有可能想扩展或修改代码，或者升级rails版本，一个良好的测试框架是验证应用基本功能正常的最好方式。这种方法的另一个好处是：它提供给未来的开发人员一个清晰的，描述功能完整的应用程序。

#### 常见错误8: 阻塞外部服务调用

Rails服务的第三方提供者通常很容易通过gems集成它们的服务到你的应用中，但是如果你的外部服务中断或运行非常慢，应该怎么办呢?

为了避免被这些调用阻塞，与其在正常的请求进程中直接调用这些服务，不如把它们移到后台工作队列中，一些流行的gems包可以解决这个问题：

* [Delayed job](https://github.com/collectiveidea/delayed_job)
* [Resque](https://github.com/resque/resque)
* [Sidekiq](http://sidekiq.org/)

如果将不切实际或不可行的委托进程放在一个后台作业队列，对那些不可避免的情况或当外部服务出现问题的时候，你需要确保应用程序有足够的错误处理和故障转移机制。你也应该测试应用在没有外部服务的情况下，不会出现意料之外的后果。

#### 常见错误9: 不敢改动现有的数据库迁移

Rails的`database migration mechanism`允许你创建指令去自动添加和删除数据库表和行，这是管理应用数据库模式变化的一个非常好的方式。虽然这适用于项目的开始阶段，随着时间的推移，数据库创建过程可能会花费较长的时间，有时还会出现迁移错位，插入顺序错误等情况。

Rails创建一个当前数据库模式的映射文件，被称作`db/schema.rb`，当数据库migration运行的时候，它就会被更新。当没有migration的时候，`schema.rb`文件可以通过运行`rake db:schema:dump`生成。一个常见的错误是把一个新的migration放进源代码库中，但是却没有更新对应的`schema.rb`文件。

当迁移已经失控，运行时间过长，或不再正常创建数据库，开发者不应该害怕清除旧的迁移目录，拷贝一个新的模式，并从那里继续。建立一个新的开发环境将需要一个`rake db:schema:load`而不是`rake db:migrate`，这是大多数开发人员所依赖的。

Rials Guide中也有这些讨论。

#### 常见错误10: 将敏感信息放入源代码库中

Rails框架可以轻松创建安全的应用程序，某些是通过使用一个`secret token`与浏览器安全会话。尽管现在这个token存储在`config/secrets.yml`, 文件从生产服务器的环境变量读取token，但以前的rails版本在`config/initializers/secret_token.rb`包含token。这个文件经常被错误的加进源码库，当这种情况发生的时候，任何可以访问代码库的人都可以轻松的危害你应用中的所有用户。

你应该确保你的存储库配置文件排除了带有token的文件。你的生产服务器可以从环境变量中获取它们的token，或者通过[dotenv gem](https://github.com/bkeepers/dotenv)。

## 小结

Rails是一个强大的框架，在构建强健的web应用过程中，它隐藏了很多丑陋的细节。然而，随着web应用开发越来越快，开发人员应该注意潜在的设计和编码缺陷，在应用规模逐渐增长的情况下，代码依然有良好的扩展性和可维护性。

开发人员还需要注意那些使他们的应用程序变得慢，不可靠，更不安全的问题，在整个开发过程，研究框架并确保充分理解架构设计和编码，以确保生产高质量和高性能的应用程序是非常重要的。