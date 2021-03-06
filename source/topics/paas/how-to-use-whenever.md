One option for running scheduled tasks (cron jobs) on Aptible is to use the [Whenever gem](https://github.com/javan/whenever). Whenever helps manage the following problems, which can be challenging when managing crontab files directly:

1. Using `ENV` variables in cron jobs.
2. Logging cron job output to a predictable location.
3. Ensuring that crontab files are always properly formatted.

To use Whenever in your app, take the following steps. In our example, we will assume that the Rake task "db:awesome" needs to run every day at 2:00 UTC.

1. Add a `config/schedule.rb` file to your app's repo. The [Whenever README](https://github.com/javan/whenever#example-schedulerb-file) has examples of the schedule file format, but for example:

    ```ruby
    # schedule.rb
    set :output, '/var/log/whenever.log'

    # Inherit all ENV variables
    ENV.each { |k, v| env(k, v) }

    every 1.day, at: '2:00 am' do
      rake 'db:awesome'
    end
    ```

2. Add a line in your Dockerfile to create the Whenever log file (if you choose to use one):

        RUN touch /var/log/whenever.log

3. Add an entry to your Procfile to write the crontab, start cron, and then follow the logs written to the Whenever log file:

        web: # ...
        cron: whenever -w && cron && tail -f /var/log/whenever.log

Alternatively, you can write the crontab inside the Dockerfile (`RUN whenever -w`). If you do so, just make sure you're [appropriately accessing ENV variables](/topics/paas/how-to-access-environment-variables-inside-dockerfile) inside the Dockerfile. Also, remove `whenever -w &&` from your Procfile.