## Ansible Role: systemd

This role provides a simple yet flexible way to manage all systemd unit files.

## Features

- Create all systemd [unit files](https://www.freedesktop.org/software/systemd/man/systemd.unit.html).
- Create template units.
- Set all unit directives.
- Enable/Disable each unit.
- Restart units on change.

## Requirements

- Ansible 2.8 or higher.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
systemd_units: []
```
A list of dictionaries. For each unit, you must provide a `name`, all other keys are listed in the [syntax heading bellow](#Syntax).

```yaml
systemd_units_restart_changed: yes
```
Whether to restart the unit on file change, you can also specify the parameter `restart_changed` for each unit. Template units will always be skipped.

```yaml
systemd_units_enabled: yes
```
Whether to enable units on boot, you can also specify the parameter `enabled` for each unit. Template units will always be skipped.

### Syntax

A systemd_unit has all the parameters of the [systemd module](https://docs.ansible.com/ansible/latest/modules/systemd_module.html) plus multiple keys ending with `_section`, you can specify as many sections as you want. Each section is a dictionary of [configuration directives](https://www.freedesktop.org/software/systemd/man/systemd.directives.html). If no section is provided, an empty file will be created, use this as an effective way to fully disable a unit.

```yaml
- name: foo.service
  scope: user
  # other systemd module parameters...
  Unit_section:
    Description: Foo
  Service_section:
    ExecStart: /usr/sbin/foo-daemon
  Install_section:
    WantedBy: multi-user.target
```

The unit above will create a `foo.service` file with following content:

```ini
[Unit]
Description=Foo

[Service]
ExecStart=/usr/sbin/foo-daemon

[Install]
WantedBy=multi-user.target
```

A directive key can be a `string`, a `list` or a `dictionary`, each key type will be converted as follows:

```yaml
ExecStart: "/usr/bin/foo-daemon"
Wants:
  - foo@1.service
  - foo@2.service
Environment:
  VAR1: word1 word2
  VAR2: word3
  VAR3: $word 5 6
```

```ini
ExecStart=/usr/bin/foo-daemon
Wants=foo@1.service
Wants=foo@2.service
Environment="VAR1=word1 word2"
Environment="VAR2=word3"
Environment="VAR3=$word 5 6"
```

**Convert camel_case keys to PascalCase**

```yaml
systemd_convert_snake_case_keys: yes
```
This variable allows you to write all directives in `snake_case`. When enabled, this role will convert all `snake_case` directives to `PascalCase`, e.g. `exec_start: "/usr/bin/foo-daemon"` will be converted to `ExecStart=/usr/bin/foo-daemon`. If you don't like this feature you can disable it.

```yaml
systemd_readable_snake_case_keys: # see defaults/main.yml
```
This role maintains a dictionary of readable `snake_case` directives. In fact, some directives are abbreviated, to write `CPUShares` in snake case, you have to write `c_p_u_shares`. Thanks to this variable, you can to write `cpu_shares`.

## Dependencies

None.

## Example Playbook

This is a sample playbook file for creating and starting a service unit `sleep.service`.

```yaml
- hosts: all
  become: true
  vars:
    systemd_units:
      - name: sleep.service
        Service_section:
          ExecStart: /bin/sleep 300
        Install_section:
          WantedBy: multi-user.target
  roles:
    - { role: anasbouzid.systemd, tags: [systemd] }
```

This is a sample `systemd_units` variable for creating and starting a target unit `sleep.target` that instantiates 3 service templates.

```yaml
systemd_units:
  - name: sleep@.service
    Unit_section:
      Description: Sleep for %i seconds
      PartOf: sleep.target
    Service_section:
      Restart: always
      ExecStart: /bin/sleep %i
    Install_section:
      WantedBy: multi-user.target
  - name: sleep.target
    Unit_section:
      Wants:
        - sleep@10.service
        - sleep@20.service
        - sleep@30.service
    Install_section:
      WantedBy: multi-user.target
```

The same variable can be written in `snake_case`.

```yaml
systemd_units:
  - name: sleep@.service
    unit_section:
      description: Sleep for %i seconds
      part_of: sleep.target
    service_section:
      restart: always
      exec_start: /bin/sleep %i
    install_section:
      wanted_by: multi-user.target
  - name: sleep.target
    unit_section:
      wants:
        - sleep@10.service
        - sleep@20.service
        - sleep@30.service
    install_section:
      wanted_by: multi-user.target
```

To organize your code, your may split units by category.

```yaml
systemd_services: []
systemd_targets: []
systemd_sockets: []
systemd_units: "{{ systemd_services + systemd_targets + systemd_sockets }}"
```

Or you may group them by functionality.

```yaml
systemd_nginx_units: []
systemd_queue_units: []
systemd_php_units: []
systemd_units: "{{ systemd_nginx_units + systemd_queue_units + systemd_php_units }}"
```

## License

MIT
