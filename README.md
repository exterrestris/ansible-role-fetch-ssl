# Ansible role `exterrestris.fetch_ssl`

An Ansible role for setting up a cron job to fetch an SSL certificate via SSH and copy it to specified locations for reuse.

## Requirements

- SSH access to certificate source using password-less public key authentication
- Full certificate chain in PEM format
- Private key in PEM format
## Role Variables

### Configuration
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `fetch_ssl_ssh_host` | *Required* | Hostname/IP address from where to fetch the certificate |
| `fetch_ssl_ssh_user` | *Required* | Username to use on `fetch_ssl_ssh_host` |
| `fetch_ssl_certs` | `[]` | List of certificates to fetch and install |
| `fetch_ssl_run_now` | `no` | Run script immediately |

#### `fetch_ssl_certs[]`
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `cert` | *Required* | Path to the full certificate chain in PEM format |
| `key` | *Required* | Path to the private key in PEM format |
| `install_to` | `[]` | List of paths to copy certificates to |
| `send_to` | `[]` | List of remote paths to copy certificates to |

#### `fetch_ssl_certs[].install_to[]`
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `cert` | *Required* | Local destination path for the full certificate chain |
| `key` | *Required* | Local destination path for the private key |

#### `fetch_ssl_certs[].send_to[]`
| Variable | Default | Comments |
| :--- | :--- | :--- |
| `host` | *Required* | Remote host to send the certificate to |
| `user` | *Required* | Username to use on `host` |
| `cert` | *Required* | Remote destination path for the full certificate chain |
| `key` | *Required* | Remote destination path for the private key |

#### Script

| Variable | Default | Comments |
| :--- | :--- | :--- |
| `fetch_ssl_script_dir` | `"/usr/local/sbin/"` | Install script into directory |
| `fetch_ssl_script_name` | `"install-ssh-cert.sh"` | File name for script |
| `fetch_ssl_script_owner` | `"root"` | Owner for script |
| `fetch_ssl_script_group` | `"root"` | Group for script |
| `fetch_ssl_script_mode` | `"0700"` | Mode/permissions for script |
| `fetch_ssl_tmp_cert` | `"/tmp/fetch_ssl_tmp_cert"` | Temp file for certificate chain. Will be reused for each certificate |
| `fetch_ssl_tmp_key` | `"/tmp/fetch_ssl_tmp_key"` | Temp file for private key. Will be reused for each private key |
| `fetch_ssl_cron_name` | `'install ssl certificate'` | Name of cron job |
| `fetch_ssl_cron_file` | `'install-ssl-certificate'` | Filename for cron job |
| `fetch_ssl_cron_frequency` | `"daily"` | Cron frequency |
| `fetch_ssl_generate_ssh_key` | `yes` | Generate an SSH key for script to use | 
| `fetch_ssl_generated_ssh_key` | `"id_fetch_ssl"` | Name of generated SSH key | 
| `fetch_ssl_ssh_key` | `/root/.ssh/{`*`fetch_ssl_generated_ssh_key`*`\|id_rsa}` | Path to SSH key for script to use |
| `fetch_ssl_ssh_key_install_remote` | `yes` | Install specified SSH key on remote hosts. Requires remote hosts to be defined in inventory |

## Example - Installing Let's Encrypt wildcard certificate issued to pfSense

pfSense can automatically request and install a Let's Encrypt certificate using the [ACME](https://docs.netgate.com/pfsense/en/latest/packages/acme/index.html) package. If you request a wildcard certificate, you can reuse this certificate on other devices instead of dealing with managing additional certificates on a per-device basis

The ACME package saves the issued certificate in a variety of formats, including the full chain required by Proxmox

### Requirements

- ACME package configured within pfSense to issue and automatically renew a wildcard certificate
- The `Write Certificates` option within `Services / Acme / Settings / General Settings` enabled to copy the issued certificates to the `/conf/acme/` directory
- SSH access to pfSense using public-key authentication for a user with read access to `/conf/acme/`

### Configuration

```Yaml
fetch_ssl_ssh_host: "pfsense-host"
fetch_ssl_ssh_user: admin
fetch_ssl_certs:
  - cert: "/conf/acme/certificate-name.fullchain"
    key : "/conf/acme/certificate-name.key"
    install_to:
      - cert: /copy/certificate/to/path
        key: /copy/key/to/path
fetch_ssl_run_now: yes
```

Replace `pfsense-host` with the hostname or IP of your pfSense router, and `certificate-name` with the name you gave your certficate when configuring it within the ACME module