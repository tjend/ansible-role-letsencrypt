# Ansible Role: ansible-role-letsencrypt

This role installs and configures the lego letsencrypt client to create https certificates for integration with nginx. Both DNS-01 and HTTP-01 letsencrypt verification can be used.

Note that if the nginx snippet will serve both IPv6 and IPv4, you should use the "ipv6only=false" parameter in your default vhost - see https://stefanchrist.eu/blog/2015_01_21/Using%20ipv6only%20in%20Nginx.xhtml and http://nginx.org/en/docs/http/ngx_http_core_module.html#listen.

If you don't provide the account json and key in the 'letsencrypt_accounts' variable, a new account json and key files will be created. You can then reuse the newly created account json and key by adding them to the 'letsencrypt_accounts' variable.

# Example of using dns verification
    roles:
    - ansible-role-letsencrypt # put after nginx

    vars:
      letsencrypt_accounts:
        - account: letsencrypt_account1@domain1.com
          cloudflare_email: <cloudflare email address>
          cloudflare_api_key: <cloudflare api key>
          json: |
            {
              "email": "letsencrypt_account1@domain1.com",
              "registration": {
                ...
                "terms_of_service": "https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf"
              }
            }
          key: |
            -----BEGIN EC PRIVATE KEY-----
            ...
            -----END EC PRIVATE KEY-----
        - account: letsencrypt_account2@domain2.com
      letsencrypt_certificates:
        - account: letsencrypt_account1@domain1.com
          domains: [ primary-domain.com, other-domains.com ]
        - account: letsencrypt_account2@domain2.com
          domains: [ primary-domain2.com, other-domains2.com ]

      nginx_vhosts:
        extra_parameters: |
          # include letsencrypt https config(the wildcard allows nginx to start before letsencrypt has run)
          include snippets/primary-domain.com.https.conf*;

# Example of using http verification
    roles:
    - ansible-role-letsencrypt # put after nginx

    vars:
      letsencrypt_accounts:
        - account: letsencrypt_account1@domain1.com
          json: |
            {
              "email": "letsencrypt_account1@domain1.com",
              "registration": {
                ...
                "terms_of_service": "https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf"
              }
            }
          key: |
            -----BEGIN EC PRIVATE KEY-----
            ...
            -----END EC PRIVATE KEY-----
      letsencrypt_certificates:
      - account: letsencrypt_account1@domain1.com
        domains: [ primary-domain.com, other-domains.com ]
        http_verification: true

      nginx_vhosts:
        extra_parameters: |
          # include acme .well-known config(the wildcard allows nginx to start before letsencrypt has run)
          include snippets/acme_well_known.conf*;
          # include letsencrypt https config(the wildcard allows nginx to start before letsencrypt has run)
          include snippets/primary-domain.com.https.conf*;
