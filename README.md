# Vault-Provision

The Ansible role for install and configure vault and then provision variables on it. Module supports RHEL/Centos7/Fedora for now. The role will be extended in the future and will support Debian/Ubuntu and other operating systems.

## Requirements

This role require Ansible 2.0.

## Role Variables


Variable part is devided into three sections:

#### General part

```
### ########################################################### ###
### General section                                             ###
### ########################################################### ###
vault_app_mirror: 'https://releases.hashicorp.com' # Package repository
vault_app_platform: 'linux_amd64' # Which one platform are you interested in 
vault_app_install_dir: '/opt/vault' # Instalation directory where vault will be installed

vault_app: vault # Vault zip package name
vault_app_ver: '0.6.4' # Version of vault application
vault_app_checksum: sha256:04d87dd553aed59f3fe316222217a8d8777f40115a115dac4d88fac1611c51a6 # Checksum of the Vault zip package

vault_app_name : '{{vault_app}}_{{vault_app_ver}}_{{vault_app_platform}}' # Vault's package name to download
vault_app_zip : '{{vault_app_name}}.zip' # Vault's zip package to download
vault_app_url : '{{vault_app_mirror}}/{{vault_app}}/{{vault_app_ver}}/{{vault_app_zip}}' # Full URL to get the Vault package

method: http # Whether https or http should be use for provisioning process
vault_domain_name: vault.your.domain # Vault domain name or ip address
vault_api_version: v1 # Vault's REST API version
content_type: application/json # The content type of the http body request

vault_user: vault # Group which is allowed to run vault service
vault_group: vault # User which is allowed to run vault service

backend_type: file # backend used by you. All possible supported backends are listed at https://www.vaultproject.io/docs/config/index.html website
```

#### Configuration part



Variables for Configuration part are well described at vault website `https://www.vaultproject.io/docs/config/index.html`. Basicly variable names in this module reflect these in vault config documentation. You can define more than one vault_configuration.backend variable but just one will be use and you have to specify which one in `backend_type` variable in general section. The module is flexible and `backend_type` variable can be whatever backend which is supperted by Vault. In your config please keep only variables which is used by you. Rest of the variables can be commented out or just removed. For now all possible configuration variables are mention below:

```
### ########################################################### ###
### Configuration section                                       ###
### ########################################################### ###

vault_configuration:
  backend:
    consul:
      path: vault
      address: 127.0.0.1:8500
      scheme:
      check_timeout:
      disable_registration:
      service:
      service_tags:
      token:
      max_parallel:
      tls_skip_verify:
      tls_min_version:
      tls_ca_file:
      tls_cert_file:
      tls_key_file:
    etcd:
      path:
      address:
      sync:
      ha_enabled:
      username:
      password:
      tls_ca_file:
      tls_cert_file:
      tls_key_file:
    zookeeper:
      path:
      address:
      auth_info:
      znode_owner:
    dynamodb:
      access_key:
      secret_key:
      table:
      read_capacity:
      write_capacity:
      session_token:
      endpoint:
      region:
      max_parallel:
      ha_enabled:
      recovery_mode:
    s3:
      bucket: bucket_name
      access_key: AKIAJWVN5Z4FOFT7NLNA
      secret_key: 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
      session_token:
      endpoint:
      region: us-east-1
    gcs:
      bucket:
      credentials_file:
      max_parallel:
    azure:
      accountName:
      accountKey:
      container:
      max_parallel:
    swift:
      container:
      username:
      password:
      auth_url:
      tenant:
      max_parallel:
    mysql:
      username:
      password:
      address:
      database:
      table:
      tls_ca_file:
    postgresql:
      connection_url:
      table:
    inmem:
    file:
      path: /opt/vault/storage
    ha_options:
      redirect_addr:
      cluster_addr:
      disable_clustering:
  listener: #"tcp" and "atlas" are valid values
    tcp:
      address: 0.0.0.0:8200
      tls_disable: 1
      cluster_address:
      tls_cert_file:
      tls_key_file:
      tls_min_version:
    atlas:
      endpoint:
      infrastructure:
      node_id:
      token:
    telemetry:
      statsite_address:
      statsd_address:
      disable_hostname:
      circonus_api_token:
      circonus_api_app:
      circonus_api_url:
      circonus_submission_interval:
      circonus_submission_url:
      circonus_check_id:
      circonus_check_force_metric_activation:
      circonus_check_instance_id:
      circonus_check_search_tag:
      circonus_check_display_name:
      circonus_check_tags:
      circonus_broker_id:
      circonus_broker_select_tag:
  ha_backend:
  cluster_name:
  cache_size:
  disable_cache: false
  disable_mlock: false
  default_lease_ttl:
  max_lease_ttl:
  ui:
```

