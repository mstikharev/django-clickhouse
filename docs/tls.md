## TLS configuration

This library supports TLS (including mutual TLS) via the underlying `infi.clickhouse_orm` fork with TLS support.

- Upstream TLS docs: [infi.clickhouse_orm TLS guide](https://github.com/mstikharev/infi.clickhouse_orm/blob/develop/docs/tls.md)

### What is supported

- HTTPS transport to ClickHouse (`https://` in `db_url`)
- Server certificate verification with custom CA bundle
- Optional client certificate authentication (mTLS)

Parameters understood and forwarded to the ORM:
- `verify_ssl_cert` — boolean; verify server certificate (recommended)
- `ca` — path to CA bundle file for server verification
- `client_cert` — path to client certificate (PEM)
- `client_key` — path to client private key (PEM)

You can provide these either directly in the database config, or as query parameters in the `db_url`.

### Configure via direct parameters

Add TLS fields to your `CLICKHOUSE_DATABASES` entry:

```python
CLICKHOUSE_DATABASES = {
    'default': {
        'db_name': 'default',
        'db_url': 'https://clickhouse.example.com:8443',
        'username': 'ch_user',
        'password': '***',
        'verify_ssl_cert': True,
        'ca': '/etc/ssl/certs/ca.pem',
        'client_cert': '/etc/ssl/certs/client.crt',
        'client_key': '/etc/ssl/private/client.key',
    },
}
```

Notes:
- Use `https://` in `db_url` to enable TLS.
- If ClickHouse uses a private/self-signed CA, set `ca` to the CA file.
- For mutual TLS, provide both `client_cert` and `client_key`.

### Configure via connection string parameters

You can also pass TLS options as query params on `db_url`:

```python
CLICKHOUSE_DATABASES = {
    'default': {
        'db_name': 'default',
        'db_url': (
            'https://clickhouse.example.com:8443/'
            '?verify_ssl_cert=1'
            '&ca=/etc/ssl/certs/ca.pem'
            '&client_cert=/etc/ssl/certs/client.crt'
            '&client_key=/etc/ssl/private/client.key'
        ),
        'username': 'ch_user',
        'password': '***',
    },
}
```

Notes:
- Boolean values like `verify_ssl_cert` accept `1/0`, `true/false`.
- URL-encode values if paths contain special characters.

### Troubleshooting

- Certificate verify failed: provide a proper `ca` bundle, and keep `verify_ssl_cert=True`.
- Connection works without TLS but fails with HTTPS: ensure the ClickHouse HTTP interface is configured for TLS and the port is correct.
- Client auth required: ensure both `client_cert` and `client_key` are readable by the application.

### Security recommendations

- Keep `verify_ssl_cert` enabled; avoid disabling verification in production.
- Restrict permissions on `client_key` files.
- Prefer CA-signed server certificates; rotate certificates before expiry.


