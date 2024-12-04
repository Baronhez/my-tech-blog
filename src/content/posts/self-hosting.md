---
title: Self-Hosting Guide
published: 2022-08-28
description: It is really easy. And if you ever meet someone who tells you otherwise, they're lying.
tags: [Docker, Server, Linux]
category: Server
draft: false
---
I know, you want to self host, for instance, your own webpage, but every time you search for info, people only recommend you to use sh\*t like NO-IP and third party providers such as Heroku. But that's not what you want, right? You have a Rashberry Pi or some old PC and you intent to host your own services, like your webpage, a torrent server or even some filebrowser to manage storage.

Ok, then I will save you a lot of hours or researching how to mount everything, how to deal with nginx, how to get https certificates... and I will show you the best way of setting up everything.

## Prepare your server machine

Firtsly, I will start by getting an ISO of Ubuntu Server (you can use another ISOs, like arch linux or Nix, if you are brave enough). 

[https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso](https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso)

Then, use some program, like [Rufus](https://github.com/pbatard/rufus/releases/download/v3.17/rufus-3.17.exe) or [Balena Etcher](https://www.balena.io/etcher/) to flash that ISO into an USB or SD Card (in case you're using a Rashberry Pi).

If you are not familiar with this, take a look into [this tutorial](https://www.youtube.com/watch?v=Wt0Q-DBejIw).

Ok, now we have an USB or SD Card flashed with our favourite OS for our server. Now it is time to insert that into the server we will be using. If it's possible I recommend you to connect that device to a monitor, in order to see what is happening and make some SSH configurations.

Once the operating system is installed, we will create a new user and get rid of the default user. (Using the root user for everything is a bad idea).

```
sudo adduser Username
sudo usermod -aG sudo Username
```

## How to connect to our machine

Next, we will configure SSH to be able to connect to our device from another pc. This way it is easier to manage the server (and we will be able to disconnect the monitor from the server).

I have another guide about how configure SSH to be secure and ready to use, go ahead a [check it](https://blog.jonthan.xyz/how-to-configure-ssh-to-be-secure/).

## Unattended upgrades (optional)

If you don't want automated updates, skip this section.

Ok, now that we are able to connect to the server using SSH, and SSH is configure to be secure, it is time to install our first packages.

```
sudo apt install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
# Now, select yes to enable unattended-upgrades.
# Once you finish with that, you can change some lines in the configuration of unattended-upgrades.
nano /etc/apt/apt.conf.d/10periodic
```

For instance, you can configure something like this:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";
```

## Email your logs to yourself (optional)

If you don't want to email your logs, skip this section.

If you want to keep track of your logs and email them to you, then you can use [Logwatch](https://linux.die.net/man/8/logwatch).

To install and configure logwatch, run these commands:

```
sudo apt-get install logwatch
nano /etc/cron.daily/00logwatch
```

Then, add this line:

```
/usr/sbin/logwatch --output mail --mailto yourmail@gmail.com --detail high  
```

## Installation of Docker and Docker-Compose

The rest of the guide focuses on using docker and docker-compose to deploy services. If you want to deploy your services manually by configuring nginx and cert-manager, go ahead and try it yourself. If you want to do it using kubernetes, I will publish a guide as soon as possible.

It's time to install docker and docker compose.

You can do it manually:

```
sudo apt update && sudo apt upgrade
sudo apt install apt-transport-https ca-certificates curl software-properties-common  
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"  
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose  
sudo chmod +x /usr/local/bin/docker-compose
```

Or you can run this command, which runs a script to install Docker automatically:

```
curl -fsSL https://get.docker.com | sh
```

The content of the executed bash script is (at 10/02/2023) as follows:

```
#!/bin/sh
set -e
# Docker CE for Linux installation script
#
# See https://docs.docker.com/engine/install/ for the installation steps.
#
# This script is meant for quick & easy install via:
#   $ curl -fsSL https://get.docker.com -o get-docker.sh
#   $ sh get-docker.sh
#
# For test builds (ie. release candidates):
#   $ curl -fsSL https://test.docker.com -o test-docker.sh
#   $ sh test-docker.sh
#
# NOTE: Make sure to verify the contents of the script
#       you downloaded matches the contents of install.sh
#       located at https://github.com/docker/docker-install
#       before executing.
#
# Git commit from https://github.com/docker/docker-install when
# the script was uploaded (Should only be modified by upload job):
SCRIPT_COMMIT_SHA="66474034547a96caa0a25be56051ff8b726a1b28"

# strip "v" prefix if present
VERSION="${VERSION#v}"

# The channel to install from:
#   * nightly
#   * test
#   * stable
#   * edge (deprecated)
DEFAULT_CHANNEL_VALUE="stable"
if [ -z "$CHANNEL" ]; then
CHANNEL=$DEFAULT_CHANNEL_VALUE
fi

DEFAULT_DOWNLOAD_URL="https://download.docker.com"
if [ -z "$DOWNLOAD_URL" ]; then
DOWNLOAD_URL=$DEFAULT_DOWNLOAD_URL
fi

DEFAULT_REPO_FILE="docker-ce.repo"
if [ -z "$REPO_FILE" ]; then
REPO_FILE="$DEFAULT_REPO_FILE"
fi

mirror=''
DRY_RUN=${DRY_RUN:-}
while [ $# -gt 0 ]; do
case "$1" in
--mirror)
mirror="$2"
shift
;;
--dry-run)
DRY_RUN=1
;;
--*)
echo "Illegal option $1"
;;
esac
shift $(( $# > 0 ? 1 : 0 ))
done

case "$mirror" in
Aliyun)
DOWNLOAD_URL="https://mirrors.aliyun.com/docker-ce"
;;
AzureChinaCloud)
DOWNLOAD_URL="https://mirror.azure.cn/docker-ce"
;;
esac

command_exists() {
command -v "$@" > /dev/null 2>&1
}

# version_gte checks if the version specified in $VERSION is at least
# the given CalVer (YY.MM) version. returns 0 (success) if $VERSION is either
# unset (=latest) or newer or equal than the specified version. Returns 1 (fail)
# otherwise.
#
# examples:
#
# VERSION=20.10
# version_gte 20.10 // 0 (success)
# version_gte 19.03 // 0 (success)
# version_gte 21.10 // 1 (fail)
version_gte() {
if [ -z "$VERSION" ]; then
return 0
fi
eval calver_compare "$VERSION" "$1"
}

# calver_compare compares two CalVer (YY.MM) version strings. returns 0 (success)
# if version A is newer or equal than version B, or 1 (fail) otherwise. Patch
# releases and pre-release (-alpha/-beta) are not taken into account
#
# examples:
#
# calver_compare 20.10 19.03 // 0 (success)
# calver_compare 20.10 20.10 // 0 (success)
# calver_compare 19.03 20.10 // 1 (fail)
calver_compare() (
set +x

yy_a="$(echo "$1" | cut -d'.' -f1)"
yy_b="$(echo "$2" | cut -d'.' -f1)"
if [ "$yy_a" -lt "$yy_b" ]; then
return 1
fi
if [ "$yy_a" -gt "$yy_b" ]; then
return 0
fi
mm_a="$(echo "$1" | cut -d'.' -f2)"
mm_b="$(echo "$2" | cut -d'.' -f2)"
if [ "${mm_a#0}" -lt "${mm_b#0}" ]; then
return 1
fi

return 0
)

is_dry_run() {
if [ -z "$DRY_RUN" ]; then
return 1
else
return 0
fi
}

is_wsl() {
case "$(uname -r)" in
*microsoft* ) true ;; # WSL 2
*Microsoft* ) true ;; # WSL 1
* ) false;;
esac
}

