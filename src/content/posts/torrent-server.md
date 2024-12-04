---
title: How to mount a torrent server to watch movies and series for free
published: 2022-09-16
description: If someone wants to charge you for using this on their home server, they are ripping you off. It is super easy to set up.
tags: [Docker, Server, Linux, Hosting, Torrent]
category: Server
draft: false
---
You want to deploy a self-hosted media server without the hassle of reading tons of documentation. Well, this is a straight-forward guide to deploy everything in just five minutes. Let's get into it! (I'm assuming you know the basics of how to manage Docker and a terminal shell).

## Step #1: Install Docker and Docker-Compose.

\[MANUAL\] First, install docker and docker-compose:

```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt install docker-ce
sudo usermod -aG docker <username>
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

\[FASTEST\] Or run this command:

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

## Step #2: Set up the directory tree.

The fastest way to set this up is to follow the [Trash Guides recommendation](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/).

data  
├── torrents  
│ ├── books  
│ ├── movies  
│ ├── music  
│ └── tv  
├── usenet  
│ ├── incomplete  
│ └── complete  
│ ├── books  
│ ├── movies  
│ ├── music  
│ └── tv  
└── media  
├── books  
├── movies  
├── music  
└── tv   

I recommend you to not create the data directory in / (because permissions issues). We will assign different volumes to the containers depending on what the services need, just as [Trash Guides](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/) recommends.

## Step #3: Create Docker-Compose file.

Ok, now it's time to deploy our services with  you will see a **docker-compose.yml** file. Edit the content with this command:

```
nano docker-compose.yml
```

Edit this content (You have to enter your time zone, PUID and PGID. It is not necessary to modify the UMASK):

```
services:
  bazarr:
    image: ghcr.io/hotio/bazarr
    container_name: bazarr
    restart: unless-stopped
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - <your-path-to-data>/bazarr:/config
      - <your-path-to-data>/data/media:/data/media
    environment:
      - TZ=Europe/Madrid
      - PUID=1000
      - PGID=1000
      - UMASK=002
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.bazarr.entrypoints=web-secure"
      - "traefik.http.routers.bazarr.rule=Host(`bazarr.<your-domain>`)"
      - "traefik.http.routers.bazarr.tls=true"
      - "traefik.http.routers.bazarr.tls.certresolver=default"
      - "traefik.http.services.bazarr.loadbalancer.server.port=6767"
      - "traefik.http.routers.bazarr.middlewares=bazarr-auth"
      - "traefik.http.middlewares.bazarr-auth.basicauth.users=<your-user-and-password>"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

  jellyfin:
    image: linuxserver/jellyfin:latest
    container_name: jellyfin
    restart: always
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - <your-path-to-data>/jellyfin:/config
      - <your-path-to-data>/data/media:/data/media
    environment:
      - TZ=Europe/Madrid
      - PUID=1000
      - PGID=1000
      - UMASK=002
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.jellyfin.entrypoints=web-secure"
      - "traefik.http.routers.jellyfin.rule=Host(`jellyfin.<your-domain>`)"
      - "traefik.http.routers.jellyfin.tls=true"
      - "traefik.http.routers.jellyfin.tls.certresolver=default"
      - "traefik.http.services.jellyfin.loadbalancer.server.port=8096"
      - "traefik.http.routers.jellyfin.middlewares=jellyfin-auth"
      - "traefik.http.middlewares.jellyfin-auth.basicauth.users=<your-user-and-password>"
      - "traefik.http.routers.jellyfin.middlewares=secHeader@file"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

  jellyseerr:
     image: fallenbagel/jellyseerr:latest
     container_name: jellyseerr
     environment:
          - LOG_LEVEL=debug
          - TZ=Europe/Madrid
     ports:
          - 5055:5055
     volumes:
          - <your-path-to-data>/jellyseerr:/app/config
     restart: unless-stopped
     labels:
          - "traefik.enable=true"
          - "traefik.http.routers.jellyseerr.entrypoints=web-secure"
          - "traefik.http.routers.jellyseerr.rule=Host(`jellyseerr.<your-domain>`)"
          - "traefik.http.routers.jellyseerr.tls=true"
          - "traefik.http.routers.jellyseerr.tls.certresolver=default"
          - "traefik.http.services.jellyseerr.loadbalancer.server.port=5055"
          - "traefik.http.routers.jellyseerr.middlewares=jellyseerr-auth"
          - "traefik.http.middlewares.jellyseerr-auth.basicauth.users=<your-user-and-password>"
          - "traefik.http.routers.jellyseerr.middlewares=secHeader@file"
          - "traefik.docker.network=traefik"
     networks:
          - traefik
  prowlarr:
    image: ghcr.io/hotio/prowlarr
    container_name: prowlarr
    restart: unless-stopped
    volumes:
      - <your-path-to-data>/prowlarr:/config
    environment:
      - TZ=Europe/Madrid
      - PUID=1000
      - PGID=1000
      - UMASK=002
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.prowlarr.entrypoints=web-secure"
      - "traefik.http.routers.prowlarr.rule=Host(`prowlarr.<your-domain>`)"
      - "traefik.http.routers.prowlarr.tls=true"
      - "traefik.http.routers.prowlarr.tls.certresolver=default"
      - "traefik.http.services.prowlarr.loadbalancer.server.port=9696"
      - "traefik.http.routers.prowlarr.middlewares=prowlarr-auth"
      - "traefik.http.middlewares.prowlarr-auth.basicauth.users=<your-user-and-password>"
      - "traefik.http.routers.prowlarr.middlewares=secHeader@file"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

  qbittorrent:
    container_name: qbittorrent
    image: ghcr.io/hotio/qbittorrent
    ports:
      - 8080:8080
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Madrid
    volumes:
      - <your-path-to-data>/qbittorrent:/config
      - <your-path-to-data>/data/torrents:/data/torrents
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.qbittorrent.entrypoints=web-secure"
      - "traefik.http.routers.qbittorrent.rule=Host(`qbittorrent.<your-domain>`)"
      - "traefik.http.routers.qbittorrent.tls=true"
      - "traefik.http.routers.qbittorrent.tls.certresolver=default"
      - "traefik.http.services.qbittorrent.loadbalancer.server.port=8080"
      - "traefik.http.routers.qbittorrent.middlewares=secHeader@file"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

  radarr:
    image: cr.hotio.dev/hotio/radarr:latest
    container_name: radarr
    restart: unless-stopped
    volumes:
      - <your-path-to-data>/radarr:/config
      - /etc/localtime:/etc/localtime:ro
      - <your-path-to-data>/data:/data
    environment:
      - TZ=Europe/Madrid
      - PUID=1000
      - PGID=1000
      - UMASK=002
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.radarr.entrypoints=web-secure"
      - "traefik.http.routers.radarr.rule=Host(`radarr.<your-domain>`)"
      - "traefik.http.routers.radarr.tls=true"
      - "traefik.http.routers.radarr.tls.certresolver=default"
      - "traefik.http.services.radarr.loadbalancer.server.port=7878"
      - "traefik.http.routers.radarr.middlewares=radarr-auth"
      - "traefik.http.middlewares.radarr-auth.basicauth.users=<your-user-and-password>"
      - "traefik.http.routers.radarr.middlewares=secHeader@file"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

  readarr:
    container_name: readarr
    image: ghcr.io/hotio/readarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Madrid
    volumes:
      - <your-path-to-data>/readarr:/config
      - /etc/localtime:/etc/localtime:ro
      - <your-path-to-data>/data:/data
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.readarr.entrypoints=web-secure"
      - "traefik.http.routers.readarr.rule=Host(`readarr.<your-domain>`)"
      - "traefik.http.routers.readarr.tls=true"
      - "traefik.http.routers.readarr.tls.certresolver=default"
      - "traefik.http.services.readarr.loadbalancer.server.port=8787"
      - "traefik.http.routers.readarr.middlewares=readarr-auth"
      - "traefik.http.middlewares.readarr-auth.basicauth.users=<your-user-and-password>"
      - "traefik.http.routers.readarr.middlewares=secHeader@file"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

  sonarr:
    image: cr.hotio.dev/hotio/sonarr:latest
    container_name: sonarr
    restart: always
    volumes:
      - <your-path-to-data>/sonarr:/config
      - /etc/localtime:/etc/localtime:ro
      - <your-path-to-data>/data:/data
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Madrid
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.sonarr.entrypoints=web-secure"
      - "traefik.http.routers.sonarr.rule=Host(`sonarr.<your-domain>`)"
      - "traefik.http.routers.sonarr.tls=true"
      - "traefik.http.routers.sonarr.tls.certresolver=default"
      - "traefik.http.services.sonarr.loadbalancer.server.port=8989"
      - "traefik.http.routers.sonarr.middlewares=sonarr-auth"
      - "traefik.http.middlewares.sonarr-auth.basicauth.users=<your-user-and-password>"
      - "traefik.http.routers.sonarr.middlewares=secHeader@file"
      - "traefik.docker.network=traefik"
    networks:
      - traefik

