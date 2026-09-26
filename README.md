# Ansible Role: openbao_config

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-openbao_config)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-openbao_config)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-openbao_config)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-openbao_config/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-openbao_config/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-openbao_config/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-openbao_config/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for managing OpenBao policies, identities, KV v2 secrets and
explicit credential operations.

## Purpose

This role manages OpenBao API configuration from the Ansible controller. It
reconciles named ACL policies,
AppRole and userpass identities, KV version 2 mounts and secret values. API
resources not listed in the inputs are preserved.
The separate secret_id entry point registers, optionally verifies and revokes
credentials.

## Scope

### Managed

- ACL policy contents and explicit removal
- AppRole and userpass authentication mounts and lease tuning
- Application AppRoles, optional Role IDs, token policies, TTLs and CIDR
  restrictions
- Userpass users with initial passwords, token policies and CIDR restrictions
- KV version 2 mounts, reconciled secret values and one-time generation of
  missing keys
- Explicit registration, optional login verification and revocation of
  individual Secret IDs

### Not Managed

- Server installation, TLS, seal configuration, self-initialization and
  audit-device configuration
- KV version upgrades, PKI, database or SSH engine configuration
- Automatic resource pruning or replacement of existing mount types
- Backups, snapshots, recovery workflows and root-token generation
- Ansible Vault encryption, editing and distribution of caller credentials

## Requirements

- OpenBao 2.4 or newer, initialized and unsealed, with HTTPS reachable from the
  controller.
- An AppRole authorized for the selected operation. The optional jomrr.openbao
  bootstrap provides the ansible/ mount and ansible-admin AppRole and policy.
- The jomrr.openbao Galaxy role dependency provisions integration test servers
  only; it is not run by this role's normal entry points. Set
  OPENBAO_MOLECULE_SERVER_ROLE to a local checkout when testing unpublished
  server changes.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `openbao_config_url`

Type: `str`. Required: `true`.

HTTPS API origin reachable from the controller, without a trailing slash or
path.

### `openbao_config_ca_file`

Type: `path`. Required: `false`.

CA bundle on the controller; omitted to use the controller's system trust store.

### `openbao_config_auth_mount`

Type: `str`. Required: `false`.

Authentication mount of the caller-supplied AppRole.

Default:

```yaml
openbao_config_auth_mount: ansible
```

### `openbao_config_role_id`

Type: `str`. Required: `true`.

Role ID of the caller-supplied AppRole for the selected operation.

### `openbao_config_secret_id`

Type: `str`. Required: `true`.

Caller-supplied AppRole Secret ID from Ansible Vault outside this OpenBao
instance.

### `openbao_config_timeout`

Type: `int`. Required: `false`.

HTTP request timeout in seconds.

Default:

```yaml
openbao_config_timeout: 30
```

### `openbao_config_policies`

Type: `list`. Required: `false`.

ACL policies managed by name; omitted policies are preserved.

Default:

```yaml
openbao_config_policies: []
```

### `openbao_config_auth_methods`

Type: `list`. Required: `false`.

AppRole and userpass authentication mounts; deletion also removes their
identities and credentials.

Default:

```yaml
openbao_config_auth_methods: []
```

### `openbao_config_secrets_engines`

Type: `list`. Required: `false`.

KV version 2 mounts; deletion permanently removes the mount and its secrets.

Default:

```yaml
openbao_config_secrets_engines: []
```

### `openbao_config_approles`

Type: `list`. Required: `false`.

Application AppRoles; Secret IDs are managed only through the explicit secret_id
entry point.

Default:

```yaml
openbao_config_approles: []
```

### `openbao_config_users`

Type: `list`. Required: `false`.

Userpass users; passwords are set only at creation and omitted users are
preserved.

Default:

```yaml
openbao_config_users: []
```

### `openbao_config_kv_secrets`

Type: `list`. Required: `false`.

KV v2 secrets with reconciled data and generated values; writes use check and
set.

Default:

```yaml
openbao_config_kv_secrets: []
```

### `openbao_config_secret_id_mount`

Type: `str`. Required: `false`.

AppRole mount used by the secret_id entry point.

