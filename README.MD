# netdata

Role for deploy `netdata` packages and `netdata` configuration

## Requirements

* Ansible 9.0.0+;

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
      db:
        - memory_deduplication_(ksm): 'no'
      plugin:
        - path: 'proc'
          /sys/class/infiniband: 'no'
        - path: 'proc:/proc/stat'
          per_cpu_core_utilization: 'yes'
        - path: 'proc:/proc/net/dev'
          bandwidth_for_all_interfaces: 'yes'
          packets_for_all_interfaces: 'yes'
          errors_for_all_interfaces: 'yes'
          drops_for_all_interfaces: 'yes'
          fifo_for_all_interfaces: 'yes'
        - path: 'proc:/proc/diskstats'
          path_to_device_mapper: '/dev/null'
          performance_metrics_for_disks_with_major_253: 'yes'
        - path: 'perf'
          command_options: 'cycles instructions'
      logs:
        - daemon: 'journal'
          debug: 'journal'
          collector: 'journal'
          access: 'journal'
          health: 'journal'
      cloud:
        - enabled: 'no'
      health:
        - enabled: 'no'
      statsd:
        - enabled: 'no'
      ml:
        - enabled: 'no'
      web:
        - bind_to: "{{ ansible_host }}"
      host_labels: "{{ lookup('ansible.builtin.vars',
        'netdata_settings_host_labels', default='') }}"
# exporting.conf settings
      exporting_settings:
# prometheus exporter settings
        - prometheus:
# enable netdata meta labels
            - send_automatic_labels: 'yes'
      plugins_settings:
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
# The go.d/x509check module (see documentation for supported options)
              x509check:
                - name: 'redhat.com'
                  source: 'file:///var/lib/lego/certificates/redhat.com.crt'
                  update_every: 600
                  autodetection_retry: 60
                  check_full_chain: yes
                - name: 'ansible.com'
                  source: 'https://ansible.com:443'
# The go.d/prometheus module (see documentation for supported options)
              prometheus:
# Enable frr_exporter metrics (with our name)
                - name: 'frr'
                  url: 'http://127.0.0.1:9342/metrics'
                  update_every: 10
                  autodetection_retry: 5
# Disable default auto-detection job (disable default -local name)
                - name: 'frr_exporter_local'
                  url: 'http://127.0.0.1:999/metrics'
                  update_every: 1
                  autodetection_retry: 0
# Disable default auto-detection job (just disable this exporter)
                - name: 'smartctl_exporter_local'
                  url: 'http://127.0.0.1:999/metrics'
                  update_every: 1
                  autodetection_retry: 0
# The go.d/storcli module (see documentation for supported options)
              storcli:
                - name: 'storcli'
                  update_every: 600
# The go.d/adaptec_raid module (see documentation for supported options)
              adaptec_raid:
                - name: 'adaptec_raid'
                  update_every: 600
# The go.d/apcupsd module (see documentation for supported options)
              apcupsd:
                - name: 'local'
                  address: '127.0.0.1:3551'
                - name: 'remote'
                  address: '100.100.100.100:3551'
                  timeout: '3'
                  update_every: '1'
                  autodetection_retry: '0'
                  timeout: '2'
```