is_darwin() {
case "$(uname -s)" in
*darwin* ) true ;;
*Darwin* ) true ;;
* ) false;;
esac
}

deprecation_notice() {
distro=$1
distro_version=$2
echo
printf "\033[91;1mDEPRECATION WARNING\033[0m\n"
printf "    This Linux distribution (\033[1m%s %s\033[0m) reached end-of-life and is no longer supported by this script.\n" "$distro" "$distro_version"
echo   "    No updates or security fixes will be released for this distribution, and users are recommended"
echo   "    to upgrade to a currently maintained version of $distro."
echo
printf   "Press \033[1mCtrl+C\033[0m now to abort this script, or wait for the installation to continue."
echo
sleep 10
}

get_distribution() {
lsb_dist=""
# Every system that we officially support has /etc/os-release
if [ -r /etc/os-release ]; then
lsb_dist="$(. /etc/os-release && echo "$ID")"
fi
# Returning an empty string here should be alright since the
# case statements don't act unless you provide an actual value
echo "$lsb_dist"
}

echo_docker_as_nonroot() {
if is_dry_run; then
return
fi
if command_exists docker && [ -e /var/run/docker.sock ]; then
(
set -x
$sh_c 'docker version'
) || true
fi

# intentionally mixed spaces and tabs here -- tabs are stripped by "<<-EOF", spaces are kept in the output
echo
echo "================================================================================"
echo
if version_gte "20.10"; then
echo "To run Docker as a non-privileged user, consider setting up the"
echo "Docker daemon in rootless mode for your user:"
echo
echo "    dockerd-rootless-setuptool.sh install"
echo
echo "Visit https://docs.docker.com/go/rootless/ to learn about rootless mode."
echo
fi
echo
echo "To run the Docker daemon as a fully privileged service, but granting non-root"
echo "users access, refer to https://docs.docker.com/go/daemon-access/"
echo
echo "WARNING: Access to the remote API on a privileged Docker daemon is equivalent"
echo "         to root access on the host. Refer to the 'Docker daemon attack surface'"
echo "         documentation for details: https://docs.docker.com/go/attack-surface/"
echo
echo "================================================================================"
echo
}

