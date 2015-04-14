# monit-wrapper

This cookbook simplifies setting up services using Monit.

## Example

In `my_cookbook/templates/default/monit/my_service.conf.erb`:

```ruby
check process <%= @service_name %>
  matching '<%= @cmd_line_pattern %>'
  every 1 cycles
  start program "/bin/bash -c 'exec <%= @cmd_line %>'"
    as uid <%= @user %> as gid <%= @user %>
  stop program "/usr/bin/pkill -u <%= @user %> -f '<%= @cmd_line_pattern %>'"
    as uid <%= @user %> as gid <%= @user %>
```

In `my_cookbook/recipes/default.rb`:

```ruby

my_service_name = '...'
command_line = '/usr/local/bin/my_service_executable --port 3456'

monit_wrapper_monitor my_service_name do
  template_cookbook 'my_cookbook'
  template_source 'monit/my_service.conf.erb'
  variables cmd_line: command_line,
            cmd_line_pattern: command_line,
            user: user
end

monit_wrapper_notify_if_not_running monit_service_name

monit_wrapper_service my_service_name do
  subscribes :restart, "monit-wrapper_monitor[#{my_service_name}]", :delayed
  subscribes :restart, "monit-wrapper_notify_if_not_running[#{my_service_name}]", :delayed
  subscribes :restart, "package[#{my_service_name}]", :delayed
end
```

## License

Apache License 2.0

https://www.apache.org/licenses/LICENSE-2.0