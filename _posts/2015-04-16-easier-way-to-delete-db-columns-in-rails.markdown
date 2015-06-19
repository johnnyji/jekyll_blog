---
layout: post
title: "Easier Way to Delete DB Columns in Rails"
date: 2015-04-16
categories: Rails
---

Morning lecture time here at [Lighthouse Labs][lighthouse], and I've found the solution to one of the more annoying problems I've been having. How to remove migrations from a project without having to generate a drop migration file. (So your migrate folder stays slim and clean)

Suppose I created a migration called `add_gender_to_users`, but I later decide that my app doesn't want to specify user gender. So what do I do?

I can either create ANOTHER migration called `remove_gender_from_users`, or I can run any of these easier tasks:
<br><br>

1. Run rake `db:migrate:down VERSION=<your migration version number>` on the migration BEFORE the one you want to delete, now go into your migrate folder and delete the migration you no longer need and then run `rake db:migrate`, so your schema will now be updated but this time without that migration file that you deleted.

2. In your rails console, run `ActiveRecord::Migration.remove_column :table_name, :column_name`, this will effectively update your database, now all you need to do is remove that migration file before updating your schema.


[lighthouse]: http://lighthouselabs.ca