Default:

```yaml
openbao_config_secret_id_mount: ansible
```

### `openbao_config_secret_id_role`

Type: `str`. Required: `false`.

AppRole name used by the secret_id entry point.

Default:

```yaml
openbao_config_secret_id_role: ansible-admin
```

### `openbao_config_secret_id_value`

Type: `str`. Required: `false`.

Required by secret_id; random Secret ID of at least 32 characters to register or
revoke.

### `openbao_config_secret_id_role_id`

Type: `str`. Required: `false`.

Required by secret_id when state is present and openbao_config_secret_id_verify
is true.

### `openbao_config_secret_id_verify`

Type: `bool`. Required: `false`.

Log in from the controller with a present Secret ID and revoke the verification
token.
Disable explicitly when the controller is outside the application AppRole's
allowed source networks.

Default:

```yaml
openbao_config_secret_id_verify: true
```

### `openbao_config_secret_id_state`

Type: `str`. Required: `false`.

Explicit secret_id operation; present registers and optionally verifies, absent
revokes the selected credential.

Default:

```yaml
openbao_config_secret_id_state: present
```

## Check Mode

Check mode reads a running instance and reports resource changes without
applying them.

- AppRole login acquires a short-lived token even in check mode; the token is
  revoked at the end of the invocation.
- The secret_id entry point does not register or revoke credentials in check
  mode.
- KV changes do not generate values or write secrets in check mode.
- There is no offline simulation for an unreachable or uninitialized server.

## Service Behavior

API configuration runs on the controller and does not install packages or
restart OpenBao.

## Security Notes

- Supply the management Secret ID from Ansible Vault outside the managed OpenBao
  instance.
- Sessions are revoked even when an API operation fails. HTTP redirects are
  disabled, TLS validation remains enabled, and authentication and credential
  responses are hidden from task output.
- The management identity is highly privileged, including policy creation. Token
  lifetimes limit individual sessions but do not expire a reusable Secret ID;
  register replacements and explicitly revoke old credentials.
- Built-in policy names and the ansible management mount are protected from
  normal resource lists. Removing a secrets-engine mount with state=absent
  destroys its data; removing an auth mount removes its identities and
  credentials.

## Operational Notes

- KV data keys are reconciled on every run, replacing each declared value
  including nested mappings. Additional keys are preserved. Generate maps key
  names to optional length and chars password-lookup settings; only missing keys
  are generated, with no local password file. Length defaults to 32 and chars to
  [ascii_letters, digits] for each key; override them in that key's generate
  settings. Data and generate must not overlap. Writes use the current version
  as CAS and concurrent modifications fail instead of being overwritten.
  Soft-deleted or destroyed current versions require explicit restoration or
  metadata deletion before recreation. State absent deletes metadata and every
  version permanently.
- Read secrets in the consuming playbook: store one
  community.hashi_vault.vault_login result for the Reader-AppRole with set_fact,
  pass its token to community.hashi_vault.vault_kv2_get lookups, and revoke-self
  in an always block. Protect login and secret-consuming tasks with no_log.
  These lookups require community.hashi_vault and hvac on the controller.
- User passwords are required on creation and never sent during updates, even
  when the input changes. AppRole secret_id_bound_cidrs and AppRole/userpass
  token_bound_cidrs default to empty lists without source restrictions. Set them
  per item; empty lists remove existing bindings. CIDRs must match the API
  representation; no address normalization is performed. Use bare addresses for
  individual hosts, such as 192.0.2.10 or ::1. Host suffixes /32 and /128 are
  rejected before API mutation.
- Item state defaults to present for policies, authentication mounts,
  secrets-engine mounts, AppRoles, users and KV secrets. Both mount lease TTLs
  default to 0, inheriting the server defaults. Set overrides on the individual
  item. AppRoles default to mount approle, token_ttl 900 and token_max_ttl 3600
  seconds, secret_id_ttl 0 (no expiry), and secret_id_num_uses 0 (unlimited
  use). These defaults have no role-wide override variables.
- Run one invocation per OpenBao instance at a time, normally in a localhost
  play. API addresses have no trailing slash or path. Resource names and mount
  paths use lowercase letters, digits, underscores and hyphens. A mount may be
  declared only once; each AppRole is identified by its name within its mount.
