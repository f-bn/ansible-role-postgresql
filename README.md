## Ansible role - Postgresql

### General informations

Ansible role to install and configure standalone PostgreSQL server instance.

**Table of Contents**

- [General informations](#general-informations)
- [Role variables](#role-variables)
- [Examples](#examples)
- [Install and use this role](#install-and-use-this-role)

**Supported Platforms**

This role only supports Ubuntu Server LTS platforms:

  - Ubuntu 22.04 *Jammy Jellyfish*

**Supported PostgreSQL releases**

  - PostgreSQL 16.x - **default**
  - PostgreSQL 15.x
  - PostgreSQL 14.x

**Requirements**

  - None

**References**

  - PostgreSQL : https://www.postgresql.org/

### Role variables

#### General

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_version`              | `16.0`                       | Defines the package version of PostgreSQL to install             |
| `postgresql_role`                 | `primary`                    | Defines the role of the instance (`primary` or `standby`). This adjust few settings to prepare instance for replication depending on the given role. Set this to `primary` for standalone instance(s) |
| `postgresql_cluster_name`         | `null`                       | Sets a name that identifies this database cluster (instance) for various purposes. The cluster name appears in the process title for all server processes in this cluster. Moreover, it is the default application name for a standby connection |
| `postgresql_root_dir`             | `/var/lib/postgresql`        | Defines the root directory where everything related to the PostgreSQL instance is stored |
| `postgresql_home_dir`             | `{{ postgresql_root_dir }}`  | Defines the home directory for the `postgres` user               |
| `postgresql_data_dir`             | `{{ postgresql_root_dir }}/{{ postgresql_release }}/data` | Defines the directory where PostgreSQL will store database data |
| `postgresql_backups_dir`          | `{{ postgresql_root_dir }}/{{ postgresql_release }}/backups` | Defines the directory where PostgreSQL backups are stored |
| `postgresql_config_dir`           | `/etc/postgresql`            | Defines the directory where to store the different PostgreSQL configuration files |
| `postgresql_wal_dir`              | `""`                         | If the variable is not empty, defines the directory where PostgreSQL WALs will be stored (outside of the data directory) |
| `postgresql_repo_url`             | see [defaults](../defaults/main.yml) | Defines the URL to the PostgreSQL APT repository         |
| `postgresql_repo_gpgkey_url`      | see [defaults](../defaults/main.yml) | Defines the URL to the GPG key for the PostgreSQL APT repository |
| `postgresql_packages`             | see [defaults](../defaults/main.yml) | Defines the list of server packages to install for PostgreSQL (can be overrided to add extensions packages i.e) |
| `postgresql_shutdown_mode`        | `fast`                       | Defines the shutdown mode to use when stopping instance service. Valid values are `smart` and `fast` |
| `postgresql_restart_on_changes`   | `false`                      | If set to `true`, restart the instance service when server configuration changes |

#### Instance initialization

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_initdb_checksum_enabled`| `false`                      | If set to `true`, enable checksuming on data pages to help detect corruption by the I/O system that would otherwise be silent. Enabling checksums may incur a noticeable performance penalty. |
| `postgresql_initdb_extra_opts`    | `""`                         | Extra args to pass to the `initdb` command                       |

#### Connections and authentications

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_listen_addresses`     | `localhost`                  | Specifies the TCP/IP address(es) on which the server is to listen for connections from client applications. The value takes the form of a comma-separated list of host names and/or numeric IP addresses. |
| `postgresql_port`                 | `5432`                       | The TCP port the server listens on                               |
| `postgresql_max_connections`      | `100`                        | Determines the maximum number of concurrent connections to the database server |
| `postgresql_superuser_reserved_connections` | `3`                | Determines the number of connection “slots” that are reserved for connections by PostgreSQL superusers |
| `postgresql_authentication_timeout` | `1min`                     | Maximum amount of time allowed to complete client authentication |
| `postgresql_password_encryption`  | `scram-sha-256`              | When a password is specified in `CREATE ROLE` or `ALTER ROLE`, this parameter determines the algorithm to use to encrypt the password |
| `postgresql_unix_socket_directories`| `/run/postgresql`          | Specifies the directory of the Unix-domain socket(s) on which the server is to listen for connections from client applications. Multiple sockets can be created by listing multiple directories separated by commas |
| `postgresql_unix_socket_group`    | `-`                          | Sets the owning group of the Unix-domain socket(s). (The owning user of the sockets is always the user that starts the server.) In combination with the parameter `unix_socket_permissions` this can be used as an additional access control mechanism for Unix-domain connections |
| `postgresql_unix_socket_permissions`| `0777`                     | Sets the access permissions of the Unix-domain socket(s). Unix-domain sockets use the usual Unix file system permission set |

#### SSL/TLS

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_ssl_enabled`          | `false`                      | If set to `true`, enable SSL connections to the database         |
| `postgresql_ssl_ca_file`          | `''`                         | Specifies the name of the file containing the SSL server certificate authority (CA) |
| `postgresql_ssl_cert_file`        | `server.crt`                 | Specifies the name of the file containing the SSL server certificate |
| `postgresql_ssl_crl_file`         | `''`                         | Specifies the name of the file containing the SSL client certificate revocation list (CRL) |
| `postgresql_ssl_key_file`         | `server.key`                 | Specifies the name of the file containing the SSL server private key |
| `postgresql_ssl_min_protocol_version`| `TLSv1.2`                 | Specifies the minimum SSL/TLS protocol version to use (`TLSv1`, `TLSv1.1`, `TLSv1.2` or `TLSv1.3`) |

#### Resource usage (except WAL)

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_shared_buffers`       | `128MB`                      | Sets the amount of memory the database server uses for shared memory buffers |
| `postgresql_temp_buffers`         | `8MB`                        | Sets the maximum amount of memory used for temporary buffers within each database session |
| `postgresql_work_mem`             | `4MB`                        | Sets the base maximum amount of memory to be used by a query operation (such as a sort or hash table) before writing to temporary disk files |
| `postgresql_hash_mem_multiplier`  | `1.0`                        | Used to compute the maximum amount of memory that hash-based operations can use |
| `postgresql_maintenance_work_mem` | `64MB`                       | Specifies the maximum amount of memory to be used by maintenance operations, such as `VACUUM`, `CREATE INDEX` and `ALTER TABLE ADD FOREIGN KEY` |
| `postgresql_max_prepared_transactions` | `0`                     | Sets the maximum number of transactions that can be in the `prepared` state simultaneously. Setting this parameter to zero (which is the default) disables the prepared-transaction feature. This parameter can only be set at server start. If you are not planning to use prepared transactions, this parameter should be set to zero to prevent accidental creation of prepared transactions. |
| `postgresql_effective_io_concurrency`| `1`                       | Sets the number of concurrent disk I/O operations that PostgreSQL expects can be executed simultaneously. Raising this value will increase the number of I/O operations that any individual PostgreSQL session attempts to initiate in parallel. The allowed range is `1` to `1000`, or zero to disable issuance of asynchronous I/O requests |
| `postgresql_maintenance_io_concurrency`| `10`                    | Similar to `postgresql_effective_io_concurrency`, but used for maintenance work that is done on behalf of many client sessions |
| `postgresql_max_worker_processes` | `8`                          | Sets the maximum number of background processes that the system can support |
| `postgresql_max_parallel_workers_per_gather`| `2`                | Sets the maximum number of workers that can be started by a single `Gather` or `Gather Merge` node. |
| `postgresql_max_parallel_maintenance_workers`| `2`               | Sets the maximum number of parallel workers that can be started by a single utility command. Currently, the parallel utility commands that support the use of parallel workers are `CREATE INDEX` only when building a B-tree index, and `VACUUM` without `FULL` option |
| `postgresql_max_parallel_workers` | `8`                          | Sets the maximum number of workers that the system can support for parallel operations |

#### Write-Ahead Log

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_wal_level`            | `replica`                    | Determines how much information is written to the WAL            |
| `postgresql_wal_sync_method`      | `fdatasync`                  | Method used for forcing WAL updates out to disk (see documentation for possible values) |
| `postgresql_wal_compression`      | `off`                        | Defines the compression method used for WAL compression if enabled. Valid values are: `on` or `off` for **PostgreSQL version =< 14.x**. For **PostgreSQL 15.x and later**, new options are available : `pglz`, `lz4` (if PostgreSQL was compiled with `--with-lz4`) or `zstd` (if PostgreSQL was compiled with `--with-zstd`) |
| `postgresql_wal_buffers`          | `-1`                         | The amount of shared memory used for WAL data that has not yet been written to disk. The default setting of `-1` selects a size equal to 1/32nd (about 3%) of `shared_buffers`, but not less than `64kB` nor more than the size of one WAL segment, typically `16MB` |
| `postgresql_synchronous_commit`   | `on`                         | Specifies how much WAL processing must complete before the database server returns a "success" indication to the client. Valid values are `remote_apply`, `on` (the default), `remote_write`, `local`, and `off`.|
| `postgresql_checkpoint_timeout`   | `5min`                       | Maximum time between automatic WAL checkpoints                   |
| `postgresql_recovery_prefetch`    | `try`                        | Whether to try to prefetch blocks that are referenced in the WAL that are not yet in the buffer pool, during recovery. Valid values are `off`, on and `try` (the default). **PostgreSQL 15.x and later required** |
| `postgresql_wal_decode_buffer_size`| `512kB`                     | Defines a limit on how far ahead the server can look in the WAL, to find blocks to prefetch. **PostgreSQL 15.x and later required** |
| `postgresql_archive_mode_enabled` | `false`                      | When `archive_mode` is enabled, completed WAL segments are sent to archive storage by setting `archive_command` |
| `postgresql_archive_library`      | `null`                       | Defines the library to use for archiving completed WAL file segments. If set to an empty string (the default), archiving via shell is enabled, and `archive_command` is used. Otherwise, the specified shared library is used for archiving. **PostgreSQL 15.x and later required** |
| `postgresql_archive_command`      | `null`                       | The local shell command to execute to archive a completed WAL file segment |
| `postgresql_restore_command`      | `null`                       | The local shell command to execute to retrieve an archived segment of the WAL file series. This parameter is required for archive recovery, but optional for streaming replication |

#### Replication

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_max_wal_senders`      | `10`                         | Specifies the maximum number of concurrent connections from standby servers or streaming base backup clients (`primary` node only) |
| `postgresql_max_replication_slots`| `10`                         | Specifies the maximum number of replication slots that the server can support (`primary` node only) |
| `postgresql_synchronous_standby_names`| `""`                     | Specifies a list of standby servers that can support synchronous replication |
| `postgresql_hot_standby`          | `true`                       | Specifies whether or not you can connect and run queries during recovery (`standby` node(s) only ) |

#### Query tuning

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_effective_cache_size` | `4GB`                        | Specifies the planner's assumption about the effective size of the disk cache that is available to a single query |

#### Reporting and logging

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_log_destination`      | `stderr`                     | PostgreSQL logging method (`stderr`, `csvlog` or `syslog`)       |
| `postgresql_logging_collector_enabled` | `false`                 | This parameter enables the logging collector, which is a background process that captures log messages sent to stderr and redirects them into log files |
| `postgresql_log_directory`        | `log`                        | When `logging_collector` is enabled, this parameter determines the directory in which log files will be created |
| `postgresql_log_filename`         | `postgresql-%Y-%m-%d_%H%M%S.log` | When `logging_collector` is enabled, this parameter sets the file names of the created log files |
| `postgresql_log_rotation_age`     | `1d`                         | When `logging_collector` is enabled, this parameter determines the maximum amount of time to use an individual log file, after which a new log file will be created |
| `postgresql_log_min_messages`     | `warning`                    | Controls which message levels are written to the server log (`DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `LOG`, `FATAL` and `PANIC`) |
| `postgresql_log_min_error_statement` | `error`                   | Controls which SQL statements that cause an error condition are recorded in the server log (`DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `INFO`, `NOTICE`, `WARNING`, `ERROR`, `LOG`, `FATAL` and `PANIC`) |
| `postgresql_log_min_duration_statement` | `-1`                   | Causes the duration of each completed statement to be logged if the statement ran for at least the specified amount of time |
| `postgresql_log_checkpoints`      | `off`                        | Causes checkpoints and restartpoints to be logged in the server log. Some statistics are included in the log messages, including the number of buffers written and the time spent writing them |
| `postgresql_log_connections`      | `off`                        | Causes each attempted connection to the server to be logged, as well as successful completion of both client authentication (if necessary) and authorization |
| `postgresql_log_disconnections`   | `off`                        | Causes session terminations to be logged. The log output provides information similar to `postgresql_log_connections`, plus the duration of the session |
| `postgresql_log_duration`         | `off`                        | Causes the duration of every completed statement to be logged     |
| `postgresql_log_line_prefix`      | `%m [%p] `                   | This is a `printf`-style string that is output at the beginning of each log line. `%` characters begin "escape sequences" that are replaced with status information |
| `postgresql_log_lock_waits`       | `off`                        | Controls whether a log message is produced when a session waits longer than `deadlock_timeout` to acquire a lock |
| `postgresql_log_statement`        | `none`                       | Controls which SQL statements are logged. Valid values are `none` (off), `ddl`, `mod`, and `all` (all statements) |
| `postgresql_log_timezone`         | `Etc/UTC`                    | Sets the time zone used for timestamps written in the server log  |
| `postgresql_syslog_facility`      | `LOCAL0`                     | If `postgresql_log_destination` is set to `syslog`, this parameter determines the syslog facility to be used |
| `postgresql_syslog_ident`         | `postgres`                   | When logging to syslog is enabled, this parameter determines the program name used to identify PostgreSQL messages in syslog logs |
| `postgresql_syslog_sequence_numbers` | `true`                    | If set to `true`, each message will be prefixed by an increasing sequence number |
| `postgresql_syslog_split_messages` | `true`                      | If set to `true`, this parameter determines how messages are delivered to syslog. When `true`, messages are split by lines, and long lines are split so that they will fit into 1024 bytes. When `false`, PostgreSQL server log messages are delivered to the syslog service as is, and it is up to the syslog service to cope with the potentially bulky messages |

#### Statistics

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_track_activities`     | `true`                       | Enables the collection of information on the currently executing command of each session, along with its identifier and the time when that command began execution |
| `postgresql_track_activity_query_size`| `1024`                   | Specifies the amount of memory reserved to store the text of the currently executing command for each active session, for the `pg_stat_activity.query` field |
| `postgresql_track_counts`         | `true`                       | Enables collection of statistics on database activity (required by autovacuum) |
| `postgresql_track_io_timing`      | `false`                      | Enables timing of database I/O calls. This parameter is off by default, as it will repeatedly query the operating system for the current time, which may cause significant overhead on some platforms |
| `postgresql_track_wal_io_timing`  | `false`                      | Enables timing of WAL I/O calls. This parameter is off by default, as it will repeatedly query the operating system for the current time, which may cause significant overhead on some platforms |
| `postgresql_track_functions`      | `none`                       | Enables tracking of function call counts and time used. Specify `pl` to track only procedural-language functions, `all` to also track SQL and C language functions. The default is `none`, which disables function statistics tracking |
| `postgresql_stats_temp_directory` | `pg_stat_tmp`                | Sets the directory to store temporary statistics data in (only available on PostgreSQL version >= 14) |
| `postgresql_compute_query_id`     | `auto`                       | Enables in-core computation of a query identifier. Valid values are `off`, `on`, `auto` and `regress`. |

#### Autovacuum

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_autovacuum_enabled`   | `true`                       | Controls whether the server should run the autovacuum launcher daemon |
| `postgresql_autovacuum_max_workers` | `3`                        | Specifies the maximum number of autovacuum processes (other than the autovacuum launcher) that may be running at any one time |
| `postgresql_autovacuum_naptime`   | `1min`                       | Specifies the minimum delay between autovacuum runs on any given database |

#### Client connection defaults

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_timezone`             | `Etc/UTC`                    | Sets the time zone for displaying and interpreting time stamps   |
| `postgresql_client_encoding`      | `UTF8`                       | Sets the client-side encoding (character set)                    |
| `postgresql_locale`               | `C.UTF-8`                    | Sets the locale                                                  |
| `postgresql_shared_preload_libraries` | `[]`                     | Specifies one or more shared libraries to be preloaded at server start |

#### Lock management

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_deadlock_timeout`     | `1s`                         | This is the amount of time to wait on a lock before checking to see if there is a deadlock condition |
 
#### Error handling

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_restart_after_crash`  | `true`                       | When set to on, which is the default, PostgreSQL will automatically reinitialize after a backend crash |

#### pg_hba

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_hba_auth_local`       | `peer`                       | Specifies the authentication method for local users via Unix-domain socket connections used in `pg_hba.conf` (`local` lines) |
| `postgresql_hba_auth_host`        | `scram-sha-256`              | Specifies the authentication method for local users via TCP/IP connections used in `pg_hba.conf` (`host` lines) |
| `postgresql_hba_entries`          | `[]`                         | Entries to set inside the `pg_hba.conf` file to manage Host-Based Access |

#### pg_ident

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_ident_entries`        | `[]`                         | Entries to set inside the `pg_ident.conf` file to manage User Name maps |

#### Miscellaneous

| Name                              | Default                      | Description                                                      |
| :-------------------------------- | :--------------------------- | :--------------------------------------------------------------- |
| `postgresql_extra_configurations` | `[]`                         | Defines a list of extra configuration directives to append in the server configuration (i.e extensions settings) |

### Examples

Examples
--------

#### Manage PostgreSQL host-based access (pg_hba)

```YAML
# Local authentication
postgresql_hba_auth_local: peer         # for local connections via Unix-domain socket
postgresql_hba_auth_host: scram-sha-256 # for local connections via TCP/IP

# Custom entries
postgresql_hba_entries:
- { type: host,  db: 'mydb',        user: 'myuser',      address: '0.0.0.0/0',         method: 'scram-sha-256' }
- { type: host,  db: 'mydb1,mydb2', user: 'mike',        address: '192.168.122.0/24',  method: 'md5' }
- { type: local, db: 'telegraf',    user: 'telegraf',    method: 'peer' }
- { type: host,  db: 'replication', user: 'replication', address: '192.168.122.10/32', method: 'scram-sha-256' }
- { type: host,  db: 'mydb3',       user: 'myuser',      address: '192.168.122.10/32', method: 'ldap', auth_options: 'ldapserver=ldap.example.net ldapbasedn="dc=example, dc=net" ldapsearchfilter="(|(uid=$username)(mail=$username))"' }
- { type: hostssl, db: mydb,        user: 'myuser',      address: '192.168.122.0/24',  method: 'scram-sha-256' }
```

More informations in the official `pg_hba` [documentation](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html).

#### Manage PostgreSQL user name maps (pg_ident)

When using an external authentication system such as Ident or GSSAPI, the name of the operating system user that initiated the connection might not be the same as the database user (role) that is to be used. In this case, a user name map can be applied to map the operating system user name to a database user. 

```YAML
postgresql_ident_entries:
- { mapname: 'omicron', system_username: 'bryanh', pg_username: 'bryanh' }
- { mapname: 'omicron', system_username: 'bryanh', pg_username: 'guest1' }
- { mapname: 'mymap', system_username: '/^(.*)@mydomain\.com$' pg_username: '\1' }
- { mapname: 'mymap', system_username: '/^(.*)@otherdomain\.com$', pg_username: 'guest' }
```

More informations in the official User Name Maps [documentation](https://www.postgresql.org/docs/current/auth-username-maps.html)

#### Manage PostgreSQL synchronous streaming replication parameters

By default, PostgreSQL streaming is asynchronous, but you can configure it to be synchronous if needed :

```YAML
# You need to define a unique name for each replica node
postgresql_cluster_name: "server1"

# Enable synchronous replication (there are multiples options such as 'on', 'remote_write' and 'remote_apply')
postgresql_synchronous_commit: 'on'

# Define the nodes that should be replicated synchronously
postgresql_synchronous_standby_names: "server1, server2, server3"
```

You can also define different topologies for synchronous replication :

```YAML
# This will cause each commit to wait for replies from three higher-priority standbys chosen 
# from standby servers server1, server2, server3 and server4. The standbys whose names appear 
# earlier in the list are given higher priority and will be considered as synchronous.
postgresql_synchronous_standby_names: 'FIRST 3 (server1, server2, server3, server4)'

# This will cause synchronous commit to wait for reply from any 2 standby servers
postgresql_synchronous_standby_names: 'ANY 2 (*)'
```

More informations in the official replication [documentation](https://www.postgresql.org/docs/current/runtime-config-replication.html)

#### Manage SSL/TLS connections

```YAML
postgresql_ssl_enabled: true
postgresql_ssl_ca_file: '/etc/postgresql/root.crt'
postgresql_ssl_cert_file: '/etc/postgresql/server.crt'
postgresql_ssl_key_file: '/etc/postgresql/server.key'
```

Once the variable is defined and the certificates are available for the PostgreSQL server, you can connect to the instance using TLS :

```shell
$ psql -h dbhost.domain.tld -U postgres "sslmode=require sslrootcert=/path/to/root.crt"

psql (15.1 (Ubuntu 15.1-1.pgdg22.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

postgres=#
```

More informations in the official SSL/TLS [documentation](https://www.postgresql.org/docs/current/ssl-tcp.html)

#### Manage PostgreSQL extensions

You can enable/disable database extensions using this role. Firstly, to use some extensions, you need to install 3rd-party packages. You can use the `postgresql_extra_packages` variable for that :

```YAML
# Here we want to enable at least 'pg_cron' and 'pg_repack', therefore we need to install 2 additional packages
postgresql_packages:
  - "postgresql-{{ postgresql_release }}={{ postgresql_version }}-1.pgdg{{ ansible_distribution_version }}+1"
  - "postgresql-client-{{ postgresql_release }}={{ postgresql_version }}-1.pgdg{{ ansible_distribution_version }}+1"
  - python3-psycopg2
  - "postgresql-{{ postgresql_release }}-cron"
  - "postgresql-{{ postgresql_release }}-repack"
```

Then, some extensions requires some libraries to be loaded on server startup, but also some custom settings must be present in the server configuration file :

```YAML
postgresql_shared_preload_libraries:
  - pg_stat_statements
  - pg_prewarm
  - pg_cron

postgresql_extra_configurations:
  - 'pg_stat_statements.max = 10000'
  - 'pg_stat_statements.track = all'
  - 'pg_prewarm.autoprewarm = true'
  - 'pg_prewarm.autoprewarm_interval = 300s'
  - 'cron.use_background_workers = on'
```

#### Use of a dedicated directory/disk for WALs

Some use-cases may require a specific folder to store WALs (i.e on a different disk for performance reasons). You can configure this behavior by simply specifying a folder using the `postgresql_wal_dir` variable :

```YAML
# The folder will be created (with permissions adjusted) and the '--waldir' flag will be passed
# to the initdb command.
postgresql_wal_dir: "{{ postgresql_root_dir }}/{{ postgresql_release }}/wal"
```

[Return to main page](../README.md)

### Install and use this role

* Install the role using the command-line :

  ```shell
  $ ansible-galaxy role install git+https://github.com/f-bn/ansible-role-postgresql.git
  ```

* You can also install the role in your projects using a `requirements.yml` file and `ansible-galaxy` command-line :

  ```YAML
  $ cat requirements.yml
  ---
  roles:
    - name: postgresql
      src: https://github.com/f-bn/ansible-role-postgresql.git
      scm: git
      version: '1.0.0'

  $ ansible-galaxy install-f -r requirements.yml
  ```

* Once the role is installed, you can use it in your playbooks :

  ```yaml
  - name: Deploy
    hosts: <hosts>
    roles:
      - role: postgresql
  ```