# Check if this is a forked Linux distro
check_forked() {

# Check for lsb_release command existence, it usually exists in forked distros
if command_exists lsb_release; then
# Check if the `-u` option is supported
set +e
lsb_release -a -u > /dev/null 2>&1
lsb_release_exit_code=$?
set -e

# Check if the command has exited successfully, it means we're in a forked distro
if [ "$lsb_release_exit_code" = "0" ]; then
# Print info about current distro
cat <<-EOF
You're using '$lsb_dist' version '$dist_version'.
EOF

# Get the upstream release info
lsb_dist=$(lsb_release -a -u 2>&1 | tr '[:upper:]' '[:lower:]' | grep -E 'id' | cut -d ':' -f 2 | tr -d '[:space:]')
dist_version=$(lsb_release -a -u 2>&1 | tr '[:upper:]' '[:lower:]' | grep -E 'codename' | cut -d ':' -f 2 | tr -d '[:space:]')

# Print info about upstream distro
cat <<-EOF
Upstream release is '$lsb_dist' version '$dist_version'.
EOF
else
if [ -r /etc/debian_version ] && [ "$lsb_dist" != "ubuntu" ] && [ "$lsb_dist" != "raspbian" ]; then
if [ "$lsb_dist" = "osmc" ]; then
# OSMC runs Raspbian
lsb_dist=raspbian
else
# We're Debian and don't even know it!
lsb_dist=debian
fi
dist_version="$(sed 's/\/.*//' /etc/debian_version | sed 's/\..*//')"
case "$dist_version" in
11)
dist_version="bullseye"
;;
10)
dist_version="buster"
;;
9)
dist_version="stretch"
;;
8)
dist_version="jessie"
;;
esac
fi
fi
fi
}

do_install() {
echo "# Executing docker install script, commit: $SCRIPT_COMMIT_SHA"

if command_exists docker; then
cat >&2 <<-'EOF'
Warning: the "docker" command appears to already exist on this system.

If you already have Docker installed, this script can cause trouble, which is
why we're displaying this warning and provide the opportunity to cancel the
installation.

If you installed the current Docker package using this script and are using it
again to update Docker, you can safely ignore this message.

You may press Ctrl+C now to abort this script.
EOF
( set -x; sleep 20 )
fi

user="$(id -un 2>/dev/null || true)"

sh_c='sh -c'
if [ "$user" != 'root' ]; then
if command_exists sudo; then
sh_c='sudo -E sh -c'
elif command_exists su; then
sh_c='su -c'
else
cat >&2 <<-'EOF'
Error: this installer needs the ability to run commands as root.
We are unable to find either "sudo" or "su" available to make this happen.
EOF
exit 1
fi
fi

if is_dry_run; then
sh_c="echo"
fi

# perform some very rudimentary platform detection
lsb_dist=$( get_distribution )
lsb_dist="$(echo "$lsb_dist" | tr '[:upper:]' '[:lower:]')"

if is_wsl; then
echo
echo "WSL DETECTED: We recommend using Docker Desktop for Windows."
echo "Please get Docker Desktop from https://www.docker.com/products/docker-desktop"
echo
cat >&2 <<-'EOF'

You may press Ctrl+C now to abort this script.
EOF
( set -x; sleep 20 )
fi

case "$lsb_dist" in

ubuntu)
if command_exists lsb_release; then
dist_version="$(lsb_release --codename | cut -f2)"
fi
if [ -z "$dist_version" ] && [ -r /etc/lsb-release ]; then
dist_version="$(. /etc/lsb-release && echo "$DISTRIB_CODENAME")"
fi
;;