- Main reconciles declared configuration, users and KV secrets. Use
  ansible.builtin.include_role with tasks_from=secret_id to register or revoke a
  credential. Both entry points require openbao_config_url,
  openbao_config_role_id and openbao_config_secret_id.
- To change the management Secret ID, store a new random value in Ansible Vault
  alongside the old value. Invoke secret_id with state=present using the old
  login and the new value; the role verifies the new login while retaining the
  old credential. Switch openbao_config_secret_id and
  openbao_bootstrap_secret_id to the new value, then invoke secret_id with
  state=absent for the old value. The currently used credential cannot be
  revoked.
- The secret_id entry point also provisions application credentials by selecting
  their mount, AppRole and Role ID. Login verification defaults to enabled and
  consumes one use of a usage-limited Secret ID. Set
  openbao_config_secret_id_verify=false when the controller is outside the
  application's secret_id_bound_cidrs. In that case no test login is attempted,
  no use is consumed, and openbao_config_secret_id_role_id is optional.
  Registration alone does not prove that the application can log in.
  Verification tokens must permit self-revocation; the standard default policy
  provides this permission. The management session is always revoked
  independently.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |

## Example Playbook

### Configure an application AppRole and KV v2 mount

The management AppRole was created by jomrr.openbao self-initialization.
The application policy permits access only beneath its own KV prefix.

```yaml
---
- name: Configure OpenBao resources
  hosts: localhost
  gather_facts: false
  roles:
    - role: jomrr.openbao_config
      vars:
        openbao_config_url: https://bao.example.org:8200
        openbao_config_ca_file: /etc/ssl/certs/internal-ca.pem
        openbao_config_role_id: ansible-controller
        openbao_config_secret_id: >-
          {{ vault_openbao_management_secret_id }}
        openbao_config_policies:
          - name: application
            rules: |-
              path "applications/data/example/*" {
                capabilities = ["create", "read", "update"]
              }
        openbao_config_auth_methods:
          - path: approle
            type: approle
        openbao_config_approles:
          - name: application
            role_id: example-application
            token_policies: [application]
        openbao_config_secrets_engines:
          - path: applications
            type: kv
```

### Register a replacement management credential

Keep the old login valid until Ansible Vault consumers have switched to the
new credential.
After switching, invoke the same entry point with the old value and state=absent.

```yaml
- name: Register and verify the replacement Secret ID
  ansible.builtin.include_role:
    name: jomrr.openbao_config
    tasks_from: secret_id
  vars:
    openbao_config_secret_id_value: >-
      {{ vault_openbao_next_secret_id }}
    openbao_config_secret_id_role_id: ansible-controller
    openbao_config_secret_id_state: present
```

### Reconcile generated and declared KV keys

```yaml
- name: Configure application secrets
  ansible.builtin.include_role:
    name: jomrr.openbao_config
  vars:
    openbao_config_kv_secrets:
      - mount: applications
        path: example/database
        generate:
          password: {length: 32, chars: [ascii_letters, digits]}
        data:
          username: example
```

### Register an application credential without a controller test login

```yaml
- name: Register a credential restricted to the application's network
  ansible.builtin.include_role:
    name: jomrr.openbao_config
    tasks_from: secret_id
  vars:
    openbao_config_secret_id_mount: approle
    openbao_config_secret_id_role: application
    openbao_config_secret_id_value: "{{ vault_application_secret_id }}"
    openbao_config_secret_id_verify: false
```

## References

- [Vault login lookup](https://docs.ansible.com/ansible/latest/collections/community/hashi_vault/vault_login_lookup.html)
- [KV v2 lookup](https://docs.ansible.com/ansible/latest/collections/community/hashi_vault/vault_kv2_get_lookup.html)
- [OpenBao KV v2 API](https://openbao.org/docs/api/secret/kv/kv-v2/)
- [OpenBao userpass API](https://openbao.org/docs/api/auth/userpass/)
- [OpenBao AppRole API](https://openbao.org/docs/api/auth/approle/)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
