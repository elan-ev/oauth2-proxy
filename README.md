# oauth2-proxy Ansible Role

[![lint](https://github.com/elan-ev/oauth2-proxy/actions/workflows/lint.yml/badge.svg?branch=main)](https://github.com/elan-ev/oauth2-proxy/actions/workflows/lint.yml?branch=main)
[![molecule](https://github.com/elan-ev/oauth2-proxy/actions/workflows/molecule.yml/badge.svg?branch=main)](https://github.com/elan-ev/oauth2-proxy/actions/workflows/molecule.yml?branch=main)

This Ansible role deploys [oauth2-proxy](https://oauth2-proxy.github.io/oauth2-proxy/). oauth2-proxy is a reverse proxy and static file server that provides authentication using Providers (Google, GitHub, and others) to validate accounts by email, domain or group. You can run this role multiple times with different configurations, e.g. one per domain.


## Dependencies

No dependencies required.

## Role Variables

- `oauth2_proxy_version`:
    - default: latest
    - oauth2-proxy release version, should match the exact git tag/release name including 'v' in front.
    Values are listed at the [oauth2-proxy release GitHub page](https://github.com/oauth2-proxy/oauth2-proxy/releases/).
    A special value `latest` will provide always the newest release version
- `oauth2_proxy_http_address`:
    - default: "127.0.0.1:4180"
    - `[http://]<addr>:<port>` or `unix://<path>` or `fd:<int>` (case insensitive) to listen on for HTTP clients.
    Unix sockets are created with default system umask mode, which can be overridden,
    e.g. `unix://my-socket,mode=0777`. Square brackets are required for ipv6 address, e.g. `http://[::1]:4180`
- `oauth2_proxy_reverse_proxy`:
    - default: false
    - Are we running behind a reverse proxy, controls whether headers like X-Real-IP are accepted and
    allows X-Forwarded-{Proto,Host,Uri} headers to be used on redirect selection
- `oauth2_proxy_trusted_proxy_ips`:
    - default: ["0.0.0.0/0", "::/0"]
    - List of direct proxy IPs or CIDR ranges allowed to supply `X-Forwarded-*` headers when `oauth2_proxy_reverse_proxy` is enabled.
    Include every trusted proxy hop that can appear in XFF.
    If not set, OAuth2 Proxy preserves backwards compatibility by trusting all source IPs (0.0.0.0/0, ::/0) and logs a warning at startup.
- `oauth2_proxy_redirect_url`:
    - default: "https://{{ inventory_hostname }}/oauth2/callback"
    - The OAuth Redirect URL
- `oauth2_proxy_service_domain`:
    - default: `<hostname of oauth2_proxy_redirect_url>[_<port of oauth2_proxy_redirect_url>]`
    - This variable is used internally to separate multiple instances of oauth2-proxy. The oauth2-proxy configuration name, log file name and systemd service name will contain this value. The default value should be fine and is not intended to be changed by user. But in some rare cases, where you have naming conflicts, you can set it to a different value.
- `oauth2_proxy_email_domains`:
    - default: ["*"]
    - Authenticate emails with the specified domains. Use * to authenticate any email
- `oauth2_proxy_client_id`:
    - required: true
    - The OAuth Client ID
- `oauth2_proxy_client_secret`:
    - required: true
    - The OAuth Client Secret
- `oauth2_proxy_scope`:
    - default: "openid email profile"
    - OAuth scope specification. Every provider has a default list of scopes which will be used in case no scope is configured.
- `oauth2_proxy_cookie_name`:
    - default: "_oauth2_proxy"
    - The name of the cookie that the oauth_proxy creates.
- `oauth2_proxy_cookie_secret`:
    - required: true
    - The seed string for secure cookies (optionally base64 encoded)
- `oauth2_proxy_cookie_domains`:
    - default: ""
    - Optional cookie domains to force cookies to (e.g. .yourcompany.com).
    The longest domain matching the request's host will be used (or the shortest cookie domain if there is no match).
- `oauth2_proxy_cookie_expire`:
    - default: "168h"
    - Expire timeframe for cookie. If set to 0, cookie becomes a session-cookie which will expire when the browser is closed.
- `oauth2_proxy_cookie_refresh`:
    - default: "0"
    - Refresh the cookie after this duration; 0 to disable; not supported by all providers
- `oauth2_proxy_extra_config`: 
    - default: {}
    - Additional configurations as dictionary. You can find all configuration options
    in the [oauth2-proxy documentation](https://oauth2-proxy.github.io/oauth2-proxy/configuration/overview). Example:
    ```yaml
    oauth2_proxy_extra_config:
        approval_prompt: "auto"
        oidc_issuer_url: "https://idp.edu.org"
        skip_oidc_discovery: false
        skip_provider_button: true
    ```

## Example Playbook

A minimal example to use oauth2-proxy as OAuth proxy behind nginx:

```yaml
- hosts: servers
  become: true
  roles:
    - role: elan.oauth2_proxy
      # required arguments
      oauth2_proxy_client_id: myapp
      oauth2_proxy_client_secret: specific_oauth_client_secret
      # cookie secret generated with 
      # `python -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'`
      oauth2_proxy_cookie_secret: g56ymUz4zvUKtVBdFW3rrE7zn_6ul3GNTT1tCjwP3s0=
      # additional arguments
      oauth2_proxy_extra_config:
        provider: oidc
        provider_display_name: "My OIDC Provider"
        oidc_issuer_url: https://idp.myedu.org/oauth2/default
        skip_provider_button: true

```

# Development Environment

For linting and role development you can use the tools defined in [development requirements](.dev_requirements.txt).
You can quickly install them in a python virtual environment like this:

```sh
# Create a virtual environment
python -m venv venv
# Activate the virtual environment
. venv/bin/activate
# Install the dependencies
pip install -r .dev_requirements.txt
```

This way, you can run the linter (`ansible-lint`).

For development and testing you can use [molecule](https://molecule.readthedocs.io/en/latest/).
With podman as driver you can install it like this – preferably in a virtual environment:

```bash
pip install -r .dev_requirements.txt
```

Then you can *create* the test instances, apply the ansible role (*converge*) and *destroy* the test instances with these commands:

```bash
molecule create
molecule converge
molecule destroy
```

Or simply run `molecule test` to do all steps at once.

If you want to inspect a running test instance, use `molecule login --host <instance_name>` where you replace `<instance_name>` with the desired value.
