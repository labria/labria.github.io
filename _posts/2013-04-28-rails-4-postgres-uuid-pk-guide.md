---
layout: post
title: "Rails 4 Postgres UUID Primary Key Guide"
description: ""
category: 
tags: [rails4, postgresql]
---
{% include JB/setup %}

## Basic setup
As noted in my previous post, Rails 4 now natively supports UUID primary keys in PostgreSQL. To create a table with a UUID simply create a migration (even simpler than I posted, [Aaron Patterson](http://twitter.com/tenderlove) pointed that I can just pass `id: :uuid` to the create_table method):
{% highlight ruby %}
create_table :users, id: :uuid do |t|
  t.string :name
  t.timestamps
end
{% endhighlight %}

This will execute the following SQL:
{% highlight sql %}CREATE TABLE "users" ("id" uuid PRIMARY KEY DEFAULT uuid_generate_v4(), "name" character varying(255), "created_at" timestamp, "updated_at" timestamp){% endhighlight %}

While this is good enough to get started, there are some quirks along the way that we'll need to overcome.
## Sorting
If you're not new to Rails, then you're used to the fact the calling `User.first` will fetch the first User object ever created. This is due to the fact that the default sorting is by `id`, which is usually a `AUTO_INCEREMENT` integer column, so the first one will have the smallest id. When we switch to UUID for PK, this property will be lost, because the UUID is generated randomly (by default) and does not guarantee sequentiality. There are two ways to overcome this (if you _do_ need to overcome this):
* Add a default scope to sort by `created_at`
* Switch to a different (from the default) UUID generation algorithm

While the first one is obvious, the second one needs some explanation.
There are [5 methods for generating a UUID](https://en.wikipedia.org/wiki/UUID). The default version used in Rails is version 4, a completely random ID. We can switch to version 1, which is the combination of the machine's MAC address and a timestamp, to get sequential UUID stings. This algorithm is somewhat less secure, because it exposes the generator machine MAC address and creation time, but this is not a really big security threat if you ask me. To switch the UUID generation to this algorithm we need to modify our migration:
{% highlight ruby %}
create_table :users, id: false do |t|
  t.primary_key :id, :uuid, :default => 'uuid_generate_v1()'
  t.string :name
  t.timestamps
end
{% endhighlight %}
This will force the generated UUIDs to be sequential and preserve the sorting we're used to.
## Relations
Since UUID primary keys are per-table and not a global setting (there is a plugin in the works to make it global), Rails has no way to tell if the primary key of a table is a integer or a UUID. Because of this, `t.references :user` will still generate a integer `user_id` field, which will obviously not work. The solution is simple:
{% highlight ruby %}
create_table :posts, id: :uuid do |t|
  t.string :title
  t.uuid :user_id
  t.timestamps
end
{% endhighlight %}
The same goes for polymorphic relations.
## Schema dumping
As Aaron mentioned on HN, ruby schema dumping does not work as expected yet (I noticed that it does not dump the :default for the primary key), you'll have to use SQL dumping for now.

## Enabling the extension
If your PostgreSQL user is a superuser, you can enable the extension right from a migration:
{% highlight ruby %}
enable_extension 'uuid-ossp'
{% endhighlight %}