#### Provision part

```
### ########################################################### ###
### Provision section                                           ###
### ########################################################### ###

backends: # Define backends which will be created
  - backend_name: secret # Backend named secret will be created if doesn't exist
    type: generic # Type of the backend
    path: secret # Vault internal path where backend will be mounted 
    secrets: # Set of data container which will be created
      - container_name: authcredentials # Data container named authcredentials
        data: # Comprise all keys and values which will be provisioned into secret/authcredentials
          username: password 
          david: superstrongpasswordfordavid
          daniel: superstrongpasswordfordaniel
      - container_name: privatekeys # Data container named privatekeys
        data: # Comprise all keys and values which will be provisioned into secret/privatekeys
          key_name: encryptedkey
          key_for_mail: enctryptedkeyformail
  - backend_name: test # Backend named test will be created if doesn't exist
    type: generic # Type of the backend
    path: test # Vault internal path where backend will be mounted
    secrets: # Set of data container which will be created
      - container_name: test_container # Data container named test_container
        data: # Comprise all keys and values which will be provisioned into test/test_container
          important_data: importantdatavalue
          another_important_data: anotherimportantdatavalue

initialization: # PGP keys for initialization process
  root_token_pgp_key: mQENBFhZFd4BCADGj9gcnJHRwQk1UL8BxTugxmJ6uk82S837LWqzrsDv18zObBQK8bmTFps4x6DrUbLbxzHRB7kxbR5g/KN2qCqMYcYkKgLqMqo9XaS4vCriQKCFaZ3FWG99/TxUurPpy+crVCDun8UMGmWZCvRu7UBJQsAF1htukVhdEXAd1116WRD6W5k/+X6B6FvSAD7ud/PWUENdtLxiRgBVhRCHWPlJvdA7I7EaBjF5TMbwFP4wlSHxbdrWZ0z9wGk5FFbtl1qlNaojyJR1Gs3a7T7lB7y1dr6Y727JAYHP5uzQo06F0nfQm58sXtmeBnyhOHZ04BzXBik+Uoh9Bh1Uj7em0cP5ABEBAAG0T0tyenlzenRvZiBTemV3Y3p5ayAoRW5jcnlwdGlvbiBrZXkgZm9yIEhhc2hpY29ycCBWYXVsdCkgPGsuc3pld2N6eWtAa2Fpbm9zLmNvbT6JAT4EEwECACgFAlhZFd4CGwMFCQHhM4AGCwkIBwMCBhUIAgkKCwQWAgMBAh4BAheAAAoJEMOj6FrFv4ik9bsH/3gmLKh8r2xhsbA79bEzBjNPDX9G6qeHfea+RMRjLH5yvNH9/Hnjwkqn3LIAdpzacldrjZMEpqS5z310gxk+cyNB/dFe/Wqmh1UQljDQ354qAZAtQ7OLuypVsg+tI8aIRQieCEC4Ck/h8xnAK8AoOHb29vR+BMNdqp+TNrDrAeYAKchmJiWcYxt5YqT4AP5JRzT82o2Sms3ihyFe1KxgcNbrNjJpEMoCnu8l5e1RrlIeMU8Hm5Mms1ZvausPfFlsa76shhOCZMfZ2acsuuTRqx8ed8UrSmDqAiia60BupsfRfMkqf/GdkEEESoRKhz1EHAUQiC9Flshyl/CjWqCIQCi5AQ0EWFkV3gEIAKLrtH904COoJhOjdqUQNSGu0TSsNik2Hd/v5A5KxG7a2iiqLOD73VvINaTHpwUjkWR3OcgRuqsnshST/qg3T1t8gghON70Pg6nKEPbDGabhA19fQ64woSvY6I2ZTv/XY0CxxDAVNN2O3rpl6ZnUgQBVjd/u4Nx3cydf30MnPtipb9bJ1ISZvWtMAYxAeiqd/6rcj/BXSQBt05ffDwPBd5+7pL7gQrJTa51icggKq06KwTfSdi0cPmqvOenKZB0exrp2jtxw1+QIhlhJJyv33fxaJHZaFqlh7vn6j7e03J4UaVPl5h/fnNg3i1uJJW9E6ECfAwRy394r7V1LUZ/GiEMAEQEAAYkBJQQYAQIADwUCWFkV3gIbDAUJAeEzgAAKCRDDo+haxb+IpJFmB/99Ss1U/dJA/XFwWofj3iEDM3W7OJoahYJLD7eOo3lPT8vNnLEVOPkSPjvaggkSsM3Xp5WZw2/oQ3p5KuIE2bSGK/Ok8zEAjEbn5vT4GNTBSH070OAQEHD5dEcYmh9EKSnitPXlLMOG/szsnReimBjLXRibU2wvuMeY7QO58K9kHm1K9cpNWuU5h8M4CMlBATvfl4ZeIBm6K9gmqKKxkF+7Mw1U8XG/OZv6GQwbMER3h7cdsor9LsOZTIdZW7bj9Iyh42hZHMcWq1E2VvHuklwJdpZfKlUs6b/6WwSPsypLLfHnO6C/G726MXH2JuZe5/fFpn+skxgbYNZ1y1BTnyvs # Public part of PGP key for vault root token encryption. After initialization process outputted root token is encrypted by this PGP public key
  secret_threshold: 1 # Define how many keys is required to unseal the vault (This value have to be less then length of the below pgp_keys array )
  pgp_keys: # Array defined how many keys will be created. The keys will be encrypted by below PGP keys
   - mQENBFhZFd4BCADGj9gcnJHRwQk1UL8BxTugxmJ6uk82S837LWqzrsDv18zObBQK8bmTFps4x6DrUbLbxzHRB7kxbR5g/KN2qCqMYcYkKgLqMqo9XaS4vCriQKCFaZ3FWG99/TxUurPpy+crVCDun8UMGmWZCvRu7UBJQsAF1htukVhdEXAd1116WRD6W5k/+X6B6FvSAD7ud/PWUENdtLxiRgBVhRCHWPlJvdA7I7EaBjF5TMbwFP4wlSHxbdrWZ0z9wGk5FFbtl1qlNaojyJR1Gs3a7T7lB7y1dr6Y727JAYHP5uzQo06F0nfQm58sXtmeBnyhOHZ04BzXBik+Uoh9Bh1Uj7em0cP5ABEBAAG0T0tyenlzenRvZiBTemV3Y3p5ayAoRW5jcnlwdGlvbiBrZXkgZm9yIEhhc2hpY29ycCBWYXVsdCkgPGsuc3pld2N6eWtAa2Fpbm9zLmNvbT6JAT4EEwECACgFAlhZFd4CGwMFCQHhM4AGCwkIBwMCBhUIAgkKCwQWAgMBAh4BAheAAAoJEMOj6FrFv4ik9bsH/3gmLKh8r2xhsbA79bEzBjNPDX9G6qeHfea+RMRjLH5yvNH9/Hnjwkqn3LIAdpzacldrjZMEpqS5z310gxk+cyNB/dFe/Wqmh1UQljDQ354qAZAtQ7OLuypVsg+tI8aIRQieCEC4Ck/h8xnAK8AoOHb29vR+BMNdqp+TNrDrAeYAKchmJiWcYxt5YqT4AP5JRzT82o2Sms3ihyFe1KxgcNbrNjJpEMoCnu8l5e1RrlIeMU8Hm5Mms1ZvausPfFlsa76shhOCZMfZ2acsuuTRqx8ed8UrSmDqAiia60BupsfRfMkqf/GdkEEESoRKhz1EHAUQiC9Flshyl/CjWqCIQCi5AQ0EWFkV3gEIAKLrtH904COoJhOjdqUQNSGu0TSsNik2Hd/v5A5KxG7a2iiqLOD73VvINaTHpwUjkWR3OcgRuqsnshST/qg3T1t8gghON70Pg6nKEPbDGabhA19fQ64woSvY6I2ZTv/XY0CxxDAVNN2O3rpl6ZnUgQBVjd/u4Nx3cydf30MnPtipb9bJ1ISZvWtMAYxAeiqd/6rcj/BXSQBt05ffDwPBd5+7pL7gQrJTa51icggKq06KwTfSdi0cPmqvOenKZB0exrp2jtxw1+QIhlhJJyv33fxaJHZaFqlh7vn6j7e03J4UaVPl5h/fnNg3i1uJJW9E6ECfAwRy394r7V1LUZ/GiEMAEQEAAYkBJQQYAQIADwUCWFkV3gIbDAUJAeEzgAAKCRDDo+haxb+IpJFmB/99Ss1U/dJA/XFwWofj3iEDM3W7OJoahYJLD7eOo3lPT8vNnLEVOPkSPjvaggkSsM3Xp5WZw2/oQ3p5KuIE2bSGK/Ok8zEAjEbn5vT4GNTBSH070OAQEHD5dEcYmh9EKSnitPXlLMOG/szsnReimBjLXRibU2wvuMeY7QO58K9kHm1K9cpNWuU5h8M4CMlBATvfl4ZeIBm6K9gmqKKxkF+7Mw1U8XG/OZv6GQwbMER3h7cdsor9LsOZTIdZW7bj9Iyh42hZHMcWq1E2VvHuklwJdpZfKlUs6b/6WwSPsypLLfHnO6C/G726MXH2JuZe5/fFpn+skxgbYNZ1y1BTnyvs # Public part of PGP key for Vault key encryption.

policies: # Array of the policies that will be created
  - name: testpolicy # Name of the policy
    data: # Contain rules
      rules: "path \"secret/authcredentials\" {\n  policy = \"read\"\n}\npath \"test/test_container\" {\n  policy = \"read\"\n}\npath \"auth/token/lookup-self\" {\n  policy = \"read\"\n}" # Contain rules in plaintext HCL format

tokens: # Array of the tokens that will be created
  - display_name: test_token # Name of the token
    id: 950938df-50cb-7a24-faeh-b4f2a23g3b6f # Id of the token
    policies: # Array of the policies attached to token
      - testpolicy # Name of the policy attached to token
```

