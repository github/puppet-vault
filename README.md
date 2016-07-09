# puppet-vault

[![Build Status](https://travis-ci.org/rhoml/puppet-vault.svg?branch=master)](https://travis-ci.org/rhoml/puppet-vault)
[![Puppet
Forge](http://img.shields.io/puppetforge/v/rhoml/vault.svg)](https://forge.puppetlabs.com/rhoml/vault)
[![Dependency Status](https://gemnasium.com/badges/github.com/rhoml/puppet-vault.svg)](https://gemnasium.com/github.com/rhoml/puppet-vault)

# Overview

This is a puppet module to install Hashicorp's [vault project](https://www.vaultproject.io) to keep your secrets safe. This module doesn't build the Vault packages which should be pretty easy to do using fpm.

Documentation for Vault can be found on their [site](https://www.vaultproject.io/docs/config/index.html). Take into consideration:
* You can only define one storage backend, listener and telemetry on the config file.
* Other configurations should be set up using Vault API or CLI.

# Install Vault

````
include ::vault
````

# Configure Vault using Hiera

This module enables you to use hiera to configure your Vault server. It also allows you to use module [data](https://github.com/rhoml/puppet-vault/blob/master/data/common.yaml).

````
vault::config_hash:
    backend:
        consul:
            address: '127.0.0.1:8500'
            advertise_addr: "http://%{::ipaddress_eth0}"
            path: 'vault/'
    listener:
        tcp:
            address: "%{::fqdn}:8200"
            tls_disable: true
    telemetry:
        statsite_address: '127.0.0.1:8125'
        disable_hostname: true
    disable_mlock: true
vault::manage_user: true
vault::package_ensure: 'latest'
vault::vault_user: 'vault'
vault::restart_cmd: '/etc/init.d/vault restart'
````

To run multiple listeners, for example disabling TLS on 127.0.0.1, but requiring TLS from external hosts
````
    listener:
        - tcp:
            address: "127.0.0.1:8200"
            tls_disable: true
        - tcp:
            address: "%{::fqdn}:8200"
            tls_cert_file: "/path/to/cert.pub"
            tls_key_file:  "/path/to/private.key"
````

[Vault Enterprise customers using a PKCS#11 HSM](https://atlas.hashicorp.com/help/vault/hsm/configuration) might do...
```
    hsm:
        pkcs11:
            lib: "/path/to/libpkcs11.so"
            slot: "0"
            key_label: "vault"
            pin: "Goofus commits secrets to repos. Gallant uses $VAULT_HSM_PIN"
```
The puppet-vault module uses a SysV init script. Those wishing to avoid putting the PIN in Hiera in plaintext could, for example, create an `/etc/default/vault`, owned by root and only readable by root, looking like...
```
export VAULT_HSM_PIN="correct horse battery staple"
```

# Uninstalling Vault

Ensure the following hiera key is present so Vault can be correctly uninstalled

```
vault::package_ensure: absent
```

# See also

* [hiera-vault](https://github.com/jsok/hiera-vault)
* [consul](https://github.com/solarkennedy/puppet-consul)
