cepces
=========
* Install and configure cepces to automatically get machine certificates issued and renewed
* Remove existing CA configuration
* Add CA configuration and request certificates

Requirements
------------
The windows CA needs to be setup correctly with the roles and additional configuration. A good place to start is to look at the szenarios at the cepces project. https://github.com/openSUSE/cepces/wiki/Scenarios.
The linux machine needs to be domain joined via sssd.

Role Variables
--------------
Defaults:

CEPCES

Git is used for distributions that are missing a package for cepces.
- `cepces_git_repository`: [default: `https://github.com/openSUSE/cepces`]: Git repository to deploy cepces from
- `cepces_git_version`: [default: `master`]: Git repository branch or tag to use
- `cepces_git_dest`: [default: `/opt/cepces`]: Destination for git clone

- `cepces_config_dest`: [default: `/etc/cepces/cepces.conf`]: Destination for the cepces configurationfile
- `cepces_server`: [required]: Hostname of the Certificate Enrollment Policy (CEP) Server. Used to construct the URL to the CEP endpoint
- `cepces_type`: [default: `Policy`]:  It can be either "Policy" where the server endpoint specified later will be used to look up enrollment endpoints, or it can be "Enrollment" to specify a direct endpoint (bypassing XCEP altogether.)
- `cepces_auth`: [default: `Kerberos`]: The authentication mechanism used for connecting to the service endpoint. Possible options are Anonymous, Kerberos, UsernamePassword and Certificate
- `cepces_endpoint`: [default: `https://${server}/ADPolicyProvider_CEP_${auth}/service.svc/CEP`]: The final URL of the Policy Provider endpoint
- `cepces_cas`: [optional]: Path to a CA bundle or directory containing certificates used for the endpoint. This requires python-requests >= 2.9
- `cepces_poll_interval`: [default: `3600`]: Time in seconds before re-checking if the certificate has been issued
- `cepces_openssl_seclevel`: [default: `2`]: openssl secruity level. This can be lowered if you encounter the error '[SSL: DH_KEY_TOO_SMALL] dh key too small'
- `cepces_kerberos_keytab`: [optional]: Use the specified keytab. If unspecified, the system default is used
- `cepces_kerberos_realm`: [optional]: An optional explicit realm to use. If unspecified, the default system realm is used
- `cepces_kerberos_ccache`: [default: `True`]: Initialize a credential cache. If this is disabled, a valid credential cache is required prior launching the application. If enabled, a temporary in-memory cache is created, and released when the application ends
cepces_kerberos_principals`: [default]: A list of principals to try when requesting a ticket 
```
cepces_kerberos_principals:
  - '${shortname}$$'
  - '${SHORTNAME}$$'
  - 'host/${SHORTNAME}'
  - 'host/${fqdn}'
```
cepces_kerberos_enctypes`: [default]: A list of encryption types to use
```
cepces_kerberos_enctypes:
  - 'des-cbc-crc'
  - 'des-cbc-md5'
  - 'arcfour-hmac'
  - 'aes128-cts-hmac-sha1-96'
  - 'aes256-cts-hmac-sha1-96'
```
- `cepces_kerberos_delegate`: [default: `True`]: When the webenrollment services and the CA aren't installed on the same machine you will get a access denied error if kerberos delegation is disabled
- `cepces_certificate_certfile`: [optional]: Use the following client certificate, given as OpenSSL format certificate file. The issuer CA certificate of this client certificate must be included in the AD NTAuth container
- `cepces_certificate_keyfile`: [optional]: Use the following client certificate key, given as OpenSSL format private key file without passphrase protection

Remove existing CAs:

It is possible to remove existing CAs configuration if you define them in a list like so:
```
cepces_remove_ca:
  - ca1
  - ca2
```

Certificate requests via certmonger:

It is possible to request certificates by defining them in a dictionary list. Here are the minimum required options.
```
cepces_requests:
  - name: REQUEST_NAME
    ca_name: CA_NAME
    ca_template: TEMPLATE_NAME
    cert_file: CERT_PATH
    cert_key: KEY_PATH
```
Now I list all the options possible inside the dictionary:
- `name`: [required]: Name for the certificate request that will be created
- `ca_name`: [required]: Name for the CA that will link to the cepces binary
- `ca_template`: [required]: Certificate template to use
- `cert_file`: [required]: Path to the certificate file
- `cert_key`: [required]: Path to the certificate key
- `cert_perm`: [default: `0600`]: Permissions to set on the certificate
- `key_perm`: [default: `0600`]: Permissions to set on the certificate key
- `extra_flags`: [optional]: Appended string for all the other getcert request options that are not predefined 
- `folder_owner`: [default: `root`]: Owner of the directory holding the certificate and key
- `folder_group`: [default: `root`]: Group of the directory holding the certificate and key
- `folder_mode`: [default: `0644`]: Permissions of the directory holding the certificate and key


Dependencies
------------

Example Playbook
----------------
```
    - hosts: systems
      vars:
        cepces_server: ca.domain.example
      roles:
        - cepces
```

Common Issues
-------------
1. No suitable key found in keytab

This error can be caused by different issues. The ones I found are:
- Hostname is longer than 15 characters
- Default realm is not configured in /etc/krb5.conf or as cepces_kerberos_realm
- Keytab is stored in a different place, this can be configured with cepces_kerberos_keytab

Author Information
------------------

Christian Erb - cerb91@gmail.com
