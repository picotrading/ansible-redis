redis
=====

Ansible Role which installs and configures Redis server.

The configuraton of the role is done in such way that it should not be necessary
to change the role for any kind of configuration. All can be done either by
changing role parameters or by declaring completely new configuration as a
variable. That makes this role absolutely universal. See the examples below for
more details.

Please report any issues or send PR.


Examples
--------

```
---

# Example of how to use the role with default parameters
- hosts: myhost1
  roles:
    - redis

# Example how to change some of the default parameters
- hosts: myhost2
  vars:
    # Change the bind IP
    redis_general_bind: 192.168.1.100
  roles:
    - redis

# Example of how to add additional parameter
- hosts: myhost3
  vars:
    redis_replication_2_4:
      # We must copy the default set of parameters. We can use the default
      # variable as its value.
      slave-serve-stale-data: "{{ redis_replication_slave_serve_stale_data }}"
      # This is the parameter we want to add to the default configuration
      slave-priority: 100
  roles:
    - redis

# You can also define completely new configuration from scratch
- hosts: myhost4
  vars:
    redis_config:
      daemonize: "yes"
      pidfile: /var/run/redis/redis.pid
      port: 6379
      bind: 127.0.0.1
      # Here continue with the list of parameters...
      ...
  roles:
    - redis
```

