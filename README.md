# Ansible Role: ansible-role-letsencrypt

This role installs and configures the lego letsencrypt client to create https certificates for integration with nginx. Both DNS-01 and HTTP-01 letsencrypt verification can be used.

Note that if the nginx snippet will serve both IPv6 and IPv4, you should use the "ipv6only=false" parameter in your default vhost - see https://stefanchrist.eu/blog/2015_01_21/Using%20ipv6only%20in%20Nginx.xhtml and http://nginx.org/en/docs/http/ngx_http_core_module.html#listen.

If you don't provide the account key in the 'letsencrypt_accounts' variable, a new account key will be created. You can then reuse the newly created account key by adding them to the 'letsencrypt_accounts' variable.

# Example of using dns verification
    roles:
    - ansible-role-letsencrypt # put after nginx

    vars:
      letsencrypt_accounts:
        - account_email: <letsencrypt account email>
          account_number: <letsencrypt account number>
          cloudflare_email: <cloudflare email address>
          cloudflare_api_key: <cloudflare api key>
          key: |
            -----BEGIN EC PRIVATE KEY-----
            ...
            -----END EC PRIVATE KEY-----
      letsencrypt_certificates:
        - account: <letsencrypt account email>
          domains: [ primary-domain.com, other-domains.com ]

      nginx_vhosts:
        extra_parameters: |
          # include letsencrypt https config(the wildcard allows nginx to start before letsencrypt has run)
          include snippets/primary-domain.com.https.conf*;

# Example of using http verification
    roles:
    - ansible-role-letsencrypt # put after nginx

    vars:
      letsencrypt_accounts:
        - account_email: <letsencrypt account email>
        - account_number: <letsencrypt account number>
          key: |
            -----BEGIN EC PRIVATE KEY-----
            ...
            -----END EC PRIVATE KEY-----
      letsencrypt_certificates:
        - account: <letsencrypt account email>
          domains: [ primary-domain.com, other-domains.com ]
          http_verification: true

      nginx_vhosts:
        extra_parameters: |
          # include acme .well-known config(the wildcard allows nginx to start before letsencrypt has run)
          include snippets/acme_well_known.conf*;
          # include letsencrypt https config(the wildcard allows nginx to start before letsencrypt has run)
          include snippets/primary-domain.com.https.conf*;
