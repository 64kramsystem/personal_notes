# Chef

- [Chef](#chef)
  - [Client](#client)
  - [Resource concepts](#resource-concepts)
    - [Attributes precedence/merging](#attributes-precedencemerging)
    - [Basic concepts](#basic-concepts)
    - [Subscribe/notify](#subscribenotify)
    - [Available variables](#available-variables)
    - [Lazy resource attribute loading](#lazy-resource-attribute-loading)
  - [Resources](#resources)
    - [`execute`](#execute)
      - [Execute as alternate user](#execute-as-alternate-user)
    - [`bash`](#bash)
    - [`cron`](#cron)
    - [File-related (`file`, `cookbook_file`, `directory`](#file-related-file-cookbook_file-directory)
    - [`link` (symlink)](#link-symlink)
    - [`line` (manual file editing)](#line-manual-file-editing)
    - [`mount`](#mount)
    - [`package`/`dpkg_package`](#packagedpkg_package)
    - [`remote_file`](#remote_file)
    - [`remote_dir` (copy directory trees)](#remote_dir-copy-directory-trees)
    - [`service`](#service)
    - [`systemd_unit`](#systemd_unit)
      - [User units](#user-units)
    - [User passwords metadata](#user-passwords-metadata)
  - [Tools](#tools)
    - [Knife](#knife)
      - [Installation on machines with Omnitruck-installed Chef](#installation-on-machines-with-omnitruck-installed-chef)
  - [Useful stuff](#useful-stuff)
    - [Define resource dynamically](#define-resource-dynamically)
    - [Setting env vars when not supported](#setting-env-vars-when-not-supported)

## Client

```sh
# --why-run        : dry run
# --format minimal : display reduced output, with changes summarized at the bottom (but no content changes)
#
chef-client --why-run --format minimal
```

## Resource concepts

### Attributes precedence/merging

WATCH OUT! It's necessary to specify `depends` in the cookbook metadata, in order to have the depending cookbook's attributes available.

Lower to higher:

- A default attribute located in a cookbook attribute file
- A default attribute located in a recipe
- A default attribute located in an environment
- A default attribute located in role
- A force_default attribute located in a cookbook attribute file
- A force_default attribute located in a recipe
- A normal attribute located in a cookbook attribute file
- A normal attribute located in a recipe
- An override attribute located in a cookbook attribute file
- An override attribute located in a recipe
- An override attribute located in a role
- An override attribute located in an environment
- A force_override attribute located in a cookbook attribute file
- A force_override attribute located in a recipe
- An automatic attribute identified by Ohai at the start of the chef-client run

Merging:

- An environment default Array attribute will merge the values with a recipe default one
- An environment overrider Array attribute will replace the values of a recipe default one

Commands:

```rb
# Idiomatic hash merge. WATCH OUT! This is a reverse merge, that is, the second hash is treated as
# "default values".
#
default[:attribute] = Chef::Mixin::DeepMerge.deep_merge(node[:myattribute], {foo: :bar})

# Array appending
#
default[:attribute] += [:foo, :bar]
```

### Basic concepts

```ruby
# Lazily execute a command
lazy { <commands> }

# Conditional resource execution
only_if { ::File.exist?(extract_path) }
not_if { ::File.exist?(extract_path) }

# Always use both; if `group` is omitted, it will default to `root`.
user  "application"
group "application"
```

### Subscribe/notify

By default, notifications are `:delayed` (executed at the end of the Chef client run).

WATCH OUT! A subscription depends on the subscribed resource *change*; it won't work as intended for resources that are in a binary state (e.g. packages).  
When using a package configuration pattern, subscribe to the configfile change, not the package's.

```ruby
file '/etc/nginx/ssl/example.crt' do
  mode  '0600'
  owner 'root'
end

service 'nginx' do
  action :nothing

  # Don't forget that the :subscribes action is the action of the enclosing resource!
  # Only one action per statement can be subscribed.
  # If the subscribed resource doesn't exist, there's no side effect.
  #
  subscribes :reload, 'file[/etc/nginx/ssl/example.crt]', :immediately

  # This is rare, but a package may actually not enable the service; in this case, depending on the
  # package (not to the config) is appropriate.
  #
  subscribes :enable, 'dpkg_packge[nginx]', :immediately
end

# Notification (push).
#
template '/path/to/myconfig.json' do
  # An error is raised if the notified resource doesn't exist.
  notifies :restart, "service[myservice]", :immediately
end
```

### Available variables

```ruby
Chef::Config[:cookbook_path]      # Cookbook path, usable in recipes
```

### Lazy resource attribute loading

Force a block to be evaluated during the execution stage; very useful to gather values of resources created by chef.

```rb
file '/etc/audit/rules.d/audit.rules' do
  content lazy { "-a always,exit -F arch=b64 -S execve -F auid=#{`id -u myuser`.chomp}" }
end
```

## Resources

The default attribute values (e.g. `action`) are the ones specified in the example resource, unless specified.

### `execute`

Execute a single command; don't set multiple `command` properties, otherwise they will be all executed even if one of them fails.

WATCH OUT!! The default shell is dash; using bashisms, e.g. `rm /{foo,bar}`, won't work, and with the mentioned example, no errors were even raised.

```ruby
execute 'apache_configtest' do
  command '/usr/sbin/apachectl configtest' # default: resource name
  action  :run
  login   true # Run as login shell; see subsection below.
end
```

#### Execute as alternate user

The `execute` resource runs a non-login shell by default; this may have unintended side effects. In order to run a login shell:

- either use `login: true`
- or set the related env variables, e.g. `HOME`; see [this article](https://saveriomiroddi.github.io/Chef-properly-run-a-resource-as-alternate-user) for a full list.

```rb
execute 'apache_configtest' do
  command '/usr/sbin/apachectl configtest'
end
```

### `bash`

Similar to `execute` resource; the difference is that the `code` content is written to a temporary file; use this when executing multiple commands.

Either concatenate the commands with `&&` or add `set -o errexit`.

```ruby
bash 'extract_module' do
  cwd    ::File.dirname(src_filepath)
  code   <<~SH
    set -o errexit

    command1
    command2
  SH
  action :run
end
```

### `cron`

```ruby
cron 'job_name' do
  command          String

  minute           Integer, String         # default value: "*"
  hour             Integer, String         # default value: "*"
  day              Integer, String         # default value: "*"
  month            Integer, String         # default value: "*"
  weekday          Integer, String, Symbol # default value: "*"

  user             String                  # default value: "root"

  environment      Hash
  home             String
  mailto           String
  path             String
  shell            String
  time             Symbol
  time_out         Hash

  action           :create                 # also supports :delete
end
```

### File-related (`file`, `cookbook_file`, `directory`

```ruby
cookbook_file '/etc/systemd/system.conf' do
  source    'etc/systemd/system.conf'
  mode      '0644'        # best to always specify; the default (not in all case) is 0777 + umask -> typically 0755.
end
```

Delete files/dirs found via glob:

```rb
# Files are found during compilation.
#
Dir[glob].each do |path|
  # WATCH OUT!!! Symlinks cause an error! Must use :link resource (if needed, test via `File.symlink?`).
  #
  file path do
    action :delete
  end
end
```

Watch out! One can't use two resources with the same name, for creation and (cleanup-time) deletion, as they will at least cause problems with notifications.

```ruby
directory '/etc/apache2' do
  mode      '0755'        # default = 0777 + umask -> typically 0755; if recursive, the permission are applied to the entire created tree
  recursive false         # If true and :create, owner/group apply only to the leaf!
  owner     'willy'
  group     'willy'
  action    :create       # :delete (don't forget :recursive=true!)
end

# Create a directory tree, owned by the given user.
#
Pathname.new(full_path).descend do |path|
  directory path do
    owner     "myuser"
    group     "myuser"
    # WATCH OUT! May want to skip the existing paths, if they'as they're owned by root.
    not_if { Dir.exist?(path) }
  end
end
```

### `link` (symlink)

```rb
link '/tmp/file' do
  to '/etc/file'
end
```

### `line` (manual file editing)

If manual editing is required, **do not use Chef::Util::FileEdit** as widely suggested (see https://git.io/JURBC), instead, use this cookbook.

Examples (see [repo](https://supermarket.chef.io/cookbooks/line)):

```ruby
# Append a line, if it doesn't exist; matches exactly.
#
append_if_no_line "title" do
  path "/path/to/file"
  line "exact_line_content"
end

# Replaces a line, or appends it; matches a pattern.
#
replace_or_add "title" do
  path              "/path/to/file"
  pattern           "regex.*"
  line              "replacement"

  remove_duplicates false
  replace_only      false            # don't append, if there are no matches
end
```

### `mount`

```ruby
# Generates:
#
# "fs-id:/dir /path/to/mountdir efs defaults,tls,accesspoint=fsap-id 0 0"
#
mount "/path/to/mountdir" do
  device  "fs-id:/dir"
  fstype  "efs"                               # default: `auto`
  options "defaults,tls,accesspoint=fsap-id"
  pass    0                                   # default: 2

  action [:enable, :mount]
end
```

### `package`/`dpkg_package`

There is currently no way of installing a local deb package, and pull the dependencies (the `gdebi` cookbook does it, but it's unmaintained).

```ruby
package 'package_name' do
  action :install                         # :purge, ...
  version '1.3.4-2'                       # optional; ignores hold status, so requires the option below
  options  "--allow-change-held-packages" # without this, upgrading a held package will cause an error
end

# Version with multiple packages; it installs them faster, and has a more compact output.
# The `version` attributes takes an Array, in this case.
#
package %w(package1 package2)

# Install a deb package from a local file.
# Ignores the hold status.
# Can't install from HTTP (as of Sep/2024, documentation is ambiguous in a place).
#
# WATCH OUT! Passing multiple package names+versions to it is supported, but it seems it doesn't work
# as intended - in one case, it invoked `dpkg -i` for a package, even if unnecessary (see below).
#
dpkg_package 'package_name' do
  source '/path/to/local_filename.deb'
end

# Conditionally hold a package.
#
execute "hold #{name}" do
  command    "apt-mark hold #{name}"
  not_if     { !!`dpkg --get-selections #{name}`[/\bhold$/] }
end
```

Working replacement of multipackage dpkg_package resource:

```rb
packages_to_install = -> do
  package_filenames                      # hash name => filename
    .filter_map do |name, filename|
      deb_version = `dpkg -I #{filename}`[/^ Version: (\S+)/, 1] || raise("Version not found for .deb #{filename}")
      # We need to be reasonably fast here, but dpkg's status needs to be inspected carefully.
      installed_version = `dpkg -s #{name} 2> /dev/null`[/^Status: [^\n]+ installed$.+^Version: ([^\n]+)/m, 1]

      [name, filename] if deb_version != installed_version
    end
    .to_h
end

dpkg_package package_filenames.keys do
  package_name lazy { packages_to_install[].keys }
  source lazy { packages_to_install[].values }
  not_if { packages_to_install[].empty? }
end
```

### `remote_file`

```ruby
remote_file local_tarball_filename do
  source node[:mysql_server][:tarball_uri]
end
```

### `remote_dir` (copy directory trees)

```rb
remote_directory '/path/to/dest' do
  source 'path/to/source'
  owner  'myuser'
  group  'myuser'
end
```

### `service`

If handling a systemd unit, use the [specific resource](#systemd_unit).

### `systemd_unit`

The resource can be specified without content, useful for example to enable the unit.

WATCH OUT!! Just because a unit is enabled by the vendor, it doesn't mean that it will actually be enabled!!

```ruby
# WATCH OUT! The `.service` suffix is necessary.
#
systemd_unit 'nmon.service' do
  # WATCH OUT!! This does *not* create a user unit; as of May/2023, it seems that user units must be
  # manually created.
  #
  user 'myuser'

  content <<~UNIT
    [Unit]
    Description=Nmon system stats recording

    [Service]
    Type=forking

    StandardOutput=syslog
    StandardError=syslog
    SyslogIdentifier=nmon

    ExecStart=$nmon_location -F #{node[:monitoring][:nmon_stats_file]} -s #{node[:monitoring][:nmon_collection_interval]}
    ExecReload=/bin/kill -HUP $MAINPID

    Restart=on-failure

    [Install]
    WantedBy=multi-user.target
  UNIT

  # Systemd can be overly strict when verifying units, so in certain cases it is preferable not to verify the unit.
  #
  verify false

  action [:create, :enable, :reload_or_restart] # Can also :delete (systemd reload is triggered as usual)
end
```

#### User units

User units are a bloody nightmare. The `execute` resource makes systemd complain about not connecting to the bus, even with `login` set, and command `systemctl ... --machine=app@.host`; using a script is the easiest solution.

```rb
file myservice_unit_file do
  # No need to set User/Group for user units in the unit definition.
  content mycontent
  owner   'myuser'
  group   'myuser'
  mode    '0644'
end

# Can't use `enable --now`, because it doesn't restart the service if it's running already.
#
execute 'enable myservice.service' do
  # Must run as sudo!
  command    'systemctl enable --user --machine=myuser@ myservice.service'
  action     :nothing
  subscribes :run, "file[#{resque_web_unit_file}]", :immediately
end

execute 'restart...' # same, with `restart` instead of `enable`.
```

### User passwords metadata

In order to get password metadata from the users, the `ruby-shadow-ruby32` gem is required:

```rb
gem_package "ruby-shadow-ruby32" { action :install }
require 'shadow'

pass_meta = Shadow::Passwd.getspnam(login)
pass_meta.sp_max                             # password age
Date.new(1970, 1, 1) + pass_meta.sp_lstchg   # expiry date
```

## Tools

### Knife

Execute code on nodes from the commandline:

```sh
# Generally speaking, it's good practice to print the node name, as it's easy to miss them in the search
# and not notice.
#
# `exec` can also receive a string, using the `-E` option, but heredoc is clearer.
# Including recipes doesn't work.

knife exec << 'RUBY'
  # Can pass :all to match all the nodes.
  # transform() is often used, but it's undocumented (it's located at `chef-X.Y.Z/lib/chef/shell/model_wrapper.rb`);
  # a node is saved depending on the truthiness of the value returned by the block.
  #
  nodes.search('name:web*') do |node|
    # Chef::Node [documentation](rubydoc.info/gems/chef/Chef/Node).
    #
    puts "- #{node.name}: #{node.run_list}"

    # Chef::RunList [documentation](rubydoc.info/gems/chef/Chef/RunList).
    #
    node.run_list.reset!('role[app_server]')

    node.save
  end
RUBY
```

Manipulate a node attribute:

```sh
# There is no command for attributes manipulation, so we use the strategy above.
#
knife exec --exec 'nodes.search("name:vmhannesbenson.ts.int") { |n| n.normal_attrs.delete("system_base"); n.save(); puts "- [√] #{node.name}" }'
```

Other commands:

```sh
knife node edit --all $node_name
```

#### Installation on machines with Omnitruck-installed Chef

From v17, the knife tool is separate, since the Chef ecosystem has changed design.

For systems configured with the old design, the knife gem is a PITA, because it's not available for installation via omnitruck tool, and when installed via gem, it pulls all the Chef gems, which conflict with the Chef debian package (installed via Omnitruck).

Fortunately, the solution is simple - install it as `chef_gem`, then symlink it (it will use the bundled Ruby):

```sh
ln -s /opt/chef/embedded/bin/knife /usr/bin/knife
```

As of Jun/2023, the Chef gems are not compatible with Ruby 3.2, and must be manually patched:

```sh
find "$(ruby -e 'print Gem::Specification.find_by_name("knife").gem_dir')" -name '*.rb' -type f | xargs perl -i -pe 's/\buntaint\b/itself/'
find "$(ruby -e 'print Gem::Specification.find_by_name("chef").gem_dir')"  -name '*.rb' -type f | xargs perl -i -pe 's/File\.exist\Ks//'
```

An alternative solution that has been explored, is to install the chef gem binaries to a separate dir - something like:

```sh
gem install chef-vault chef-config:=17.0.242 chef-utils:=17.0.242 chef:=17.0.242 --bindir=/opt/chef-gem
gem install knife --version '=17.0.246' # keep reference: 17.0.242
```

Besides being a hack, it depends on the system Ruby, which is a source of problems.

## Useful stuff

### Define resource dynamically

Example of defining symlink resources, based on the content of a directory:

```rb
ruby_block "Ruby link resource definitions" do
  block do
    Dir.glob("/usr/lib/myruby/versions/myver/bin/*")).each do |binary_path|
      symlink_path = File.join("/usr/bin", File.basename(binary_path))

      Chef::Resource::Link.new(symlink_path, run_context).tap do |link_resource|
        link_resource.to         binary_path
        link_resource.run_action :create
      end
    end
  end
end
```

### Setting env vars when not supported

Use this workaround:

```rb
ruby_block "Set MAKE env var to CPU threads count" do
  block    { ENV["MAKE"] = "make -j #{node[:cpu][:total]}" }
  action   :nothing
  notifies :run, "gem_package[mygem]", :before
end

gem_package "mygem"

ruby_block "Unset MAKE env var" do
  block    { ENV["MAKE"] = nil }
  action   :nothing
  notifies :run, "gem_package[mygem]", :immediately
end
```
