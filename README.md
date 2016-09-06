# Rails Sidekiq Guide
A brief guide on sidekiq implementation in rails application.
### Gem Configuartion
You can install sidekiq directly into your system by running this command,
`gem install sidekiq`
Or you can add `gem 'sidekiq'` in you Gemfile and run `bundle` or `bundle install`.
Use a Worker to execute a background task.

Once you have a worker class defined, you can call its `perform_async` method to run the task in the background:

`SampleWorker.perform_async({record_id: 10})`

The code will be executed in the background, in a separate thread to the server process.You also need to have an instance of [Redis](http://redis.io/topics/quickstart) available and running on your server to deal with background and scheduled tasks.


### Define a Worker
The basic unit of Sidekiq is a Worker class. This provides an interface to the rest of the application for executing a pre-defined chunk of work or a common task.

Workers should be put in **app/workers/** which means you can define a worker like this:
app/workers/sample_worker.rb

```ruby
class SampleWorker
  include Sidekiq::Worker
  include Sidekiq::Schedulable
  
  def perform(args)
    # Logic defining your task
  end 
end 
```
 
 When executing a job, Sidekiq will call the perform method with any arguments passed in during queuing. Because arguments are serialized to Redis, it is best practice to pass in primitives as opposed to complex objects (e.g. a record ID, not the entire record object itself).

* Note that worker objects are just instances of a ruby class, and can therefore use methods, variables, and more just like any other ruby object.
 
### Calling Workers Methods
Just like creating a worker, calling worker method is simple and plain job. You can call worker method from controller and model. Call worker like:

`GenerateReport.perform_async(name, count)`

This will automatically initiate background worker to run in background thread.

### How to execute a background at a specific datetime
You can specify when to run a specific worker, by `perform_in(interval, *args)` or `perform_at(timestamp, *args)` instead of `perform_async`.

```ruby
SampleWorker.perform_in(10.days, record_id: 10)
SampleWorker.perform_at(10.days.from_now, record_id: 10)
```

### Start SideKiq In Development
Sidekiq runs using redis. For starting sidekiq one has to start redis-server first. Sidekiq can be started in development by following statement:

`bundle exec sidekiq`

### Sidekiq On Production
Starting sidekiq in production is different from development as in production you will need to demonize the sidekiq server. After demonizing the redis-server in production, you can start sidekiq as:

`RAILS_ENV=production nohup bundle exec sidekiq &`

`nohup` write the logs of sidekiq in nohup file and & demonize the server.

### Sidekiq Dashboard
Another wonderful feature that comes with sidekiq gem is its web dashboard. You can use this dashboard to monitor the jobs and see all threads in working. For adding sidekiq dashboard in application, add this in your routes.rb

```ruby
require 'sidekiq/web'
mount Sidekiq::Web => '/sidekiq'
```

Dashboard will open at domain/sidekiq. But this will give dashboard access to all authorized/unauthorized users. For giving dashboard acces only to authorized users add sidekiq in routes.rb like this:

```ruby
authenticate :user, lambda { |u| u.admin? } do
  mount Sidekiq::Web => '/sidekiq'
end
```
