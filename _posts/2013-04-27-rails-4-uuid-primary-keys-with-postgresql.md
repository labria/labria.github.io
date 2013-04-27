---
layout: post
title: "Rails 4 UUID primary keys with PostgreSQL"
description: ""
category: 
tags: [rails4, postgresql]
---
{% include JB/setup %}

UPD: See my second, more detailed [post](/2013/04/28/rails-4-postgres-uuid-pk-guide/) about the matter.

Aaron recently added [uuid primary key support](https://github.com/rails/rails/commit/bc8ebefe9825dbff2cffa29ff015a1e7a31f9812) to Rails4 edge. To try it in your app simply create a migration like this:
{% highlight ruby %}
create_table :users, id: false do |t|
  t.primary_key :id, :uuid
  t.string :name
  t.timestamps
end
{% endhighlight %}
Rails now automatically sets the default value to `uuid_generate_v4()` for the primary_key.

You'll of course need to run `CREATE EXTENSION "uuid-ossp";` in Postgres, either manually or with a migration via a `execute` statement.