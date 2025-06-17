# Rake/Capistrano

- [Rake/Capistrano](#rakecapistrano)
  - [Rake](#rake)
    - [Use other binaries as Ruby (eg. lldb)](#use-other-binaries-as-ruby-eg-lldb)
    - [Minitest](#minitest)
  - [Capistrano](#capistrano)
    - [Functions](#functions)
    - [Tasks/Actions](#tasksactions)
    - [Target hosts](#target-hosts)
      - [Hooks](#hooks)
    - [Useful stuff](#useful-stuff)
      - [Reboot servers](#reboot-servers)

## Rake

### Use other binaries as Ruby (eg. lldb)

If one needs Rake to use another binary, eg. lldb, it's possible to [hack the `RUBY` constant](https://git.io/JDKA9):

```
RUBY='lldb ruby --' rake test                             # via env var
FileUtils.const_set(:RUBY, "lldb #{FileUtils::RUBY} --")  # from the Ruby interpreter
```

### Minitest

Convenient task for running specified test suites/unit tests:

```rb
# Test suites are assumed to be under any subdirectory level of `test`, and with
# a filename ending with `_test.rb`.
#
# Arguments:
#
# - :test_suite_prefix: test suite prefix, in glob format; can include slashes.
#   defaults to '*'.
#   example: `foo/bar` will match `test/**/foo/bar_test.rb`
# - :test_bare_name: test name (without prefix); if nil, all the UTs are run.
#   defaults to run all the UTs.
#   example: `empty_array` will run `test_empty_array`
#
task :test, [:test_suite_prefix, :test_bare_name] => test_files + [:compile] do |_, args|
  Rake::TestTask.new do |t|
    test_suite_prefix = args.test_suite_prefix || '*'

    t.libs << "test"
    t.test_files = FileList["test/**/#{test_suite_prefix}_test.rb"]

    # This is somewhat hacky, but TestTask doesn't leave many... options 😬
    # See source (`rake-$version/lib/rake/testtask.rb`).
    #
    t.options = "-ntest_#{args.test_bare_name}" if args.test_bare_name

    t.verbose = true
    t.warning = true
  end
end
```

## Capistrano

### Functions

```rb
# Conditional
#
if test("[[ -f /path/to/file ]]"); ...; end

# Execute.
# WATCH OUT! Raises an error, if program fails. Use `test` in order to ignore the exit status.
#
execute :program, 'param1 param2'

# Execute and capture the output
# WATCH OUT! Raises an error, if program fails; see workaround below
#
output = capture :program, 'param1 param2'
output = capture :program, 'param1 param2 || true' # ignore exit with error
```

### Tasks/Actions

In order to rewrite a task, the previous one must be cleared:

```rb
Rake::Task['qualified_task_name'].clear_actions
```

If other tasks' callbacks invoke the it:

- if it's redefined, it will be invoked;
- otherwise, it's not invoked, and no error is raised.

Watch out! If the task is invoked by an :after hook, the hook won't be cleared; see [this question](https://stackoverflow.com/q/22712240).

### Target hosts

```rb
# on() takes also an array of names.
#
on [hostname1, hostname2] { |host| ... }

# The host where a given action is performed is passed by on(), so that it can be used to restrict execution.
#
on roles(:myrole) do |host|
  if capture("command") =~ /failure/
    execute 'handling'

    # This won't be work as intended (no scoping)!
    #
    # When filtering hosts via on(), don't forget use to_s(), otherwise an odd error is raised.
    #
    on host.to_s do
      invoke 'my:task'
    end
  end
end
```

#### Hooks

- `before "deploy:starting"`      : right before the deployment starts
- `after  "deploy:starting"`      : immediately after the deployment starts
- `before "deploy:updating"`      : before the deployment updates the code
- `after  "deploy:updated"`       : after the code has been updated on the server
- `before "deploy:publishing"`    : before the release is published (i.e., before the current symlink is updated)
- `after  "deploy:published"`     : after the release has been published
- `before "deploy:finishing"`     : before the deployment process is complete
- `after  "deploy:finished"`      : after the deployment process is complete

- `before "deploy" `              : before the whole process
- `after  "deploy" `              : after the whole process

- `before "deploy:reverting"`     : before the deploy is rolled back to a previous version
- `after  "deploy:reverted"`      : after the deploy has been rolled back

- `before/after "deploy:migrate"` : If you need to run tasks before or after the database migrations

### Useful stuff

#### Reboot servers

Can't use a straight reboot/poweroff, because when a connection fails on a host (due to shutdown), Capistrano may not invoke the command on the other hosts.

```sh
# Use `poweroff` to shutdown.
#
# Without `AccuracySec`, interval is 1 minute
# Without `RemainAfterElapse`, the unit is automatically re-runs
#
capistrano_shell $servers <<< `systemd-run --on-active=5s --timer-property=AccuracySec=1s --timer-property=RemainAfterElapse=yes /bin/systemctl reboot`
```
