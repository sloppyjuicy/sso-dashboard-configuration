# sso-dashboard-configuration

![Build Status](https://github.com/mozilla-iam/sso-dashboard-configuration/workflows/Test%20Configuration/badge.svg)

![Codebuild Status](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiUWVHQlJNT2FjckNEcUFtUzI4VVR3ZlBTYjRCYnl4SWhWcUx0TTFEMUMzWmFMM3N2eGdLOFJMTUl6NkNtQTFkRVdXa2RzSEQ5SGYvZWRZMW01Q2cvcXhRPSIsIml2UGFyYW1ldGVyU3BlYyI6IjZjWmVyRWdkRDFFVTllRksiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

# How it works...

`apps.yml` is used both for [first stage access control](https://github.com/mozilla-iam/mozilla-iam#2-stage-access-validation)
and SSO Dashboard visibility settings.

See: [Access File documentation](https://github.com/mozilla-iam/cis/blob/master/docs/AccessFile.md)
for a complete reference. `apps.yml` is deployed to an S3 bucket by CI and made
available via the CDN at https://cdn.sso.mozilla.com/apps.yml

## Fields reference

This is a list of available fields.

```
  - application:
      ### RP Identification settings
      # This is just a name for the RP, easier for humans when reading this file
      name: "Example RP Name"

      # This is the access provider's client_id for this RP
      client_id: "xzc2030239xzxc"

      # This is the access provider name (OP: Open Id Connect Provider)
      op: auth0

      ### SSO Dashboard display settings
      # This is the URL that a user must visit to be logged into the RP. This URL would
      # either be the URL of the login button on the site (if it has one), or the URL
      # that a user gets redirected to when they visit a protected page while unauthenticated.
      url: "https://rp.example.net/oidc/authenticate"

      # A custom logo to be displayed for this RP on the SSO Dashboard
      logo: "example.png"

      # If true, will be displayed on the SSO Dashboard
      display: true

      # An URL that people can bookmark on the SSO Dashboard to login directly to that RP (i.e. not the RP frontpage)
      vanity_url: ['/an-easy-to-remember-url']

      ### Security settings
      # The list of users and groups allowed to access this RP.
      # If both authorize_users and authorized_groups are empty, everyone is allowed
      # If one is empty and the other has content, only the members of the non empty one are allowed
      # If both have content, the union of everyone in both are allowed
      # https://github.com/mozilla-iam/auth0-deploy/blob/4ac5cb4959fc93a668fcc9909ce33eac2eb8416c/rules/AccessRules.js#L173-L181
      # https://github.com/mozilla-iam/sso-dashboard/blob/a0d66f10b28654b40722f4a9e773069a8f84c629/dashboard/models/user.py#L138-L152
      # This is used by the SSO Dashboard for display purposes and for first stage access control by the Access Provider
      authorized_users: []
      authorized_groups: []

      ## Mappings to standard levels (https://infosec.mozilla.org/guidelines/risk/standard_levels) AAL
      ## values below are available at the IAM well-known endpoint
      ## (https://auth.mozilla.org/.well-known/mozilla-iam)
      # AAI is Authenticator Assurance Indicator: A Standard level which indicates the amount confidence in the
      # authentication mechanism used is required to access this RP. It is enforced by the Access Provider.
      # E.g. "MEDIUM may mean 2FA required"
      AAL: "MAXIMUM"
```

# Git workflow

In order to publish a change you must:

1. Clone the repository
2. Pull request to `master` and get it approved.
3. Have the PR merged to the `master` branch which will cause CI to deploy to
   the *dev* S3 bucket called `sso-dashboard.configuration` which can then be
   seen at https://sso.allizom.org/dashboard and is used by the Auth0 dev tenant
   rules from https://cdn.sso.allizom.org/apps.yml
4. Tag the commit in `master` that should be deployed to *production* with a
   conforming tag name. A conforming tag name for a production deploy is in the
   format of [1.2.3-prod](https://github.com/mozilla-iam/sso-dashboard-configuration/blob/26f1e5c7d8512b1a447dbac0e981fa3afbf3c346/deploy.sh#L11).
   Applying this tag will cause CI to deploy to the
   `sso-dashboard.configuration-prod` S3 bucket used by production. The easiest
   way to tag and trigger a production deployment is to [create a 'release'](https://github.com/mozilla-iam/sso-dashboard-configuration/releases),
   we are using semantic versioning (0.0.1) for this repo.

# CI Pipeline

This GitHub repo has a webhook configured which triggers the `apps_yml` AWS
CodeBuild job in the `mozilla-iam` AWS account in `us-west-2`. This CodeBuild
job follows the [`buildspec.yml`](buildspec.yml) which calls [`deploy.sh`](deploy.sh)
to deploy the change.
