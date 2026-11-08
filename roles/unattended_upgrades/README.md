# unattended_upgrades

Installs `unattended-upgrades` and fixes a Raspberry-Pi-OS-specific origin
gap - see `tasks/main.yml` for the actual fix and why it's needed. This
file is for verifying it's actually working, which the role itself
doesn't check.

## Checking it's actually configured

```bash
cat /etc/apt/sources.list.d/*.sources
apt-cache policy | grep -B1 release   # shows the real o=/n=/l= fields to match on
apt-config dump Unattended-Upgrade::Origins-Pattern   # confirm both the stock Debian
                                                        # entries AND the custom ones
                                                        # are present, not one or the
                                                        # other - list-type apt.conf
                                                        # options merge/append across
                                                        # files in apt.conf.d/, read in
                                                        # lexical order, rather than
                                                        # replacing
```

## Checking it's actually working

Not just that the command exits `0` - whether the origin matching rule
actually catches what it's meant to:

```bash
sudo unattended-upgrade --dry-run --debug 2>&1 | grep "Allowed origins"
```

## Two non-obvious gotchas if a scheduled run doesn't seem to have done anything

Surfaced during review of the upstream PR this fix is based on, not
specific to this custom config:

- `apt-daily-upgrade.timer` → `apt.systemd.daily` has
  `ExecStartPre=-/usr/lib/apt/apt-helper wait-online`; if that check
  fails/times out, the run silently no-ops. Check
  `systemctl status apt-daily-upgrade.timer` / the journal after a
  scheduled run, don't just assume it worked.
- `/var/lib/apt/periodic/upgrade-stamp` suppresses
  `apt.systemd.daily install` if it thinks it already ran today - delete
  it if you need to force a real test run rather than waiting for the
  timer.