networks:
    traefik:
        driver: bridge
        name: traefik
```

If you want secure headers, go to [my post](https://blog.jonthan.xyz/self-hosting-guide/) about traefik configuration.  
Otherwise, the line _"traefik.http.routers.<service>.middlewares=secHeader@file"_ won't work.

To create a user+password pair, follow this guide and use _htpasswd_: [https://doc.traefik.io/traefik/middlewares/http/basicauth/](https://doc.traefik.io/traefik/middlewares/http/basicauth/)  
You can use htpasswd online: [https://hostingcanada.org/htpasswd-generator/](https://hostingcanada.org/htpasswd-generator/)  

## Step #4: Run your containers!

```
sudo docker-compose up -d
```

## Step #5: Let's set up our indexers.

First of all, I want to clarify that it is not necessary to have a domain name. Knowing the IP of your server, and which ports you have to access to connect to the web interfaces, is more than enough. However, you will have to expose those ports to the internet...

I leave to you to decide how to do it: you can use Nginx reverse proxy, Nginx manager, traefik reverse proxy in Docker, traefik as IngressController...

Ok, first we have to connect to Prowlarr (use http:<yourIP>:9696, in case you don't want to use trusted certificates or domain names):

![](https://blog.jonthan.xyz/media/posts/19/server-2.png)

This is the webUI of prowlarr. Let's add a New Indexer

![](https://blog.jonthan.xyz/media/posts/19/server-3.png)

I use this one for anime movies. You can use anyone.

![](https://blog.jonthan.xyz/media/posts/19/server-4.png)

In this case, for this indexer the default values should be enough. Check the enable box.

![](https://blog.jonthan.xyz/media/posts/19/server-5.png)

I use this one too.

![](https://blog.jonthan.xyz/media/posts/19/server-6.png)

Usually, for NZB indexers you need an API Key.

![](https://blog.jonthan.xyz/media/posts/19/server-7.png)

Now, go to Settings -> Apps to add Applications

![](https://blog.jonthan.xyz/media/posts/19/server-8.png)

We have to select Radarr (movies) and Sonarr (series) for this guide. There are a few more as you can see (for instance, Lidarr is for music).

![](https://blog.jonthan.xyz/media/posts/19/server-10.png)

You have to set your radar API Key. As it is written, is generated by Radarr in Settings -> General. Click on "Test" after that.

![](https://blog.jonthan.xyz/media/posts/19/i.PNG)

That's the API Key you have to use. Same goes for Sonarr and its API Key.

![](https://blog.jonthan.xyz/media/posts/19/server-13.png)

The same goes for Sonarr. Input your Sonarr API Key. Click on "Test" after that.

![](https://blog.jonthan.xyz/media/posts/19/server-14.png)

Your applications tab should look like this.

![](https://blog.jonthan.xyz/media/posts/19/server-15.png)

Wait a few minutes. Then go to Settings -> Indexers inside Sonarr and Radarr and you should see something like this.

## Step #6: Set up your torrent client.

"But I want to use usenet instead of torrents". Ok, well, I will make a more complete guide taking usenet into account. But for now, let's stick with torrents.

In any case, [here](https://morrismotel.com/servarr-pt3a-choosing-your-indexers/) you have a "Pros and cons" list.

And [here](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/) you have some additional documentation.

Ok, in order to connect to the qbittorrent webUI, we have to use port 9095 (you can change this in the **docker-compose.yml** file).

Connect to http://<yourIP>:9095, then enter the default username (admin) and the default password (adminadmin).

![](https://blog.jonthan.xyz/media/posts/19/server-16.png)

Enter in Tools -> Options. Then, in the Connection tab, mark the same options as me. I'm not going to explain why this works, maybe in the next guide.

![](https://blog.jonthan.xyz/media/posts/19/server-17.png)

Mark this option in case your server is at home, but should be disabled for several security reasons.

![](https://blog.jonthan.xyz/media/posts/19/server-18.png)

Now, in the BitTorrent tab, mark this.

![](https://blog.jonthan.xyz/media/posts/19/server-27.png)

Lastly, my Downloads tab.

## Step #7: Set up Radarr and Sonarr.

Radarr and Sonarr are very similar. So the configuration we're going to do is the exact same for both. We're going to use Radarr as the example but the configuration is the same for Sonarr as well. Let's get started.

Go to http://<yourIP>:7878, then go to Settings -> Download Clients, click the add button:

![](https://blog.jonthan.xyz/media/posts/19/server-20.png)

You should see something like this.

![](https://blog.jonthan.xyz/media/posts/19/server-21.png)

If you want to use usenet, select for instance NZBGet and fill the information.

![](https://blog.jonthan.xyz/media/posts/19/server-22.png)

To configure bittorrent, use the login info to fill the gaps.

![](https://blog.jonthan.xyz/media/posts/19/server-23.png)

Repeat this process for Sonarr.

## Step #8: Set your quality profiles.

This step is important, as important as people have created [tools](https://github.com/recyclarr/recyclarr) to optimise this step, using [special configurations](https://github.com/recyclarr/recyclarr/wiki/Configuration-Reference).

I leave [here](https://trash-guides.info/Radarr/Radarr-Quality-Settings-File-Size/) some good documentation.

Go to the Profiles tab, in Radarr and Sonarr:

![](https://blog.jonthan.xyz/media/posts/19/server-24.png)

I leave here my "Any" profile for both services:

![](https://blog.jonthan.xyz/media/posts/19/server-25.png)

Sonar "Any" Profile

![](https://blog.jonthan.xyz/media/posts/19/server-26.png)

Radarr "Any" Profile

Now, let's get some series and movies. In Radarr or Sonarr, go to the home page, search for the movie or tv show you want to watch, and select it. You should see a screen like this one:

![](https://blog.jonthan.xyz/media/posts/19/server-11.png)

Select your preferred quality profile, and click on "Add Movie".

Then, click on the movie poster.

![](https://blog.jonthan.xyz/media/posts/19/server-12.png)

Go to the search tab and select your preferred option based on size and format.

In this guide I will use [Jellyfin](https://github.com/jellyfin/jellyfin), but feel free to use any other, like [Plex](https://www.plex.tv/es/media-server-downloads/), [qflood](https://github.com/jesec/flood)... just change the **docker-compose.yml** to deploy your selected app.

For Jellyfin, go to http://<yourIP>:8096.

![](https://blog.jonthan.xyz/media/posts/19/server-9.png)

Login screen. Just enter your username and password when prompted.

![](https://blog.jonthan.xyz/media/posts/19/server-19.png)

You should be seeing something similar to this.

Ok, if you click on "Movies", there will be a list of your downloaded movies displayed.

![](https://blog.jonthan.xyz/media/posts/19/server-28.png)

You can even filter your movies and series depending of genres, make collections, connect to your smart tv to display there the movie you want to see...

And that's all, folks! You have a torrent site running in your server with no hassle at all.

As soon as possible I will upload a more complex configuration, with bazarr for subtitles, and quality settings for both sonarr and radarr using recyclarr, also using jellyseer for managing user requests.

 Enjoy the piracy!

## Recommendations

Set a bazarr service to get subtitles. I will make a guide about it as soon as possible.

Set a service to manage user request. I use Jellyseer for this purpose. I will make a guide about it too. Pinky promise.

Do not run a torrent site in a server you don't own, like an AWS instance. They can ban you for that.

If you want to tweak Jellyfin a bit, with themes and stuff, check this [video](https://www.youtube.com/watch?v=DivZcyoY6bs).

If you prefer to [use traefik as a reverse proxy](https://blog.jonthan.xyz/self-hosting-guide/) and qflood as a media server, you can use this **docker-compose.yml** file instead the one already written above:

```
version: '3'
  services:
    qflood:
      image: cr.hotio.dev/hotio/qflood:release-4.3.9--4.7.0
      environment:
        - PUID=1000
        - PGID=100
        - UMASK=002
        - TZ=Europe/Madrid
        - FLOOD_AUTH=true
      ports:
        - "51413:51413"
      volumes:
        - '/your/pathstorage/data/qflood:/config'
        - '/your/path:/data'
        - '/storage/shared/bittorrent:/downloads'
      labels:
      - traefik.enable=true
      - traefik.http.routers.flood.entryPoints=web-secure
      - traefik.http.routers.flood.rule=Host(`yourdomain`)
      - traefik.http.services.flood.loadbalancer.server.port=3000
      - traefik.http.routers.flood.service=flood
      - traefik.http.routers.qbittorrent.entryPoints=web-secure
      - traefik.http.routers.qbittorrent.rule=Host(`yourdomain`)
      - traefik.http.services.qbittorrent.loadbalancer.server.port=8080
      - traefik.http.routers.qbittorrent.service=qbittorrent
      networks:
        - rflood
      restart: unless-stopped
    prowlarr:
      image: cr.hotio.dev/hotio/prowlarr:nightly
      ports:
        - "9696:9696"
      environment:
        - PUID=1000
        - PGID=1000
        - UMASK=002
        - TZ=Europe/Madrid
      volumes:
        - /your/path:/config
      labels:
        - traefik.enable=true
        - traefik.http.routers.prowlarr.entryPoints=web-secure
        - traefik.http.routers.prowlarr.rule=Host(`yourdomain`)
        - traefik.http.services.prowlarr.loadbalancer.server.port=9696
      networks:
        - rflood
      restart: unless-stopped
    sonarr:
      image: cr.hotio.dev/hotio/sonarr
      ports:
      - "8989:8989"
      environment:
      - PUID=1000
      - PGID=100
      - UMASK=002
      - TZ=Europe/Madrid
      volumes:
      - './config/arr/sonarr/config:/config'   
      - '/storage/shared/bittorrent:/storage'
      labels:
      - traefik.enable=true
      - traefik.http.routers.sonarr.entryPoints=web-secure
      - traefik.http.routers.sonarr.rule=Host(`yourdomain`)
      - traefik.http.services.sonarr.loadbalancer.server.port=8989
      networks:
      - rflood
      restart: unless-stopped
    radarr:
      image: cr.hotio.dev/hotio/radarr
      ports:
      - "7878:7878"
      environment:
      - PUID=1000
      - PGID=100
      - UMASK=002
      - TZ=Europe/Madrid
      volumes:
      - './config/arr/radarr/config:/config'
      - '/storage/shared/bittorrent:/storage'
      labels:
      - traefik.enable=true
      - traefik.http.routers.radarr.entryPoints=web-secure
      - traefik.http.routers.radarr.rule=Host(`yourdomain`)
      - traefik.http.services.radarr.loadbalancer.server.port=7878
      networks:
      - rflood
      restart: unless-stopped
    bazarr:
      image: hotio/bazarr
      ports:
      - "6767:6767"
      environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Europe/Madrid
      volumes:
      - ./config/arr/bazarr/config:/config
      - /storage/shared/bittorrent/media:/storage/media
      labels:
      - traefik.enable=true
      - traefik.http.routers.bazarr.entryPoints=web-secure
      - traefik.http.routers.bazarr.rule=Host(`yourdomain`)
      - traefik.http.services.bazarr.loadbalancer.server.port=6767      
      networks:
      - rflood      
      restart: unless-stopped
  networks:
    rflood:
      driver: bridge
