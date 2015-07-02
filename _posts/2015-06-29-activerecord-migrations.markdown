---
layout: post
title: "ActiveRecord and Migrations"
date: 2015-06-29 23:10:52
tags: activerecord ruby rails sql
categories: ruby rails activerecord
---
Migrations in Rails can cause headaches; to avoid headaches, please use SQL
statements and avoid writing to your models in any way. As soon as you add in
something like `User.save`, you're bound to run into issues running migrations
from scratch later on.

Prepared statements allow us to safely insert or update our data, and we can be
confident that the state of the database will be compatible with the SQL you are
executing.

{% highlight ruby %}
class DoubleUserBalance < ActiveRecord::Migration
  def up
    # Double user's balance
    prepared = ActiveRecord::Base.connection.raw_connection.prepare("
      UPDATE users
      SET balance = balance * ?
    ")
    prepared.execute(2)
    prepared.close
  end

  def down
    # Halve user's balance
    prepared = ActiveRecord::Base.connection.raw_connection.prepare("
      UPDATE users
      SET balance = balance * ?
    ")
    prepared.execute(1/2)
    prepared.close
  end
end
{% endhighlight %}

This may at first seem a bit paranoid, as you can always use `rake db:setup` to
load the database directly from `schema.rb`. However, this will most likely bite
you when you least expect it - for example, if you deploy code to your staging
server daily, but to your production server weekly; you will eventually find that
migrations which worked when run individually, spontaneously combust when run as
a group. This can be caused by table column information not being reloaded
between migrations (forcing you to add in `User.reset_column_information` when
you discover that it's broken!).

Reverse *(down)* migrations are also essential to create and test. You never
know when you might need to hastily rollback that prematurely deployed release.
