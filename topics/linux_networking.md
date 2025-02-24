# Linux networking

- [Linux networking](#linux-networking)
  - [High-level services](#high-level-services)
    - [wget](#wget)
    - [curl](#curl)
  - [rsync](#rsync)
    - [Sending emails](#sending-emails)
    - [SSH/utilities](#sshutilities)
    - [SSL/Certificates](#sslcertificates)
    - [PGP (GnuPG/gpg)](#pgp-gnupggpg)
      - [Key servers](#key-servers)
    - [Samba (SMB)](#samba-smb)
  - [Network manager client (`nmcli`)](#network-manager-client-nmcli)
  - [Lower level networking](#lower-level-networking)
    - [SSL](#ssl)
    - [DNS](#dns)
    - [Netcat (nc)](#netcat-nc)
    - [Diagnose/measure traffic](#diagnosemeasure-traffic)
    - [Sniff network traffic](#sniff-network-traffic)
    - [Port scan (nmap)](#port-scan-nmap)
    - [Unix Sockets](#unix-sockets)
    - [Disable ipv6](#disable-ipv6)
  - [Interfaces](#interfaces)
    - [Basic configuration](#basic-configuration)
    - [NetworkManager](#networkmanager)
      - [Changing Network manager DNS (18.04+)](#changing-network-manager-dns-1804)
    - [Netplan](#netplan)
    - [`ip` tool](#ip-tool)
      - [iproute2](#iproute2)
      - [Create a TUN/TAP device](#create-a-tuntap-device)
    - [Wi-Fi](#wi-fi)

## High-level services

Find process listening on port:

```sh
sudo lsof -i :$port
fuser -n tcp -v $port               # [n]amespace; necessary; can only specify one
sudo netstat -lnp | grep :8000      # [l]istening only (convenient); n: no resolution, p: program names
```

### wget

Options:

- `--user-agent=Mozilla`      : use this is a website refuses download
- `-P|--directory-prefix $dir`: download to a specific directory (`P`refix)

### curl

Options:

- `-d`, `--data`
- `-X`, `--request <command>`
- `-H`, `--header <header/@file>`
- `-i`, `--include`: include headers in the response
- `-s`, `--silent`: quiet mode - errors are not displayed; best used with `-S`
- `-S`, `--show-error`: display errors also in quiet mode

```sh
# Bare form: implies `-X GET`
#
curl "$URL"

# `-d` implies `-X POST`.
# In this case, the data format must be specified.
#
json_data=$(
  jq -n \
    --arg channel "$channel" \
    --arg text "$text" \
    '{ channel: $channel, text: $text }'
)
curl \
  -H "Authorization: Bearer $token" \
  -H 'Content-type: application/json; charset=utf-8' \
  -d "$json_data" \
  "https://slack.com/api/chat.postMessage"

# Send an HTTP request for a specific format
#
curl -H 'Accept: application/halo+json' "$URL"
```

## rsync

- `-n|--dry-run`   : test (dry run)
- `--delete`       : delete from destination; if `--exclude` is specified, it applies to both source and destination
`--chmod=o-rwx -p` : preserve permissions
- `-l|--links`     : copy symlinks as symlinks
- `-r`             : recursive
- `-z`             : compress

`-ptgoD`				  : preserve [p]ermissions, modification [t]imes, [g]roup, [o]wner (super user only), [D]evices/specials
`-HAX`            : preserve [H]ard links, [A]cls (implies -p), e[X]tended attributes
`-a`      				: works as cp -a; implies `rl` and `ptgoD`, but not `HAX`

- `-P|--progress` : applies per-file; not good as global indication
- `--info=progress2 --no-inc-recursive` : best way of showing global progress (!! but still inaccurate !!)
- `-h|--human-readable` : good to use with progress indicators

- `--exclude $glob` : examples:
                      `*.foo`       # any file
                      `foo/`        # any directory
                      `/foo`        # root directory
                      `foo/*/bar`   # any file two levels under a `foo`
                      `foo/**/bar`  # any file at any level under a `foo`

`--link-dest=$old_bkup_dir` : create dest_dir as hard linked copy of $old_bkup_dir, then apply the changes to the modified files dereferencing them

```sh
# Copy the the structure of a relative path. Use `--relative`, and place a dot path `./`:
#
rsync -av --relative "/target/run/./systemd/resolve" "/mnt/run"

# "Move-merge" a directory into another (mv doesn't allow this)
#
rsync -av --remove-source-files $from $to

# Resume partial files (and after, checks that they're the same).
# When syncing with a server, it must have rsync installed.
# If the file is very large, verification can take a long time; one can use `--append`, at the cost
# of not verifying the data.
# WATCH OUT! In one case, `--append-verify` the copy was not correct.
#
rsync --append-verify myserver:clonezilla.iso clonezilla.iso

# Exclude (glob)
#
rsync --exclude=.git parsec-benchmark/ /dest                     # exclude at any level, unless the excluded directory
                                                                 # is passed as source
rsync --exclude=parsec-benchmark/.git parsec-benchmark/ /dest    # exclude only the root one

# Inclusion is a holy mess. In some cases, e.g. syncing dir `**/a`, it's simpler to use bash features
# (note the subshell!).
# Examples using `--include`: https://stackoverflow.com/q/15687755.
#
(shopt -s globstar; rsync -av --relative source/./**/a dest)

# Ownership is preserved, when running as sudo (consider that it's id-based). In order to change it:
#
sudo rsync -og --chown=$user:$group $from $to

# Specify a custom ssh command (e.g. for the port) + auto ssh password
#
sshpass -p 'fedora_rocks!' rsync -av -e 'ssh -p 10000' --progress --delete . riscv@localhost:parsec-benchmark/
```

The destination user is the current user, unless rsync is run as sudo, in which case, ownership is preserved.

### Sending emails

The `mail` package is `bsd-mailx` in Ubuntu; it can't send attachments:

```sh
echo body | mail -s subject [-c cc_addr] recipient
```

Mutt is a non-default, but convenient, client:

```sh
# Use /dev/null to send an empty email.
# `copy=no`: don't store `$HOME/sent` file
#
mutt -e 'set copy=no' -a attachment.xz -s 'Subject' destination@mail.com <<< "Body $(hostname)"
```

### SSH/utilities

```sh
openssl rsa -in $private_key -pubout   # Generate a public key from private
ssh-keygen -y -f $private_key          # Generate a public key from private, for SSH usage; the `-e` formats are not appropriate

ssh-copy-id -i $private_key $user@$host # Authorize a user (key) on a host!
ssh-keygen [-E md5] -lf $pub_key_file   # Display the hash (SHA256/MD5) of public key (requires file)

ssh-add -D                             # Delete cached keys. Useful if there are problems in authenticating!
ssh-add                                # adds the ssh key password to the authentication agent (e.g. gnome-keyring)

<RETURN>~.                             # terminate an ssh session

openssl rsa -in $private_key -out $private_key_dec # decrypt a key
ssh-keygen -p -f $private_key                      # encrypt a key (or change passphrase)
ssh-keygen [-t $type] [-O bits=$bits]              # generate ssh key pair (defaults: RSA/3072 bits)

ssh-keyscan -H $address > ~/.ssh/known_hosts                                     # Programmatically add fingerprints to known_hosts
ssh-keygen [-E md5] -lf $public_key                                              # Print public keys fingerprint (use md5 for legacy)
openssl pkey -in $private_key -pubout -outform DER | openssl md5 -c              # !!! Print fingerprint, for EC2 !!!

openssl s_client -connect $url:$port < /dev/null | openssl x509  -noout -enddate # Get the expiry of an ssl certificate

# Run a program in the primary display, from an SSH section
# The `xhost` command may be necessary (e.g. for xrandr), as the display is restricted only to the logged in user;
# without it, the error `Authorization required, but no authorization protocol specified` is raised.
#
sudo -u "$SUDO_USER" xhost + > /dev/null
DISPLAY=:0 teamviewer
```

sshpass usages:

```sh
sshpass -p 'fedora_rocks!' ssh -p 10000 riscv@localhost

# sshfs
echo -n 'fedora_rocks!' > /tmp/pwdfile
sshfs -p 10000 riscv@localhost: /media/saverio/temp -o ssh_command="sshpass -f /tmp/pwdfile ssh"
```

Exit hung ssh session: tap `↵~.`.

If some text "sticks" when scrolling history during SSH sessions, look into PS1 - for example, broken escape sequences can cause issues.

### SSL/Certificates

For an automatic (and locally trusted) solution, see [mkcert](https://github.com/FiloSottile/mkcert).

Create a self-signed certificate, manual procedure:

```sh
# Generate a key (insert a 4-letters password; it will be discarded in subsequent steps)
openssl genrsa -des3 -out server.key 2048

# Remove the password
openssl rsa -in server.key -out server.key.insecure
mv -f server.key.insecure server.key

# Create the CSR; use `*.$domain.com` as Common Name.
openssl req -new -key server.key -out server.csr

# Create the certificate
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# Display informations
openssl x509 -in server.crt -noout -text
```

### PGP (GnuPG/gpg)

Main commands (`gpg...`); `$key_id` can be email or key id. See the [Key Servers section](#key-servers) for important notes.

```sh
--gen-key                                            # generate a gpg keypair (store in the database)

--armor --export[-secret-key] $key_id                # export a key
--import [$pubkey]                                   # import a key (also secret); accepts stdin (don't specify $pubkey)

printf $'fpr\nsign\n'   | gpg --command-fd 0 --edit-key $key_id  # self-sign a key
printf $'trust\n5\ny\n' | gpg --command-fd 0 --edit-key $key_id  # ultimately trust a key [required by some programs]

printf $'passwd' | gpg --command-fd 0 --edit-key $key_id   # remove passphrase -> INTERACTIVE!

--delete[-secret]-key $key_id

--list[-secret]-keys
--fingerprint [$key_id]

--recipient $key_id --encrypt [--output $destfile]   # encrypt from stdin
--decrypt [file] [--output $destfile]

--detach-sign [--default-key $key_id] $doc           # don't include the original; use default key for specifying to key
--clearsign [--default-key $key_id] $doc             # don't compress the original, useful for ascii files
--verify [$sig] $doc                                 # see https://security.stackexchange.com/a/45534 about key not trusted

--keyserver $keyserver_address --search-key $key_id  # search key, by email, on a keyserver
```

WATCH OUT GnuPG private keys are composed of a "master" and a "subordinate" key. Both are necessary. If for some reason, a master key is missing the `sec` entry in the listings will show as `sec#`.

Snippets:

```sh
# Encode a group of files; gpg can also input from stdin.
#
find . -name *.log.gz | xargs -I {} gpg -r phony@recipient.com [--output {}.xxx] --encrypt {}

# In order to generate a key in a headless environment, one must use the unattended form (otherwise, gpg tries to ask the password via GUI, causing confusing errors like "gpg: agent_genkey failed: Permission denied"):

gpg --batch --gen-key << 'CFG'
%no-protection
Key-Type: default
Subkey-Type: default
Name-Real: My Name
Name-Email: my@address.com
Expire-Date: 0
CFG

# Delete expired/revoked secret keys (adapted from https://superuser.com/a/1631427).
# Key codes: `sec`ret, `pub`lic, `e`xpired, `r`evoked.
#
gpg --list-secret-keys --with-colons |
  awk -F: '$1 == "sec" && ($2 == "e" || $2 == "r") { print $5 }' |
  xargs gpg --batch --yes --delete-keys
```

#### Key servers

Key servers are surprisingly terrible (timeouts, usability, correct practices...).

WATCH OUT: once a key has been published, the master key is required in order to replace an existing public key; without it, nothing can be done, aside waiting for the expiry.

The best choice is [The HKPS pool](http://hkps.pool.sks-keyservers.net) (see [Stack Overflow](https://superuser.com/a/228033)).

When searching keys in servers via fingerprint, the prefix `0x` must be added.

### Samba (SMB)

```sh
# Mount a share.
# WATCH OUT!! Requires the package `cifs-utils`, otherwise an odd error `mount: /path/to/mountdir: bad option
# is raised; additionally fstab SMB mount works without it.
#
mount -t cifs "//192.168.178.20/clonezy" /home/partimag -o user="saverio",vers=3.0

# Run a custom configuration, for testing pursposes. Debug levels >2 are very verbose.
#
smbd --interactive --configfile smb.conf --debuglevel 2

# Create a guest (no user/password) share, read only, browseable. Access from windows at \\ip\share_name.
#
aptitude install samba

mkdir /path/to/share
chmod 777 /path/to/share	# inverse of `writeable`

# WATCH OUT!! wide links require the package `samba-vfs-modules`.
#
cat >> /etc/samba/smb.conf <<CFG
[share_name]			      # don't forget to change this!
path = /path/to/share
guest ok = yes			    # choose this...
valid users = myuser		# ... or this
# browseable = yes      # default
# read only = yes       # default
force user = myuser     # avoid created files to be owned by root

# Set all the below to allow symlinks
follow symlinks = yes
wide links = yes
unix extensions = no
CFG

Note that permissions are managed as file system level (after the `read only` check).

# !!! Try and see if the protocol is supported on desired clients !!!
#
perl -i -pe 's/(\[global\]\n)/$1        min protocol = SMB2_10\n/'

smbpasswd -a myuser		# set this if a user was configured

systemctl restart smbd
```

## Network manager client (`nmcli`)

```sh
# Connect to networkg; the command is synchronous.
#
nmcli con up id $conn_id

# If the connection is not active, there is no output.
# Exit status is success in either in/active case.
#
nmcli con show --active $conn_id
```

## Lower level networking

### SSL

```sh
# Test an SSL connection
#
# CAfile is needed in order to get local issuer certificate.
#
echo HEAD / | openssl s_client -quiet -CAfile /etc/ssl/certs/ca-certificates.crt -host wexfordopera.ticketsolve.com -port 443
```

### DNS

```sh
# Find currently used nameserver
#
nslookup $address
```

### Netcat (nc)

```sh
nc $address $port           # simulate telnet
nc -l -k -p $port           # [l]isten to port; [k]eep listening after a connection terminates
```

Examples:

```sh
# Bidirectional proxy (3000→3001→3000)
#
mkfifo loop.pipe
cat loop.pipe | nc -l -p 3000 | nc localhost 3001 > loop.pipe

# Wait until a port is open.
#
while ! nc -z localhost 9200; do sleep 0.5; done
```

### Diagnose/measure traffic

```sh
# Measure traffic/connections on a certain port
tcptrack -i $interface port $port

# Display all the connections on a certain network interface
iftop [-i $interface]

# Continuous connectivity test (like traceroute) (performance/speed)
#
mtr $host
```

### Sniff network traffic

```sh
# Deferred connection sniffing. Capture (w/save) and printing are on separate steps
# Good reference: https://danielmiessler.com/study/tcpdump/
#
# `w`  : save to file
# `vvv`: very very verbose. `v` is the very minimum (eg. it displays checksum errors)
# `n` : don't resolve host/port names
#
# sNN: save NN bytes. the default is full packets, which can potentially cause loss of packets.
# i: interface; otherwise, it's detected automatically; use `any` for all the interfaces
#
# `host hostname.com`: filter by host
# `net 1.2.3.4/24`: filter by subnet
#
tcpdump -w realex.pcap -s96 -vvv -n host epage.payandshop.com
tail -c +0 -f realex.pcap | tcpdump -qns 0 -X -r -

# Live connection listening, very simple; outgoing packets only
#
tcpdump -n src host 192.168.0.103

# Live connection listening, with more complex rules logic!
# Note that it's HTTPS, so the content won't be meaningful.
#
# The second is the same as the first, but in extended form, to show complex rules.
#
# `X`  : print packet content
# (rules): note that we're inside quotes. alternatively, brackets can be escaped.
# BOOLEAN OPERATORS LOWERCASE!!
#
tcpdump -vvv -nn -X -s256 'port 443 and host 82.195.158.81'
tcpdump -vvv -nn -X -s256 'port 443 and (src host 82.195.158.81 or dst host 82.195.158.81)'
```

### Port scan (nmap)

```sh
nmap 192.168.0.0/24		                      # scan all open ports in a subnet
nmap -oG - -p 22 192.168.0.0/24 | grep open	# scan a single port, with [o]output for [G]rep, filtering only machines where the [p]ort 22 is open
```

### Unix Sockets

Test if a socket is connected:

```sh
socat -u OPEN:/dev/null UNIX-CONNECT:/tmp/mysql.sock 2> /dev/null
```

### Disable ipv6

```sh
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

## Interfaces

### Basic configuration

Sample system network configuration (`/etc/network/interfaces`):

```
auto lo
iface lo inet loopback

auto enp42s0
iface enp42s0 inet static
        address 10.0.2.15
        netmask 255.255.255.0
        broadcast 10.0.2.255
        gateway 10.0.2.2

auto enp43s0
iface enp43s0 inet dhcp
```

Simpler static parameters could be used (`address .../24` and `gateway ...` only), but they may not be compatible with all the systems.

For USB network interfaces, use [`allow-hotplug`](https://lists.debian.org/debian-user/2017/09/msg00911.html).

Recent Linux distros have persistent names; in the past, udev rules were used (`70-local-persistent-net.rules`).

### NetworkManager

When adding hooks, the scripts must be executable!

#### Changing Network manager DNS (18.04+)

On 18.04 Desktop, one needs to modify the Network manager connection; see https://askubuntu.com/a/1104305/46091.

### Netplan

```sh
ETHERNET_ITF=enp42s0
ETHERNET_IP=192.168.178.199

cat > /etc/netplan/01-$ETHERNET_ITF.yaml <<YAML
network:
  version: 2
  renderer: networkd
  ethernets:
    $ETHERNET_ITF:
      addresses:
        - $ETHERNET_IP/24
      gateway4: ${ETHERNET_IP%.*}.1
      nameservers:
        addresses: [${ETHERNET_IP%.*}.1]
YAML

netplay apply
```

### `ip` tool

#### iproute2

Show all the interfaces, even if unconfigured:

```sh
ip link show
```

Manually setup an ethernet card on Ubuntu server:

```sh
ip addr add 192.168.0.2/24 dev enp42s0
ip link set enp42s0 up
ip route add default via 192.168.0.1
echo 'nameserver 8.8.8.8' > /etc/resolv.conf
```

For a permanent configuration, use Netplan.

#### Create a TUN/TAP device

```sh
# $devname: mytap, $ip_mask: 192.168.0.1/24

ip tuntap add mod etap name $devname user $USER          # create a TT device, in tunneling mode
ip link set $devname up                                  # activate it
ip addr add $ip_mask dev $devname                        # set the IP address
iptables -t nat -a POSTROUTING -a $ip_mask -j MASQUERADE # enable internet packets to reach the ip mask by mapping IP addresses to the device
sysctl -w net.ipv4.ip_forward = 1

ip tuntap [list]                                         # list TT devices
ip tuntap del mode tap name $devname                     # delete the device
```

### Wi-Fi

Get connection frequency:

```sh
# Get the current wifi connection frequency.
#
iwlist $adapter frequency
iwconfig [$adapter]
```

Measure speed:

```sh
iperf -s                 # on the server
iperf -c $iperf_server   # on the client
```