debian|raspbian)
dist_version="$(sed 's/\/.*//' /etc/debian_version | sed 's/\..*//')"
case "$dist_version" in
11)
dist_version="bullseye"
;;
10)
dist_version="buster"
;;
9)
dist_version="stretch"
;;
8)
dist_version="jessie"
;;
esac
;;

centos|rhel|sles)
if [ -z "$dist_version" ] && [ -r /etc/os-release ]; then
dist_version="$(. /etc/os-release && echo "$VERSION_ID")"
fi
;;

*)
if command_exists lsb_release; then
dist_version="$(lsb_release --release | cut -f2)"
fi
if [ -z "$dist_version" ] && [ -r /etc/os-release ]; then
dist_version="$(. /etc/os-release && echo "$VERSION_ID")"
fi
;;

esac

# Check if this is a forked Linux distro
check_forked

# Print deprecation warnings for distro versions that recently reached EOL,
# but may still be commonly used (especially LTS versions).
case "$lsb_dist.$dist_version" in
debian.stretch|debian.jessie)
deprecation_notice "$lsb_dist" "$dist_version"
;;
raspbian.stretch|raspbian.jessie)
deprecation_notice "$lsb_dist" "$dist_version"
;;
ubuntu.xenial|ubuntu.trusty)
deprecation_notice "$lsb_dist" "$dist_version"
;;
fedora.*)
if [ "$dist_version" -lt 33 ]; then
deprecation_notice "$lsb_dist" "$dist_version"
fi
;;
esac

