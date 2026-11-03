# Winlogbeat forwarder role

Install Winlogbeat OSS on Windows Server and forward selected Windows Event
Logs to a Graylog Beats input.

## Requirements

### Control node

- Ansible 2.15 or later.
- `ansible.windows` and `community.windows` collections.

```bash
ansible-galaxy collection install ansible.windows community.windows
ansible-doc ansible.windows.win_template >/dev/null
```

### Managed Windows host

- Windows Server 2019, 2022, or 2025.
- WinRM access.
- Network access to the Graylog Beats input, normally TCP port `5044`.

### Graylog

Create a Beats input before deployment. Configure it to avoid adding a Beats
`type` prefix to fields if you depend on the documented field names.

## Installation

```bash
ansible-galaxy role install tuxbull.winlogbeat_forwarder
```

```yaml
- name: Forward Windows Event Logs to Graylog
  hosts: windows
  gather_facts: true
  roles:
    - role: tuxbull.winlogbeat_forwarder
      vars:
        winlogbeat_graylog_host: graylog.example.com
        winlogbeat_graylog_port: 5044
```

## Variables

| Variable | Default | Description |
|---|---:|---|
| `winlogbeat_version` | `9.4.2` | Winlogbeat OSS release. |
| `winlogbeat_graylog_host` | `graylog.example.com` | Graylog Beats host. |
| `winlogbeat_graylog_port` | `5044` | Graylog Beats port. |
| `winlogbeat_event_log_groups` | all built-in groups | Curated event log groups to collect. |
| `winlogbeat_event_logs` | unset | Complete manual event-channel override. |
| `winlogbeat_download_mode` | `target` | `target` or `delegated` ZIP retrieval. |
| `winlogbeat_service_state` | `started` | Desired service state. |
| `winlogbeat_service_enabled` | `true` | Start Winlogbeat on boot. |

Use a narrow selection where it fits the host role:

```yaml
winlogbeat_event_log_groups:
  - baseline
  - powershell
  - task_scheduler
```

In delegated mode, the download delegate retrieves the ZIP, the controller
fetches it to its cache, then copies it to Windows targets. The ZIP checksum
is always verified on the Windows target before extraction.

## Uninstall

```yaml
- name: Remove Windows event forwarding
  hosts: windows
  gather_facts: true
  tasks:
    - ansible.builtin.include_role:
        name: tuxbull.winlogbeat_forwarder
        tasks_from: uninstall.yml
```

The uninstall task stops and removes the service, then removes the installation,
data, log, and temporary download directories.

## Validation

After a deployment, verify the Windows service and rendered configuration:

```powershell
Get-Service winlogbeat
& 'C:\Program Files\Winlogbeat\current\winlogbeat.exe' test config -c 'C:\Program Files\Winlogbeat\current\winlogbeat.yml' -e
```

For receiver proof, search Graylog by a unique controlled event marker and
verify `source`, `winlog_channel`, `winlog_provider_name`, `winlog_event_id`,
and `agent_type:winlogbeat`.

## License

Apache License 2.0