### Another required variables:

During the first cycle of running this role the initialization process is performed. The unseal keys and vault root token are outputted and these values are encrypted by pgp_keys. The provisioning process is skipped. You have to depcypt the unseal keys. Then you have to unseal in some way (ssh to the machine and unseal vault by providing the decrypted keys or you define some jenkins job in continues integration tool which do it for you). The way how would you like to do it is up to you.

When you have Vault unsealed then decrypt vault root token and put decrypted value as a vault_token variable. Then you can define this value in your ansible in vars or you can call ansible with extra vars parameter like this:  

`ansible-playbook -i "<ip_address>," vault.yml --extra-vars '{"vault_token":"de4cdf73-3b4d-4762-09f5-97088e147c1d"}'`

or the other way is to store vault_token value in continues integration tool and use this value in you jenkins job or Jenkinsfile.

If you provide correct vault token and the vault is unsealed, during the second cycle the role will provision the vault. All backends, containers, keys and values, policies, tokens will be created.

## Dependencies

The role has no external dependencies.

## Example Playbook

You have to at least define your own variables in your playbook:

```---
method: http # Suggest to use https
vault_domain_name: vault.your.domain
vault_configuration:
  backend:
    file:
      path: /opt/vault/storage
  listener: #"tcp" and "atlas" are valid values
    tcp:
      address: 0.0.0.0:8200
      tls_disable: 1
  disable_cache: false
  disable_mlock: false
```
You have to also define backends, secrets, policies which you would like to put into Vault and of course you have to define some initialization values like pgp keys for Vault keys encryption, pgp key for Vault root token encryption and secret threshold which define how many keys you/your team have to provide to unseal the Vault. Secret threshold must be less than length of the `pgp_keys` array. Let's adopt below variables as you want.

