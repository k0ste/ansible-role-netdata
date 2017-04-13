ansible-netdata
=================

Role for deploy netdata configuration.

Ansible versions
-----------------

Ansible 2.2 is required.

Extra
------

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
netdata_settings:
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
