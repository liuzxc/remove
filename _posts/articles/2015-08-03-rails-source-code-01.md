---
layout: post
title: Rails 源码解读（一）rails console
excerpt:
comments: true
share: true
categories: articles
---

rails console 命令时我们经常使用的，它可以让我们通过命令行很方便的与 Rails 应用进行交互，类似于 irb，
其实，rails console 的底层就是用 irb 来实现的，让我们来扒一扒相关的源码。

{% highlight ruby %}
#rails/railties/lib/rails/commands.rb
ARGV << '--help' if ARGV.empty?

aliases = {
  "g"  => "generate",
  "d"  => "destroy",
  "c"  => "console",
  "s"  => "server",
  "db" => "dbconsole",
  "r"  => "runner"
}

help_message = <<-EOT
Usage: rails COMMAND [ARGS]

The most common rails commands are:
 generate    Generate new code (short-cut alias: "g")
 console     Start the Rails console (short-cut alias: "c")
 server      Start the Rails server (short-cut alias: "s")
 dbconsole   Start a console for the database specified in config/database.yml
             (short-cut alias: "db")
 new         Create a new Rails application. "rails new my_app" creates a
             new application called MyApp in "./my_app"
......
EOT

command = ARGV.shift
command = aliases[command] || command

case command
when 'plugin'
  require "rails/commands/plugin"
when 'generate', 'destroy'
  ...
when 'console'
  ...
{% endhighlight %}

ARGV 是一个数组，它包含了从标准处输出传来的命令行参数。例如在命令行输入以下命令的时候：

`rails console --sandbox`

ARGV 的值是： `["console", "--sandbox"]`

当 ARGV 为空的时候，即 Rails 命令后面不加任何参数，程序会将 `－－help` 参数自动添加到ARGV 数组中。

{% highlight ruby %}
command = ARGV.shift
command = aliases[command] || command
{% endhighlight %}

ARGV 中的第一个元素被弹出，并传递给 command 变量，case 语句根据 command 的值选择相应的分支。

{% highlight ruby %}
when 'console'
  require 'rails/commands/console'
  options = Rails::Console.parse_arguments(ARGV)

  # RAILS_ENV needs to be set before config/application is required
  ENV['RAILS_ENV'] = options[:environment] if options[:environment]

  # shift ARGV so IRB doesn't freak
  ARGV.shift if ARGV.first && ARGV.first[0] != '-'

  require APP_PATH
  Rails.application.require_environment!
  Rails::Console.start(Rails.application, options)
{% endhighlight %}

进入 console 分支后，此时的 ARGV 数组中只剩下一个元素 `--sandbox`, 因为元素 "console" 已经在之前被弹出，接着 Rails::Console 类中的 parse_arguments 方法会去解析 ARGV：

{% highlight ruby %}
#rails/railties/lib/rails/commands/console.rb
require 'optparse'
...
 OptionParser.new do |opt|
  opt.banner = "Usage: rails console [environment] [options]"
  opt.on('-s', '--sandbox', 'Rollback database modifications on exit.') { |v| options[:sandbox] = v }
  opt.on("-e", "--environment=name", String,
          "Specifies the environment to run this console under (test/development/production).",
          "Default: development") { |v| options[:environment] = v.strip }
  opt.on("--debugger", 'Enable the debugger.') { |v| options[:debugger] = v }
  opt.parse!(arguments)
end

if arguments.first && arguments.first[0] != '-'
  env = arguments.first
  if available_environments.include? env
    options[:environment] = env
  else
    options[:environment] = %w(production development test).detect {|e| e =~ /^#{env}/} || env
  end
end

options
{% endhighlight %}

通过 ruby 开发命令行工具，optparse 是个非常好的选择，大多数的 ruby 命令行工具的开发都会使用到它。从上面的代码中我们可以看出：

* `--sandbox` 表明任何对于数据库的修改操作在退出的时候都会被回滚；
* rails console 的环境有三种：test/development/production，默认环境是 development

如果第一个参数包含了 '－'，则直接将参数返回，否则就说明该参数是用于指定环境的，rails 提供的环境可以在 `config/environments`
目录中配置，并不仅仅限于 test/development/production 这三种，但如果没有指定自己特有的环境，则在默认的 test/development/production 中选择。

`ARGV.shift if ARGV.first && ARGV.first[0] != '-'`

在参数被解析完毕之后，如果 ARGV 中包含 '－'，则需要把参数弹出，避免将其继续传递给 irb，因为 irb 并不需要也不理解该参数。接下来
就要准备启动 rails console 了。

{% highlight ruby %}
#rails/railties/lib/rails/commands/console.rb
module Rails
  class Console
    class << self
      def start(*args)
        new(*args).start ＃初始化操作
      end
    ......
    end

    def initialize(app, options={})
      @app     = app
      @options = options

      app.sandbox = sandbox?
      app.load_console

      @console = app.config.console || IRB
    end
    ......
    def start
      require_debugger if debugger?
      set_environment! if environment?

      #进入rails console后的提示信息
      if sandbox?
        puts "Loading #{Rails.env} environment in sandbox (Rails #{Rails.version})"
        puts "Any modifications you make will be rolled back on exit"
      else
        puts "Loading #{Rails.env} environment (Rails #{Rails.version})"
      end

      if defined?(console::ExtendCommandBundle)
        console::ExtendCommandBundle.send :include, Rails::ConsoleMethods
      end
      console.start  #启动irb
    end
    ......
{% endhighlight %}

`@console = app.config.console || IRB`

该行代码表明了 rails conlose 的底层实现就是通过 irb 完成的。

`console.start` 相当于 `IRB.satrt`