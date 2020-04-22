# Ansible role for Blackbox exporter

This role will setup [Blackbox exporter](https://github.com/prometheus/blackbox_exporter) on any Linux machine using systemd.

## Requirements

- systemd on the target host
- gnu-tar on Mac deployer host (`brew install gnu-tar`)

## Role Variables

- `blackbox_exporter_version`: the version that will be downloaded and installed. (`0.14.0`).

The role will download the Blackbox exporter release on the deployer and upload the binary on the target host.

If the `/usr/local/bin/blackbox_exporter` binary already exists, the role will skip the install steps. You can force them (to update, for instance), by setting `blackbox_exporter_force_install` to `true`.

- `blackbox_exporter_system_group`: system user that will run the exporter (`blackbox-exporter`)
- `blackbox_exporter_system_user`: system group that will run the exporter (`{{ blackbox_exporter_system_group }}`)
- `blackbox_exporter_listen_address`: the address the exporter will listen on (`0.0.0.0:9115`)
- `blackbox_exporter_service_flags`: extra flags passed to the binary via the systemd unit. (`{}`)
- `blackbox_exporter_configuration_modules`: you can define your modules here. See the [documentation](https://github.com/prometheus/blackbox_exporter/blob/master/CONFIGURATION.md) for more details, as well as `defaults/main.yml`, which contains some ready-to-use modules.

## Example playbook

```yaml
---

- hosts: myhost
  roles: blackbox-exporter
  vars:
    blackbox_exporter_web_listen_address: "127.0.0.1:9115"
```

**Most of the config is done on Prometheus**. See the [docs](https://github.com/prometheus/blackbox_exporter#prometheus-configuration).

If you are using my [Prometheus role](https://github.com/angristan/ansible-prometheus), here's a sample config:

```yaml
- hosts: myhost
  roles: prometheus
  vars:
  prometheus_scrape_configs:
    - job_name: 'blackbox_exporter'
      metrics_path: /probe
      params:
        module: [http_2xx]
      static_configs:
        - targets:
          - https://angristan.xyz
          - https://angristan.fr
      relabel_configs:
        - source_labels: [__address__]
          target_label: __param_target
        - source_labels: [__param_target]
          target_label: instance
        - target_label: __address__
          replacement: 127.0.0.1:9115
```

## License

MIT. See LICENSE for more details.

## Credit

This role is largely inspired by [cloudalchemy/ansible-blackbox-exporter](https://github.com/cloudalchemy/ansible-blackbox-exporter).

## Author Information

See my other Ansible roles at [angristan/ansible-roles](https://github.com/angristan/ansible-roles).
