# Ansible Role: Multi-Redis

[![Build Status](https://travis-ci.com/davidalger/ansible-role-multi-redis.svg?branch=master)](https://travis-ci.com/davidalger/ansible-role-multi-redis)

Installs multiple instances of Redis 5 from [IUS Community Project](http://ius.io) RPMs on RHEL / CentOS 7 and from AppStreams on RHEL / CentOS 8. Where archived verions of redis are required (such as Redis 3.0, 3.1 or 3.2), the ius-archive repository may be enabled to facilitate installing older versions on EL 7 (versions older than 5.0 are not supported on EL 8)

Currently this role installs redis pre-configured with defaults tuned for Magento. Some of these defaults may be different if this role is used to deploy redis in a non-Magento environment. See `defaults/main.yml` for details.

## Requirements

None.

## Role Variables

    redis_version: 5

On EL 7 any redis version supported by IUS RPMs may be specified: 30, 32, etc. For EL 8 only newer versions such as '5' (and later, once they are released as additional module streams) are supported.

See `defaults/main.yml` for complete list of variables available to customize the redis services.

## Dependencies

* `davidalger.repo_ius`
* `davidalger.selinux`

## Example Playbook

* Production triple-redis deployment:

        - { role: redis, tags: redis, redis_instance: { name: obj, port: 6379 }}
        - { role: redis, tags: redis, redis_instance: { name: fpc, port: 6381 }}
        - { role: redis, tags: redis, redis_instance: { name: ses, port: 6380, save: yes }}

* Stage triple-redis deployment:

        - { role: redis, tags: redis, redis_instance: { name: stage-obj, port: 6389 }}
        - { role: redis, tags: redis, redis_instance: { name: stage-fpc, port: 6391 }}
        - { role: redis, tags: redis, redis_instance: { name: stage-ses, port: 6390, save: yes }}

* Customized Instance Configs

    Override defaults and supply `maxmemory-policy` in the instance config instead of the base config:

        # Key/value hash of settings for /etc/redis-{{ redis_instance.name }}.conf
        redis_instance_config:
          - maxmemory-policy: "volatile-lru"

        # Key/value hash of settings for /etc/redis-base.conf
        redis_config:
          - daemonize: "yes"
          - timeout: "0"
          - loglevel: "notice"
          - databases: "2"
          - rdbcompression: "no"
          - dbfilename: "dump.rdb"
          - appendonly: "no"
          - appendfsync: "everysec"
          - no-appendfsync-on-rewrite: "no"
          - slowlog-log-slower-than: "10000"
          - slowlog-max-len: "1024"
          - list-max-ziplist-entries: "512"
          - list-max-ziplist-value: "64"
          - set-max-intset-entries: "512"
          - zset-max-ziplist-entries: "128"
          - zset-max-ziplist-value: "64"
          - activerehashing: "yes"
          - slave-serve-stale-data: "yes"
          - auto-aof-rewrite-percentage: "100"
          - auto-aof-rewrite-min-size: "64mb"
          - tcp-backlog: "511"
          - tcp-keepalive: "0"
          - repl-disable-tcp-nodelay: "no"

    Then when calling the role, specify the instance config value per role where it needs to be different

        - { role: redis, tags: redis, redis_instance: { name: obj, port: 6379 }, redis_maxmemory: 8gb, redis_instance_config: [{maxmemory-policy: allkeys-lru}] }
        - { role: redis, tags: redis, redis_instance: { name: fpc, port: 6381 }, redis_maxmemory: 8gb, redis_instance_config: [{maxmemory-policy: allkeys-lru}] }
        - { role: redis, tags: redis, redis_instance: { name: ses, port: 6380, save: yes }, redis_maxmemory: 8gb}

## License

This work is licensed under the MIT license. See LICENSE file for details.

## Author Information

This role was created in 2017 by [David Alger](http://davidalger.com/) with contributions from [Matt Johnson](https://github.com/mttjohnson/).