# Run setup for each distro accordingly
case "$lsb_dist" in
ubuntu|debian|raspbian)
pre_reqs="apt-transport-https ca-certificates curl"
if ! command -v gpg > /dev/null; then
pre_reqs="$pre_reqs gnupg"
fi
apt_repo="deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] $DOWNLOAD_URL/linux/$lsb_dist $dist_version $CHANNEL"
(
if ! is_dry_run; then
set -x
fi
$sh_c 'apt-get update -qq >/dev/null'
$sh_c "DEBIAN_FRONTEND=noninteractive apt-get install -y -qq $pre_reqs >/dev/null"
$sh_c 'mkdir -p /etc/apt/keyrings && chmod -R 0755 /etc/apt/keyrings'
$sh_c "curl -fsSL \"$DOWNLOAD_URL/linux/$lsb_dist/gpg\" | gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg"
$sh_c "chmod a+r /etc/apt/keyrings/docker.gpg"
$sh_c "echo \"$apt_repo\" > /etc/apt/sources.list.d/docker.list"
$sh_c 'apt-get update -qq >/dev/null'
)
pkg_version=""
if [ -n "$VERSION" ]; then
if is_dry_run; then
echo "# WARNING: VERSION pinning is not supported in DRY_RUN"
else
# Will work for incomplete versions IE (17.12), but may not actually grab the "latest" if in the test channel
pkg_pattern="$(echo "$VERSION" | sed "s/-ce-/~ce~.*/g" | sed "s/-/.*/g")"
search_command="apt-cache madison 'docker-ce' | grep '$pkg_pattern' | head -1 | awk '{\$1=\$1};1' | cut -d' ' -f 3"
pkg_version="$($sh_c "$search_command")"
echo "INFO: Searching repository for VERSION '$VERSION'"
echo "INFO: $search_command"
if [ -z "$pkg_version" ]; then
echo
echo "ERROR: '$VERSION' not found amongst apt-cache madison results"
echo
exit 1
fi
if version_gte "18.09"; then
search_command="apt-cache madison 'docker-ce-cli' | grep '$pkg_pattern' | head -1 | awk '{\$1=\$1};1' | cut -d' ' -f 3"
echo "INFO: $search_command"
cli_pkg_version="=$($sh_c "$search_command")"
fi
pkg_version="=$pkg_version"
fi
fi
(
pkgs="docker-ce${pkg_version%=}"
if version_gte "18.09"; then
# older versions didn't ship the cli and containerd as separate packages
pkgs="$pkgs docker-ce-cli${cli_pkg_version%=} containerd.io"
fi
if version_gte "20.10" && [ "$(uname -m)" = "x86_64" ]; then
# also install the latest version of the "docker scan" cli-plugin (only supported on x86 currently)
pkgs="$pkgs docker-scan-plugin"
fi
if version_gte "20.10"; then
pkgs="$pkgs docker-compose-plugin docker-ce-rootless-extras$pkg_version"
fi
if version_gte "23.0"; then
pkgs="$pkgs docker-buildx-plugin"
fi
if ! is_dry_run; then
set -x
fi
$sh_c "DEBIAN_FRONTEND=noninteractive apt-get install -y -qq $pkgs >/dev/null"
)
echo_docker_as_nonroot
exit 0
;;
centos|fedora|rhel)
if [ "$(uname -m)" != "s390x" ] && [ "$lsb_dist" = "rhel" ]; then
echo "Packages for RHEL are currently only available for s390x."
exit 1
fi
if [ "$lsb_dist" = "fedora" ]; then
pkg_manager="dnf"
config_manager="dnf config-manager"
enable_channel_flag="--set-enabled"
disable_channel_flag="--set-disabled"
pre_reqs="dnf-plugins-core"
pkg_suffix="fc$dist_version"
else
pkg_manager="yum"
config_manager="yum-config-manager"
enable_channel_flag="--enable"
disable_channel_flag="--disable"
pre_reqs="yum-utils"
pkg_suffix="el"
fi
repo_file_url="$DOWNLOAD_URL/linux/$lsb_dist/$REPO_FILE"
(
if ! is_dry_run; then
set -x
fi
$sh_c "$pkg_manager install -y -q $pre_reqs"
$sh_c "$config_manager --add-repo $repo_file_url"

if [ "$CHANNEL" != "stable" ]; then
$sh_c "$config_manager $disable_channel_flag docker-ce-*"
$sh_c "$config_manager $enable_channel_flag docker-ce-$CHANNEL"
fi
$sh_c "$pkg_manager makecache"
)
pkg_version=""
if [ -n "$VERSION" ]; then
if is_dry_run; then
echo "# WARNING: VERSION pinning is not supported in DRY_RUN"
else
pkg_pattern="$(echo "$VERSION" | sed "s/-ce-/\\\\.ce.*/g" | sed "s/-/.*/g").*$pkg_suffix"
search_command="$pkg_manager list --showduplicates 'docker-ce' | grep '$pkg_pattern' | tail -1 | awk '{print \$2}'"
pkg_version="$($sh_c "$search_command")"
echo "INFO: Searching repository for VERSION '$VERSION'"
echo "INFO: $search_command"
if [ -z "$pkg_version" ]; then
echo
echo "ERROR: '$VERSION' not found amongst $pkg_manager list results"
echo
exit 1
fi
if version_gte "18.09"; then
# older versions don't support a cli package
search_command="$pkg_manager list --showduplicates 'docker-ce-cli' | grep '$pkg_pattern' | tail -1 | awk '{print \$2}'"
cli_pkg_version="$($sh_c "$search_command" | cut -d':' -f 2)"
fi
# Cut out the epoch and prefix with a '-'
pkg_version="-$(echo "$pkg_version" | cut -d':' -f 2)"
fi
fi
(
pkgs="docker-ce$pkg_version"
if version_gte "18.09"; then
# older versions didn't ship the cli and containerd as separate packages
if [ -n "$cli_pkg_version" ]; then
pkgs="$pkgs docker-ce-cli-$cli_pkg_version containerd.io"
else
pkgs="$pkgs docker-ce-cli containerd.io"
fi
fi
if version_gte "20.10" && [ "$(uname -m)" = "x86_64" ]; then
# also install the latest version of the "docker scan" cli-plugin (only supported on x86 currently)
pkgs="$pkgs docker-scan-plugin"
fi
if version_gte "20.10"; then
pkgs="$pkgs docker-compose-plugin docker-ce-rootless-extras$pkg_version"
fi
if version_gte "23.0"; then
pkgs="$pkgs docker-buildx-plugin"
fi
if ! is_dry_run; then
set -x
fi
$sh_c "$pkg_manager install -y -q $pkgs"
)
echo_docker_as_nonroot
exit 0
;;
sles)
if [ "$(uname -m)" != "s390x" ]; then
echo "Packages for SLES are currently only available for s390x"
exit 1
fi
if [ "$dist_version" = "15.3" ]; then
sles_version="SLE_15_SP3"
else
sles_minor_version="${dist_version##*.}"
sles_version="15.$sles_minor_version"
fi
opensuse_repo="https://download.opensuse.org/repositories/security:SELinux/$sles_version/security:SELinux.repo"
repo_file_url="$DOWNLOAD_URL/linux/$lsb_dist/$REPO_FILE"
pre_reqs="ca-certificates curl libseccomp2 awk"
(
if ! is_dry_run; then
set -x
fi
$sh_c "zypper install -y $pre_reqs"
$sh_c "zypper addrepo $repo_file_url"
if ! is_dry_run; then
cat >&2 <<-'EOF'
WARNING!!
openSUSE repository (https://download.opensuse.org/repositories/security:SELinux) will be enabled now.
Do you wish to continue?
You may press Ctrl+C now to abort this script.
EOF
( set -x; sleep 30 )
fi
$sh_c "zypper addrepo $opensuse_repo"
$sh_c "zypper --gpg-auto-import-keys refresh"
$sh_c "zypper lr -d"
)
pkg_version=""
if [ -n "$VERSION" ]; then
if is_dry_run; then
echo "# WARNING: VERSION pinning is not supported in DRY_RUN"
else
pkg_pattern="$(echo "$VERSION" | sed "s/-ce-/\\\\.ce.*/g" | sed "s/-/.*/g")"
search_command="zypper search -s --match-exact 'docker-ce' | grep '$pkg_pattern' | tail -1 | awk '{print \$6}'"
pkg_version="$($sh_c "$search_command")"
echo "INFO: Searching repository for VERSION '$VERSION'"
echo "INFO: $search_command"
if [ -z "$pkg_version" ]; then
echo
echo "ERROR: '$VERSION' not found amongst zypper list results"
echo
exit 1
fi
search_command="zypper search -s --match-exact 'docker-ce-cli' | grep '$pkg_pattern' | tail -1 | awk '{print \$6}'"
# It's okay for cli_pkg_version to be blank, since older versions don't support a cli package
cli_pkg_version="$($sh_c "$search_command")"
pkg_version="-$pkg_version"
fi
fi
(
pkgs="docker-ce$pkg_version"
if version_gte "18.09"; then
if [ -n "$cli_pkg_version" ]; then
# older versions didn't ship the cli and containerd as separate packages
pkgs="$pkgs docker-ce-cli-$cli_pkg_version containerd.io"
else
pkgs="$pkgs docker-ce-cli containerd.io"
fi
fi
if version_gte "20.10"; then
pkgs="$pkgs docker-compose-plugin docker-ce-rootless-extras$pkg_version"
fi
if version_gte "23.0"; then
pkgs="$pkgs docker-buildx-plugin"
fi
if ! is_dry_run; then
set -x
fi
$sh_c "zypper -q install -y $pkgs"
)
echo_docker_as_nonroot
exit 0
;;
*)
if [ -z "$lsb_dist" ]; then
if is_darwin; then
echo
echo "ERROR: Unsupported operating system 'macOS'"
echo "Please get Docker Desktop from https://www.docker.com/products/docker-desktop"
echo
exit 1
fi
fi
echo
echo "ERROR: Unsupported distribution '$lsb_dist'"
echo
exit 1
;;
esac
exit 1
}

