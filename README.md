# Ansible Role: openbao_config

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-openbao_config)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-openbao_config)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-openbao_config)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-openbao_config/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-openbao_config/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-openbao_config/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-openbao_config/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for managing OpenBao AppRoles, ACL policies, KV v2 mounts and
explicit credential operations.

## Purpose

This role manages OpenBao API configuration from the Ansible controller. It
reconciles named ACL policies,
AppRole mounts and roles, and KV version 2 mounts. API resources not listed in
the inputs are preserved.
Separate recovery and secret_id entry points perform deliberate credential
operations.

## Scope

### Managed

- ACL policy contents and explicit removal
- AppRole authentication mounts and lease tuning
- Application AppRoles, optional Role IDs, token policies and TTLs
- KV version 2 secrets engine mounts and lease tuning
- Initial PGP-encrypted recovery-key export through an explicit entry point
- Explicit registration, login verification and revocation of individual Secret
  IDs

### Not Managed

- Server installation, TLS, seal configuration, self-initialization and
  audit-device configuration
- Secret values, KV version upgrades, PKI, database or SSH engine configuration
- Automatic resource pruning or replacement of existing mount types
- Rotation of existing recovery keys or automatic root-token generation
- Ansible Vault encryption, editing and distribution of caller credentials

## Requirements

- OpenBao 2.4 or newer, initialized and unsealed, with HTTPS reachable from the
  controller.
- A management AppRole authorized for the declared configuration and credential
  operations. The optional jomrr.openbao bootstrap provides the ansible/ mount
  and ansible-admin AppRole and policy.
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

CA bundle on the controller; omitted to use the system trust store.

### `openbao_config_auth_mount`

Type: `str`. Required: `false`.

Authentication mount of the controller management AppRole.

Default:

```yaml
openbao_config_auth_mount: ansible
```

### `openbao_config_role_id`

Type: `str`. Required: `true`.

Role ID of the controller management AppRole.

### `openbao_config_secret_id`

Type: `str`. Required: `true`.

Management Secret ID from Ansible Vault outside this OpenBao instance.

### `openbao_config_timeout`

Type: `int`. Required: `false`.

HTTP request timeout in seconds.

Default:

```yaml
openbao_config_timeout: 30
```

### `openbao_config_state`

Type: `str`. Required: `false`.

Default desired state for resource items without an explicit state.

Default:

```yaml
openbao_config_state: present
```

### `openbao_config_approle_mount`

Type: `str`. Required: `false`.

Default mount for managed application AppRoles.

Default:

```yaml
openbao_config_approle_mount: approle
```

### `openbao_config_mount_default_lease_ttl`

Type: `int`. Required: `false`.

Default mount lease TTL in seconds; zero inherits the server default.

Default:

```yaml
openbao_config_mount_default_lease_ttl: 0
```

### `openbao_config_mount_max_lease_ttl`

Type: `int`. Required: `false`.

Maximum mount lease TTL in seconds; zero inherits the server default.

Default:

```yaml
openbao_config_mount_max_lease_ttl: 0
```

### `openbao_config_token_ttl`

Type: `int`. Required: `false`.

Initial token TTL in seconds for managed AppRoles.

Default:

```yaml
openbao_config_token_ttl: 900
```

### `openbao_config_token_max_ttl`

Type: `int`. Required: `false`.

Maximum token TTL in seconds for managed AppRoles.

Default:

```yaml
openbao_config_token_max_ttl: 3600
```

### `openbao_config_secret_id_ttl`

Type: `int`. Required: `false`.

Secret ID TTL in seconds for managed AppRoles; zero requires explicit
revocation.

Default:

```yaml
openbao_config_secret_id_ttl: 0
```

### `openbao_config_secret_id_num_uses`

Type: `int`. Required: `false`.

Login limit for managed AppRole Secret IDs; zero allows repeated use.

Default:

