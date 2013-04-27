---
layout: post
title: "Rails 4 UUID primary keys with PostgreSQL"
description: ""
category: 
tags: [rails4, postgresql]
---
{% include JB/setup %}

Aaron recently added [uuid primary key support](https://github.com/rails/rails/commit/bc8ebefe9825dbff2cffa29ff015a1e7a31f9812) to Rails4 edge. To try it in your app simply create a migration like this:
	
	create_table :users, id: false do |t|
	  t.primary_key :id 
	  t.string :name
	  t.timestamps
	end		

You'll of course need to run `CREATE EXTENSION "uuid-ossp";` in Postgres, either manually or with a migration via a `execute` statement.