# wrapped up in a function so that we have some protection against only getting
# half the file during "curl | sh"
do_install

```

Now, assuming that everything went well, we should have docker and docker-compose installed in our server.  It's time to test docker-compose by deploying a small web in our local network.

```
mkdir app
nano app/index.html
```

Insert this content into the **index.html** file:

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Docker Compose Demo</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/kognise/water.css@latest/dist/dark.min.css">
</head>
<body>

<h1>This is a Docker Compose Demo Page.</h1>
<p>This content is being served by an Nginx container.</p>

</body>
</html>
```

Save the file, and run the following command:

```
nano docker-compose.yml
```

Insert this content into the **docker-compose.yml** file:

```
version: '3.7'
services:
  web:
    image: nginx:alpine
    ports:
      - "8000:80"
    volumes:
      - ./app:/usr/share/nginx/html
```

Now, to start our aplication containerized, we must run the following command:

```
docker-compose up -d
```

That command should return an output like this one:

```
Creating network "compose-demo_default" with the default driver
Pulling web (nginx:alpine)...
alpine: Pulling from library/nginx
cbdbe7a5bc2a: Pull complete
10c113fb0c77: Pull complete
9ba64393807b: Pull complete
c829a9c40ab2: Pull complete
61d685417b2f: Pull complete
Digest: sha256:57254039c6313fe8c53f1acbf15657ec9616a813397b74b063e32443427c5502
Status: Downloaded newer image for nginx:alpine
Creating compose-demo_web_1 ... done
```

