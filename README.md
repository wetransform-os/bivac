Conplicity
==========

[![Docker Pulls](https://img.shields.io/docker/pulls/camptocamp/conplicity.svg)](https://hub.docker.com/r/camptocamp/conplicity/)
[![Build Status](https://img.shields.io/travis/camptocamp/conplicity/master.svg)](https://travis-ci.org/camptocamp/conplicity)
[![By Camptocamp](https://img.shields.io/badge/by-camptocamp-fb7047.svg)](http://www.camptocamp.com)


conplicity lets you backup all your named docker volumes using duplicity.


## Installing

```shell
$ go get github.com/camptocamp/conplicity
```

## Usage

```shell
Usage:
  conplicity [OPTIONS]

Application Options:
  -V, --version             Display version.
  -i, --image=              The duplicity docker image. (default: camptocamp/duplicity:latest) [$DUPLICITY_DOCKER_IMAGE]
  -l, --loglevel=           Set loglevel. (default: info) [$CONPLICITY_LOG_LEVEL]
  -b, --blacklist=          Volumes to blacklist in backups. [$CONPLICITY_VOLUMES_BLACKLIST]

Duplicity Options:
  -u, --url=                The duplicity target URL to push to. [$DUPLICITY_TARGET_URL]
      --full-if-older-than= The number of days after which a full backup must be performed. (default: 15D) [$CONPLICITY_FULL_IF_OLDER_THAN]
      --remove-older-than=  The number days after which backups must be removed. (default: 30D) [$CONPLICITY_REMOVE_OLDER_THAN]

Metrics Options:
  -g, --gateway-url=        The prometheus push gateway URL to use. [$PUSHGATEWAY_URL]

AWS Options:
      --aws-access-key-id=  The AWS access key ID. [$AWS_ACCESS_KEY_ID]
      --aws-secret-key-id=  The AWS secret access key. [$AWS_SECRET_ACCESS_KEY]

Swift Options:
      --swift-username=     The Swift user name. [$SWIFT_USERNAME]
      --swift-password=     The Swift password. [$SWIFT_PASSWORD]
      --swift-auth_url=     The Swift auth URL. [$SWIFT_AUTHURL]
      --swift-tenant-name=  The Swift tenant name. [$SWIFT_TENANTNAME]
      --swift-region-name=  The Swift region name. [$SWIFT_REGIONNAME]

Help Options:
  -h, --help                Show this help message
```

## Examples

### Backup all named volumes to S3

```shell
$ conplicity \
  -u s3://s3-eu-west-1.amazonaws.com/<my_bucket>/<my_dir> \
  --aws-access-key-id=<my_key_id> \
  --aws-secret-key-id=<my_secret_key>
```


### Using docker

```shell
$ docker run -v /var/run/docker.sock:/var/run/docker.sock:ro  --rm -ti \
   -e DUPLICITY_TARGET_URL=s3://s3-eu-west-1.amazonaws.com/<my_bucket>/<my_dir> \
   -e AWS_ACCESS_KEY_ID=<my_key_id> \
   -e AWS_SECRET_ACCESS_KEY=<my_secret_key> \
     camptocamp/conplicity
```

## Environment variables

### CONPLICITY_FULL_IF_OLDER_THAN

Perform a full backup if an incremental backup is requested, but the latest full backup in the collection is older than the given time. Defaults to 15D.

### CONPLICITY_REMOVE_OLDER_THAN

Delete all backup sets older than the given time. Defaults to 30D.

### CONPLICITY_VOLUMES_BLACKLIST

Comma separated list of named volumes to blacklist.

### DUPLICITY_DOCKER_IMAGE

The image to use to launch duplicity. Default is `camptocamp/duplicity:latest`.

### DUPLICITY_TARGET_URL

Target URL passed to duplicity.
The hostname and the name of the volume to backup
are added to the path as directory levels.

### PUSHGATEWAY_URL

The Url of a Prometheus pushgateway to push metrics to.

### S3 credentials

- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

### Swift credentials

- SWIFT_USERNAME
- SWIFT_PASSWORD
- SWIFT_AUTHURL
- SWIFT_TENANTNAME
- SWIFT_REGIONNAME

## Controlling backup parameters

The parameters used to backup each volume can be fine-tuned using volume labels (requires Docker 1.11.0 or greater):

- `io.conplicity.ignore=true` ignores the volume
- `io.conplicity.full_if_older_than=<value>` sets the time period after which a full backup is performed. Defaults to the `CONPLICITY_FULL_IF_OLDER_THAN` environment variable value


## Providers


Conplicity detects automatically the kind of data that is stored on a volume and adapts its backup strategy to it. The following providers and associated strategies are currently supported:

* PostgreSQL: Run `pg_dumpall` before backup
* MySQL: Run `mysqldump` before backup
* OpenLDAP: Run `slapcat` before backup
* Default: Backup volume data as is

