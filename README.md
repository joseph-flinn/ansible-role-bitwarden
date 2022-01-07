# Ansible Role: Bitwarden

[![CI](https://github.com/e-breuninger/ansible-role-bitwarden/actions/workflows/ci.yml/badge.svg)](https://github.com/e-breuninger/ansible-role-bitwarden/actions/workflows/ci.yml)

Deploy Bitwarden with Docker and Docker-Compose using the `bitwarden.sh`.

This role is an automated wrapper around the Bitwarden setup scripts. 
It makes heavy use of handlers to trigger reconfigure and update tasks. 

If you need any task not covered by the role it's totally fine to use the setup script on the machine directly.
Use the official docs as reference: https://bitwarden.com/help/article/install-on-premise/

## Table of content

* [Usage](#Usage)
* [Default Variables](#default-variables)
  * [bitwarden_domain_name](#bitwarden_domain_name)
  * [bitwarden_global_env](#bitwarden_global_env)
  * [bitwarden_ssl_mode](#bitwarden_ssl_mode)
  * [bitwarden_nginx_cert_path](#bitwarden_nginx_cert_path)
  * [bitwarden_nginx_key_path](#bitwarden_nginx_key_path)
  * [bitwarden_lets_encrypt_email](#bitwarden_lets_encrypt_email)
  * [bitwarden_setup_config](#bitwarden_setup_config)
  * [bitwarden_version](#bitwarden_version)
  * [bitwarden_test_install_script](#bitwarden_test_install_script)
* [Dependencies](#dependencies)
* [Known Issues](#known-issues)
* [License](#license)
* [Author](#author)

---

## Usage

The role is currently not in Ansible Galaxy due to an issue of connecting out Github organisation with Galaxy.
As soon this is fixed you will be able to use it via Galaxy. 

Add the following to your `requirements.yml`

```YAML
roles:
  - src: git+https://github.com/e-breuninger/ansible-role-bitwarden.git
    name: breuninger.bitwarden
    version: 0.1.0
```

Add the role to your playbook:

```YAML
- hosts: server
  roles:
    - { role: breuninger.bitwarden }
```

### SSL Modes

The Bitwarden setup script allows for four different ways of setting up SSL (or the lack thereof): a user provided SSL
cert, an SSL cert that is created by Let's Encrypt, a self-signed cert generated by the setup container, and no SSL (not
recommended for installs being used normally). 

#### User Provided SSL

To maintain backwards compatibility, this is the default mode for this role. While the Bitwarden setup script allows for
untrusted certs provided by the user, this role requires it to be trusted (signed by a CA, not self signed). 

```YAML
- hosts: server
  roles:
    - role: breuninger.bitwarden
      vars:
        bitwarden_ssl_mode: provided
        bitwarden_nginx_cert_path: /path/to/ssl/cert
        bitwarden_nginx_cert_key: /path/to/ssl/key
```

If an untrusted-user-provided-cert usecase is needed, it can be added with a new ssl_mode and corresponding inputs in 
`defaults/main.yml`.


#### Let's Encrypt SSL

Use the Certbot SSL integration that comes with the Bitwarden setup script

```YAML
- hosts: server
  roles:
    - role: breuninger.bitwarden
      vars:
          bitwarden_ssl_mode: lets_encrypt
          bitwarden_lets_encrypt_email: alice@email.com
```

#### Generated Self-signed Cert 

The Bitwarden setup script allows for generating a self-signed SSL cert to utilize SSL, but from an untrusted source.
The two methods above are better for running Bitwarden in a Production environment. Please choose from one of them
instead of using this option, unless absolutely necessary.

```YAML
- hosts: server
  roles:
    - role: breuninger.bitwarden
      vars:
        bitwarden_ssl_mode: generate
```

#### No SSL

Please heavily consider your use case before using this option. One legitimate usecase for this is SSL termination at a
reverse proxy.

```YAML
- hosts: server
  roles:
    - role: breuninger.bitwarden
      vars:
        bitwarden_ssl_mode: disable
```

## Default Variables

### bitwarden_domain_name

Domain name witch used for hole bitwarden setup

#### Default value

```YAML
bitwarden_domain_name: localhost
```

### bitwarden_global_env

Map of global Bitwarden environment variables. Each entire is mapped to the global.override.env. See https://bitwarden.com/help/article/environment-variables/

#### Default value

```YAML
bitwarden_global_env: {}
```

#### Example usage

```YAML
bitwarden_global_env:
  globalSettings__mail__smtp__host: localhost
  globalSettings__mail__smtp__port: 25
```

### bitwarden_ssl_mode

Mode of SSL to use while installing (required). Options: `provided`, `generate`, `lets_encrypt`, `disable`.

#### Default value

```YAML
bitwarden_ssl_mode: provided
```

### bitwarden_nginx_cert_path

Path of the certificate file used for the Nginx container (required if `bitwarden_ssl_mode == "provided"`). The user of the role is responsible for providing a valid certificate file. File is copied from the provided location to Bitwardens user home in order to garantue the correct mapping inside the container.

#### Default value

```YAML
bitwarden_nginx_cert_path:
```

### bitwarden_nginx_key_path

Path of the key file used for the Nginx container (required if `bitwarden_ssl_mode == "provided"`). The user of the role is responsible for providing a valid key file. File is copied from the provided location to Bitwardens user home in order to garantue the correct mapping inside the container.

#### Default value

```YAML
bitwarden_nginx_key_path:
```

### bitwarden_lets_encrypt_email

Email for Let's Encrypt account. Used by Let's Encrypt to communicate cert expiration notifications

#### Default value

```YAML
bitwarden_lets_encrypt_email: 
```

### bitwarden_setup_config

Map of Bitwarden setup configuration values to override. Use this to change values in the generated config.yml file from Bitwarden.

#### Default value

```YAML
bitwarden_setup_config: {}
```

#### Example usage

```YAML
bitwarden_setup_config:
  database_docker_volume: true
```

### bitwarden_version

#### Default value

```YAML
bitwarden_version: v1.43.0
```

### bitwarden_test_install_script

A flag to disable downloading the `bitwarden.sh` script. Used in cases where the Let's Encrypt ssl_mode needs to be 
tested without fear of hitting the Let's Encrypt rate limit. Or to test changes to the `bitwarden.sh` or `run.sh`
scripts. Hopefully this flag can be added to the `bitwarden.sh` script in the future instead of being used here.

#### Default value

```YAML
bitwarden_test_install_script: false
```

## Dependencies

None.

## Known issues

### Bitwarden version

Bitwarden has a different version in the setup files than in the tagged version of the repo may indicates.
This is due to their release strategy, which always increases the actual version only in the master. We are already in talks with Bitwarden and hope for a different mode of release.

Install and configure bitwarden on premise in docker-compose fashion.


## License

MIT

## Author

Marco Frese <marco.frese@breuninger.de>
