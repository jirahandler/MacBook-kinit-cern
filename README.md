# MacBook CERN Lxplus Kerberos Settings

This repo contains configs and instructions for using kerberos authentication on MacOS Sonoma

## But First

Please don't use `krb5` packages from `homebrew`, `cask`, or even `anaconda`. 

Use the one and only `/usr/bin/kinit`

## System `krb5` config

Your Macbook should ideally have these settings in `/etc/krb5.conf`

If such a file doesn't exist, create one using `sudo`


```bash
# Filename: krb5.conf
# This is needed for MacoS to login to RHEL9 machines at CERN
#
# This file can be used if your OS cannot configure kerberos site ..
# to use, simply do
#
#  (for bash and zsh, place it in .bashrc and .zshrc respectively)
#   export KRB5_CONFIG=$ATLAS_LOCAL_ROOT_BASE/user/krb5-MacOS.conf
#

[libdefaults]
  default_realm = CERN.CH
  ticket_lifetime = 24h
  renew_lifetime = 7d
  forwardable = true
  proxiable = true
  default_etypes = aes256-cts-hmac-sha1-96 aes256-cts aes128-cts des3-cbc-sha1 des-cbc-md5 des-cbc-crc
  
[realms]
  CERN.CH = {
    default_domain = cern.ch
    kpasswd_server = afskrb5m.cern.ch
    admin_server = afskrb5m.cern.ch
    kdc = tcp/cerndc.cern.ch
  }

[domain_realm]
  cern.ch = CERN.CH
  .cern.ch = CERN.CH
```

>[!NOTE]
> for bash and zsh, place it in .bashrc and .zshrc respectively
> ```bash
> export KRB5_CONFIG=$ATLAS_LOCAL_ROOT_BASE/user/krb5-MacOS.conf
> ```

## User `ssh` config

Your Macbook should ideally have these settings in `~/.ssh/config`

If such a file doesn't exist, create one WITHOUT `sudo`

```bash
# This is a template 
#  copy and rename this as ~/.ssh/config and make the changes as indicated
# This is documented in 
#  https://twiki.atlas-canada.ca/bin/view/AtlasCanada/Password-lessSsh

IgnoreUnknown RequiredRSASize,UseKeychain

# it may be that these P1 machines are accessible only on cern.ch domains ...
#  you can always tunnel in: ssh -A -J lxtunnel.cern.ch atlasgw.cern.ch

Host lxplus*
  GSSAPIAuthentication yes 
  GSSAPIDelegateCredentials yes 
  PubkeyAuthentication no 
  ForwardX11 no
  # User <your lxplus username if different>

# important: create ~/.ssh/controlmasters first so that it exists
host lxtunnel lxtunnel.cern.ch
  Hostname lxtunnel.cern.ch
  PubkeyAuthentication no
  ForwardX11 yes
  ControlPath ~/.ssh/controlmasters/%r@%h:%p
  ControlMaster auto
  ControlPersist 10m
  GSSAPIAuthentication yes
  GSSAPIDelegateCredentials yes
  ServerAliveInterval 60
  ServerAliveCountMax 2
  DynamicForward 8090
  # User <your lxplus username if different>

Host *
  UseKeychain yes
  AddKeysToAgent yes
  # uncomment the line after you create your public key, instructions:
  #   https://twiki.atlas-canada.ca/bin/view/AtlasCanada/Password-lessSsh
  #IdentityFile ~/.ssh/id_rsa
```

## Once `ssh` config is created

Execute the following as a regular user, assuming you have ssh-keys from `$USER@lxplus.cern.ch:~/.ssh/*`

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/config ~/.ssh/id_rsa.pub
ssh-add --apple-use-keychain ~/.ssh/id_rsa
```

## Usage

For lxplus, you may now use kerberos authentication and it should not ask you for a password.
Get a valid ticket (lasts for 24h at most) - MacOS users with kerberos stored in Keychains do not need to do this step

```bash
kinit <your lxplus username>@CERN.CH
```

With a valid ticket (`klist` will show validity):
```bash

ssh lxplus.cern.ch
```

>[!WARNING]
>Not having the right `krb5` and/or `ssh` config will show `~/.bash_profile` permissions error upon `ssh`

## Useful kerberos commands on MacOS

1. To check the ticket details, do `klist -v`

The result should be as follows:

```bash
Credentials cache: API:<Token HEX above>
        Principal: <your_CERN_username>@CERN.CH
    Cache version: 0

Server: krbtgt/CERN.CH@CERN.CH
Client: <your_CERN_username>@CERN.CH
Ticket etype: aes256-cts-hmac-sha1-96, kvno 5
Ticket length: 2748
Auth time:  Jul 28 14:48:15 2024
Start time: Jul 28 16:48:24 2024
End time:   Jul 29 16:48:17 2024
Renew till: Aug  4 14:48:08 2024
Ticket flags: enc-pa-rep, pre-authent, initial, renewable, proxiable, forwardable
Addresses: addressless
```
2. To destroy a ticket, do `kdestroy -c API:<Token HEX above>`

## What to do if all of the above fails?

STEP 1: Bug your office mate, mate!
STEP 2: Open a CERN SNOW ticket
STEP 3: The first rule of `kinit` is you don't talk about STEP 3
STEP 4: goto (deprecated) STEP 3; inf loop
