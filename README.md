# oauth2-proxy Ansible Role

[![lint](https://github.com/elan-ev/oauth2-proxy/actions/workflows/lint.yml/badge.svg?branch=main)](https://github.com/elan-ev/oauth2-proxy/actions/workflows/lint.yml?branch=main)
[![molecule](https://github.com/elan-ev/oauth2-proxy/actions/workflows/molecule.yml/badge.svg?branch=main)](https://github.com/elan-ev/oauth2-proxy/actions/workflows/molecule.yml?branch=main)

This Ansible role deploys [oauth2-proxy](https://oauth2-proxy.github.io/oauth2-proxy/). oauth2-proxy is a reverse proxy and static file server that provides authentication using Providers (Google, GitHub, and others) to validate accounts by email, domain or group. This role can be run multiple times with different configurations: for example, different settings can be applied to different domains.


## Dependencies

This Ansible module requires the `community.general` collection to be installed, which Ansible will install by default. The Python library `tomlkit` is also needed.

## Role Variables

- `oauth2_proxy_version`:
    - default: latest
    - oauth2-proxy release version, should match the exact git tag/release name including 'v' in front.
    Values are listed at the [oauth2-proxy release GitHub page](https://github.com/oauth2-proxy/oauth2-proxy/releases/).
    A special value `latest` will provide always the newest release version

- `oauth2_proxy_config`:
    - default: {}
    - Oauth2-proxy configurations as dictionary. You can find all configuration options
    in the [oauth2-proxy documentation](https://oauth2-proxy.github.io/oauth2-proxy/configuration/overview).

    example:
    ```yaml
    oauth2_proxy_config:
        http_address: "127.0.0.1:4181"
        redirect_url: https://{{ inventory_hostname }}/oauth2/callback
        email_domains: ["*"]
        provider: oidc
        oidc_issuer_url: https://idm.myedu.org
        skip_oidc_discovery: true
        login_url: https://idm.myedu.org/idp/profile/oidc/authorize
        redeem_url: https://idm.myedu.org/idp/profile/oidc/end-session
        oidc_jwks_url: https://idm.myedu.org/idp/profile/oidc/keyset
        client_id: myotherapp
        client_secret: other_secure_hash
        scope: openid email profile
        # cookie secret generated with
        # `python -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'`
        cookie_secret: g56ymUz4zvUKtVBdFW3rrE7zn_6ul3GNTT1tCjwP3s0==
        skip_provider_button: true
    ```
    Some configuration values are required and should be provided.
    Please consult [oauth2-proxy documentation](https://oauth2-proxy.github.io/oauth2-proxy/configuration/overview).

## Example Playbook

The following example shows a common OpenID Connect (OIDC) configuration to be used behind a reverse proxy such as nginx:

```yaml
- hosts: servers
  become: true
  roles:
    - role: elan.oauth2_proxy
      oauth2_proxy_config:
        redirect_url: https://{{ inventory_hostname }}/oauth2/callback
        email_domains: ["*"]
        provider: oidc
        oidc_issuer_url: https://idm.myedu.org
        client_id: myotherapp
        client_secret: other_secure_hash
        scope: openid email profile
        # cookie secret generated with
        # `python -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'`
        cookie_secret: g56ymUz4zvUKtVBdFW3rrE7zn_6ul3GNTT1tCjwP3s0==
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
