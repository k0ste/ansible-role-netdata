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
# Install/upgrade netdata package or not.
netdata_install_packages: 'true'
# Enable netdata after deploy or not.
netdata_enable: 'true'
# Restart netdata after deploy or not.
netdata_restart: 'true'
# 'present' (do nothing if package is already installed) or 'latest' (always
# upgrade to last version).
netdata_package_state: 'latest'

netdata:
- global:
  - memory_mode: 'save'
    web_files_owner: 'root'
    web_files_group: 'netdata'
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
```