```yaml
openbao_config_secret_id_num_uses: 0
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

AppRole authentication mounts; deleting a mount also removes its roles and
credentials.

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

### `openbao_config_recovery_pgp_keys`

Type: `list`. Required: `false`.

Required by recovery; base64-encoded binary PGP public keys, one per share, in
recipient order.

### `openbao_config_recovery_threshold`

Type: `int`. Required: `false`.

Required by recovery; number of shares required for the recovery quorum.

### `openbao_config_recovery_output`

Type: `path`. Required: `false`.

Required by recovery; absolute export filename in an existing private controller
directory with mode 0700.
Stores only PGP-encrypted shares; use a path outside source repositories and
back it up independently.

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

Required by secret_id when state is present; Role ID used to verify the
registered credential.

### `openbao_config_secret_id_state`

Type: `str`. Required: `false`.

Explicit secret_id operation; present registers and verifies, absent revokes the
selected credential.

Default:

```yaml
openbao_config_secret_id_state: present
```

## Managed Files

- `openbao_config_recovery_output` Explicit recovery export on the controller,
  mode 0600, containing PGP-encrypted shares and cluster metadata.

## Check Mode

Check mode reads a running instance and reports resource changes without
applying them.

- AppRole login acquires a short-lived token even in check mode; the token is
  revoked at the end of the invocation.
- Recovery and Secret ID entry points do not generate, register, revoke or
  export keys in check mode.
- There is no offline simulation for an unreachable or uninitialized server.

## Service Behavior

API configuration does not install packages or restart OpenBao. All HTTP
operations run on the controller.

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
- Recovery exports contain only PGP-encrypted shares. Private PGP keys belong to
  the custodians and are never supplied to this role. Recovery keys cannot
  substitute for the static seal key when decrypting storage.

## Operational Notes

- Run one invocation per OpenBao instance at a time, normally in a localhost
  play. API addresses have no trailing slash or path. Resource names and mount
  paths use lowercase letters, digits, underscores and hyphens. A mount may be
  declared only once; each AppRole is identified by its name within its mount.
- Main reconciles only configuration. Use ansible.builtin.include_role with
  tasks_from=recovery to generate initial recovery keys, or tasks_from=secret_id
  to register or revoke a credential. All entry points require
  openbao_config_url, openbao_config_role_id and openbao_config_secret_id.
- For recovery, supply one base64-encoded binary PGP public key per share, the
  quorum and an absolute output filename in an existing controller directory
  with mode 0700 outside source repositories. Export public keys with gpg
  --export and encode those binary bytes as base64; do not pass ASCII-armored
  keys or key identifiers.
- Recovery records a recipient manifest at the output path before creating
  server-side PGP-encrypted backups. It then saves the encrypted shares
  atomically with mode 0600, verifies the saved contents and deletes the server
  backup. Retry the same invocation after an interruption, preserving the
  manifest and output path. Existing keys without a matching local manifest and
  encrypted shares locally or in the server backup cause failure; they are never
  silently replaced. A manifest without a backup field is not a completed
  export. Back up the export independently and distribute each encrypted share
  to its corresponding custodian.
- This entry point creates the first recovery key set only. Existing
  recovery-key rotation, export validation by custodians and quorum ceremonies
  use the authenticated OpenBao rotation workflow. An existing local export is
  not proof that externally rotated keys still match it; update custody records
  after any external rotation.
- To change the management Secret ID, store a new random value in Ansible Vault
  alongside the old value. Invoke secret_id with state=present using the old
  login and the new value; the role verifies the new login while retaining the
  old credential. Switch openbao_config_secret_id and
  openbao_bootstrap_secret_id to the new value, then invoke secret_id with
  state=absent for the old value. The currently used credential cannot be
  revoked.
- The secret_id entry point also provisions application credentials by selecting
  their mount, AppRole and Role ID. Login verification consumes one use of a
  usage-limited Secret ID. Token policies must permit self-revocation; the
  standard default policy provides this permission.
- For emergency administration, use the OpenBao generate-root quorum workflow
  with the recovery custodians. Protect its OTP or PGP recipient, submit the
  required shares, use the resulting token only for recovery, then revoke it.
  Neither normal convergence nor the recovery export creates or retains a root
  token.

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

### Export initial recovery keys

Connection variables use the same values as the preceding example.
The private export directory already exists.

```yaml
- name: Export PGP-encrypted recovery shares
  ansible.builtin.include_role:
    name: jomrr.openbao_config
    tasks_from: recovery
  vars:
    openbao_config_recovery_pgp_keys: >-
      {{ recovery_custodian_public_keys }}
    openbao_config_recovery_threshold: 3
    openbao_config_recovery_output: /srv/private/openbao-recovery.json
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

## References

- [OpenBao AppRole API](https://openbao.org/docs/api/auth/approle/)
- [OpenBao recovery-key generation](https://openbao.org/docs/api/system/rotate/init/)
- [OpenBao recovery-key backups](https://openbao.org/docs/api/system/rotate/backup/)
- [OpenBao generate-root](https://openbao.org/docs/commands/operator/generate-root/)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