```backends:
  - backend_name: secret
    type: generic
    path: secret
    secrets:
      - container_name: authcredentials
        data:
          username: password
          david: superstrongpasswordfordavid
          daniel: superstrongpasswordfordaniel
      - container_name: privatekeys
        data:
          key_name: encryptedkey
          key_for_mail: enctryptedkeyformail
  - backend_name: test
    type: generic
    path: test
    secrets:
      - container_name: test_container
        data:
          important_data: importantdatavalue
          another_important_data: anotherimportantdatavalue

initialization:
  root_token_pgp_key: mQENBFhZFd4BCADGj9gcnJHRwQk1UL8BxTugxmJ6uk82S837LWqzrsDv18zObBQK8bmTFps4x6DrUbLbxzHRB7kxbR5g/KN2qCqMYcYkKgLqMqo9XaS4vCriQKCFaZ3FWG99/TxUurPpy+crVCDun8UMGmWZCvRu7UBJQsAF1htukVhdEXAd1116WRD6W5k/+X6B6FvSAD7ud/PWUENdtLxiRgBVhRCHWPlJvdA7I7EaBjF5TMbwFP4wlSHxbdrWZ0z9wGk5FFbtl1qlNaojyJR1Gs3a7T7lB7y1dr6Y727JAYHP5uzQo06F0nfQm58sXtmeBnyhOHZ04BzXBik+Uoh9Bh1Uj7em0cP5ABEBAAG0T0tyenlzenRvZiBTemV3Y3p5ayAoRW5jcnlwdGlvbiBrZXkgZm9yIEhhc2hpY29ycCBWYXVsdCkgPGsuc3pld2N6eWtAa2Fpbm9zLmNvbT6JAT4EEwECACgFAlhZFd4CGwMFCQHhM4AGCwkIBwMCBhUIAgkKCwQWAgMBAh4BAheAAAoJEMOj6FrFv4ik9bsH/3gmLKh8r2xhsbA79bEzBjNPDX9G6qeHfea+RMRjLH5yvNH9/Hnjwkqn3LIAdpzacldrjZMEpqS5z310gxk+cyNB/dFe/Wqmh1UQljDQ354qAZAtQ7OLuypVsg+tI8aIRQieCEC4Ck/h8xnAK8AoOHb29vR+BMNdqp+TNrDrAeYAKchmJiWcYxt5YqT4AP5JRzT82o2Sms3ihyFe1KxgcNbrNjJpEMoCnu8l5e1RrlIeMU8Hm5Mms1ZvausPfFlsa76shhOCZMfZ2acsuuTRqx8ed8UrSmDqAiia60BupsfRfMkqf/GdkEEESoRKhz1EHAUQiC9Flshyl/CjWqCIQCi5AQ0EWFkV3gEIAKLrtH904COoJhOjdqUQNSGu0TSsNik2Hd/v5A5KxG7a2iiqLOD73VvINaTHpwUjkWR3OcgRuqsnshST/qg3T1t8gghON70Pg6nKEPbDGabhA19fQ64woSvY6I2ZTv/XY0CxxDAVNN2O3rpl6ZnUgQBVjd/u4Nx3cydf30MnPtipb9bJ1ISZvWtMAYxAeiqd/6rcj/BXSQBt05ffDwPBd5+7pL7gQrJTa51icggKq06KwTfSdi0cPmqvOenKZB0exrp2jtxw1+QIhlhJJyv33fxaJHZaFqlh7vn6j7e03J4UaVPl5h/fnNg3i1uJJW9E6ECfAwRy394r7V1LUZ/GiEMAEQEAAYkBJQQYAQIADwUCWFkV3gIbDAUJAeEzgAAKCRDDo+haxb+IpJFmB/99Ss1U/dJA/XFwWofj3iEDM3W7OJoahYJLD7eOo3lPT8vNnLEVOPkSPjvaggkSsM3Xp5WZw2/oQ3p5KuIE2bSGK/Ok8zEAjEbn5vT4GNTBSH070OAQEHD5dEcYmh9EKSnitPXlLMOG/szsnReimBjLXRibU2wvuMeY7QO58K9kHm1K9cpNWuU5h8M4CMlBATvfl4ZeIBm6K9gmqKKxkF+7Mw1U8XG/OZv6GQwbMER3h7cdsor9LsOZTIdZW7bj9Iyh42hZHMcWq1E2VvHuklwJdpZfKlUs6b/6WwSPsypLLfHnO6C/G726MXH2JuZe5/fFpn+skxgbYNZ1y1BTnyvs
  secret_threshold: 1
  pgp_keys:
   - mQENBFhZFd4BCADGj9gcnJHRwQk1UL8BxTugxmJ6uk82S837LWqzrsDv18zObBQK8bmTFps4x6DrUbLbxzHRB7kxbR5g/KN2qCqMYcYkKgLqMqo9XaS4vCriQKCFaZ3FWG99/TxUurPpy+crVCDun8UMGmWZCvRu7UBJQsAF1htukVhdEXAd1116WRD6W5k/+X6B6FvSAD7ud/PWUENdtLxiRgBVhRCHWPlJvdA7I7EaBjF5TMbwFP4wlSHxbdrWZ0z9wGk5FFbtl1qlNaojyJR1Gs3a7T7lB7y1dr6Y727JAYHP5uzQo06F0nfQm58sXtmeBnyhOHZ04BzXBik+Uoh9Bh1Uj7em0cP5ABEBAAG0T0tyenlzenRvZiBTemV3Y3p5ayAoRW5jcnlwdGlvbiBrZXkgZm9yIEhhc2hpY29ycCBWYXVsdCkgPGsuc3pld2N6eWtAa2Fpbm9zLmNvbT6JAT4EEwECACgFAlhZFd4CGwMFCQHhM4AGCwkIBwMCBhUIAgkKCwQWAgMBAh4BAheAAAoJEMOj6FrFv4ik9bsH/3gmLKh8r2xhsbA79bEzBjNPDX9G6qeHfea+RMRjLH5yvNH9/Hnjwkqn3LIAdpzacldrjZMEpqS5z310gxk+cyNB/dFe/Wqmh1UQljDQ354qAZAtQ7OLuypVsg+tI8aIRQieCEC4Ck/h8xnAK8AoOHb29vR+BMNdqp+TNrDrAeYAKchmJiWcYxt5YqT4AP5JRzT82o2Sms3ihyFe1KxgcNbrNjJpEMoCnu8l5e1RrlIeMU8Hm5Mms1ZvausPfFlsa76shhOCZMfZ2acsuuTRqx8ed8UrSmDqAiia60BupsfRfMkqf/GdkEEESoRKhz1EHAUQiC9Flshyl/CjWqCIQCi5AQ0EWFkV3gEIAKLrtH904COoJhOjdqUQNSGu0TSsNik2Hd/v5A5KxG7a2iiqLOD73VvINaTHpwUjkWR3OcgRuqsnshST/qg3T1t8gghON70Pg6nKEPbDGabhA19fQ64woSvY6I2ZTv/XY0CxxDAVNN2O3rpl6ZnUgQBVjd/u4Nx3cydf30MnPtipb9bJ1ISZvWtMAYxAeiqd/6rcj/BXSQBt05ffDwPBd5+7pL7gQrJTa51icggKq06KwTfSdi0cPmqvOenKZB0exrp2jtxw1+QIhlhJJyv33fxaJHZaFqlh7vn6j7e03J4UaVPl5h/fnNg3i1uJJW9E6ECfAwRy394r7V1LUZ/GiEMAEQEAAYkBJQQYAQIADwUCWFkV3gIbDAUJAeEzgAAKCRDDo+haxb+IpJFmB/99Ss1U/dJA/XFwWofj3iEDM3W7OJoahYJLD7eOo3lPT8vNnLEVOPkSPjvaggkSsM3Xp5WZw2/oQ3p5KuIE2bSGK/Ok8zEAjEbn5vT4GNTBSH070OAQEHD5dEcYmh9EKSnitPXlLMOG/szsnReimBjLXRibU2wvuMeY7QO58K9kHm1K9cpNWuU5h8M4CMlBATvfl4ZeIBm6K9gmqKKxkF+7Mw1U8XG/OZv6GQwbMER3h7cdsor9LsOZTIdZW7bj9Iyh42hZHMcWq1E2VvHuklwJdpZfKlUs6b/6WwSPsypLLfHnO6C/G726MXH2JuZe5/fFpn+skxgbYNZ1y1BTnyvs

policies:
  - name: testpolicy
    data:
      rules: "path \"secret/authcredentials\" {\n  policy = \"read\"\n}\npath \"test/test_container\" {\n  policy = \"read\"\n}\npath \"auth/token/lookup-self\" {\n  policy = \"read\"\n}"

tokens:
  - display_name: test_token
    id: 950938df-50cb-7a24-faeh-b4f2a23g3b6f
    policies:
      - testpolicy
```

