# ansible-netdata

Role for deploy `netdata` and `netdata` configuration.

## Requirements

* Ansible 2.8+;

## Extra

netdata options is not validate and pass as-is. For all possible options
request netdata.conf from running netdata `http://localhost:19999/netdata.conf`.

Configuration is not validated after deploy! Netdata loads this dictionary and
then when the code needs a value from it, it just looks up the `name` in the
dictionary at the proper `section`. In all places, in the code, there are both
the `names` and their `default values`, so if something is not found in the
configuration file, the default is used.

Example configuration
-------------------------

```yaml
---
netdata:
# Enable netdata after deploy or not.
- enable: 'true'
# Restart netdata after deploy or not.
  restart: 'true'
# Install/upgrade netdata package or not.
  install_package: 'true'
# 'present' (do nothing if package is already installed) or 'latest' (always
# upgrade to last version).
  package_state: 'latest'
# netdata.conf settings.
  settings:
  - global:
    - memory_mode: 'save'
      stock_config_directory: "{{ netdata_stock_config_directory | default(omit) }}"
      web_files_owner: "{{ netdata_settings_web_files_owner | default('netdata') }}"
      web_files_group: "{{ netdata_settings_web_files_group | default('netdata') }}"
      memory_deduplication_(ksm): 'no'
      error_log: 'stderr'
      access_log: 'none'
  - plugins:
    - checks: 'no'
      node.d: 'no'
      fping: 'no'
  - health:
    - enabled: 'no'
  - backend:
    - enabled: 'no'
  - registry:
    - enabled: 'yes'
  - statsd:
    - enabled: 'no'
  # Special section, if lmv label assigned and another application doesn't have
  # this label, we can rename family label. Useful for work with 3rd party
  # exporters.
  - disk_family:
    - 'dm-0'
    - 'dm-1'
# conf.d settings
  - plugins_settings:
# pyhond.d plugins settings.
    - python:
# smartd_log plugin settings.
      - smartd_log:
        - log_path: '/var/log/smartd'
# Age should match with smartd interval (default is 30 min).
          age: '60'
# nginx plugin settings.
        nginx:
        - url: 'http://127.0.0.1/nginx_status'
```
