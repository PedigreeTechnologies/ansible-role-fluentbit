Ansible role Fluentbit
=========

This role installs and configures Fluentbit.

Requirements
------------

* This role is only tested with Ansible >= 2.9.

Role Variables
--------------

This role tries to keep the same default configuration as if you manually install
Fluentbit.
You can find more information about each option in the [Fluentbit documentation](
https://docs.fluentbit.io/manual/v/1.6/administration/configuring-fluent-bit/configuration-file).

Variables used for the installation:

| Variables                       | Default value                               | Description                                                 |
|---------------------------------|---------------------------------------------|-------------------------------------------------------------|
| fluentbit_pkg_name              | fluent-bit                                  | The Fluentbit APT package name.                             |
| fluentbit_pkg_version           | 2.1.9                                          | Install a specific version of the package.                  |
| fluentbit_pkg_version_hold      | "{{ fluentbit_pkg_version \| default(False) \| ternary(True, False) }}" | Lock package version to prevent accidental updates. By default, `True` if `fluentbit_pkg_version` is defined, `False` otherwise. |
| fluentbit_svc_name              | fluent-bit                                  | The Fluentbit service name to start/stop the daemon.        |


Variables used for the general configuration:

| Variables                        | Default value     | Description                                               |
|----------------------------------|-------------------|-----------------------------------------------------------|
| fluentbit_svc_flush              | 5                 | Flush time in *seconds.nanoseconds* format.               |
| fluentbit_svc_grace              | 5                 | Set the grace time is *seconds*.                          |
| fluentbit_svc_daemon             | "off"             | On/Off value to specify if Fluentbit runs as a Deamon. Should be Off when using the provided Systemd unit. |
| fluentbit_svc_logfile            | ""                | Absolute path for an optional log file. Log to stdout if not specified. |
| fluentbit_svc_loglevel           | error             | Set the logging verbosity level.                          |
| fluentbit_svc_parsers_file       | ["parsers.conf"]  | List of paths for *parsers* configuration files.          |
| fluentbit_svc_plugins_file       | ["plugins.conf"]  | List of paths for *plugins* configuration files.          |
| fluentbit_svc_streams_file       | []                | List of paths for *stream processors* configuration files.|
| fluentbit_managed_parsers_enable | "{{ ((_fluentbit_parsers \| length) or (_fluentbit_mlparsers \| length)) \| bool }}" | Define if Ansible should create/update a custom file for user defined parser. |
| fluentbit_managed_parsers_file   | "{{ fluentbit_conf_directory }}/managed-parsers.conf"                                | File where the custom parsers are defined if needed. |
| fluentbit_svc_http               | {}                | Dictionary for HTTP built-in server configuration.        |
| fluentbit_svc_storage            | {}                | Dictionary for storage/buffer configuration.              |
| fluentbit_svc_limit_open_files   | Undefined         | Configure LimitNOFILE for systemd service if defined      |

Other variables:

| Variables               | Default value | Description                                               |
|-------------------------|---------------|-----------------------------------------------------------|
| fluentbit_dbs_path      | ""            | A path for a directory that will be created by the role. For example to store your input DB files. |

Variables for log processing:

| Variables               | Default value                                                                      | Description                                    |
|-------------------------|------------------------------------------------------------------------------------|------------------------------------------------|
| fluentbit_inputs       | n/a    | List of dictionaries defining all log inputs.  |
| fluentbit_filters      | n/a   | List of dictionaries defining all log filters. |
| fluentbit_outputs      | n/a   | List of dictionaries defining all log outputs. |
| fluentbit_parsers      | n/a   | List of dictionaries defining all log managed parser. |
| fluentbit_multi_parsers    | n/a | List of dictionaries defining all log managed multiline parser. |



Each dictionary is used to define one `[INPUT]`, `[FILTER]`, `[OUTPUT]`, `[PARSER]` or `[MULTILINE_PARSER]` section
in the Fluentbit configuration file or in managed parser file. Each configuration section
is configured with key/value couples, so the dictionary's keys are used as
configuration keys and values as values.

For example:
```
fluentbit_nginx_input:
  - name: tail
    path: /var/log/nginx/access.log

fluentbit_kernel_input:
  name: tail
  path: /var/log/kern.log
```

will create:
```
[INPUT]
  name tail
  path /var/log/nginx/access.log
[INPUT]
  name tail
  path /var/log/kern.log
```

It allows you to define variables in multiple group_vars and cumulate them for
hosts in multiples groups without the need to rewrite the complete list.

Ansible dictionaries can't have the same key multiple times. This can be a problem
to use stuff like `record`.
To overcome this issue, keys with multiple values are set as lists, and the jinja
configuration handles these keys differently. The keys that are handled as lists are
Add, Record, and Rename

Fir example:
```
my_test:
    Name: modify
    Match: general.app
    Add: ['app lineage', 'type my_app', 'path /home/ec2-user/my_app/app.log']
```

will create:
```
[FILTER]
  name modify
  match general.app
  Add app lineage
  Add type my_app
  Add path /home/ec2-user/my_app/app.log
```

Dependencies
------------

None

License
-------

BSD

Author Information
------------------

[BIMData.io](https://bimdata.io/)
