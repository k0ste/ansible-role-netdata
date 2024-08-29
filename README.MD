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
        - hostname: 'example.com'
    - db:
        - memory_deduplication_(ksm): 'no'
    - plugin:
        - path: 'proc'
          /sys/class/infiniband: 'no'
        - path: 'proc:/proc/net/dev'
          bandwidth_for_all_interfaces: 'yes'
          packets_for_all_interfaces: 'yes'
          errors_for_all_interfaces: 'yes'
          drops_for_all_interfaces: 'yes'
          fifo_for_all_interfaces: 'yes'
        - path: 'proc:/proc/diskstats'
          path_to_device_mapper: '/dev/null'
          performance_metrics_for_disks_with_major_253: 'yes'
    - cloud:
        - enabled: 'no'
    - health:
        - enabled: 'no'
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
# go.d plugin settings
    - golang:
        - local:
# This go.d modules will be explicity enabled or disabled
            - modules:
                - wireguard: 'no'
                - nvme: 'no'
                - prometheus: 'no'
# The go.d/ping module (see documentation for supported options)
          ping:
            - name: 'inet'
              privileged: true
              hosts:
                - '8.8.8.8'
                - '1.1.1.1'
            - name: 'telegram'
              privileged: true
              network: 'ipv4'
              hosts:
                - 'api.telegram.org'
# The go.d/storcli module (see documentation for supported options)
          storcli:
            - name: 'storcli'
              update_every: 600
# The go.d/adaptec_raid module (see documentation for supported options)
          adaptec_raid:
            - name: 'adaptec_raid'
              update_every: 600
# charts.d plugin settings
    - charts:
# The charts.d/apcupsd module (see documentation for supported options)
        - apcupsd:
            - sources:
                - local: '127.0.0.1:3551'
                  remote: '1.2.3.4:3551'
              timeout: '3'
              update_every: '1'
              priority: '90000'
              retries: '10'
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
```
