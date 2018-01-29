---
layout: post
title: 使用 RSpec 测试 Rails 程序（1）
excerpt: rspec
categories: blog
comments: true
share: true
---

虽然`test::unit`测试框架是rails的官方标配，但是毋庸置疑的是，Rspec才是rails社区最流行的测试框架。

有趣的是，在毕业后的两年时间里，我所工作的两家公司均使用了Rspec，而获得的感受却是完全不同的。我工作的第一家公司使用了一套用ruby开发的持续集成系统（Continuous Integration System)，100%的Rspec测试覆盖率，当你对系统程序做了任何改动之后，只要运行一遍测试用例就知道结果如何，基本上不需要担心改动会引入很严重的bug。而我工作的第二家公司使用rails开发了一套应用程序，打开源代码的时候，我看到了spec目录，可里面却空空如也，而这带来的结果是显而易见的，你不停的在忙于救火，对系统的任何改动都战战兢兢，也许轻微的改动都可能导致灾难性的后果，大量的时间都浪费在重复测试上，极大影响开发效率。

本文以rails登录系统作为例子来学习Rspec的使用。

#### 安装RSpec

在Gemfile中添加如下代码：

```ruby
group :development, :test do
	gem "rspec-rails"
end
```

然后运行bundle install

一般在生产环境下不会跑测试用例，所以只在测试和开发环境下安装RSpec

> 如果你不想使用rails自带的`test::unit`测试框架，你只需要删除test目录和相关的引用即可

gem包安装好之后，需要手动安装RSpec

```vim
$ rails g rspec:install
    create  .rspec  //配置文件
    create  spec    //存放测试文件
    create  spec/spec_helper.rb
```

spec目录的结构需要和app目录的结构保持一致，测试文件的命名规则：<modle class>_spec.rb

> 添加`--format documentation`到.rspec，可以格式化输出测试信息

运行RSpec

```vim
➜ rspec
No examples found.

Finished in 0.00009 seconds
0 examples, 0 failures
```

#### 搭建测试数据库

rails默认使用数据库sqlite，如果想使用mysql，可以在`database.yml`中配置，

```yaml
test:
	adapter: mysql2
	encoding: utf8
	reconnect: false
	database: contacts_test
	pool: 5
	username: root
	password:
	socket: /tmp/mysql.sock
```

运行 `rake db:create:all` 创建测试数据库
运行 `rake db:test:clone` 把开发数据库的结构复制到测试数据库

#### 测试用例

Rspec属于BDD测试（行为驱动测试），即测试先行， 所以在下面的例子中，会先写测试用例，再实现相应的功能。

> 很多人都会争论BDD和TDD孰优孰劣，我相信没有那一种测试方法是占有绝对优势的，虽然DHH大神曾说TDD已死，但是对于国内大多数开发人员而言，BDD只是个新鲜的名词而已，很少有人将其用于实践当中。而BDD需要大量的实践，这样才可以帮助你提高开发效率和质量，否则只会是适得其反。

首先创建一个user表来存放用户数据

```bash
rails g migration model User name:string email:string
```

自动生成如下代码：

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :email

      t.timestamps
    end
  end
end

class User < ActiveRecord::Base
end
```

我们需要实现一些基本的数据验证功能，只有合法的数据才可以被存入数据库，创建一个`user_spec.rb`的测试文件，用于存放测试用例。

Rspec测试文件的基本结构：

```ruby
require 'spec_helper' #测试文件必须添加该行

describe <类名> do
#describe: 描述你期望这个类的表现是什么，它通常包含了一组测试用例的集合
	it "is invalid without user name" do
    #一个it表示一条测试用例，双引号中需要注明测试的内容，后面的do关键字是必须的，不可以省略
		user = User.new(name: "", email: "jack@163.com") #在测试文件中你可以使用任意的ruby代码
		expect(user).not_to be_valid  #这句可以翻译为"期望用户不是合法的"
	end
end
```

> `to`, `not_to`, `be_valid` 是Rspec实现的匹配器，使用rspec写测试用例必须熟悉常用的匹配器，可以从[rspec-expectations](https://github.com/rspec/rspec-expectations)查找更多的匹配器。

接下来让我们尝试写第一个测试用例：验证用户名和邮箱不为空

```ruby
require 'spec_helper'

describe User do
	it "is invalid without user name" do
		user = User.new(name: "", email: "jack@163.com")
		expect(user).not_to be_valid
	end

	it "is invalid without email" do
		user = User.new(name: "john", email: "")
		expect(user).not_to be_valid
	end
end
```

运行测试用例：

```vim
➜ rspec
User
  is invalid without user name (FAILED - 1)
  is invalid without email (FAILED - 2)

