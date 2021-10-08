# Rake/Capistrano

- [Rake/Capistrano](#rakecapistrano)
  - [Rake](#rake)
    - [Minitest](#minitest)
  - [Capistrano](#capistrano)
    - [Tasks/Actions](#tasksactions)

## Rake

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

### Tasks/Actions

In order to rewrite a task, the previous one must be cleared:

```ruby
Rake::Task['qualified_task_name'].clear_actions
```

If other tasks' callbacks invoke the it:

- if it's redefined, it will be invoked;
- otherwise, it's not invoked, and no error is raised.

Watch out! If the task is invoked by an :after hook, the hook won't be cleared; see [this question](https://stackoverflow.com/q/22712240).