#### Playbook can look like:

```
---
- name: Provision vault
  hosts: vault.your.domain
  roles:
  - vault-provision
```

## Decrypt Vault keys and root token

During the first cycle of provisioning you get keys and root token: 

```
TASK [vault-provision : debug] *************************************************
ok: [vaultprovision] => {
    "initialization": {
        "cache_control": "no-store",
        "changed": false,
        "connection": "close",
        "content_length": "1899",
        "content_type": "application/json",
        "date": "Wed, 28 Dec 2016 17:56:58 GMT",
        "json": {
            "keys": [
                "c1c04c03822c554bc14d713401080024fc6ac55a5966819561129600f055d8fc837e3371955b586887408d3d82c2eb4c897678137daaa39fddc0b1062d199db654b69bce20aebbade8fc7b20d2cf9cafc129a5e321542f332d37ecc1357314ead64eaa25131723197b90827c7c63fb7465ba8ae1ef762cc7dcb583df7246d95c7593efae3f7391e39dcb023f9d7933d6a553d510e30c643dc78939218cef2bca650e20fe7aa494157bde4a0c26a733321a597adc2b65c59110d408c764f3cdcc3082beccb89ff3103e6cc1c81f83dc00628cd5201edc3d36f58d6c2921798b7d2803254ebe3414eeef2bb923a0baaa21016c57193ab6e9abfe2f7a0baf7181200980d9bab7f1f2cd12c80a43aad455d2e001e4cf59665b131fd56e5a793f0fd5670891e1fb87e0bfe0f7e16799e084e2352a6027e0dce66f4b05d594f172c58717660a4ea46adf3bc240e134950c07f0c302d1458f5dccc14d95fc669004891903cd1a5a9ac2dcd9f25564238990cda2e0e3a57006cf9de0f4e498ddc6d2349e201378da85eceb979fa6e246ae4ea7e174a100"
            ],
            "keys_base64": [
                "wcBMA4IsVUvBTXE0AQgAJPxqxVpZZoGVYRKWAPBV2PyDfjNxlVtYaIdAjT2CwutMiXZ4E32qo5/dwLEGLRmdtlS2m84grrut6Px7INLPnK/BKaXjIVQvMy037ME1cxTq1k6qJRMXIxl7kIJ8fGP7dGW6iuHvdizH3LWD33JG2Vx1k++uP3OR453LAj+deTPWpVPVEOMMZD3HiTkhjO8rymUOIP56pJQVe95KDCanMzIaWXrcK2XFkRDUCMdk883MMIK+zLif8xA+bMHIH4PcAGKM1SAe3D029Y1sKSF5i30oAyVOvjQU7u8ruSOguqohAWxXGTq26av+L3oLr3GBIAmA2bq38fLNEsgKQ6rUVdLgAeTPWWZbEx/Vblp5Pw/VZwiR4fuH4L/g9+FnmeCE4jUqYCfg3OZvSwXVlPFyxYcXZgpOpGrfO8JA4TSVDAfwwwLRRY9dzMFNlfxmkASJGQPNGlqawtzZ8lVkI4mQzaLg46VwBs+d4PTkmN3G0jSeIBN42oXs65efpuJGrk6n4XShAA=="
            ],
            "root_token": "wcBMA4IsVUvBTXE0AQgAGQNSD1geNeecOe2Pta7ugXaCIIf80HEX7YJPouys2x6SzB844GG+zCEubsswFBANOgTMctTExC0Fs35vRhCTZakAkYuRLIirQXuFOxZ7avIIaCsv/sn8/R3gIRNWSiRX/m2snI3r49UkA7tJo8w/BKS9b+SQwzDwCAMNY04gXqOE++bJqY6Z/PEEebhuv7McJrA+s2SJj8zKst5DgXzm5tNhkr5/McQEntcHG/RZHBKtBbHppSLDibd98TODnHkcEm+DkhYgKhNQNEiTYzOTHoZtdwwgwd6tYfPYV9d4z/3SsEnXmRskEymJONLU4uMo6nlykNq8UxsqW3vc/xCDhNLgAeTfuOJ86dRgq0Q+2m1wxwBU4Zb34ALg5eEehOCM4nEi8KjgJOWob3JV0y74SJy1tUlXvbIeT5RwvLimvueEbOwVc1FlsuBm4k8V8ATgquRHGMHaat72Z3Q4906BTGb14thk+abhPsYA"
        },
        "msg": "OK (1899 bytes)",
        "redirected": false,
        "status": 200,
        "url": "http://192.168.60.10:8200/v1/sys/init"
    }
}
```

