# ansible-apps_squid

[![Galaxy Role](https://img.shields.io/badge/galaxy-apps_squid-purple?style=flat)](https://galaxy.ansible.com/lotusnoir/apps_squid)
[![Version](https://img.shields.io/github/release/lotusnoir/ansible-apps_squid.svg)](https://github.com/lotusnoir/ansible-apps_squid/releases/latest)
[![GitHub repo size](https://img.shields.io/github/repo-size/lotusnoir/ansible-apps_squid?color=orange&style=flat)](https://galaxy.ansible.com/lotusnoir/apps_squid)
[![downloads](https://img.shields.io/ansible/role/d/lotusnoir/apps_squid)](https://galaxy.ansible.com/lotusnoir/apps_squid)
[![License](https://img.shields.io/badge/license-Apache--2.0-brightgreen?style=flat)](https://opensource.org/licenses/Apache-2.0)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Description

Deploy [squid](http://www.squid-cache.org/) proxy system using ansible.

This role installs Squid and builds its configuration from small, included fragments instead of one monolithic `squid.conf`: general settings are rendered into `squid.conf`, while ACLs, sources and destinations are generated as separate files under `/etc/squid/conf.d/` driven by a single structure, `squid_groups`. A config reload (`squid -k reconfigure`) is triggered only when a relevant fragment changes; a full restart only happens for `squid.conf` or secret changes.

## Requirements
- `community.general` collection, required when a `squid_groups` source entry is a hostname that is neither a raw IP/CIDR nor an existing inventory host (resolved via the `community.general.dig` lookup): `ansible-galaxy collection install community.general`
- `squid-langpack` package available on the target if `squid_error_directory` is set (source of the error page templates)
- `gather_facts: true` is mandatory — the role checks `ansible_facts.distribution`/`distribution_version` against the platforms declared in `meta/main.yml` and gracefully skips the host (`meta: end_host`) if unsupported

## Role variables

See [variables](/defaults/main.yml) for the full list with raw defaults.

### General settings

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_visible_hostname` | `{{ inventory_hostname }}` | Value of the `visible_hostname` directive. |
| `squid_port` | `3128` | Value of `http_port`. Can also be `ip:port`, e.g. `10.0.0.5:3128`. |
| `squid_core_dump_dir` | *(unset)* | `coredump_dir` directive, e.g. `/var/spool/squid`. |
| `squid_via` | *(unset)* | `via` directive (`on`/`off`). |

### Resource limits

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_max_filedescriptors` | *(unset)* | Sets `max_filedescriptors` **and** OS-level limits: writes `/etc/security/limits.d/squid.conf` (soft/hard `nofile`) and a systemd override (`LimitNOFILE=`) under `squid.service.d/`, then reloads the daemon and restarts Squid. |
| `squid_cache_mem` | *(unset)* | `cache_mem` directive, e.g. `2 GB`. |
| `squid_memory_pools` | *(unset)* | `memory_pools` (`on`/`off`). |
| `squid_workers` | *(unset)* | `workers` directive (SMP workers count). |
| `squid_maximum_object_size_in_memory` | *(unset)* | `maximum_object_size_in_memory`, e.g. `32 MB`. |
| `squid_maximum_object_size` | *(unset)* | `maximum_object_size`, e.g. `512 MB`. |

### HTTP / SSL tuning

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_httpd_suppress_version_string` | *(unset)* | Hides the Squid version in responses (`on`/`off`). |
| `squid_client_persistent_connections` | *(unset)* | `client_persistent_connections` (`on`/`off`). |
| `squid_server_persistent_connections` | *(unset)* | `server_persistent_connections` (`on`/`off`). |
| `squid_connect_timeout` | *(unset)* | `connect_timeout`, e.g. `15 seconds`. |
| `squid_request_timeout` | *(unset)* | `request_timeout`, e.g. `1 minutes`. |
| `squid_read_timeout` | *(unset)* | `read_timeout`, e.g. `2 minutes`. |
| `squid_client_lifetime` | *(unset)* | `client_lifetime`, e.g. `30 minutes`. |

### Anti-DDoS hardening

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_request_body_max_size` | *(unset)* | `request_body_max_size` directive. |
| `squid_reply_body_max_size` | *(unset)* | Rendered as `reply_body_max_size <value> all`. |

### DNS settings

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_dns_nameservers` | *(unset)* | `dns_nameservers` directive. |
| `squid_dns_timeout` | *(unset)* | `dns_timeout`, e.g. `5 seconds`. |
| `squid_positive_dns_ttl` | *(unset)* | `positive_dns_ttl`, e.g. `8 hours`. |
| `squid_negative_ttl` | *(unset)* | `negative_ttl`, e.g. `4 minutes`. |
| `squid_forwarded_for` | *(unset)* | `forwarded_for` (`on`/`off`/`delete`/`truncate`). |

### Authentication (auth_param)

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_auth_param_secret` | `""` | If non-empty, written to `/etc/squid/ldap.secret` (owner `proxy`, mode `0600`) — triggers a config test + Squid restart. |
| `squid_auth_param` | `{}` | List of `{name, type, value, comment}` → one `auth_param <name> <type> <value>` line each. Typically used to declare an LDAP `basic` auth helper (program path, realm, credentialsttl, casesensitive…). |

### Base ACLs (squid_acls)

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_external_acl_type` | `""` | Raw line inserted as-is (e.g. an `external_acl_type ldap_group ...` definition). |
| `squid_localnet_name` | `"localnet"` | Name of the ACL used for the global allow-lists (`squid_global_domains` / `squid_global_ips`). The ACL itself must be declared through `squid_acls`. |
| `squid_acls` | see defaults | List of `{name, type, value, comment}` → one `acl <name> <type> <value>` line each. Ships with safe defaults: `localnet` (RFC1918 ranges), `SSL_ports`, `Safe_ports` and the `CONNECT` method. |

### Sources / destinations groups (squid_groups)

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_generate_groups_from_inventory` | `false` | Must be `true` to activate the 'inventory groups → destinations' pattern (source files generated per referenced Ansible inventory group). |
| `squid_single_vm_domains` | `[]` | List of DNS suffixes tried (in order) to resolve a single host/VM entry that isn't a raw IP or a known inventory host, via `getent hosts`. |
| `squid_groups` | `[]` | List of group definitions. Each entry matches one of 3 patterns depending on which keys are set: (A) `name` + `source_inventory_groups` + `destination_file` + `destinations`; (B) `name` + `source_file` + `sources` + `destination_file` + `destinations`; (C) `name` (hostname/IP) + `destinations` only. See notes below. |
| `squid_global_domains` | `[]` | Domains allowed for everyone matching `squid_localnet_name` → `global_domains.txt` + `acl global_domains`. |
| `squid_global_ips` | `[]` | Same, for IP destinations → `global_ips.txt` + `acl global_ips`. |

### HTTP access rules

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_http_access_generate` | `false` | If `true`, auto-generates `http_access allow <group.name> dom_<group.name>` for every entry in `squid_groups`. |
| `squid_http_access` | see defaults | List of `{access, type, scope}` → one `http_access <access> <type> <scope>` line each. Ships with safe defaults denying unsafe ports and CONNECT to non-SSL ports. Always rendered before the generated group rules; `http_access deny all` is always appended last. |
| `squid_always_direct` | `{}` | List of `{access, value}` → `always_direct <access> <value>` (e.g. bypass peers for FTP). |
| `squid_deny_info` | `""` | `deny_info` directive — custom deny page mapping. |

### Cache

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_cache_enable` | `false` | If `false` (default), hardens Squid with `cache deny all` (no on-disk caching). If `true`, uses `squid_cache_dir`. |
| `squid_cache_dir` | `"ufs /var/spool/squid 10240 16 256"` | `cache_dir` directive, only applied when `squid_cache_enable: true`. |
| `squid_cache_replacement_policy` | `""` | `cache_replacement_policy`, e.g. `heap LFUDA`. |

### Refresh patterns

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_refresh` | `""` | If non-empty, enables the standard `refresh_pattern` block (ftp, gopher, cgi-bin/`?`, default `.`). |

### SNMP

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_snmp_enable` | `false` | Enables the SNMP block (`snmp_port`, `acl snmppublic`, `snmp_access`). |
| `squid_snmp_port` | `3401` | `snmp_port` directive. |
| `squid_snmp_community` | `"localhost"` | SNMP community string. |
| `squid_snmp_incoming_address` | `"127.0.0.1"` | `snmp_incoming_address` directive. |

### Annotations (squid_enable_notes)

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_enable_notes` | `false` | If `true`, generates `conf.d/notes.conf` (included from `squid.conf`) with one `note rule_matched <name> <acl>` per group in `squid_groups`, plus `manager`, `global_domain`/`global_ip` and a catch-all `no_rule_set`. Pairs with a custom `squid_log_format` using `%{rule_matched}note`. |

### Logging & custom error pages

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_error_directory` | `""` | If set, runs the error-pages generation task: copies/customizes pages from `/usr/share/squid-langpack/<squid_error_lang>/` into this directory, then sets `error_directory`. |
| `squid_error_lang` | `"fr"` | Source language folder under `squid-langpack` used to generate custom error pages. |
| `squid_error_message` | *(unset, `!unsafe` string)* | Template content written into every generated error page body, e.g. `PROXY_SQUID_ERROR type=${f} url=%U client=%i time=%t`. |
| `squid_log_format` | *(unset, `!unsafe` string)* | Custom `logformat` directive name + pattern, e.g. a `readable_with_rule` format including `%{rule_matched}note`. |
| `squid_log_access` | `"stdio:/var/log/squid/access.log squid"` | `access_log` directive. Switch the format name to match `squid_log_format` if you define one. |
| `squid_log_cache` | `"/var/log/squid/cache.log"` | `cache_log` directive. |
| `squid_log_store` | `"stdio:/var/log/squid/store.log"` | `cache_store_log` directive. |
| `squid_logfile_rotate` | `0` | `logfile_rotate` directive. |

### Catch-all

| Variable | Default | Description |
|----------|---------|--------------|
| `squid_various` | `""` | Raw text appended verbatim at the end of `squid.conf` — use for any directive not covered by a dedicated variable. |

### Framework variable (not in defaults/main.yml)

| Variable | Default | Description |
|----------|---------|--------------|
| `success_run_flag` | *(undefined)* | If defined and truthy, writes an idempotency timestamp file to `/var/cache/ansible/<role_name>` after a successful run (Debian/RedHat only). Not required for the role to work. |

### Notes

> Safe ACL/access defaults are shipped out of the box: `localnet` (RFC1918 ranges), `SSL_ports`, `Safe_ports`, the `CONNECT` method, plus a base `http_access` denying CONNECT to non-SSL ports and unsafe ports. Override `squid_acls` / `squid_http_access` entirely if you need different hardening rules.
>
> `squid_localnet_name` only *names* the ACL used for global allow-lists; make sure an ACL with that name is actually declared via `squid_acls`.
>
> `squid_groups` supports three patterns depending on which keys you set: (A) inventory groups → destinations, (B) a YAML list of IPs/hostnames → destinations, (C) a single host/IP → destinations. Mixing patterns across different entries in the same list is supported.
>
> `rules.conf` always ends with `http_access deny all` — the default-deny is built in regardless of what you configure.
>

## Generated files layout

```
/etc/squid/squid.conf
/etc/squid/ldap.secret                                   # if squid_auth_param_secret is set
/etc/squid/generate_error_pages.sh                        # if squid_error_directory is set
/etc/squid/conf.d/
├── acls_sources.conf
├── acls_destinations.conf
├── rules.conf
├── notes.conf                                            # if squid_enable_notes: true
├── sources/
│   ├── inventory_groups/<ansible_group>.txt               # Pattern A
│   ├── groups/<source_file>.txt                           # Pattern B
│   └── hosts/<name>.txt                                   # Pattern C
└── destinations/
    ├── global/global_domains.txt | global_ips.txt         # squid_global_domains / squid_global_ips
    ├── groups/<destination_file>.txt                       # Pattern A & B
    └── hosts/<name>.txt                                    # Pattern C

```

## Tags

| Tag | Scope |
|-----|-------|
| `always` | OS support check, package install, directory tree creation, success flag. |
| `errors` | Custom error pages generation. |
| `config` | LDAP secret, squid.conf, final syntax validation. |
| `rules` | ACLs, destinations/sources files, rules.conf, notes.conf. |
| `inventory` | Inventory-group source files (Pattern A). |
| `global` | global_domains.txt / global_ips.txt. |

## Handlers

| Handler | Description |
|---------|--------------|
| `Reload daemon` | `systemctl daemon-reload` (after a systemd override change). |
| `Test squid configuration` | `squid -k parse` (after squid.conf or LDAP secret change). |
| `Restart squid` | Full service restart. |
| `Reload squid` | `squid -k reconfigure` (after any conf.d/* fragment change — no downtime). |

## Examples

Minimal example. With all variables left at their defaults, this installs Squid on port 3128 with safe base ACLs/access rules but **no allowed destination** — you must add `squid_groups` or `squid_global_domains` / `squid_global_ips` to actually let traffic through.

```yaml
---
- hosts: apps_squid
  become: true
  become_method: sudo
  gather_facts: true
  roles:
    - role: ansible-apps_squid # or lotusnoir.base.apps_squid
```

Hardened proxy combining custom ACLs, LDAP authentication, the three `squid_groups` patterns, a global allow-list, annotated logs and custom error pages.

```yaml
---
- hosts: apps_squid
  become: true
  become_method: sudo
  gather_facts: true
  roles:
    - role: ansible-apps_squid # or lotusnoir.base.apps_squid
vars:
    squid_port: 3128
    squid_max_filedescriptors: "65535"
    squid_cache_mem: "2 GB"

    squid_acls:
      - { name: "localnet", type: "src", value: "10.0.0.0/8" }
      - { name: "SSL_ports", type: "port", value: "443" }
      - { name: "Safe_ports", type: "port", value: "80", comment: "http" }
      - { name: "Safe_ports", type: "port", value: "443", comment: "https" }
      - { name: "CONNECT", type: "method", value: "CONNECT" }

    squid_http_access:
      - { access: "deny", type: "!Safe_ports", scope: "" }
      - { access: "deny", type: "CONNECT", scope: "!SSL_ports" }

    squid_auth_param_secret: "{{ vault_squid_ldap_password }}"
    squid_auth_param:
      - { name: "basic", type: "program", value: "/usr/lib/squid/basic_ldap_auth -v 3 -b \"dc=example,dc=com\" -D \"cn=squid,ou=services,dc=example,dc=com\" -W /etc/squid/ldap.secret -f \"(&(objectClass=person)(sAMAccountName=%s))\" ldap.example.com" }
      - { name: "basic", type: "realm", value: "Squid Proxy" }
      - { name: "basic", type: "credentialsttl", value: "4 hours" }

    squid_generate_groups_from_inventory: true
    squid_single_vm_domains:
      - ".prod.example.com"
    squid_groups:
      - name: debian_hosts
        source_inventory_groups:
          - os_debian12
        destination_file: repo_debian
        destinations:
          - deb.debian.org
          - security.debian.org
      - name: batch_servers
        source_file: batch_vms
        sources:
          - 10.1.2.3
          - 10.1.2.0/24
        destination_file: access_batch
        destinations:
          - .google.com
          - .microsoft.com
      - name: vm-proxytestclient-staging-01
        destinations:
          - .github.com

    squid_http_access_generate: true

    squid_global_domains:
      - .ubuntu.com
      - .canonical.com

    squid_enable_notes: true
    squid_log_format: !unsafe 'readable_with_rule %{%d/%b/%Y:%H:%M:%S %z}tl %6tr %>a %<la:%<lp %Ss/%03>Hs %<st %rm %ru %>rp %un %Sh/%<A %mt %dt rule=%{rule_matched}note %{User-Agent}>h'
    squid_log_access: "stdio:/var/log/squid/access.log readable_with_rule"

    squid_error_directory: "/etc/squid/errors"
    squid_error_lang: "en"
    squid_error_message: !unsafe 'PROXY_SQUID_ERROR type=${f} url=%U client=%i time=%t'
```

## License

This project is licensed under Apache License. See [LICENSE](/LICENSE) for more details.

## Author Information

- [Philippe LEAL](https://github.com/lotusnoir)