Finished in 0.02447 seconds
2 examples, 2 failures
......
```

由于未添加验证功能，所以测试用例全部失败。

```ruby
class User < ActiveRecord::Base
	validates :name, presence: true
	validates :email, presence: true
end
```

添加验证功能之后再运行测试用例，

```vim
➜ rspec
User
  is invalid without user name
  is invalid without email

Finished in 0.0169 seconds
2 examples, 0 failures
```

测试用例全部成功！

很多人都会有这样一个疑问：`new`方法是不会触发数据验证的，而且在`rails console`中可以new一个用户名或邮箱为空的user，那测试用例是不应该运行成功的，对么？

通过对rspec源码的查看，我们可以找到答案：

```ruby
module Spec
  module Rails
    module Matchers
      class BeValid  #:nodoc:
        def matches?(model)
          @model = model
          @model.valid?
        end
        ........
      end

      def be_valid
        BeValid.new
      end
    end
  end
end
```

`be_valid`方法会调用`ActiveRecord::Validations`的`valid?`方法，而这个方法会主动触发数据验证，无论你是使用`new`或`create`创建记录。

```vim
➜ rails c --sandbox
Loading development environment in sandbox (Rails 3.2.15)
Any modifications you make will be rolled back on exit
irb(main):001:0> user = User.new(name: "", email: "jack@163.com")
=> #<User id: nil, name: "", email: "jack@163.com", created_at: nil, updated_at: nil>
irb(main):002:0> user.errors
=> #<ActiveModel::Errors:0x007fe59b2cab90 @base=#<User id: nil, name: "", email: "jack@163.com", created_at: nil, updated_at: nil>, @messages={}>
irb(main):003:0> user.valid?
=> false
irb(main):004:0> user.errors
=> #<ActiveModel::Errors:0x007fe59b2cab90 @base=#<User id: nil, name: "", email: "jack@163.com", created_at: nil, updated_at: nil>, @messages={:name=>["can't be blank"]}>
```

> 在`rails console --sandbox`中对数据库做的任何操作，在退出的时候都会被回滚。

当user对象被创建的时候，第一次调用`errors`方法，`@messages`为空；当调用`valid?`方法之后再次调用`errors`，`@messages`就不为空了，因为`valid?`方法触发了数据验证。

所以，之前的测试用例也可以这样写：

```ruby
require 'spec_helper'

describe User do
	it "is invalid without user name" do
		user = User.new(name: "", email: "jack@163.com")
		user.valid?
		expect(user.errors[:name]).to include("can't be blank")
	end

	it "is invalid without email" do
		user = User.new(name: "john", email: "")
		user.valid?
		expect(user.errors[:email]).to include("can't be blank")
	end
end
```

运行测试用例：

```vim
➜  rspec
User
  is invalid without user name
  is invalid without email

Finished in 0.03445 seconds
2 examples, 0 failures
```

同样运行成功!

接着我们添加验证字段长度的测试用例（假设name长度不超过20，邮箱长度不超过50）

```ruby
describe User do
	it "is invalid if name is too long" do
		user = User.new(name: "test", email: "test@test.com")
		user.name = "n" * 21
		expect(user).not_to be_valid
	end

	it "is invalid if email is too long" do
		user = User.new(name: "test", email: "test@test.com")
		user.email = "e" * 51
		expect(user).not_to be_valid
	end
end
```

添加邮箱格式验证：

```ruby
describe User do
	it "is invalid with a error format email" do
		user = User.new(name: "test", email: "test@test.com")
		invalid_email = %w(
				test1@@163.com
				test2@163
				test3163.com
				test+4@163.com
						  )
		invalid_email.each do |email|
			user.email = email
			expect(user).not_to be_valid
		end
	end
end
```

添加邮箱唯一性验证：

```ruby
describe User do
	it "is invalid with a duplication email" do
		user = User.new(name: "test", email: "test@test.com")
		user1 = user.clone
		user1.save
		user.valid?
		expect(user.errors.messages[:email]).to include("has already been taken")
	end
end
```

写完测试用例之后，我们来实现相应的功能：

```ruby
class User < ActiveRecord::Base
  validates :name, presence: true, length: {maximum: 20}
  validates :email, presence: true, length: {maximum: 50},
                    format: { with: /\A[\w]+@[a-z\d]+\.[a-z]/ },
                    uniqueness: true
end
```

运行测试用例可以验证我们的功能是否符合预期，

```vim
➜ rspec
User
  is invalid with a error format email
  is invalid if name is too long
  is invalid without email
  is invalid with a duplication email
  is invalid if email is too long
  is invalid without user name