In the `keys_base64` array of the received json you can see your keys encrypted by the provided earlier PGP or GPG keys in base64 format. To decrypt that you have to decode your base64
formated value and then decrypt by gpg1 tool (MacOS) or gpg tool (Linux).

`echo "wcBMA4IsVUvBTXE0AQgAJPxqxVpZZoGVYRKWAPBV2PyDfjNxlVtYaIdAjT2CwutMiXZ4E32qo5/dwLEGLRmdtlS2m84grrut6Px7INLPnK/BKaXjIVQvMy037ME1cxTq1k6qJRMXIxl7kIJ8fGP7dGW6iuHvdizH3LWD33JG2Vx1k++uP3OR453LAj+deTPWpVPVEOMMZD3HiTkhjO8rymUOIP56pJQVe95KDCanMzIaWXrcK2XFkRDUCMdk883MMIK+zLif8xA+bMHIH4PcAGKM1SAe3D029Y1sKSF5i30oAyVOvjQU7u8ruSOguqohAWxXGTq26av+L3oLr3GBIAmA2bq38fLNEsgKQ6rUVdLgAeTPWWZbEx/Vblp5Pw/VZwiR4fuH4L/g9+FnmeCE4jUqYCfg3OZvSwXVlPFyxYcXZgpOpGrfO8JA4TSVDAfwwwLRRY9dzMFNlfxmkASJGQPNGlqawtzZ8lVkI4mQzaLg46VwBs+d4PTkmN3G0jSeIBN42oXs65efpuJGrk6n4XShAA==" | base64 -D | gpg1 -d`

