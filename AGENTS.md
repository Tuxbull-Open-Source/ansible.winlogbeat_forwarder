# Repository instructions

This repository provides `tuxbull.winlogbeat_forwarder`, a Windows-only role
that installs Winlogbeat OSS and forwards selected Windows Event Logs to a
Graylog Beats input. Keep it focused on Windows Event Log collection and Beats
forwarding; Linux log shipping belongs to a separate role.

## Safety and secrets

- Do not commit Windows inventories, connection passwords, API tokens, real
  internal host names, private addresses, or generated event markers.
- Keep public examples on placeholder host names such as `graylog.example.com`.
- Preserve the artifact SHA-512 validation and run `winlogbeat.exe test config`
  before starting or restarting the service.
- Use the bundled install/uninstall PowerShell scripts for the service; do not
  invent an unsupported service installation command.

## Validation and release

- Test through a Galaxy-style harness using the FQN
  `tuxbull.winlogbeat_forwarder`.
- For live Windows QA: verify WinRM and the Beats port first; run install and a
  zero-change second converge; independently check service state and config;
  emit one unique controlled Application event; prove its receipt through the
  Graylog API using the event marker plus `source`, `winlog_channel`,
  `winlog_provider_name`, `winlog_event_id`, and `agent_type`; then perform
  uninstall/cleanup verification and final reinstall/idempotency.
- The manual Windows lab must be left in the forwarding state unless explicitly
  requested otherwise.
- Before a release PR, run YAML parsing, role syntax validation, relevant
  controller collection checks, and `git diff --check`.
- Add the exact SemVer release target to `CHANGELOG.md` before creating a PR.
