OULibraries.tls-cert
=========

Utility role for wrangling TLS certificates. eg. letsencrypt, olde-style CSRs, and self-signed certs.
Places certificates in local vaultfiles so other roles may decrypt and push them out to remote hosts.

Things get placed in
```
{{ tls_cert_path }}/{{ item.common_name }}
```

All certs start off the same way, by generating a private key and a CSR. They diverge from there.
Everybody gets
```
{{ tls_cert_path }}/{{ item.common_name }}/privkey.vault.yml
{{ tls_cert_path }}/{{ item.common_name }}/request.csr
```

* self-signed runs x509 to wrap things up immediately, with a self-signed certificate at:
```
{{ tls_cert_path }}/{{ item.common_name }}/fullchain.pem
```

* olde-style expects you to place the leaf certificate like so:
```
{{ tls_cert_path }}/{{ item.common_name }}/cert.pem
```

and then rerun the role to generate the chain files.

* acme expects you to have a remote server already configured to:
  1.  serve out an acme challenge from the specified hostname on port 80
  2.  allow you to rsync a file over directly to the challenge directory with no interaction

  it will:
  1. Create an acme account key if you don't have one already
  2. request a challenge
  3. push the file containing the answer to the remote server
  4. respond to the challenge
  5. write out the leaf certificate

By default it points to the letsencrypt staging infrastructure,
so you'll probably want to point it at something that will actually
sign certs for you, such as the letsencrypt production infrastructure:
```
https://acme-v01.api.letsencrypt.org/directory
```

tls-cert uses the same process to generate chainfiles for acme and old-style
certificates.

It will follow the chain of trust from the leaf cert
and download each intermediate along the way. It will then convert each
intermediate certificate to pem-formatted file. It will then concatenate
the intermediate certificates into a chain file. It will then concatenate the
chain file and the leaf certificate into a full chain, resulting in the
follwing files of immediate interest:
```
{{ tls_cert_path }}/{{ item.common_name }}/privkey.vault.yml
{{ tls_cert_path }}/{{ item.common_name }}/cert.pem
{{ tls_cert_path }}/{{ item.common_name }}/chain.pem
{{ tls_cert_path }}/{{ item.common_name }}/fullchain.pem
```

It also leaves some less-interesting artifacts that we don't clean out because
their presence allows ansible to skip multiple steps that would be executed for
each certificate on every run.
```
{{ tls_cert_path }}/{{ item.common_name }}/chain.*.crt
{{ tls_cert_path }}/{{ item.common_name }}/chain.*.pem
```

Requirements
------------

* python >= 2.6
* openssl
* ansible >= 2.2

Role Variables
--------------

```
tls_cert_path: where all the magic happens. defaults to "{{ role_path }}/../../tls_certs" which works out to tls_certs living next to roles dir.
tls_cert_acme_remaining_days: How close to expiration to you wish to renew? default is 10 days
tls_cert_country:
tls_cert_state:
tls_cert_locality:
tls_cert_organization:
tls_cert_department:
tls_cert_acme_account_email: specifically for acme cert handling


tls_certificates:
  - common_name: localhost
    type: self-signed
    alt_names:
      - localhost.localdomain
      - vagrant.localdomain
  - common_name: yoursite.example.com
    type: olde-style
    alt_names:
      - "*.yoursite.example.com"
  - common_name: freecerts.example.com
    type: acme
    acme_dir: 'https://acme-staging.api.letsencrypt.org/directory'
    alt_names:
      - freecerts1.example.com
      - freecerts2.example.com
    dest_host: 'ssh-accessible.from.your.shell.example.com'
    dest_acme_challenge_dir: '/srv/certbot' # or wherever your acme challenge dir lives
```


Dependencies
------------

TBD

Example Playbook
----------------

TBD

Troubleshooting
---------------
It can sometimes be useful to use openssl to examine your certificates and CSRs
to verify that you're getting what you request, especially with SANs.

Here's an example of checking the info on a CSR for localhost.

```
openssl req -text -noout -in tls_certs/localhost/fullchain.pem
```
Here's an example of checking the info on a certificate for localhost.

```
openssl x509 -text -noout -in tls_certs/localhost/fullchain.pem
```


License
-------

[MIT](LICENSE)

Author Information
------------------

Jason Sherman
