---
title: "Gitea"
description: "Integrating Gitea with the Authelia OpenID Connect Provider."
lead: ""
date: 2022-07-01T13:07:02+10:00
draft: false
images: []
menu:
  integration:
    parent: "openid-connect"
weight: 620
toc: true
community: true
---

## Tested Versions

* [Authelia]
  * [v4.36.3](https://github.com/authelia/authelia/releases/tag/v4.36.3)
* [Gitea]
  * [1.17.0](https://github.com/go-gitea/gitea/releases/tag/v1.17.0)

## Before You Begin

{{% oidc-common %}}

### Assumptions

This example makes the following assumptions:

* __Application Root URL:__ `https://gitea.example.com`
* __Authelia Root URL:__ `https://auth.example.com`
* __Client ID:__ `gitea`
* __Client Secret:__ `insecure_secret`
* __Authentication Name (Gitea):__ `authelia`:
    * This option determines the redirect URI in the format of
      `https://gitea.example.com/user/oauth2/<Authentication Name>/callback`.
      This means if you change this value you need to update the redirect URI.

## Configuration

### Application

To configure [Gitea] to utilize Authelia as an [OpenID Connect 1.0] Provider:

1. Expand User Options
2. Visit Site Administration
3. Visit Authentication Sources
4. Visit Add Authentication Source
5. Configure:
   1. Authentication Name: `authelia`
   2. OAuth2 Provider: `OpenID Connect`
   3. Client ID (Key): `gitea`
   4. Client Secret: `insecure_secret`
   5. OpenID Connect Auto Discovery URL: `https://auth.example.com/.well-known/openid-configuration`

{{< figure src="gitea.png" alt="Gitea" width="300" >}}

To configure [Gitea] to perform automatic user creation for the `auth.example.com` domain via [OpenID Connect 1.0]:

1. Edit the following values in the [Gitea] `app.ini`:
```ini
[openid]
ENABLE_OPENID_SIGNIN = false
ENABLE_OPENID_SIGNUP = true
WHITELISTED_URIS     = auth.example.com

[service]
DISABLE_REGISTRATION                          = false
ALLOW_ONLY_EXTERNAL_REGISTRATION              = true
SHOW_REGISTRATION_BUTTON                      = false
```

Take a look at the [See Also](#see-also) section for the cheatsheets corresponding to the sections above for their
descriptions.

### Authelia

The following YAML configuration is an example __Authelia__
[client configuration](../../../configuration/identity-providers/open-id-connect.md#clients) for use with [Gitea] which
will operate with the above example:

```yaml
identity_providers:
  oidc:
    ## The other portions of the mandatory OpenID Connect 1.0 configuration go here.
    ## See: https://www.authelia.com/c/oidc
    clients:
    - id: gitea
      description: Gitea
      secret: '$pbkdf2-sha512$310000$c8p78n7pUMln0jzvd4aK4Q$JNRBzwAo0ek5qKn50cFzzvE9RXV88h1wJn5KGiHrD0YKtZaR/nCb2CJPOsKaPK0hjf.9yHxzQGZziziccp6Yng'  # The digest of 'insecure_secret'.
      public: false
      authorization_policy: two_factor
      redirect_uris:
        - https://gitea.example.com/user/oauth2/authelia/callback
      scopes:
        - openid
        - email
        - profile
      userinfo_signing_algorithm: none
```

## See Also

- [Gitea] app.ini [Config Cheat Sheet](https://docs.gitea.io/en-us/config-cheat-sheet):
  - [OpenID](https://docs.gitea.io/en-us/config-cheat-sheet/#openid-openid)
  - [Service](https://docs.gitea.io/en-us/config-cheat-sheet/#service-service)

[Gitea]: https://gitea.io/
[OpenID Connect 1.0]: ../../openid-connect/introduction.md