This role requires [Jinja2 Encoder
Macros](https://github.com/picotrading/jinja2-encoder-macros) which must be
placed into the same directory as the playbook:

```
$ ls -1 *.yaml
site.yaml
$ git clone https://github.com/picotrading/jinja2-encoder-macros.git ./templates/encoder
```


Role variables
--------------

```
# General
redis_general_daemonize: "yes"
redis_general_pidfile: /var/run/redis/redis.pid
redis_general_port: 6379
redis_general_tcp_backlog: 511
redis_general_bind: 127.0.0.1
redis_general_unixsocket: /tmp/redis.sock
redis_general_unixsocketperm: 700
redis_general_timeout: 0
redis_general_tcp_keepalive: 0
redis_general_loglevel: notice
redis_general_logfile: /var/log/redis/redis.log
redis_general_syslog_enabled: "no"
redis_general_syslog_ident: redis
redis_general_syslog_facility: local0
redis_general_databases: 16

redis_general_2_4: &redis_general_2_4
  daemonize: "{{ redis_general_daemonize }}"
  pidfile: "{{ redis_general_pidfile }}"
  port: "{{ redis_general_port }}"
  bind: "{{ redis_general_bind }}"
  #unixsocket: "{{ redis_general_unixsocket }}"
  #unixsocketperm: "{{ redis_general_unixsocketperm }}"
  timeout: "{{ redis_general_timeout }}"
  loglevel: "{{ redis_general_loglevel }}"
  logfile: "{{ redis_general_logfile }}"
  #syslog-enabled: "{{ redis_general_syslog_enabled }}"
  #syslog-ident: "{{ redis_general_syslog_ident }}"
  #syslog-facility: "{{ redis_general_syslog_facility }}"
  databases: "{{ redis_general_databases }}"

redis_general_2_6: &redis_general_2_6
  << : *redis_general_2_4
  tcp-keepalive: "{{ redis_general_tcp_keepalive }}"

redis_general_2_8: &redis_general_2_8
  << : *redis_general_2_6
  tcp-backlog: "{{ redis_general_tcp_backlog }}"


# Snapshotting
redis_snapshotting_save:
  - 900 1
  - 300 10
  - 60 10000
redis_snapshotting_stop_writes_on_bgsave_error: "yes"
redis_snapshotting_rdbcompression: "yes"
redis_snapshotting_rdbchecksum: "yes"
redis_snapshotting_dbfilename: dump.rdb
redis_snapshotting_dir: /var/lib/redis/

redis_snapshotting_2_4: &redis_snapshotting_2_4
  save: "{{ redis_snapshotting_save }}"
  rdbcompression: "{{ redis_snapshotting_rdbcompression }}"
  dbfilename: "{{ redis_snapshotting_dbfilename }}"
  dir: "{{ redis_snapshotting_dir }}"

redis_snapshotting_2_6: &redis_snapshotting_2_6
  << : *redis_snapshotting_2_4
  stop-writes-on-bgsave-error: "{{ redis_snapshotting_stop_writes_on_bgsave_error }}"
  rdbchecksum: "{{ redis_snapshotting_rdbchecksum }}"

redis_snapshotting_2_8: &redis_snapshotting_2_8
  << : *redis_snapshotting_2_6


# Replication
redis_replication_slaveof: <masterip> <masterport>
redis_replication_masterauth: <master-password>
redis_replication_slave_serve_stale_data: "yes"
redis_replication_slave_read_only: "yes"
redis_replication_repl_diskless_sync: "no"
redis_replication_repl_diskless_sync_delay: 5
redis_replication_repl_ping_slave_period: 10
redis_replication_repl_timeout: 60
redis_replication_repl_disable_tcp_nodelay: "no"
redis_replication_repl_backlog_size: 1mb
redis_replication_repl_backlog_ttl: 3600
redis_replication_slave_priority: 100
redis_replication_min_slaves_to_write: 0
redis_replication_min_slaves_max_lag: 10

redis_replication_2_4: &redis_replication_2_4
  #slaveof: "{{ redis_replication_slaveof }}"
  #masterauth: "{{ redis_replication_masterauth }}"
  slave-serve-stale-data: "{{ redis_replication_slave_serve_stale_data }}"
  #repl-ping-slave-period: "{{ redis_replication_repl_ping_slave_period }}"
  #repl-timeout: "{{ redis_replication_repl_timeout }}"
  ##slave-priority: "{{ redis_replication_slave_priority }}"

redis_replication_2_6: &redis_replication_2_6
  << : *redis_replication_2_4
  slave-read-only: "{{ redis_replication_slave_read_only }}"
  repl-disable-tcp-nodelay: "{{ redis_replication_repl_disable_tcp_nodelay }}"

redis_replication_2_8: &redis_replication_2_8
  << : *redis_replication_2_6
  repl-diskless-sync: "{{ redis_replication_repl_diskless_sync }}"
  repl-diskless-sync-delay: "{{ redis_replication_repl_diskless_sync_delay }}"
  repl-backlog-size: "{{ redis_replication_repl_backlog_size }}"
  repl-backlog-ttl: "{{ redis_replication_repl_backlog_ttl }}"
  min-slaves-to-write: "{{ redis_replication_min_slaves_to_write }}"
  min-slaves-max-lag: "{{ redis_replication_min_slaves_max_lag }}"


# Security
redis_security_requirepass: foobared
redis_security_rename_command: []

redis_security_2_4: &redis_security_2_4
  #requirepass: "{{ redis_security_requirepass }}"
  rename-command: "{{ redis_security_rename_command }}"

redis_security_2_6: &redis_security_2_6
  << : *redis_security_2_4

redis_security_2_8: &redis_security_2_8
  << : *redis_security_2_6


# Limits
redis_limits_maxclients: 10000
redis_limits_maxmemory: 524288
redis_limits_maxmemory_policy: volatile-lru
redis_limits_maxmemory_samples: 3

redis_limits_2_4: &redis_limits_2_4
  maxclients: "{{ redis_limits_maxclients }}"
  maxmemory: "{{ redis_limits_maxmemory }}"
  maxmemory-policy: "{{ redis_limits_maxmemory_policy }}"
  maxmemory-samples: "{{ redis_limits_maxmemory_samples }}"

redis_limits_2_6: &redis_limits_2_6
  << : *redis_limits_2_4

redis_limits_2_8: &redis_limits_2_8
  << : *redis_limits_2_6


# Append mode only
redis_amo_appendonly: "no"
redis_amo_appendfilename: appendonly.aof
redis_amo_appendfsync: everysec
redis_amo_no_appendfsync_on_rewrite: "no"
redis_amo_auto_aof_rewrite_percentage: 100
redis_amo_auto_aof_rewrite_min_size: 64mb
redis_amo_aof_load_truncated: "yes"

redis_amo_2_4: &redis_amo_2_4
  appendonly: "{{ redis_amo_appendonly }}"
  appendfsync: "{{ redis_amo_appendfsync }}"
  no-appendfsync-on-rewrite: "{{ redis_amo_no_appendfsync_on_rewrite }}"
  auto-aof-rewrite-percentage: "{{ redis_amo_auto_aof_rewrite_percentage }}"
  auto-aof-rewrite-min-size: "{{ redis_amo_auto_aof_rewrite_min_size }}"

redis_amo_2_6: &redis_amo_2_6
  << : *redis_amo_2_4

redis_amo_2_8: &redis_amo_2_8
  << : *redis_amo_2_6
  appendfilename: "{{ redis_amo_appendfilename }}"
  aof-load-truncated: "{{ redis_amo_aof_load_truncated }}"


# Lua scripting
redis_lua_lua_time_limit: 5000

redis_lua_2_6: &redis_lua_2_6
  lua-time-limit: "{{ redis_lua_lua_time_limit }}"

redis_lua_2_8: &redis_lua_2_8
  << : *redis_lua_2_6


# Slow log
redis_slowlog_slowlog_log_slower_than: 10000
redis_slowlog_slowlog_max_len: 128

redis_slowlog_2_4: &redis_slowlog_2_4
  slowlog-log-slower-than: "{{ redis_slowlog_slowlog_log_slower_than }}"
  slowlog-max-len: "{{ redis_slowlog_slowlog_max_len }}"

redis_slowlog_2_6: &redis_slowlog_2_6
  << : *redis_slowlog_2_4

redis_slowlog_2_8: &redis_slowlog_2_8
  << : *redis_slowlog_2_6


# Virtual memory (deprecated)
redis_vm_vm_enabled: "no"
redis_vm_vm_swap_file: /tmp/redis.swap
redis_vm_vm_max_memory: 0
redis_vm_vm_page_size: 32
redis_vm_vm_pages: 134217728
redis_vm_vm_max_threads: 4

redis_vm_2_4:
  vm-enabled: "{{ redis_vm_vm_enabled }}"
  vm-swap-file: "{{ redis_vm_vm_swap_file }}"
  vm-max-memory: "{{ redis_vm_vm_max_memory }}"
  vm-page-size: "{{ redis_vm_vm_page_size }}"
  vm-pages: "{{ redis_vm_vm_pages }}"
  vm-max-threads: "{{ redis_vm_vm_max_threads }}"


# Latency monitor
redis_latencymon_latency_monitor_threshold: 0

redis_latencymon_2_8: &redis_latencymon_2_8
  latency-monitor-threshold: "{{ redis_latencymon_latency_monitor_threshold }}"


# Event notification
redis_eventnotif_notify_keyspace_events: ""

redis_eventnotif_2_8: &redis_eventnotif_2_8
  notify-keyspace-events: "{{ redis_eventnotif_notify_keyspace_events }}"


# Advanced config
redis_advanced_hash_max_zipmap_entries: 512
redis_advanced_hash_max_zipmap_value: 64
redis_advanced_hash_max_ziplist_entries: 512
redis_advanced_hash_max_ziplist_value: 64
redis_advanced_list_max_ziplist_entries: 512
redis_advanced_list_max_ziplist_value: 64
redis_advanced_set_max_intset_entries: 512
redis_advanced_zset_max_ziplist_entries: 128
redis_advanced_zset_max_ziplist_value: 64
redis_advanced_hll_sparse_max_bytes: 3000
redis_advanced_activerehashing: "yes"
redis_advanced_client_output_buffer_limit:
  - normal 0 0 0
  - slave 256mb 64mb 60
  - pubsub 32mb 8mb 60
redis_advanced_hz: 10
redis_advanced_aof_rewrite_incremental_fsync: "yes"

redis_advanced_2_4: &redis_advanced_2_4
  hash-max-zipmap-entries: "{{ redis_advanced_hash_max_zipmap_entries }}"
  hash-max-zipmap-value: "{{ redis_advanced_hash_max_zipmap_value }}"
  list-max-ziplist-entries: "{{ redis_advanced_list_max_ziplist_entries }}"
  list-max-ziplist-value: "{{ redis_advanced_list_max_ziplist_value }}"
  set-max-intset-entries: "{{ redis_advanced_set_max_intset_entries }}"
  zset-max-ziplist-entries: "{{ redis_advanced_zset_max_ziplist_entries }}"
  zset-max-ziplist-value: "{{ redis_advanced_zset_max_ziplist_value }}"
  activerehashing: "{{ redis_advanced_activerehashing }}"

redis_advanced_2_6: &redis_advanced_2_6
  << : *redis_advanced_2_4
  hash-max-ziplist-entries: "{{ redis_advanced_hash_max_ziplist_entries }}"
  hash-max-ziplist-value: "{{ redis_advanced_hash_max_ziplist_value }}"
  client-output-buffer-limit: "{{ redis_advanced_client_output_buffer_limit }}"
  hz: "{{ redis_advanced_hz }}"
  aof-rewrite-incremental-fsync: "{{ redis_advanced_aof_rewrite_incremental_fsync }}"

redis_advanced_2_8: &redis_advanced_2_8
  << : *redis_advanced_2_6
  hll-sparse-max-bytes: "{{ redis_advanced_hll_sparse_max_bytes }}"


# Includes
redis_includes_include: []

redis_includes:
  include: "{{ redis_includes_include }}"


# Final configs per version
redis_config_2_4: &redis_config_2_4
  general: "{{ redis_general_2_4 }}"
  snapshotting: "{{ redis_snapshotting_2_4 }}"
  replication: "{{ redis_replication_2_4 }}"
  security: "{{ redis_security_2_4 }}"
  limits: "{{ redis_limits_2_4 }}"
  "append mode only": "{{ redis_amo_2_4 }}"
  "slow log": "{{ redis_slowlog_2_4 }}"
  advanced: "{{ redis_advanced_2_4 }}"

redis_config_2_6: &redis_config_2_6
  general: "{{ redis_general_2_6 }}"
  snapshotting: "{{ redis_snapshotting_2_6 }}"
  replication: "{{ redis_replication_2_6 }}"
  security: "{{ redis_security_2_6 }}"
  limits: "{{ redis_limits_2_6 }}"
  "append mode only": "{{ redis_amo_2_6 }}"
  "lua scripting": "{{ redis_lua_2_6 }}"
  "slow log": "{{ redis_slowlog_2_6 }}"
  advanced: "{{ redis_advanced_2_6 }}"

redis_config_2_8: &redis_config_2_8
  general: "{{ redis_general_2_8 }}"
  snapshotting: "{{ redis_snapshotting_2_8 }}"
  replication: "{{ redis_replication_2_8 }}"
  security: "{{ redis_security_2_8 }}"
  limits: "{{ redis_limits_2_8 }}"
  "append mode only": "{{ redis_amo_2_8 }}"
  "lua scripting": "{{ redis_lua_2_8 }}"
  "slow log": "{{ redis_slowlog_2_8 }}"
  "latency monitor": "{{ redis_latencymon_2_8 }}"
  "event notification": "{{ redis_eventnotif_2_8 }}"
  advanced: "{{ redis_advanced_2_8 }}"

redis_config:
  << : *redis_config_2_4
```


License
-------

MIT


Author
------

Jiri Tyr
