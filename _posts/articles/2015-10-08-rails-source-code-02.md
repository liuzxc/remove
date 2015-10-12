---
layout: post
title: Rails 源码解读（二）Active Record
excerpt:
comments: true
share: true
categories: articles
---

#### Active Record 是什么？

Active Record 是 MVC 中的 M（模型），处理数据和业务逻辑。Active Record 负责创建和使用需要持久存入数据库中的数据。Active Record 实现了 Active Record 模式，是一种对象关系映射系统。

Active Record 是 Rails 最核心的功能，也是 Rails 在web开发领域取得成功的关键因素之一，它的作者也是 DHH 本人。Active Record 的主要优点有：

1. 不依赖于特定的数据库，支持多种数据库
2. 以面向对象的方式操作数据，无需直接编写sql语句
3. 事务支持
4. 简介的关联关系（例如 has_many, belongs_to)
5. 内建 validations

接下来我们看看 Active Record 的核心源码：

{% highlight ruby %}
#rails/activerecord/lib/active_record.rb
require 'active_support'
require 'active_support/rails'
require 'active_model'
require 'arel'

require 'active_record/version'

module ActiveRecord
  extend ActiveSupport::Autoload

  autoload :Base
  autoload :Callbacks
  autoload :Core
  autoload :ConnectionHandling
  autoload :CounterCache
  autoload :DynamicMatchers
  autoload :Explain
  autoload :Inheritance
  autoload :Integration
  autoload :Migration
  autoload :Migrator, 'active_record/migration'
  autoload :ModelSchema
  autoload :NestedAttributes
  autoload :Persistence
  autoload :QueryCache
  autoload :Querying
  autoload :ReadonlyAttributes
  autoload :Reflection
  autoload :RuntimeRegistry
  autoload :Sanitization
  autoload :Schema
  autoload :SchemaDumper
  autoload :SchemaMigration
  autoload :Scoping
  autoload :Serialization
  autoload :StatementCache
  autoload :Store
  autoload :Timestamp
  autoload :Transactions
  autoload :Translation
  autoload :Validations
  .....
{% endhighlight %}

ActiveRecord 模块会自动加载不同功能的模块，包括常见的数据库连接，迁移，查询，序列化，验证模块等。

以数据库查询模块Querying为例：

{% highlight ruby %}
#rails/activerecord/lib/active_record/querying.rb
module ActiveRecord
  module Querying
    delegate :find, :take, :take!, :first, :first!, :last, :last!, :exists?, :any?, :many?, to: :all
    delegate :first_or_create, :first_or_create!, :first_or_initialize, to: :all
    delegate :find_or_create_by, :find_or_create_by!, :find_or_initialize_by, to: :all
    delegate :find_by, :find_by!, to: :all
    delegate :destroy, :destroy_all, :delete, :delete_all, :update, :update_all, to: :all
    delegate :find_each, :find_in_batches, to: :all
    delegate :select, :group, :order, :except, :reorder, :limit, :offset, :joins,
             :where, :preload, :eager_load, :includes, :from, :lock, :readonly,
             :having, :create_with, :uniq, :distinct, :references, :none, :unscope, to: :all
    delegate :count, :average, :minimum, :maximum, :sum, :calculate, to: :all
    delegate :pluck, :ids, to: :all
    .......
    #   Post.find_by_sql ["SELECT title FROM posts WHERE author = ? AND created > ?", author_id, start_date]
    #   Post.find_by_sql ["SELECT body FROM comments WHERE author = :user_id OR approved_by = :user_id", { :user_id => user_id }]
    def find_by_sql(sql, binds = [])
      result_set = connection.select_all(sanitize_sql(sql), "#{name} Load", binds)
      column_types = {}

      if result_set.respond_to? :column_types
        column_types = result_set.column_types
      else
        ActiveSupport::Deprecation.warn "the object returned from `select_all` must respond to `column_types`"
      end

      result_set.map { |record| instantiate(record, column_types) }
    end
    ....
    def count_by_sql(sql)
      sql = sanitize_conditions(sql)
      connection.select_value(sql, "#{name} Count").to_i
    end
  end
end
{% endhighlight %}

delegate 方法在 rails 源码中使用的非常多，它可以把所包含对象的 public method 变成自己的。例如：

{% highlight ruby %}
class Greeter < ActiveRecord::Base
  def hello
    'hello'
  end
  def goodbye
    'goodbye'
  end
end
class Foo < ActiveRecord::Base
  belongs_to :greeter
  delegate :hello, to: :greeter
end

Foo.new.hello   # => "hello"
Foo.new.goodbye # => NoMethodError: undefined method `goodbye' for #<Foo:0x1af30c>
{% endhighlight %}

委托方法在rails中应用广泛的原因很明显，我们很难在rails代码中找到动辄几千行的代码，也没有存在某个类拥有大量的方法，这样的代码一定会让可读性和维护性变差，而rails通过把一堆方法分成不同的组放在不同的文件，然后通过代理方法把它们汇总在一起，很巧妙的避免了这个问题。例如查询模块，rails把数据查询，数据的删除和更新等操作分成不同的组放入不同的文件，再在Querying模块把它们汇集在一起。那么为什么不使用include方法呢？我们知道，如果在一个module A 中inlude另外一个module B，那么B 中的方法将成为A的实例方法。显然include和delegate有着相同的作用，但是有些情况下，我只需要B中的某个或一部分方法，使用include把所有模块的方法都取过来显然是不科学的，而delegate可以做到按需索取。

ActiveRecord提供了如此多的方法可以供我们使用，那么它又是如何连接数据库的呢？

