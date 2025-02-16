# RubyGems/Bundler

- [RubyGems/Bundler](#rubygemsbundler)
  - [Bundler](#bundler)
    - [Gemfile](#gemfile)
    - [Cache directory](#cache-directory)
    - [Commands](#commands)
    - [Context](#context)
  - [Gem](#gem)
    - [Programmatic APIs](#programmatic-apis)
    - [Gemspec](#gemspec)
    - [Packaged gems/YAML gemspec](#packaged-gemsyaml-gemspec)
    - [Gem commands](#gem-commands)

## Bundler

### Gemfile

```ruby
source 'https://rubygems.org'

# If there is a gemspec: add here `gemspec`, with rake/rspec as dev dependencies.
#
gemspec

gem "json", "2.4.1" if RUBY_VERSION == "2.0.0"

# Local repository; path can be relative.
#
gem 'sshkit-sudo', '~> 0.2.0', path: '/path/to/sshkit-sudo'

# Git repository; `branch` is optional; `ref` and `tag` are also supported.
# `glob` allows installing from a repository subdir.
#
gem 'sshkit-sudo', git: 'https://github.com/ticketsolve/sshkit-sudo', branch: 'mytest', glob: 'subdir/*.gemspec'

# If there isn't a gemspec: add here the `development` group, with rspec.
# The rake gem can be excluded unless specifically required.
#
group :development do
  # gem "rake"
  gem 'rspec', '~> 3.10.0'
end

# The test group should include gems that are used only in the test env
#
group :test do
  gem "benchmark-ips", "~> 2.7.2"
  if Gem::Version.new(RUBY_VERSION) >= Gem::Version.new("2.5.0")
    gem "coveralls_reborn", "~> 0.21.0"
    gem "simplecov", "~> 0.21.0"
  end
end

# Multi-group gem.
# This specific case is a Rails default addition.
#
group :development, :test do
  gem 'byebug'
end
```

### Cache directory

It's possible to create a cache directory (`vendor/cache`), but it's all-or-nothing; it's no possible to keep a subset of the gems.

### Commands

- `bundle $command --gemfile=$gemfile` : specify the gemfile
- `bundler package`                    : download gems into `vendor/cache`

### Context

When one wants to run outside of the bundler context:

```rb
# Watch out! It's the *environment*, that is, any subshell, not the Ruby context.
#
Bundler.with_unbundled_env { `passenger-status` }
```

If one wants to run without Bundler, it's best to invoke `ruby` without Bundler, and if one really needs it at some point, can use `require "bundler/setup"`, which will activate Bundler.

## Gem

### Programmatic APIs

```rb
# Gems dir.
#
Gem::RUBYGEMS_DIR

# Easiest way to find a gem path.
# Apparently, there's no direct way to find this via gem (bundler can)
#
Gem::Specification.find_by_name('passenger').gem_dir

# Find the path of the binary of a given gem.
# The second version is the same as the first, with the addition that it activates the gem.
#
Gem.bin_path("passenger-enterprise-server", "passenger-status"[, <version])
Gem.activate_bin_path(...)

# Find a gem version.
# The first way is more practical, as it returns a Gem::Version, and it doesn't require finding the
# VERSION location.
#
# WATCH OUT! Ruby automatically loads the Bundler version corresponding to the Gemfile.lock, if found,
# so as of the time of writing, it hasn't been possible to find the system default version (including
# using `Bundler.with_unbundled_env`).
# If the Gemfile.lock Bundler version is not installed, then the system version will be loaded (but it
# will cause a corrupted bundler error (!) in some circumstances) (Passenger context; probably, because
# it doesn't automatically upgrade Bundler when required).
#
Gem.loaded_specs['activerecord'].version
require 'zip/version'; Zip::VERSION

# Compare (gem) versions!!!
#
Gem::Version.new('3.0') > Gem::Version.new('10.0')         # string-comparison returns true (wrong)!
Gem::Version.new('6.0-rc10') > Gem::Version.new('6.0-rc2') # RCs are also supported!
Gem.ruby_version > Gem::Version.new('10.0')                # ready API to retrieve the ruby version
```

### Gemspec

```ruby
require 'sshkit/sudo/version'

# See https://piotrmurach.com/articles/writing-a-ruby-gem-specification for a good summary.
#
# The choice to include test files is arbitrary; they're not strictly required, and they don't necessarily
# provide value. When not specifying them, add them to the Gemfile (see specific section).
#
Gem::Specification.new do |spec|
  spec.name    = "sshkit-sudo-next"
  spec.version = SSHKit::Sudo::VERSION
  spec.summary = "SSHKit extension, for sudo operation with password input."
  spec.authors = ["Saverio Miroddi"]

  spec.email        = ["saverio.pub2@gmail.com"]
  spec.description  = "SSHKit extension, for sudo operation with password input."
  spec.homepage     = "https://github.com/saveriomiroddi/sshkit-sudo-next"
  spec.license      = "MIT"

  spec.files                 = Dir["lib/**/*"]
  # RDoc scans by default only `lib`.
  spec.extra_rdoc_files      = Dir["README.md", "CHANGELOG.md", "LICENSE.txt"]
  spec.require_paths         = ["lib"]
  # Binaries location; the contained files included automatically.
  spec.bindir                = "exe"
  spec.required_ruby_version = ">= 2.0.0"

  spec.add_dependency "sshkit", "~> 1.20.0"

  spec.add_development_dependency "rake", "~> 10.0"
  spec.add_development_dependency "rspec", ">= 3.0"

  if spec.respond_to?(:metadata=)
    # There isn't a comprehensive reference; see https://guides.rubygems.org/specification-reference.
    #
    # Entries are optional; they'll show in the rubygems.org page.
    #
    spec.metadata = {
      # Restrict pushes to a single host (RubyGems 2.2.0+).
      "allowed_push_host" => "https://rubygems.org",

      "bug_tracker_uri"   => "https://github.com/saveriomiroddi/sshkit-sudo-next/issues",
      "changelog_uri"     => "https://github.com/saveriomiroddi/sshkit-sudo-next/blob/master/CHANGELOG.md",
      "documentation_uri" => "https://www.rubydoc.info/gems/sshkit-sudo-next",
      "homepage_uri"      => spec.homepage,
      "source_code_uri"   => "https://github.com/saveriomiroddi/sshkit-sudo-next"

      # Other entries:
      #
      #   "mailing_list_uri"
      #   "wiki_uri"
      #   "funding_uri"
    }
  end
end
```

### Packaged gems/YAML gemspec

Packaged gems (`.gem`) are tarballs. They need to contain a gem specification, which is generally `.gemspec file`.

When the gem spec is not present, it's likely in serialized (YAML) format; in this case, one needs to:

- unpack the package in a destination dir;
- manually untar the package in a temp dir;
- gunzip the `metadata.gz` file from the temp dir into a `$gemname.gemspec` file (inside the unpacked directory)

This will yield a valid gem source 😳

### Gem commands

```sh
gem unpack $gemfile                  # Unpack gem
gem build $gem.gemspec               # Build gem
gem push $gem-*.gem                  # Publish a built gem
gem fetch $gem                       # Download a gem
gem install $gem:$version            # Install the specific version of a gem
gem install $gem -- --build-flags... # Install a gem, with build flags
gem uninstall -ax $gem               # Uninstall, without versions/binaries prompt
gem dependency $gem                  # List dependencies

# Format for operations (`fetch`, `list -r`...) on remote authenticated repositories.
# `-s` appends a source; --clear-sources clears the preceding sources.
#
gem $operation --clear-sources -s $source $gem1 {$gem2...}
```
