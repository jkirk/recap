# Recap

[![GitHub release](https://img.shields.io/github/release/rackerlabs/recap.svg)](https://github.com/rackerlabs/recap/releases/latest)
[![GitHub license](https://img.shields.io/github/license/rackerlabs/recap.svg)](https://raw.githubusercontent.com/rackerlabs/recap/master/COPYING)
[![GitHub stars](https://img.shields.io/github/stars/rackerlabs/recap.svg?style=social&label=Star)](https://github.com/rackerlabs/recap)
[![Twitter](https://img.shields.io/twitter/url/https/github.com/rackerlabs/recap.svg?style=social)](https://twitter.com/intent/tweet?text=Check%20this%20out:&url=https%3A%2F%2Fgithub.com%2Frackerlabs%2Frecap)

**recap** is a system status reporting tool. A reporting script that generates reports of various information about the server.

## Contribution

Contribution guidelines can be found in [CONTRIBUTING.md](https://github.com/rackerlabs/recap/blob/master/CONTRIBUTING.md)

## Dependencies
- bash >= 4
- coreutils
- gawk
- grep
- iotop
- iproute/iproute2
- links
- procps
- psmisc
- sysstat >= 9

## Versioning

`recap` is following the `x.y.z` versioning as defined below:

 - **x** *(major)* - Changes that prevent at least some rolling upgrades.
 - **y** *(minor)* - Changes that don't break any rolling upgrades but require closer user attention for example configuration defaults, function behavior, tools used to produce reports, among others.
 - **z** *(patch)* - Changes that are backward-compatible including features and/or bug fixes.

## Installation

It is highly recommended to make use of a package to install `recap` is the easiest way to keep it updated whenever there is a new release.

### Fedora

`recap` is available from the main Fedora repository ([spec file](https://src.fedoraproject.org/rpms/recap/blob/master/f/recap.spec)).

```
dnf install recap
```

### RHEL/CentOS

`recap` is available from the [EPEL](https://fedoraproject.org/wiki/EPEL) repository ([spec file](https://src.fedoraproject.org/rpms/recap/blob/master/f/recap.spec)).

```
yum install recap
```

### Debian / Ubuntu

At the moment there is no public repository for Debian nor Ubuntu, two options are available:

#### Build a package

This repository https://github.com/raxpkg/recap contains the debian files required to build a deb package

These are the steps:

```bash
# Install all the packages required for building the package
apt-get update
apt-get install debhelper devscripts git -y

## For Ubuntu:
apt-get install equivs -y

# Create the working dir:
mkdir recap
cd recap

# Get the debian configs
git init
git remote add origin https://github.com/raxpkg/recap.git
git fetch --no-tags origin
git checkout -qf FETCH_HEAD
git submodule update --init --recursive
export LATEST=$( git log --format="%h" --no-merges -1 )

# Build dependencies
echo "yes" | mk-build-deps --install --remove debian/control

# Get upstream recap code
git checkout --orphan upstream
git reset --hard
git remote add upstream https://github.com/rackerlabs/recap.git
git fetch -t upstream
latest_tag=$( git tag | tail -1 )
git archive ${latest_tag} -o ../recap_${latest_tag}.orig.tar.gz
tar -zxf ../recap_${latest_tag}.orig.tar.gz
git fetch --no-tags origin
git checkout ${LATEST} -- debian

# Build the package
debuild -us -uc --lintian-opts --profile debian

# Package will be created in ../recap_${latest_tag}-<RELEASE>_all.deb
# RELEASE comes from the changelog in the debian repository.
```

#### Manual install

Use the [manual installation](#manual) method.


### Manual

1. Install the required dependencies.
2. Clone this repository: `git clone https://github.com/rackerlabs/recap.git`
3. Change into the new directory: `cd recap`
4. Install the program: `sudo make install`

The information captured will be found in log files in the `/var/log/recap/` directory.

#### About the locations of the scripts

The default location of the install is `"/"` it can be overriden with `DESTDIR`, the scripts, man pages and docs are installed under "`"/usr/local"` by default, this can be overriden with `PREFIX`. The following example is a common location for most of the distributions, this will install `recap` under `/usr`:

  ```
$ sudo make PREFIX="/usr" install 
```

This other example will install `recap` under your homedirectory but using the default locations for the script, i.e. under `"~./usr/local"`:

  ```
$ make DESTDIR="~" install
```

The `Makefile` scripts attempts to detect systemd if so, the `install` option will install the systemd unit files. The install will **not** enable the timers, but it will show the commands required to enable each of the timers.  When systemd is not detected the cronjobs will be installed.

Is up to each package distribution to follow their own best practices regarding enabling/disabling the timers on install/remove of the package.

## Cron/Timers and Configuration

### Timers(systemd)

Multiple unit files are available to make use of `timers`, here the default schedules for the recap scripts:
- recap (default every 10min)
- recap-onboot (runs at boot time)
- recaplog (default: Once a day 1am)

#### Enabling timers

Each one of the timers can be enabled with:

  ```bash
  sudo systemctl enable recap.timer --now"
  sudo systemctl enable recaplog.timer --now"
  sudo systemctl enable recap-onboot.timer --now"
  ```

#### Disabling timers

Each one of the timers can be disabled with:

  ```bash
  sudo systemctl disable recap.timer --now"
  sudo systemctl disable recaplog.timer --now"
  sudo systemctl disable recap-onboot.timer --now"
  ```

### Cron

The cron file (`/etc/cron.d/recap`) is used to determine the execution time of `recap` and `recaplog`.  By default the cron execution for `recap` is enabled to run every 10 min. and `recaplog` is expected to run every day at 1 am, but those can be adjusted as needed.

### Configuration

The following variables are commented out with the defaults values in the configuration file `/etc/recap.conf` which can be overriden.

#### Settings shared by recap scripts

- **BASEDIR** - Directory where the logs are saved.

  Default: `BASEDIR="/var/log/recap"`

- **BACKUP_ITEMS** - Is the list of reports generated and used by recap scripts

  Default: `BACKUP_ITEMS="fdisk mysql netstat ps pstree resources"`


#### Settings used only by `recaplog`

- **LOG_COMPRESS** - Enable or disable log compression.

  Default: `LOG_COMPRESS=1`

- **LOG_EXPIRY** - Log files will be deleted after LOG_EXPIRY days

  Default: `LOG_EXPIRY=15`

#### Settings used only by `recap`

- **MAILTO** - Send a report to the email defined.

  Default: `MAILTO=""`

- **MIN_FREE_SPACE** - Minimum free space (in MB) required in `${BASEDIR}` to run recap, a value of 0 deactivates this check.

  Default: `MIN_FREE_SPACE=0`


#### Reports

These are the type of reports generated and their dependencies.

##### fdisk

- **USEFDISK** - Generates logs from "fdisk `${OPTS_FDISK}`"

  Default: `USEFDISK="no"`

##### mysql

- **USEMYSQL** - Generates logs from "mysqladmin status"

  Makes use of `DOTMYDOTCNF`.

  Required by: `USEMYSQLPROCESSLIST`, `USEINNODB`

  Default: `USEMYSQL="no"`

- **USEMYSQLPROCESSLIST** - Generates logs from "mysqladmin processlist"

  Makes use of `DOTMYDOTCNF` and `MYSQL_PROCESS_LIST`

  Requires: `USEMYSQL`

  Default: `USEMYSQLPROCESSLIST="no"`

- **USEINNODB** - Generates logs from "mysql show engine innodb status"

  Makes use of `DOTMYDOTCNF`

  Requires: `USEMYSQL`

  Default: `USEINNODB="no"`


##### netstat

- **USENETSTAT** - Generates network stats from "ss `${OPTS_NETSTAT}`"

  Required by: `USENETSTATSUM`

  Default: `USENETSTAT="yes"`


- **USENETSTATSUM** - Generates logs from "nstat `${OPTS_NETSTAT_SUM}`".

  Requires: `USENETSTAT`

  Default: `USENETSTATSUM="no"`

##### ps

- **USEPS** - Generates logs from "ps"

  Options can be modified in `OTPS_PS`

  Default: `USEPS="yes"`

##### pstree

- **USEPSTREE** - Generates logs from pstree

  Options can be modified in `OTPS_PSTREE`

  Default: `USEPSTREE="no"`

##### resources

- **USERESOURCES** - Generates "resources"(uptime, free, vmstat, iostat, iotop) log

  Required by: `USEDF`, `USESLAB`, `USESAR`, `USESARQ`, `USESARR`, `USEFULLSTATUS`

  Default: `USERESOURCES="yes"`


- **USEDF** - Generates logs from df

  Requires: `USERESOURCES`

  Options can be modified in `OPTS_DF`

  Default: `USEDF="yes"`

- **USESLAB** - Generates logs from the slab table.

  Requires: `USERESOURCES`

  Default: `USESLAB="no"`


- **USERSAR** - Generates logs from sar.

  Requires: `USERESOURCES`

  Default: `USESAR="yes"`

- **USESARQ** - Generates logs from "sar -q" (logs queue lenght, load data)

  Requires: `USERESOURCES`

  Default: `USESARQ="no"`

- **USESARR** - Generates logs from"sar -r" (logs memory data)

  Requires: `USERESOURCES`

  Default: `USESARR="no"`

- **USEFULLSTATUS** - Performs an http request(GET) to the URL defined in `OPTS_STATUSURL`. Needs a webserver configured to respond to this request. Nginx(nginx_status) and Apache HTTPD(server-stats) offer this functionality that needs to be enabled.

  Requires: `USERESOURCES`

  Default: `USEFULLSTATUS="no"`


#### Options

Options used by the tools generating the reports

- **DOTMYDOTCNF** - Defines the path to the mysql client configuration file

  Required by: `USEMYSQL`, `USEMYSQLPROCESSLIST`, `USEINNODB`

  Default: `DOTMYDOTCNF="/root/.my.cnf"`

- **MYSQL_PROCESS_LIST** - Format  to  display MySQL process list, options are "table" or "vertical".

  Required by: `USEMYSQLPROCESSLIST`

  Default: `MYSQL_PROCESS_LIST="table"`

- **OPTS_LINKS** - Options used by links.
  Required by: `USEFULLSTATUS`

  Default: `OPTS_LINKS="-dump"`

- **OPTS_DF** - df options

  Required by: `USEDF`

  Default: `OPTS_DF="-x nfs"`

- **OPTS_FDISK** - Option used by USEFDISK.

  Required by: `USEFDISK`

  Default: `OPTS_FDISK="-l"`

- **OPTS_FREE** - free options

  Required by: `USEFREE`

  Default: `OPTS_FREE=""`

- **OPTS_IOSTAT** - iostat options

  Required by: `USERESOURCES`

  Default: `OPTS_IOSTAT="-t -x 1 3"`

- **OPTS_IOTOP** - iotop options

  Required by: `USERESOURCES`

  Default: `OPTS_IOTOP="-b -o -t -n 3"`

- **OPTS_NETSTAT** - ss options

  Required by: `USENETSTAT`

  Default: `OPTS_NETSTAT="-atunp"`

- **OPTS_NETSTAT_SUM** - nstat options

  Required by: `USENETSTATSUM`

  Default: `OPTS_NETSTAT_SUM="-a"`

- **OPTS_PS** - ps options

  Required by: `USEPS`

  Default: `OPTS_PS="auxfww"`

- **OPTS_PSTREE** - pstree options

  Required by: `USEPSTREE`

  Default: `OPTS_PSTREE="-p"`


- **OPTS_STATUSURL** - URL to perform the http request  when USEFULLSTATUS is enabled.

  Required by: `USEFULLSTATUS`

  Default: `OPTS_STATUSURL="http://localhost:80/server-status"`

- **OPTS_VMSTAT** - vmstat options

  Required by: `USERESOURCES`

  Default: `OPTS_VMSTAT="-S M 1 3"`

## Changelog & Contributions

Information about changes and contributors is documented in the [CHANGELOG.md](https://github.com/rackerlabs/recap/blob/master/CHANGELOG.md)

## License

*recap* is licensed under the [GNU General Public License v2.0](https://raw.githubusercontent.com/rackerlabs/recap/master/COPYING)

