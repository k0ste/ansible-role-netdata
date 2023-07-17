# netdata

Role for deploy `netdata` packages and `netdata` configuration

## Requirements

* Ansible 3.0.0+;

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
# Enable netdata after deploy or not
- enable: 'true'
# Restart netdata after deploy or not
  restart: 'true'
# Install/upgrade netdata package or not
  install_package: 'true'
# 'present' (do nothing if package is already installed) or 'latest' (always
# upgrade to last version)
  package_state: 'latest'
# Allow use sudo for netdata service or not, see
# https://www.freedesktop.org/software/systemd/man/systemd.exec.html#Capabilities
  allow_sudo: 'true'
# netdata.conf settings.
  settings:
    - global:
        - memory_mode: 'save'
          stock_config_directory:
            "{{ netdata_stock_config_directory | default(omit) }}"
          web_files_owner:
            "{{ netdata_settings_web_files_owner | default('netdata') }}"
          web_files_group:
            "{{ netdata_settings_web_files_group | default('netdata') }}"
          memory_deduplication_(ksm): 'no'
          error_log: 'stderr'
          access_log: 'none'
    - plugins:
        - checks: 'no'
          node.d: 'no'
    - plugin:
        - path: 'proc:/proc/net/dev'
          bandwidth_for_all_interfaces: 'yes'
          packets_for_all_interfaces: 'yes'
          errors_for_all_interfaces: 'yes'
          drops_for_all_interfaces: 'yes'
          fifo_for_all_interfaces: 'yes'
        - path: 'proc:/proc/diskstats'
          path_to_device_mapper: '/dev/null'
          performance_metrics_for_disks_with_major_253: 'yes'
    - health:
        - enabled: 'no'
    - backend:
        - enabled: 'no'
    - registry:
        - enabled: 'yes'
    - statsd:
        - enabled: 'no'
# exporting.conf settings
  - exporting_settings:
# prometheus exporter settings
      - prometheus:
# enable netdata meta labels
          - send_automatic_labels: 'yes'
# conf.d settings
  - plugins_settings:
# pyhond.d plugins settings
      - python:
# The python.d.conf 'local' settings. Settings will be merged with distro
# python.d.conf
        - local:
# Enable / disable the whole python.d.plugin (all its modules)
          enabled: 'yes'
# Enable / Disable explicit garbage collection (full collection run). Default is
# enabled
          gc_run: 'yes'
# Garbage collection interval in seconds. Default is 300
          gc_interval: '300'
# This python.d modules will be explicity enabled or disabled
          modules:
            - ceph: 'no'
              samba: 'yes'
# smartd_log plugin settings
          smartd_log:
          - log_path: '/var/log/smartd'
# Age should match with smartd interval (default is 30 min)
            age: '60'
# nginx plugin settings
          nginx:
            - url: 'http://127.0.0.1/nginx_status'
```
