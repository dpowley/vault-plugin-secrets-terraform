# Vault Plugin: Terraform Cloud Secrets Backend [![HashiCorp](https://circleci.com/gh/hashicorp/vault-plugin-secrets-terraform.svg?style=svg)](https://circleci.com/gh/hashicorp/vault-plugin-secrets-terraform)

This is a standalone backend plugin for use with [Hashicorp
Vault](https://www.github.com/hashicorp/vault). This plugin generates:
- Revocable, time-limited [API tokens](https://www.terraform.io/cloud-docs/users-teams-organizations/api-tokens#user-api-tokens) for [Terraform Cloud users](https://www.terraform.io/cloud-docs/users-teams-organizations/users).
- Revocable, time-limited [Terraform Cloud teams](https://www.terraform.io/cloud-docs/users-teams-organizations/teams) and corresponding [API tokens](https://www.terraform.io/cloud-docs/users-teams-organizations/api-tokens#team-api-tokens).
- Manages single [API tokens](https://www.terraform.io/cloud-docs/users-teams-organizations/api-tokens#team-api-tokens) for existing [Terraform Cloud teams](https://www.terraform.io/cloud-docs/users-teams-organizations/teams).
- Manages single [API tokens](https://www.terraform.io/cloud-docs/users-teams-organizations/api-tokens#organization-api-tokens) for [Terraform Cloud organizations](https://www.terraform.io/cloud-docs/users-teams-organizations/organizations). 

Please see Terraform Cloud's documentation on [API Tokens](https://www.terraform.io/docs/cloud/users-teams-organizations/api-tokens.html) for more information on the types of API tokens offered by the Terraform Cloud API.

**Please note**: We take Vault's security and our users' trust very seriously.
If you believe you have found a security issue in Vault, _please responsibly
disclose_ by contacting us at
[security@hashicorp.com](mailto:security@hashicorp.com).

## Quick Links
- [Vault Website](https://www.vaultproject.io)
- [Terraform Cloud](https://www.terraform.io/cloud)
- [Terraform Cloud Secrets
  Docs](https://www.vaultproject.io/docs/secrets/terraform/index.html)
- [Terraform Cloud API token
  documentation](https://www.terraform.io/docs/cloud/users-teams-organizations/api-tokens.html)
- [Vault Github Project](https://www.github.com/hashicorp/vault)

## Getting Started

This is a [Vault
plugin](https://www.vaultproject.io/docs/internals/plugins.html) and is meant to
work with Vault. This guide assumes you have already installed Vault and have a
basic understanding of how Vault works.

Otherwise, first read this guide on how to [get started with
Vault](https://www.vaultproject.io/intro/getting-started/install.html).

To learn specifically about how plugins work, see documentation on [Vault
plugins](https://www.vaultproject.io/docs/internals/plugins.html).

## Usage

Please see [documentation for the
plugin](https://www.vaultproject.io/docs/secrets/terraform/index.html) on the
Vault website.

This plugin is currently built into Vault and by default is accessed at
`terraform`. To enable this in a running Vault server:

```sh 
$ vault secrets enable terraform Success! 
Enabled the terraform secrets engine at: terraform/ 
```


## Developing

If you wish to work on this plugin, you'll first need
[Go](https://www.golang.org) installed on your machine
(version 1.15.3+ is *required*).

For local dev first make sure Go is properly installed, including
setting up a [GOPATH](https://golang.org/doc/code.html#GOPATH).
Next, clone this repository into
`$GOPATH/src/github.com/hashicorp/vault-plugin-secrets-terraform`.
You can then download any required build tools by bootstrapping your
environment:

```sh
$ make bootstrap
```

To compile a development version of this plugin, run `make` or `make dev`.
This will put the plugin binary in the `bin` and `$GOPATH/bin` folders. `dev`
mode will only generate the binary for your platform and is faster:

```sh
$ make
$ make dev
```

Put the plugin binary into a location of your choice. This directory will be
specified as the
[`plugin_directory`](https://www.vaultproject.io/docs/configuration/index.html#plugin_directory)
in the Vault config used to start the server.

```json
...
plugin_directory = "path/to/plugin/directory"
...
```

Start a Vault server with this config file:
```sh
$ vault server -config=path/to/config.json ...
...
```

Once the server is started, register the plugin in the Vault server's [plugin
catalog](https://www.vaultproject.io/docs/internals/plugins.html#plugin-catalog):

```sh
$ vault write sys/plugins/catalog/terraform \
        sha256=<expected SHA256 Hex value of the plugin binary> \
        command="vault-plugin-secrets-terraform"
...
Success! Data written to: sys/plugins/catalog/terraform
```

Note you should generate a new sha256 checksum if you have made changes
to the plugin. Example using openssl:

```sh
openssl dgst -sha256 $GOPATH/vault-plugin-secrets-terraform
...
SHA256(.../go/bin/vault-plugin-secrets-terraform)= 896c13c0f5305daed381952a128322e02bc28a57d0c862a78cbc2ea66e8c6fa1
```

Enable the auth plugin backend using the secrets enable plugin command:

```sh
$ vault secrets enable -plugin-name='terraform' plugin
...

Successfully enabled 'plugin' at 'terraform'!
```

#### Tests

If you are developing this plugin and want to verify it is still
functioning (and you haven't broken anything else), we recommend
running the tests.

To run the tests, invoke `make test`:

```sh
$ make test
```

You can also specify a `TESTARGS` variable to filter tests like so:

```sh
$ make test TESTARGS='--run=TestTokenRole'
```

The tests assume environment variables are set to use in order to perform the
integration tests. Be sure to use appropriate values and be aware that real
tokens will be created and destroyed in the course of running the tests. 

Environment variables:

- `TFE_TOKEN`: (required) API token required by go-tfe
- `TEST_TF_TOKEN`: (required) API token used to configure the engine and make API calls
- `TEST_TF_ADDRESS`: (optional) HTTP API address if using Terraform Enterprise.
  Defaults to `https://app.terraform.io` for Terraform Cloud.
- `TEST_TF_ORGANIZATION`: The test organization to manage.
- `TEST_TF_TEAM_ID`: The *team ID* for the test Team to manage.
- `TEST_TF_USER_ID`: The *user ID* for the user to create dynamic tokens for.