Finished in 0.26591 seconds
6 examples, 0 failures
```

测试用例全部通过！

让我们来看下我们写的所有的测试用例：

```ruby
require 'spec_helper'

	describe User do
	it "is invalid without user name" do
		user = User.new(name: "", email: "jack@163.com")
		user.valid?
		expect(user.errors[:name]).to include("can't be blank")
	end

	it "is invalid without email" do
		user = User.new(name: "john", email: "")
		user.valid?
		expect(user.errors[:email]).to include("can't be blank")
	end

	it "is invalid if name is too long" do
		user = User.new(name: "test", email: "test@test.com")
		user.name = "n" * 21
		expect(user).not_to be_valid
	end

	it "is invalid if email is too long" do
		user = User.new(name: "test", email: "test@test.com")
		user.email = "e" * 51
		expect(user).not_to be_valid
	end

	it "is invalid with a error format email" do
		user = User.new(name: "test", email: "test@test.com")
		invalid_email = %w(
							test1@@163.com
							test2@163
							test3163.com
							test+4@163.com
						  )
		invalid_email.each do |email|
			user.email = email
			expect(user).not_to be_valid
		end
	end

	it "is invalid with a duplication email" do
		user = User.new(name: "test", email: "test@test.com")
		user1 = user.clone
		user1.save
		user.valid?
		expect(user.errors.messages[:email]).to include("has already been taken")
	end
end
```


通过这些测试用例我们发现了一个问题，每一个测试用例中都会新建一个用户，目前的user表只有两个字段，在一般的应用当中，user表的字段远不止这个数。字段越多，构造测试数据就越麻烦，相应的代码量就越多，大部分代码都是做着重复的事情，违背了DRY原则。Rspec提供了消除重复代码的利器：`before`块

让我们试着重构上面的测试用例：

```ruby
require 'spec_helper'

describe User do
	before :each do
		@user = User.new(name: "test", email: "test@test.com")
	end

	it "is invalid without user name" do
		@user.name = ''
		@user.valid?
		expect(@user.errors[:name]).to include("can't be blank")
	end

	it "is invalid without email" do
		@user.email = ''
		@user.valid?
		expect(@user.errors[:email]).to include("can't be blank")
	end

	it "is invalid if name is too long" do
		@user.name = "n" * 21
		expect(@user).not_to be_valid
	end

	it "is invalid if email is too long" do
		@user.email = "e" * 51
		expect(@user).not_to be_valid
	end

	it "is invalid with a error format email" do
		invalid_email = %w(
			test1@@163.com
			test2@163
			test3163.com
			test+4@163.com
		)
		invalid_email.each do |email|
			@user.email = email
			expect(@user).not_to be_valid
		end
	end

	it "is invalid with a duplication email" do
		user1 = @user.clone
		user1.save
		@user.valid?
		expect(@user.errors.messages[:email]).to include("has already been taken")
	end
end
```

在运行每一个测试用例之前，会先运行`before`块中的代码。此处有6个测试用例，`before`块中的代码也会被执行六次，因此我们不必在每个测试用例中创建user，减少了代码的重复。

接下来为user表建立一个`password_hash`字段，用来存放加密后的密码。

```bash
rails g migration add_password_hash_to_users password_hash:string
```

在`Gemfile`中需要添加一个gem包（在创建rails项目的时候，已经在`Gemfile`中添加了该包，只需要取消注释即可）：

```ruby
# To use ActiveModel has_secure_password
gem 'bcrypt-ruby', '~> 3.0.0'
```

`has_secure_password`方法会创建虚拟的`password`和`password_confirmation`字段

> [has_secure_password](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password)的更多信息

密码长度一般不会小于6位，让我们写出相应的测试用例：

```ruby
describe User do
	before :each do
		@user = User.new(name: "test", email: "test@test.com", password: "123456", password_confirmation: "123456")
	end
	......
	it "is invalid if password is too short" do
		@user.password = "p" * 5
		@user.password_confirmation = "p" * 5
		expect(@user).not_to be_valid
	end
end
```

相应的实现代码：

```ruby
class User < ActiveRecord::Base
  validates :name, presence: true, length: {maximum: 20}
  validates :email, presence: true, length: {maximum: 50},
                    format: { with: /\A[\w]+@[a-z\d]+\.[a-z]/ },
                    uniqueness: true
  has_secure_password
  validates :password, length: {minimum: 6}
end
```

因为添加了新的字段，你需要更新测试数据库结构，不然测试无法通过。
只需要运行`rake db:test:clone`即可。

我们建立了基本的user表，添加了数据验证和相应的测试用例。使用Rspec测试需要大量的实践，一开始你会觉得开发效率很慢，但是随着对框架越来越熟悉，你会感受到它带来的好处，并且让测试成为开发的一部分。
