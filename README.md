# step_cert

Installs the step CLI and provisions certificates and SSH certificate using a one time token.

A systemd timer and service is then configured to automatically renew each certificate.


## Platforms
* EL 7
* EL 8
* Debian buster
* Debian bullseye
## Example: Obtain a certificate and SSH certificate
```yaml
- hosts: all
  tasks:
    - import_role:
        name: step_cert
      vars:
        step_cert_ca_url: https://ca.example.com
        step_cert_provisioner: ansible
        step_cert_provisioner_pass: password
        step_cert_ca_fingerprint: 000000000000000000000
        step_cert_certs:
          - name: nginx
            cn: "{{ ansible_host | lower }}"
            crt_file: /etc/pki/tls/certs/nginx.crt
            key_file: /etc/pki/tls/private/nginx.key
            sans:
              - "{{ ansible_host | lower }}"
            renew_cmds:
              - nginx -s reload
        # Global ID to use across all SSH certs.
        step_cert_ssh_id: "{{ ansible_host | lower }}"
        # Global renew command for all SSH certs.
        step_cert_ssh_renew_cmds:
          - systemctl restart sshd
        step_cert_ssh_certs:
          - name: ed25519
            file: /etc/ssh/ssh_host_ed25519_key
          - name: ecdsa
            file: /etc/ssh/ssh_host_ecdsa_key
          - name: rsa
            file: /etc/ssh/ssh_host_rsa_key
```

## Example: Only install the cli
```yaml
- hosts: localhost
  tasks:
    - import_role:
        name: step_cert
        tasks_from: cli_install.yml
```

## Role Variables - cli_install
| Variable | Required | Type | Choices/Defaults | Comments |
|----------|----------|------|------------------|----------|
| `step_cert_version` | False | str | `latest` | The version (without `v`) of the step cli to install or `latest` for the latest release on GitHub. |
| `step_cert_cli_bin_path` | False | str | `/usr/bin/step` | The path to install `step` to. |

## Role Variables
| Variable | Required | Type | Choices/Defaults | Comments |
|----------|----------|------|------------------|----------|
| `step_cert_ca_url` | True | str | | The URL for the CA. It should start with `https://` |
| `step_cert_provisioner` | True | str | | The name of a JWK provisioner. |
| `step_cert_provisioner_pass` | True | str | | The password for the JWK provisioner. |
| `step_cert_certs` | True | list[dict] | | See [`<step_cert_certs>`](#step_cert_certs). |
| `step_cert_version` | False | str | `latest` | The version (without `v`) of the step cli to install or `latest` for the latest release on GitHub. |
| `step_cert_cli_bin_path` | False | str | `/usr/bin/step` | The path to install `step` to. |
| `step_cert_token_host` | False | str | `localhost` | The host to run `step ca token` on. |
| `step_cert_ssh_certs` | False | list[dict] | `[]` | See [`<step_cert_ssh_certs>`](#step_cert_ssh_certs). |
| `step_cert_ssh_id` | False | str | ``ansible_host`` | The default key ID of the SSH certificate. |
| `step_cert_ssh_principals` | False | list[str] | `[]` | List of default principals for SSH certificates. |
| `step_cert_ssh_not_after` | False | str | | The default time/duration of when the SSH ceritficate validity ends. If omitted or empty, the argument won&#39;t be passed. |
| `step_cert_ssh_renew_cmds` | False | list[str] | | Default list of commands to run after renewing the ssh certificate |
| `step_cert_ssh_renew_cmds_continue_on_failure` | False | bool | `True` | Run the next renew command if the previous one fails. |
| `step_cert_step_path` | False | str | `/etc/step` | The step configuration directory. |
| `step_cert_force` | False | bool | `False` | Overwrite any existing certificates with a new one. |
| `step_cert_ssh_force` | False | bool | `False` | Overwrite any existing SSH certificates with a new one. |
| `step_cert_renewal_timer` | False | str | `*:1/5` | systemd calendar expression for the renewal timer. |
| `step_cert_renewal_delay` | False | str | `5m` | Random delay for the renewal timer. |
| `step_cert_ssh_renewal_timer` | False | str | `*:1/5` | systemd calendar expression for the SSH renewal timer. |
| `step_cert_ssh_renewal_delay` | False | str | `5m` | Random delay for the SSH renewal timer. |

### `<step_cert_certs>`
A dict of certs to provision.
| Variable | Required | Type | Choices/Defaults | Comments |
|----------|----------|------|------------------|----------|
| `name` | True | str |  | Name for the certificate. Used to name the systemd service and timer. |
| `cn` | True | str |  | The CN of the cert. |
| `crt_file` | True | str |  | The path to the cert file. |
| `key_file` | True | str |  | The path to the key file. |
| `sans` | False | list[str] |  |  |
| `kty` | False | str | <ul><li>**EC**</li><li>OKP</li><li>RSA</li></ul> | The kty to build the certificate upon. |
| `renew_cmds` | False | list[str] |  | A list of commands to run after renewing the certificate |
| `renew_cmds_continue_on_failure` | False | bool | `True`  | Run the next renew command if the previous one fails. |
| `not_after` | False | str |  | The time/duration of when the ceritficate validity ends. If omitted, the argument won't be passed. |
| `owner` | False | str | `root`  |  |
| `group` | False | str | `root`  |  |
| `mode` | False | str | `0600`  |  |
| `crt_mode` | False | str | `0644`  |  |

### `<step_cert_ssh_certs>`
A dict of SSH certs to provision.
| Variable | Required | Type | Choices/Defaults | Comments |
|----------|----------|------|------------------|----------|
| `name` | True | str |  | Name for the certificate. Used to name the systemd service and timer. |
| `id` | False | str |  | The key ID of the SSH certificate |
| `file` | True | path |  | The path to the private key. If the private_key is `/etc/ssh/ssh_host_rsa_key`, this role expects the public key to have the name `/etc/ssh/ssh_host_rsa_key.pub` and will store the ssh certificate as `/etc/ssh/ssh_host_rsa_key-cert.pub`. |
| `principals` | False | list[str] | `[]`  | List of additional principals to add to the SSH certificate. |
| `not_after` | False | str |  | The time/duration of when the SSH ceritficate validity ends. If omitted or empty, the argument won't be passed. |
| `renew_cmds` | False | list[str] |  | A list of commands to run after renewing the certificate. Overrides `step_cert_ssh_renew_cmds`. |
| `renew_cmds_continue_on_failure` | False | bool |  | Run the next renew command if the previous one fails. Overrides `step_cert_ssh_renew_cmds_continue_on_failure`. |
| `owner` | False | str | `root`  |  |
| `group` | False | str | `root`  |  |
| `mode` | False | str | `0644`  |  |
