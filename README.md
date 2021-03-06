# sssd

[![Build Status](https://travis-ci.org/sgnl05/sgnl05-sssd.svg)](https://travis-ci.org/sgnl05/sgnl05-sssd)
[![Puppet Forge](https://img.shields.io/puppetforge/v/sgnl05/sssd.svg)](https://forge.puppetlabs.com/sgnl05/sssd)
[![Puppet Forge Downloads](https://img.shields.io/puppetforge/dt/sgnl05/sssd.svg)](https://forge.puppetlabs.com/sgnl05/sssd)
[![Puppet Forge Score](https://img.shields.io/puppetforge/f/sgnl05/sssd.svg)](https://forge.puppetlabs.com/sgnl05/sssd/scores)

#### Table of Contents

1. [Overview](#overview)
2. [Usage - Configuration options and additional functionality](#usage)
3. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
4. [Limitations - OS compatibility, etc.](#limitations)
5. [Credits](#credits)

## Overview

This module installs and configures SSSD (System Security Services Daemon)

[SSSD][0] is used to provide access to identity and authentication remote resource through a common framework that can provide caching and offline support to the system.

## Usage

Example configuration:

```puppet
class {'::sssd':
  config => {
    'sssd' => {
      'domains'             => 'ad.example.com',
      'config_file_version' => 2,
      'services'            => ['nss', 'pam'],
    },
    'domain/ad.example.com' => {
      'ad_domain'                      => 'ad.example.com',
      'ad_server'                      => ['server01.ad.example.com', 'server02.ad.example.com'],
      'krb5_realm'                     => 'AD.EXAMPLE.COM',
      'realmd_tags'                    => 'joined-with-samba',
      'cache_credentials'              => true,
      'id_provider'                    => 'ad',
      'krb5_store_password_if_offline' => true,
      'default_shell'                  => '/bin/bash',
      'ldap_id_mapping'                => false,
      'use_fully_qualified_names'      => false,
      'fallback_homedir'               => '/home/%d/%u',
      'access_provider'                => 'simple',
      'simple_allow_groups'            => ['admins', 'users'],
    }
  }
}
```

...or the same config in Hiera:

```yaml
sssd::config:
  'sssd':
    'domains': 'ad.example.com'
    'config_file_version': 2
    'services':
      - 'nss'
      - 'pam'
  'domain/ad.example.com':
    'ad_domain': 'ad.example.com'
    'ad_server':
      - 'server01.ad.example.com'
      - 'server02.ad.example.com'
    'krb5_realm': 'AD.EXAMPLE.COM'
    'realmd_tags': 'joined-with-samba'
    'cache_credentials': true
    'id_provider': 'ad'
    'krb5_store_password_if_offline': true
    'default_shell': '/bin/bash'
    'ldap_id_mapping': false
    'use_fully_qualified_names': false
    'fallback_homedir': '/home/%d/%u'
    'access_provider': 'simple'
    'simple_allow_groups':
      - 'admins'
      - 'users'
```

Will be represented in sssd.conf like this:

```ini
[sssd]
domains = ad.example.com
config_file_version = 2
services = nss, pam

[domain/ad.example.com]
ad_domain = ad.example.com
ad_server = server01.ad.example.com, server02.ad.example.com
krb5_realm = AD.EXAMPLE.COM
realmd_tags = joined-with-samba
cache_credentials = true
id_provider = ad
krb5_store_password_if_offline = true
default_shell = /bin/bash
ldap_id_mapping = false
use_fully_qualified_names = false
fallback_homedir = /home/%d/%u
access_provider = simple
simple_allow_groups = admins, users
```

Tip: Using 'ad' as `id_provider` require you to run 'adcli join domain' on the target node. *adcli join* creates a computer account in the domain for the local machine, and sets up a keytab for the machine.

Example:

```bash
$ sudo adcli join ad.example.com
```

Or you can use a relevant [module][1] for automation.

## Reference

#####`ensure`
Defines if sssd and its relevant packages are to be installed or removed. Valid values are 'present' and 'absent'.
Type: string
Default: present

#####`config`
Configuration options structured like the sssd.conf file. Array values will be joined into comma-separated lists.
Type: hash
Default:
```puppet
config => {
  'sssd' => {
    'config_file_version' => '2',
    'services'            => 'nss, pam',
    'domains'             => 'ad.example.com',
  },
    'domain/ad.example.com' => {
      'id_provider'       => 'ad',
      'krb5_realm'        => 'AD.EXAMPLE.COM',
      'cache_credentials' => true,
  },
}
```

#####`mkhomedir`
Set to 'true' to enable auto-creation of home directories on user login.
Type: boolean
Default: true

## Limitations

Tested on:
* Fedora 22-25
* (RHEL|CentOS|OracleLinux) 5,6,7
* Ubuntu 14.04 & 16.04
* Suse 11 & 12

## Versioning
The v1 series of this module will support both Puppet v3 and v4. The v2
series of this module will drop support for Puppet v3.

## Credits

* sssd.conf template from [walkamongus-sssd][2] by Chadwick Banning
* See CHANGELOG file for additional credits

[0]: https://docs.pagure.org/SSSD.sssd/
[1]: https://forge.puppet.com/modules?sort=rank&q=adcli
[2]: https://github.com/walkamongus/sssd
