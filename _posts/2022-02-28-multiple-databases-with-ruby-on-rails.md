---
title: Multiple databases with Ruby on Rails
published_at: 2022-02-28 12:13:14
updated_at: 2022-02-028 12:13:14
---

<p>Multiple database support was added in Rails 6.0 and now in the latest version 7.0 you can add multiple databases and replicas, automatically switch between connections, swap between the writer and replica, rails tasks and horizontal sharding. Load balancing replicas is not yet supported. So, if you have a version 6.0 and later then you are all set. But what if you are still running 5.2 or even earlier versions of Rails.</p>
<br />

#### Multiple databases in Rails 4

<p>You can keep your database configurations in one database.yml file or you can have a separate file for each database. In our example we are going to keep several database configurations in one database.yml.</p>

<pre>
defaults: &defaults
  adapter: postgresql
  encoding: unicode
  database: sdevelopment
  pool: 5
  username: isorsa
  password: password

first_database:
  development:
    database: isorsa1_development
    host: localhost
    <<: *defaults

second_database:
  development:
    database: isorsa2_development
    host: localhost
    <<: *defaults
</pre>

<p>Then, we will need to load database configurations. We will add database constants in the application.rb.</p>
<pre>
db_conf = YAML.load_file(Rails.root.join('config/database.yml'))

FIRST_DB_CONFIG = db_conf["first_database"][Rails.env]
SECOND_DB_CONFIG = db_conf["second_database"][Rails.env]
</pre>

<p>Then, if we want to run migrations on multiple database with one command we can add a new db:migrate task.</p>

<pre>
desc "Migrate multiple databases"

namespace :db do
  task :migrate do
    Rake::Task["db:migrate_first_database"].invoke
    Rake::Task["db:migrate_second_database"].invoke
  end

  task :migrate_first_database do
    ActiveRecord::Base.establish_connection(FIRST_DB_CONFIG)
    ActiveRecord::Migrator.migrate("db/migrate/first_database/")
  end

  task :migrate_second_database do
    ActiveRecord::Base.establish_connection(SECOND_DB_CONFIG)
    ActiveRecord::Migrator.migrate("db/migrate/second_database/")
  end
end
</pre>

<p>Connect your models to databases explicitly with establish_connection call.</p>

<pre>
class User < ActiveRecord::Base
  establish_connection FIRST_DB_CONFIG
end

class Document < ActiveRecord::Base
  establish_connection SECOND_DB_CONFIG
end
</pre>

<br />

#### Multiple databases in Rails 5

<p>In rails 5 we need to use ActiveRecord::MigrationContext instead of ActiveRecord::Migrator.</p>

<pre>
ActiveRecord::MigrationContext.new("db/migrate/").migrate(env_migration_version)
</pre>

<br />

#### Bonus

<p>Here is an example with a separate custom database.yml file and rake db tasks.</p>

<pre>
namespace :custom do
  desc "Configure database"
  task :set_custom do
    Rails.application.config.paths['db'] = ['custom_database']
    Rails.application.config.paths['db/migrate'] = ['custom/migrate']
    Rails.application.config.paths['config/database'] = ['config/custom_database.yml']
  end

  namespace :db do
    task migrate: :set_custom do
      Rake::Task['db:migrate'].invoke
    end

    task rollback: :set_custom do
      Rake::Task['db:rollback'].invoke
    end

    task drop: :set_custom do
      Rake::Task['db:drop'].invoke
    end

    task create: :set_custom do
      Rake::Task['db:create'].invoke
    end

    task version: :set_custom do
      Rake::Task['db:version'].invoke
    end

    task reset: :set_custom do
      Rake::Task['db:reset'].invoke
    end

    namespace :schema do
      task load: :set_custom do
        Rake::Task['db:schema:load'].invoke
      end

      task dump: :set_custom do
        Rake::Task['db:schema:dump'].invoke
      end
    end
  end
end
</pre>

<br />

#### References

<ol>
  <li>Multiple Databases with Active Record https://guides.rubyonrails.org/active_record_multiple_databases.html</li>
  <li>ActiveRecord::MigrationContext https://www.rubydoc.info/github/rails/rails/ActiveRecord/MigrationContext</li>
  <li>Padrino https://github.com/padrino/padrino-framework/blob/master/padrino-gen/lib/padrino-gen/padrino-tasks/activerecord.rb</li>
</ol>
