# Buildkite APIs

## Authentication

The Buildkite REST and GraphQL APIs expect an access token to be provided using the `Authorization` HTTP header:

```bash
curl -H "Authorization: Bearer $TOKEN" https://api.buildkite.com/v2/user
```

Generate an [access token](https://buildkite.com/user/api-access-tokens).

All webhooks contain an [`X-Buildkite-Token` header](/docs/apis/webhooks/pipelines#http-headers) which allows you to verify the authenticity of the request.

## API security

Buildkite is a member of the [GitHub secret scanning program](https://docs.github.com/en/code-security/secret-scanning/secret-scanning-partnership-program/secret-scanning-partner-program).
This service [alerts](https://docs.github.com/en/code-security/secret-scanning/secret-scanning-partnership-program/secret-scanning-partner-program#the-secret-scanning-process) us when a Buildkite personal API access token has been leaked on GitHub in a public repository.

Once Buildkite receives a notification of a publicly leaked token from GitHub, Buildkite will:

- Revoke the token immediately.
- Email the user who generated the token to let them know it has been revoked.
- Email the organizations associated with the token to let them know it has been revoked.

You can also:

- Enable GitHub secret scanning for [private repositories](https://docs.github.com/en/code-security/secret-scanning/enabling-secret-scanning-features/enabling-secret-scanning-for-your-repository).

- Generate a new [access token for your Buildkite user account](https://buildkite.com/user/api-access-tokens).

## REST API

The Buildkite REST API aims to give you complete programmatic access and control of Buildkite to extend, integrate and automate anything to suit your particular needs.

1. Generate an [API access token](https://buildkite.com/user/api-access-tokens) with as much [scope](/docs/apis/managing-api-tokens#token-scopes) as you need.
2. Make requests to https://api.buildkite.com using the token you generated in the `Authorization` header:

    ```bash
    curl -H "Authorization: Bearer $TOKEN" https://api.buildkite.com/v2/user
    ```

More information about the [REST API](/docs/apis/rest-api).

## GraphQL

The Buildkite GraphQL API provides an alternative to the REST API.
It allows for more efficient retrieval of data by enabling you to fetch multiple, nested resources in a single request.

1. Generate an [API access token](https://buildkite.com/user/api-access-tokens) with `GraphQL` scope
2. Open the [GraphQL explorer](https://graphql.buildkite.com/explorer)
3. Follow the [GraphQL tutorial](https://buildkite.com/blog/getting-started-with-graphql-queries-and-mutations)

More information about the [GraphQL API](/docs/apis/graphql-api).

## Webhooks

Webhooks allow you to monitor and respond to events within your Buildkite organization, providing a real time view of activity and allowing you to extend and integrate Buildkite into your systems.

Webhooks can be added and configured on your [organization's Notification Services settings](https://buildkite.com/organizations/-/services) page.

More information about the [webhooks](/docs/apis/webhooks).
