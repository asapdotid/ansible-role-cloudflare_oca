<p align="center"> <img src="https://user-images.githubusercontent.com/34257858/129839002-15e3f2c7-3f75-46d4-afae-0fd207d7fdde.png" width="100" height="100"></p>

<h1 align="center">
    Ansible Role Cloudflare Origin CA
</h1>

<p align="center" style="font-size: 1.2rem;">
    This ansible role install CloudFlare Origin CA certificate On Ubuntu, CentOS.
</p>

Short answer: To keep connection between Cloudflare and your severs private and secure from tampering.

Long answer:

> Cloudflare’s Flexible SSL mode is the default for Cloudflare sites on the Free plan. Flexible SSL mode means that traffic from browsers to Cloudflare will be encrypted, but traffic from Cloudflare to a site's origin server will not be. To take advantage of our [Full and Strict SSL](https://www.cloudflare.com/ssl) mode—which encrypts the connection between Cloudflare and the origin server—it’s necessary to install a certificate on the origin server.
>
> Cloudflare Blog - [Origin Server Connection Security with Universal SSL](https://blog.cloudflare.com/origin-server-connection-security-with-universal-ssl/)

### What are the benefits of Cloudflare Origin CA over Let's Encrypt?

To get certificates from [Let's Encrypt](https://letsencrypt.org/), you have to first disable Cloudflare because Cloudflare hides actual server IPs and make Let's Encrypt challenges fail. Using Cloudflare Origin CA simplifies the troubles.

### What are the benefits of Cloudflare Origin CA over other public certificates?

See [Introducing Cloudflare Origin CA](https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#whataretheincrementalbenefitsoforigincaoverpubliccertificates) on Cloudflare blog.

### include Setup nginx include configuration with CloudFlare CA root certificates

`/etc/nginx/includes.d/cloudflare-origin-ca.conf`

include above file to your server block.

```bash
# Cloudflare Origin CA
ssl_certificate /etc/ssl/cloudflare/example.com.pem;
ssl_certificate_key /etc/ssl/cloudflare/example.com.key;
ssl_client_certificate /etc/ssl/cloudflare/origin_ca_rsa_root.pem;
ssl_verify_client on;
```

## Requirements

None

## Role Variables

| Name                            | Default Value | Description                             |
| ------------------------------- | ------------- | --------------------------------------- |
| `cfca_cloudflare_origin_ca_key` | `""`          | CloudFlare origin CA key.               |
| `cfca_origin_ca_root_type`      | `rsa`         | CloudFlare CA root type `rsa` or `ecc`. |
| `cfca_origin_ca_sites_config`   | `[]`          | CloudFlare CA sites config.             |

## Dependencies

None.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
  vars:
    # better secret key use vault encryption
    cfca_cloudflare_origin_ca_key: v1.0-xxxxxxxxxxx
    cfca_origin_ca_sites_config:
      - name: example.com
        days: 7
        hostnames:
          - example.com
          - "*.example.com"
        key_size: 4096
        key_type: rsa
        overwrite: false

  roles:
    - { role: asapdotid.cloudflare_oca }
```

Detail variables:

```yaml
# Cloudflare Origin CA Key
# Not to confuse with Cloudflare Global API Key
# See: https://blog.cloudflare.com/cloudflare-ca-encryption-origin/#iiobtainyourcertificateapitoken
cfca_cloudflare_origin_ca_key: v1.0-xxxxxxxxxxx

# Indicates the desired package state.
# `latest` ensures that the latest version is installed.
# `present` does not update if already installed.
# Choices: present|latest
# Default: latest
cfca_package_state: present

# Whether to hide results of sensitive tasks which
# may include Cloudflare Origin CA Key in plain text.
# Choices: true|false
# Default: false
cfca_origin_ca_no_log: true

# Site config use CloudFlare Origin CA certificate
cfca_origin_ca_sites_config:
  - name: example.com
    # Number of days for which the issued cert will be valid. Acceptable options are: 7, 30, 90, 365 (1y), 730 (2y), 1095 (3y),
    # 5475 (15y - default)
    days: 7
    # List of fully-qualified domain names to include on the certificate as Subject Alternative Names.
    # Default: All canonical and redirect domains
    # In the above example: example.com, hi.example.com, hello.another-example.com
    hostnames:
      - example.com
      - "*.example.com"
      - "*.another-example.com"
    # Key size in bits to use for the generated key pair (Acceptable sizes: rsa: 2048|3072|4096, ecdsa: 256|384|521) (default 2048)
    key_size: 4096
    # Type of key pair to generate, either RSA or ECDSA. (rsa|ecdsa) (default "rsa")
    key_type: rsa
    # Overwrite the existing cert and/or key file
    overwrite: false
```

## License

MIT / BSD

## Author Information

[JogjaScript](https://jogjascript.com)

This role was created in 2021 by [Asapdotid](https://github.com/asapdotid).
