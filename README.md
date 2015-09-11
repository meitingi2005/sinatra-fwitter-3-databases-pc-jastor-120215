# Sinatra Fwitter 3 -  Databases

## Outline

1. Add Gems to our gemfile - `activerecord`, `sinatra-activerecord`, `rake`, `tux`, and `sqlite3`
2. Connect to our database in `config/environment.rb`
3. Create a migration for a table called `tweets` with up and down methods. The table should have columns for username and status. 
4. Migrate the database and update the tweet model so that it inherits from ActiveRecord::Base. Comment out or remove all of the methods and attributes. 
5. Update the application controller to use hash syntax to create new tweets. 

## Objectives

1. Create a connection to a sqlite database with activerecord in a Sinatra application.
2. Understand how to use ActiveRecord migrations to create a table. 
3. Update and use a model inheriting from ActiveRecord::Base. 
4. Create and persis tweets in our database

## Overview
We're back for Fwitter Part 3! We'll be incorporating a database into our application so we can persist out tweets.

## Instructions

### Setup
Fork and clone this repository to get started. 

### Updating Our Gemfile
First, we'll add some gems to our Gemfile that we'll need to setup our database. `activerecord` and `sinatra-activerecord` will setup our ActiveRecord magic for us, `rake` will help us easily create migration files for setting up our tables. We'll also be using `sqlite3` as our flavor of SQL in development.

```ruby
source "https://rubygems.org"

gem "sinatra"
gem "activerecord"
gem "sinatra-activerecord"
gem "rake"

group :development do
  gem "pry"
  gem "tux"
  gem "shotgun"
  gem "sqlite3" 
end
```

Run `bundle install` to add any needed gems and dependencies. 

### Connecting to our Database

Next, we need to setup a connection to our database. In our `environment.rb` file, add the following block of code. 

```ruby
require 'bundler'
Bundler.require

configure :development do
  set :database, "sqlite3:db/database.db"
end
```

### Setting up our Rakefile

We'll use `rake` to help us create our database and tables. Rake stands for "Ruby Make" - it's basically a way to make and automate certain tasks. From the terminal, run `rake -T` - you should see something like the following output: 

```bash
rake aborted!
No Rakefile found (looking for: rakefile, Rakefile, rakefile.rb, Rakefile.rb)
/Users/iancandy/.rvm/gems/ruby-2.2.1/bin/ruby_executable_hooks:15:in `eval'
/Users/iancandy/.rvm/gems/ruby-2.2.1/bin/ruby_executable_hooks:15:in `<main>'
(See full trace by running task with --trace)
```

This is because we don't have a Rakefile. Let's fix this error by creating one. Create a file called `Rakefile` in the root of this directory and add the following code snippets: 

```ruby
require 'sinatra/activerecord/rake'
require './config/environment' 
```

This will add the ActiveRecord rake tasks into our project. Save this file and run `rake -T` again - you should see a list of the available tasks.

```bash
rake db:create              # Creates the database from DATABASE_URL or con...
rake db:create_migration    # Create a migration (parameters: NAME, VERSION)
rake db:drop                # Drops the database from DATABASE_URL or confi...
rake db:fixtures:load       # Load fixtures into the current environment's ...
rake db:migrate             # Migrate the database (options: VERSION=x, VER...
rake db:migrate:status      # Display status of migrations
rake db:rollback            # Rolls the schema back to the previous version...
rake db:schema:cache:clear  # Clear a db/schema_cache.dump file
rake db:schema:cache:dump   # Create a db/schema_cache.dump file
rake db:schema:dump         # Create a db/schema.rb file that is portable a...
rake db:schema:load         # Load a schema.rb file into the database
rake db:seed                # Load the seed data from db/seeds.rb
rake db:setup               # Create the database, load the schema, and ini...
rake db:structure:dump      # Dump the database structure to db/structure.sql
rake db:version             # Retrieves the current schema version number
```
Nice work! If you're not sure the name of a Rake tast you want to run, you can always bring up this list with `rake -T`

 ### Creating our Migration
 
 Now, we're ready to create our migration. A migration is an ActiveRecord file which sets up our schema for us. Our migrations will create tables, add or remove columns, etc. It's common for projects to have many migration files, but for now we'll just need one: to create a table for tweets. 
 
 From the terminal, run `rake db:create_migration`. You should see the following output: 
 
 ```bash
 No NAME specified. Example usage: `rake db:create_migration NAME=create_users`
 ```
 
 What a helpful error - thanks, Rake! This says that we need to give our migration a name. In general, your migration names should describe what they are doing. Since we're creating a table called tweets, let's call this migraiton `create_tweets`. 
 
 Run the following command: `rake db:create_migration NAME=create_tweets`. This will create a directory called `db`. Inside of `db` will be a directory alled `migrate`, and inside of the `migrate` directory will be a file named someting like `20141022163315_create_tweets.rb`. The beginning is a timestamp - this is important, as it ensures that our migrations will run in order. 
 
 Inside of your migration file, ActiveRecord has stubbed out the migration for us. We have a class called `CreateTweets` which inherits from `ActiveRecord::Migration` and an empty method called `change`. Replace the `change` method with two methods - one called `up` and one called `down`. asdf

```ruby
 class CreateTweets < ActiveRecord::Migration
  def up
   
  end
  
  def down
   
  end
end
```

Inside of our `up` method, add the following block of code to create a table called `tweets`. **This is very important:** because we have a model called Tweet, we need to have a table called "tweets". ActiveRecord table names are always the plural of the model name. If we have a model called `Wolf`, it would correspond to a table called `wolves`. 

```ruby
 class CreateTweets < ActiveRecord::Migration
  def up
  	create_table :tweets do |t|
  	  t.string :username
  	  t.string :status
  	end
  end
  
  def down
   
  end
end
```

This will create a table called with three columns: an ID column which will function as the primary key gets created automatically, as well as two columns we define:  username and status.

The down method will function as the oppostie of our up, in case we need to rollback this migration. The opposite of `create_table` is `drop_table`, so let's add that.

```ruby
 class CreateTweets < ActiveRecord::Migration
  def up
  	create_table :tweets do |t|
  	  t.string :username
  	  t.string :status
  	end
  end
  
  def down
    drop_table :tweets
  end
end
```

Awesome. We're now ready to actually run this migration. From the terminal, run `rake db:migrate` - this will execute the code in our migration file. You should see something to the following output: 

```bash
== 20141022163315 CreateTweets: migrating =====================================
-- create_table(:tweets)
   -> 0.0156s
== 20141022163315 CreateTweets: migrated (0.0158s) ============================
```

Awesome! If you got an error message, read it carefully to debug your code. When you're ready, simply run rake db:migrate again. 

### Setting Up Our Model

Now that our database is setup, we need to connect our model to it. Our `Tweet` class should inherit from `ActiveRecord::Base`. 

```ruby
class Tweet < ActiveRecord::Base
  attr_accessor :username, :status
  
  ALL_TWEETS = []

  def initialize(username, status)
    @username = username
    @status = status
    ALL_TWEETS << self
  end

  def self.all
    ALL_TWEETS
  end
end
```
Here's where some more ActiveRecord magic comes in. ActiveRecord gives us tons of methods to extend the abilities of our class. 

## Resources

* [Stack Exchange](http://www.stackexchange.com) - [Some Question on Stack Exchange](http://www.stackexchange.com/questions/123)