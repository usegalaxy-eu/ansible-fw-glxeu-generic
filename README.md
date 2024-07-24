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

## Configurable playbook variables

- `firewall_restart_daemon` (default `false`): If true, restart
  firewalld as the last task in the playbook to activate the
  firewall configuration; otherwise a manual reload is required
  to actually activate the configuration.

- `firewall_reset_configuration` (default `false`): If true,
  reset the firewall rules to system defaults before applying
  our changes. This is required to actually *revoke* permissions.
  **USE WITH CAUTION**, see the section "CAVEAT EMPTOR" below!

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


## CAVEAT EMPTOR

An incoming IP packet will be matched by *exactly one* zone,
based on the networks assigned to each zone, first match wins, with
`internal` being matched before `trusted` and `public` (being bound
to destination network interfaces rather than source networks)
receiving whatever is not matched by any other zone.

THUS IT IS IMPORTANT THAT THE SUBNETS ASSIGNED TO THE ZONES BE ALL
DISJUNCT!

It might appear logical to e.g. bind an organization's whole internal IP
address space (say, 10.0.0.0/8) to the `internal` zone and the network
where the service actually lives (e.g. 10.2.3.0/24) to `trusted`, but this
is **not** how firewalld works! The source address 10.2.3.4 will be matched
in `internal`, **not** in `trusted` (see above for why).

Likewise, if you intend the services whitelisted in `internal` to be a
superset of those whitelisted in `public`, which is what you'll probably
want, you have to list them all explicitly, the list for `internal` does
**not** "inherit" from that of `public`. (See the variable documentation
above for an example.)

Last not least: remember to set `firewall_reset_configuration: true` if
you want to actually *revoke* any permissions you had previously granted via
this playbook. While the playbook does remove well-known system default
service permissions from both `internal` and `public` zones before applying
its own changes, it will, by default, only ever *add* new permissions via
the "prefix" and "services" variables listed above. (At this writing, there
is no obvious way to reset individual network or service lists to empty
with `ansible.posix.firewalld` v.1.5.4.) **USE THIS OPTION WITH CAUTION,
AS IT MIGHT AFFECT TOTALLY UNRELATED, BUT REQUIRED, FIREWALL SETTINGS MADE
BY OTHER SUBSYSTEMS, SUCH AS FORWARDING RULES FOR DOCKER CONTAINERS!**
