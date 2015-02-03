---
layout: post
title: Rails常用命令
excerpt:
categories: blog
comments: true
share: true
---

创建新的rails项目

`rails new <app_name>`

创建模型(模型的名字是单数，对应的数据表名是复数)

`rails g model User name:string age:integer`

<figure>
    <img src="/images/rails_type.png">
    <figcaption>rails中的字段类型</figcaption>
</figure>


{% highlight ruby %}
create_table 'example' do |t|
  t.integer :int                 # int (4 bytes, max 2,147,483,647)
  t.integer :int1, :limit => 1   # tinyint (1 byte, -128 to 127)
  t.integer :int2, :limit => 2   # smallint (2 bytes, max 32,767)
  t.integer :int3, :limit => 3   # mediumint (3 bytes, max 8,388,607)
  t.integer :int4, :limit => 4   # int (4 bytes)
  t.integer :int5, :limit => 5   # bigint (8 bytes, max 9,223,372,036,854,775,807)
  t.integer :int8, :limit => 8   # bigint (8 bytes)
  t.integer :int11, :limit => 11 # int (4 bytes)
end
{% endhighlight %}

<figure>
    <img src="/images/rails_reserve.png">
    <figcaption>rails中的保留字段名</figcaption>
</figure>

#### rake命令

* rake db:create 依照目前的 RAILS_ENV 環境建立資料庫
* rake db:create:all 建立所有環境的資料庫
* rake db:drop 依照目前的 RAILS_ENV 環境刪除資料庫
* rake db:drop:all 刪除所有環境的資料庫
* rake db:migrate 執行Migration動作
* rake db:rollback STEP=n 回復上N個 Migration 動作
* rake db:migrate:up VERSION=20080906120000 執行特定版本的Migration
* rake db:migrate:down VERSION=20080906120000 回復特定版本的Migration
* rake db:version 目前資料庫的Migration版本
* rake db:seed 執行 db/seeds.rb 載入種子資料
* rake db:setup does db:create, db:schema:load, db:seed
* rake db:reset does db:drop, db:setup
* rake db:schema:load 根据schema.rb创建数据库表