```

As you can see, you have total freedom to choose where you want to store your data and configuration. Set a path wherever it says **"/your/path"**. Also, the bazarr service (for subtitles) is included.

Oh, here is another possibility, if you want to expose your ports:

```
version: "3.7"
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment: &env
      - PUID=<Enter your USER_ID>
      - PGID=<Enter your GROUP_ID>
      - TZ=<Enter your time zone>
    volumes:
      - ./config/jellyfin:/config
      - ./library/series:/data/series
      - ./library/movies:/data/movies
    ports:
      - 8096:8096
      - 8920:8920 #optional
      - 7359:7359/udp #optional
      - 1900:1900/udp #optional
    restart: &restartpolicy unless-stopped
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment: *env
    volumes:
      - ./config/arr/radarr/config:/config
      - ./library/movies:/data/movies #optional
      - ./downloads:/downloads #optional
      - /etc/localtime:/etc/localtime:ro
    ports:
      - 7878:7878
    restart: *restartpolicy
  sonarr:
    container_name: sonarr
    image: cr.hotio.dev/hotio/sonarr:latest
    restart: *restartpolicy
    logging:
      driver: json-file
    network_mode: bridge
    ports:
      - 8989:8989
    environment: *env
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./config/arr/sonarr/config:/config
      - ./downloads:/downloads #optional
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=<Enter your USER_ID>
      - PGID=<Enter your GROUP_ID>
      - TZ=<Enter your time zone>
      - WEBUI_PORT=9095
    volumes:
      - ./torrent/qbittorrent/config:/config
      - ./downloads:/downloads
    ports:
      - 9095:9095
      - 6881:6881
      - 6881:6881/udp
    restart: *restartpolicy
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:develop
    container_name: prowlarr
    environment: *env
    volumes:
      - ./torrent/prowlarr/config:/config
    ports:
      - 9696:9696
    restart: *restartpolicy
```

## Credits

My friend Edu, for discovering me Servarr.

[https://wiki.servarr.com/](https://wiki.servarr.com/)

[TRaSH Guides](https://trash-guides.info/)

[https://morrismotel.com/servarr-pt3b-prowlarr-sonarr-radarr/](https://morrismotel.com/servarr-pt3b-prowlarr-sonarr-radarr/)