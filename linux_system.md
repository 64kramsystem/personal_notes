# Linux system

- [Linux system](#linux-system)
  - [Packages](#packages)
  - [Debconf](#debconf)
  - [Debian alternatives](#debian-alternatives)
  - [Environment](#environment)
  - [Systemctl](#systemctl)
  - [Terminal](#terminal)
  - [Desktop Environment: windows](#desktop-environment-windows)
  - [MIME (extensions) (file associations) handling](#mime-extensions-file-associations-handling)
    - [Adding and associating a new application](#adding-and-associating-a-new-application)
    - [Split MAFF association](#split-maff-association)

## Packages

Informations:

```sh
# Show package info; `apt` is considered not stable, and shows a warning
#
apt-cache show $package

# Check if a package is installed (**with pipefail**), in the most standard way possible
#
if grep -qP "^trash-cli\s+install$" <<< $(dpkg --get-selections); then echo installed; fi

# Advanced filtering with aptitude.
#
# Aptitude uses regexes by default, but it doesn't support all the patterns.
# In order to filter out i386 packages, use the suffix `~rnative`.
# The option `-F %p` prints only the package name.
#
aptitude search -F %p 'suld-driver2-[0-9.]+$~rnative'

# Show dependencies of a package
#
aptitude why -v $package

# Display the sources of given package
#
apt-cache policy $package

# Find the repository of a package
#
apt-cache showpkg $package

# (Extra tool) Check dependencies of installed package.
#
apt-rdepends $package
```

Package/file operations:

```sh
# Downgrade to previous version of a package
#
apt-get install $package=$version

# Install/remove package ignoring dependencies, via dpkg
#
dpkg (-i|-r) --ignore-depends=libdbd-mysql-perl libmysqlclient18

# Download the packages without installing (dependencies are not downloaded).
#
aptitude download ($packages)

# Download source code for package
#
apt-get source $package

# List the contents of a deb file
#
dpkg-deb -c $package.deb

# Unpack deb file
#
ar -vx $package.deb
tar xvf data.tar.gz

# Find which package a file belongs to
#
dpkg -S $file
```

Useful generic snippets:

```sh
# Remove linux kernels other than the current one
# Note that if there is a version newer than the current (uname -r), it will be deleted.
#
dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | xargs sudo apt-get -y purge

# Purge packages matching a pattern, only if they're installed.
# apt can't search for a pattern and an installation status in one pass
#
aptitude purge '~i ruby1.9'
apt list --installed | perl -ne 'print "$1\n" if /^(<pattern>)\//' | xargs apt purge
```

## Debconf

```sh
debconf-show packagename                      # show properties of an installed package
debconf-get-selections | grep packagename/    # more detailed, but requires `debconf-utils package`
```

Example of seeding:

```sh
echo "zfs-dkms zfs-dkms/note-incompatible-licenses note true" | debconf-set-selections
```

## Debian alternatives

Display the current info (for machine parsing).

```sh
# Sample:
#
#   Name: ruby
#   Link: /usr/bin/ruby
#   Status: auto
#   Best: /usr/bin/ruby2.6
#   Value: /usr/bin/ruby2.6
#   
#   Alternative: /usr/bin/ruby2.5
#   Priority: 9999
#   
#   Alternative: /usr/bin/ruby2.6
#   Priority: 9999
#
update-alternatives --query <program>
```

Install an alternative:

```sh
# - format: <file path> <alternative name> <alternative path> <priority>
# - 9999 is high priority, and will (likely) set the alternative as default.
# - If the specified alternative exists, it will be updated.
# - "Slaves" are alternatives that depend on the "master" (e.g. irb -> ruby)
#
update-alternatives --install /etc/mysql/my.cnf my.cnf "$(pwd)/config/my.travis.cnf" 9999 \
                    --slave   /path/to/slave    slave  /path/to/slave_file
```

Clear the alternatives:

```sh
update-alternatives --remove-all ruby
```

Example: set `/etc/alternatives/editor` to vim (requires the alternative to be installed):

```sh
update-alternatives --set editor /usr/bin/vim
```

## Environment

Current user run dir:

```sh
$XDG_RUNTIME_DIR
```

## Systemctl

List everything (with their states), including timers:

```sh
systemctl -a
```

## Terminal

Open a new tab, execute a command, and leave the tab open:

```sh
mate-terminal --tab --command 'sh -c "echo abc; zsh"'
```

## Desktop Environment: windows

Bringing a window to the front:

```sh
# Sample window names
#
xdotool getwindowname "$window_id"
# Google - Mozilla Firefox (Private Browsing) # private with website
# Mozilla Firefox                             # standard, with blank page

# Search only in the current desktop.
#
desktop_id=$(xdotool get_desktop)

# For a brief moment, xdotool changes the terminal name to the command, so if we search for a
# string, the terminal will be found as well! So one needs to make sure that the name regex doesn't
# match itself (any metacharacter will do).
#
# `--all` is important, otherwise, each token of the `--name` parameter is an independent match!
#
# The windows are listed is reverse access order (last accessed is at the bottom)
#
window_id=$(xdotool search --all --desktop "$desktop_id" --name 'Mozilla Firefox \(Private Browsing\)$' | tail -n 1)

# Bring to front.
#
xdotool windowactivate "$window_id"
```

## MIME (extensions) (file associations) handling

```sh
# Find the MIME type.
# More accurate than `file --mime-type`;  on Markdown docs for example, the type found was `text/plain`.
#
mimetype "file"

# Open a file using MIME.
# On types which are not set yet, default will be asked.
#
xdg-open "file_or_uri" # supports URI (see https://wiki.archlinux.org/index.php/default_applications)
mimeopen "file"

# Find/set application default.
#
xdg-mime query default "x-scheme-handler/http"
xdg-mime default "sublime_text.desktop" "text/markdown"

# Reset custom associations.
#
rm "~/.local/share/applications/wine*.desktop"
rm "~/.local/share/applications/mimeinfo.cache"
```

### Adding and associating a new application

```sh
# Partial; see `/usr/share/applications/firefox.desktop` for a full example.
#
cat > ~/.local/share/applications/ << 'CONF'
[Desktop Entry]
Version=1.0
Name=Sav's Browser
...
CONF

# Optional: add to MATE menu.
#
perl -i -lpe '$. == 1 && print "location:/home/saverio/.local/share/applications/browser.desktop"' ~/.config/mate-menu/applications.list

# Register!
#
xdg-mime default "browser.desktop" "x-scheme-handler/http"
xdg-mime default "browser.desktop" "x-scheme-handler/https"
```

### Split MAFF association

Associate an extension based on the filename rather than the MIME type:

```sh
cat > /usr/share/mime/packages/maff.xml << 'XML'
<?xml version="1.0" encoding="UTF-8"?>
<mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
<mime-type type="application/x-maff">
<comment>Maff File</comment>
<glob pattern="*.maff"/>
</mime-type>
</mime-info>
XML

update-mime-database /usr/share/mime
```