The result of the above command will be your decrypted key which is necessary to unseal Vault.

Let's do the same with your root_token:

`echo "wcBMA4IsVUvBTXE0AQgAGQNSD1geNeecOe2Pta7ugXaCIIf80HEX7YJPouys2x6SzB844GG+zCEubsswFBANOgTMctTExC0Fs35vRhCTZakAkYuRLIirQXuFOxZ7avIIaCsv/sn8/R3gIRNWSiRX/m2snI3r49UkA7tJo8w/BKS9b+SQwzDwCAMNY04gXqOE++bJqY6Z/PEEebhuv7McJrA+s2SJj8zKst5DgXzm5tNhkr5/McQEntcHG/RZHBKtBbHppSLDibd98TODnHkcEm+DkhYgKhNQNEiTYzOTHoZtdwwgwd6tYfPYV9d4z/3SsEnXmRskEymJONLU4uMo6nlykNq8UxsqW3vc/xCDhNLgAeTfuOJ86dRgq0Q+2m1wxwBU4Zb34ALg5eEehOCM4nEi8KjgJOWob3JV0y74SJy1tUlXvbIeT5RwvLimvueEbOwVc1FlsuBm4k8V8ATgquRHGMHaat72Z3Q4906BTGb14thk+abhPsYA" | base64 -D | gpg1 -d`

