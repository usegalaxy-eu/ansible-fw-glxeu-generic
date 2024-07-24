# usegalaxy.eu local IP filter playbook

Ansible playbook for setting up the local IP filter on a host. For
portability reasons, it uses `firewalld(1)` on Redhat-based systems;
as of this writing this playbook is not implemented for Debian-based
systems.

The firewall configuration created by this playbook distinguishes
three zones:

- A `trusted` zone: networks assigned to this zone are not subject
  to IP filtering, all traffic to/from these networks is accepted.

- An `internal` zone: ingress traffic from networks assigned to
  this zone is accepted only for services that are explicitly
  whitelisted for it; egress traffic is unrestricted.

- The `public` zone: applies to ingress traffic from any network
  that is not explicitly assigned to either the trusted or the
  internal zone. Only services whitelisted for the zone are
  allowed; egress traffic is unrestricted.

## Configurable variables

- `firewall_restart_daemon` (default `false`): If true, restart
  firewalld as the last task in the playbook to activate the
  firewall configuration; otherwise a manual reload is required
  to actually activate the configuration.

- `firewall_ip_prefix_trusted` (default is site-specific):
  A list of networks (in CIDR notation) that are assigned to
  the `trusted` zone.

- `firewall_ip_prefix_internal` (default is site-specific):
  A list of networks (in CIDR notation) that are assigned to
  the `internal` zone.

- `firewall_internal_services` (default `[http, https, ssh]`):
  The list of services whitelisted for the `internal` zone.

- `firewall_public_services` (default: `[http, https]`):
  The list of services whitelisted for the `public` zone.


