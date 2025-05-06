# Alchemy CMS with Rails 8 and Bootstrap 5 - prepared to deploy with dokku

this repository is a basic starting point to setup MVP Web-Applications, and include  
* Alchemy CMS
* Some CMS Elements, to build beautiful pages
* Rails 8
* Bootstrap 5
* Dokku deployment ready

## Getting started

### Installing Ruby and Ruby on Rails
Have a look at:
https://guides.rubyonrails.org/
have a look at Michael Hartl's excellent [tutorial](https://railstutorial.org/book/).

Make sure you install the correct versions.

### Project Setup

* Set up a local git repository
  ```sh
    git clone git@github.com:schmidie/alchemy_cms.git
  ```

* Install required gems
    ```sh
      bundle install
    ```

* Set up credentials
    ```sh 
      EDITOR="code --wait" rails credentials:edit --environment development
    ```

* set up Postgres for development environment
  * you can use the docker service (use port `54321` and update `config/database.yml` accordingly)

* set up database, load schema and seed
  ```sh
    rails db:prepare db:migrate db:seed
  ```

* you may like to adopt the config/database.yml according to your wishes

* start rails & yarn
  ```sh
    bin/dev
  ```

### Run tests
to run tests you can
```sh
rails test:all
```

### Background Processing
To start background processing (`solid_queue`) in development:
```sh
  bin/rails solid_queue:start
```

### Dokku Setup
You need to setup

* setup postgres
  
* scale at least one worker process
  ```sh
    dokku ps:scale yourapp worker=1
  ```

* setup env variables
  ``` sh
    dokku config:set --no-restart yourapp RAILS_MASTER_KEY=your-master-key
    dokku config:set --no-restart yourapp RAILS_ENV=production
  ```

* mount storage (or setup S3)
  ``` sh
  dokku storage:mount yourapp /var/lib/dokku/data/storage/yourapp/log:/app/log
  dokku storage:mount yourapp /var/lib/dokku/data/storage/yourapp/storage:/app/storage
  dokku storage:mount yourapp /var/lib/dokku/data/storage/yourapp/uploads:/app/uploads
  ```

* make sure that all storage directories (and sub directories) have correct permission. if the owner is not 32767:32767 you can change it with:
  ```sh
  sudo chown -R 32767:32767 *
  ```

## Advanced

### Separtaed Databases for SOLID (Queue, Cache & Cable)

we use a single database for app, queue, cache & cable. for small MVPs this should be fine (and you do not have to do anything).
For scaling the app this should be separated.
Therefor have a look at and change again to the original setting:

https://github.com/rails/solid_queue?tab=readme-ov-file#single-database-configuration
https://github.com/rails/solid_cable?tab=readme-ov-file#single-database-configuration
https://github.com/rails/solid_cache

this should include for all SOLIDs something like:

* database.yml
  ```yml
  +  queue:
  +    <<: *default
  +    database: storage/development_queue.sqlite3
  +    migrations_paths: db/queue_migrate
  ```

* config/environments/production.rb
  ```yml
  config.solid_queue.connects_to = { database: { writing: :queue } }
  ```