{% highlight ruby %}
#rails/activerecord/lib/active_record/connection_adapters/connection_specification.rb
module ActiveRecord
  module ConnectionAdapters
    class ConnectionSpecification #:nodoc:
      attr_reader :config, :adapter_method
      .....
        def resolve_hash_connection(spec) # :nodoc:
          spec = spec.symbolize_keys

          raise(AdapterNotSpecified, "database configuration does not specify adapter") unless spec.key?(:adapter)

          path_to_adapter = "active_record/connection_adapters/#{spec[:adapter]}_adapter"
          begin
            require path_to_adapter
          rescue Gem::LoadError => e
            raise Gem::LoadError, "Specified '#{spec[:adapter]}' for database adapter, but the gem is not loaded. Add `gem '#{e.name}'` to your Gemfile."
          rescue LoadError => e
            raise LoadError, "Could not load '#{path_to_adapter}'. Make sure that the adapter in config/database.yml is valid. If you use an adapter other than 'mysql', 'mysql2', 'postgresql' or 'sqlite3' add the necessary adapter gem to the Gemfile.", e.backtrace
          end

          adapter_method = "#{spec[:adapter]}_connection"

          ConnectionSpecification.new(spec, adapter_method)
        end
{% endhighlight %}

rails server 或 rails console启动的时候，rails会自动的去读取 database.yml 文件，此处的spec参数就是文件中的内容被转化成hash，通过读取adapter中的值确定使用哪个数据库适配器。以mysql为例：

{% highlight ruby %}
#rails/activerecord/lib/active_record/connection_adapters/mysql_adapter.rb
module ActiveRecord
  module ConnectionHandling # :nodoc:
    # Establishes a connection to the database that's used by all Active Record objects.
    def mysql_connection(config)
      config = config.symbolize_keys
      host     = config[:host]
      port     = config[:port]
      socket   = config[:socket]
      username = config[:username] ? config[:username].to_s : 'root'
      password = config[:password].to_s
      database = config[:database]

      mysql = Mysql.init
      mysql.ssl_set(config[:sslkey], config[:sslcert], config[:sslca], config[:sslcapath], config[:sslcipher]) if config[:sslca] || config[:sslkey]

      default_flags = Mysql.const_defined?(:CLIENT_MULTI_RESULTS) ? Mysql::CLIENT_MULTI_RESULTS : 0
      default_flags |= Mysql::CLIENT_FOUND_ROWS if Mysql.const_defined?(:CLIENT_FOUND_ROWS)
      options = [host, username, password, database, port, socket, default_flags]
      ConnectionAdapters::MysqlAdapter.new(mysql, logger, options, config)
    end
  end
  .....
{% endhighlight %}

rails找到mysql_adapter.rb 文件，该文件包含了获取数据库连接的相关方法。mysql_connection 方法回再次读取database.yml文件，获取数据库配置，建立数据库连接，active_record 有一个 ConnectionPool的类来管理数据库连接。

Active Record的缺点也是显而易见的，它把数据库中的数据映射成相应的对象，对于数据库的批量操作，势必会消耗大量的内存；Active Record的关联关系使用起来的很简单，但是如果对其不够了解，一个普通的操作背后可能就是大量的数据库操作，带来性能问题。Rails通过牺牲性能为代价换取了极高的开发效率，一方面使其在web开发领域占据一席之地，另一方面也让其饱受诟病，而DHH开发rails的初衷之一就是以效率至上，通过其他技术和增加硬件资源来弥补在性能上的缺失，毕竟人力资源总是比机器资源更昂贵。

#### method_missing

method_missing 在 ruby元编程 一书中被称作幽灵方法，它可以动态的创建方法. rails 大量的使用 method_missing，最重要的用途之一就是可以减少代码量。很多人会好奇为什么rails可以把数据库字段映射成响应的方法供 active record 对象调用，其背后的秘密就在于 method_missing 的使用。

{% highlight ruby %}
#rails/activerecord/lib/active_record/attribute_methods.rb
    def method_missing(method, *args, &block)
      if respond_to_without_attributes?(method, true)
        super
      else
        match = match_attribute_method?(method.to_s)
        match ? attribute_missing(match, *args, &block) : super
      end
    end

    def attribute_missing(match, *args, &block)
      __send__(match.target, match.attr_name, *args, &block)
    end
{% endhighlight %}

> 在ruby中，__send__ 可以动态的调用方法，代码在运行期间可以动态的决定到底调用哪一个方法；其另外一个作用就是它可以调用对象的私有方法。send方法和__send__的作用是一样的，只是在某些情况下，send方法容易被覆盖。

在ruby当中，method_missing 也和我们关系密切，只是它待在一个极为隐秘的地方，你几乎察觉不到它的存在。

{% highlight ruby %}
class A
end
A.new.private_methods.grep(/miss/)
=> [:method_missing]
A.new.hello
NoMethodError: undefined method `hello' for #<A:0x007f9a0911d6e0>

A.ancestors
=> [A, Object, Kernel, BasicObject]
{% endhighlight %}

ruby会沿着A的祖先链去搜寻hello方法，一直都没有找到，最后来到Kernel模块，ruby解释器要求A对象调用 method_missing 方法，该方法抛出 NoMethodError 的错误。因此ruby中method_missing和我们的关系密切，它就像一座监狱，所有不合法的方法都会被关押在这里统一处理，你可以通过覆写它，让这些不合法的方法重获自由。

>对于一般的ruby编程，我并不推荐使用 method_missing 这样的花哨技巧，因为它增加了代码的复杂度，使代码维护性降低。它很适合应用于编写框架，就像rails，把细节隐藏在背后，我们在意框架如何使用，而不会过多的关心框架的底层实现。