Now, your webpage should be accesible through your browser by typing "local-IP-of-your-server:8000".

Ok, now that we have a webpage deployed in our local network, I should share a few useful commands to manage Docker:

```
# We can see the logs of our containers
docker-compose logs
# Pause our containers
docker-compose pause
# Unpause our containers
docker-compose unpause
# Stop our containers
docker-compose stop
# Delete all the resources and volumes assigned to our containers
docker-compose down
# List our images
docker images
# Delete our images
docker image rm nginx:alpine
# Or
docker rmi nginx:alpine
# We can delete all stoped container, every network not use by al least one container, every unused image and all build cache.
docker system prune -a
# Do the same but filtering specific images
docker system prune --filter nginx:alpine
# Or just the volumes
docker system prune --volumes
# Or we can do the same but without asking for confirmation
docker system prune -a -f  
```

Now, if you want to deploy you web to the world outside your local network, you must change a few things in your router (supposing you have access to the router). It is completely necessary to set port-forwarding, because if you are mounting your server at home, your devices are connected inside your home using a private network. It is possible for us to connect to the internet thanks to [NAT](https://en.wikipedia.org/wiki/Network_address_translation). 

Each router is a different world, so you must search information about how to port-forward in your home router. 

On the other hand, if you're using some provider for your server, such as a VM in Oracle Cloud, then it should be easier. Search for your specific provider how to port-forward and that's the end of the problem.

## Using a domain name and DDNS

Ok, if you want to access your webpage easily without having to memorize a bunch of numbers, you have to buy a domain name. I recommend [https://www.namecheap.com/](https://www.namecheap.com/). It provides cheap domain names and it is also a DNS provider, but you can pick whichever you want.

If you want to point your domain name to your server, there is one problem if your server is in your home. Most ISPs (or all of them, to be honest) nowadays change your public IP from time to time. If you don't want to lose access to your webpage when that happens, we must use Dynamic DNS. It is necessary to set DDNS with your DNS provider and use a DDNS client in our server to comunicate with our DNS provider. The DDNS client will inform the DNS provider of your new IP.

I use [ddclient](https://ddclient.net/) for that, because Namecheap has a [guide](https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient/) to use it with them.

## Using HTTPS certificates from Let's Encrypt

Next step is to create HTTPS certificates for our webpage. The docker way of doing this is by configurating let's encrypt. We can do it using different methods, but the best for me is using one of the best Edge Router nowadays. I'm talking of [Traefik](https://doc.traefik.io/traefik/).

I will save you tons of documentation, because this guide is already too long. If you want to set a reverse proxy with traefik which also can generate certificates with Let's Encrypt, just use this files:

First, create an acme.json file and change its privilegies (take into account the path of the acme.json file, because you have to add that path to the **docker-compose.yaml** file:

```
touch acme.json
chmod 600 acme.json
```

Now, create a folder and two different files for traefik configuration (put your domain name where it says YOUR-DOMAIN-NAME and your preferred email where it says youremail@example.com):

```
mkdir ./config
nano ./config/traefik.yml
```

```
# traefik.yml
api:
  dashboard: true # Enable the dashboard
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: web-secure
  web-secure:
    address: ":443"
    http:
      tls:
        certResolver: default
providers:
        # In order to get this working on docker, refer to https://doc.traefik.io/traefik/providers/docker/ for more info.
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false # As seen in the documentation, "Expose containers by default through Traefik. If set to false, containers that do not have a traefik.enable=true label are ignored from the resulting routing configuration".
  file:
    filename: /etc/traefik/config.yml # In order to get all my configuration info in a separate file (dinamic.yaml), I use https://doc.traefik.io/traefik/providers/file/.
    watch: true # Watch for changes
    
certificatesResolvers: # Refer to https://doc.traefik.io/traefik/https/acme/#certificate-resolvers to more info. 
  default:
    acme:
      email: youremail@example.com 
      storage: /etc/traefik/acme/acme.json # Previously you have to create the file, change the permissions using chmod and then use a docker volume.
      keyType: 'EC384' # 
      tlsChallenge: true # https://doc.traefik.io/traefik/https/acme/#tlschallenge
```

Now, we have to configure one last yaml file (put your domain name where it says YOUR-DOMAIN-NAME):

```
nano ./config/config.yml
```

```
# config.yml
http:
  routers:
    traefik:
      rule: Host(`subdomainForTheDashboard.YOUR-DOMAIN-NAME`)
      entryPoints: 
        - "web-secure"
      service: api@internal
      middlewares:
        - secHeader
      tls:
        certResolver: default
  middlewares:
          # In order to modify the request, I use middlewares. In this case to force https.
    secHeader:
        # HSTS / Secure Headers, Useful to have a more secure experience with HTTPS
      headers:
        frameDeny: true
        sslRedirect: true
        browserXssFilter: true
        contentTypeNosniff: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 31536000
tls:
        # Now, I refer to https://doc.traefik.io/traefik/https/tls/ for information. 
        # Options allow me to to configure some parameters of the TLS connection.
  options:
    default:
            # Now, here a use the min Version tls version 1.2 just as shown in the traefik documentation (This config will get an A+ grade in https://www.ssllabs.com/ssltest/) 
      minVersion: VersionTLS12 # Refering to the Traefik Documentation, I use sniStrict because "Traefik won't allow connections from clients that do not specify a server_name extension or don't match any certificate configured on the tlsOption".
      sniStrict: true
      cipherSuites:
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
        - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384   # TLS 1.2
        - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305    # TLS 1.2
        - TLS_AES_256_GCM_SHA384                  # TLS 1.3
        - TLS_CHACHA20_POLY1305_SHA256            # TLS 1.3
        # Now I define the eliptic curves preferences for ECC cryptography
        # Refer to https://pkg.go.dev/crypto/tls#CurveID for more info
      curvePreferences:
        - CurveP521
        - CurveP384
    mintls13:
      minVersion: VersionTLS13
      sniStrict: true
```

Ok, this configuration should work flawlessly. You can add more middlewares, such as rate limit or plugins like fail2ban. I'm not covering that here right now.

Lastly, a **docker-compose.yaml**(put your domain name where it says YOUR-DOMAIN-NAME and put the path to your acme.json file and your webpage folder):

```
nano docker-compose.yaml
```

```
version: '3.7'
services:
  traefik:
    image: traefik:2.5.5
    container_name: traefik
    network_mode: host
    restart: unless-stopped
    volumes:
      - ./config/:/etc/traefik/
      - /path/to/acme.json:/etc/traefik/acme/acme.json
      - /var/run/docker.sock:/var/run/docker.sock
  web:
    image: nginx:1.21.4-alpine
    container_name: web
    restart: always
    expose:
      - "80"
    volumes:
      - /path/to/your/webpage/folder:/usr/share/nginx/html:ro
    labels:
      - traefik.enable=true
      - traefik.http.routers.webpage.entryPoints=web-secure
      - traefik.http.routers.webpage.rule=Host(`YOUR-DOMAIN-NAME`)
      - "traefik.http.routers.webpage.middlewares=secHeader@file"
```

If you want to deploy more services, just add this to every new service in a docker-compose file:

```
 labels:
      - traefik.enable=true
      - traefik.http.routers.NAME-ROUTER.entryPoints=web-secure
      - traefik.http.routers.NAME-ROUTER.rule=Host(`YOUR-DOMAIN-NAME`)
      - "traefik.http.routers.NAME-ROUTER.middlewares=secHeader@file"
```

Just remember to change NAME-ROUTER to another name each time (it is the name of the [traefik router](https://doc.traefik.io/traefik/routing/routers/)) and YOUR-DOMAIN-NAME should be a subdomain, such as **nextcloud.example.com** for a hipothetical nextcloud service.

To deploy this configuration, go to the folder where the **docker-compose.yaml** file is, and run the following command:

```
sudo docker-compose up -d
```

If you go to your domain, assuming that you previouly pointed the domain to your server IP (or the public IP of your router, in case of a home server), the webpage should be displayed in the browser as intended.

If you want to get an A+ result in the SSL test from [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/) then you have to add a CAA record.

You can find documentation about CAA records here: [https://www.namecheap.com/support/knowledgebase/article.aspx/9991/38/caa-record-and-why-it-is-needed-ssl-related/](https://www.namecheap.com/support/knowledgebase/article.aspx/9991/38/caa-record-and-why-it-is-needed-ssl-related/)

And you can find a tool to create CAA records here: [https://sslmate.com/caa/](https://sslmate.com/caa/)

That's it, you have started in the selfhosting hobby. I recommend you this [subreddit](https://www.reddit.com/r/selfhosted/).

Good luck, lads.