The result of the above command will be your decrypted key which is necessary to further Vault provisioning process - deployment backends, tokens and secrets into Vault. Like i mentioned before root token will be necessary during the second cycle of Vault provisioning and you have to pass root token to ansible as an extra vars:

`ansible-playbook -i "<ip_address>," vault.yml --extra-vars '{"vault_token":"de4cdf73-3b4d-4762-09f5-97088e147c1d"}'`

Notice: If `vault_token` variable is empty then second provision cycle will be skipped.

## Test

The tests directory contains tests for this role in the form of a Vagrant environment. After executing vagrant up in that directory, you should get an environment with one VM, attached to VirtualBox's default NAT network (Internet access is required to get vault package and dependencies).


The directory `tests/roles/vault-provision` is a symbolic link that should point to the root of this project in order to work. 

The playbook `test.yml` applies the role to a VM, setting role variables.

vagrant provision should be running twice. First cycle is for initialization process. Then you have to unseal Vault. In second cycle you need to provide correct root token (decrypted by your private part of PGP or GPG key) in `extra_vars.yml` file.


## Feedback, bug-reports, requests, ...

Issues, feature requests, ideas are appreciated and can be posted in the Issues section. Pull requests are also very welcome. Preferably, create a topic branch and when submitting, squash your commits into one (with a descriptive message).

Cheers !

## License

MIT

## Author Information

 
I attached links where you can find more information about me:

- Linkedin: https://pl.linkedin.com/in/krzysztof-szewczyk-8486a2ba
- Github account: https://github.com/christophershoemaker
- Email: szewczyk.christopher@gmail.com
