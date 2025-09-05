# Buildkite APIs

## Authentication

The Buildkite REST and GraphQL APIs expect an access token to be provided using the `Authorization` HTTP header:

```bash
curl -H "Authorization: Bearer $TOKEN" https://api.buildkite.com/v2/user
```

Generate an [access token](https://buildkite.com/user/api-access-tokens).

All webhooks for [Pipelines](/docs/apis/webhooks/pipelines#http-headers) and [Package Registries](/docs/apis/webhooks/package-registries#http-headers) contain an `X-Buildkite-Token` header which allows you to verify the authenticity of the request.

## Managing API access tokens

Learn more about how to manage Buildkite's API access tokens in [Managing API access tokens](/docs/apis/managing-api-tokens), which covers the following topics:

- The [scopes](/docs/apis/managing-api-tokens#token-scopes) which can be assigned to API access tokens.
- [Auditing](/docs/apis/managing-api-tokens#auditing-tokens) token usage.
- [Removing](/docs/apis/managing-api-tokens#removing-an-organization-from-a-token) Buildkite organization access to tokens.
- [Limiting](/docs/apis/managing-api-tokens#limiting-api-access-by-ip-address) a token's access by IP address.
- A token's [lifecycle](/docs/apis/managing-api-tokens#api-token-lifecycle) characteristics.
- Managing a token's [security](/docs/apis/managing-api-tokens#api-token-security), including [token rotation](/docs/apis/managing-api-tokens#api-token-security-rotation) and [GitHub's secret scanning program](/docs/apis/managing-api-tokens#api-token-security-github-secret-scanning-program).

## REST API

The Buildkite REST API aims to give you complete programmatic access and control of Buildkite to extend, integrate and automate anything to suit your particular needs.

1. Generate an [API access token](https://buildkite.com/user/api-access-tokens) with as much [scope](/docs/apis/managing-api-tokens#token-scopes) as you need.
2. Make requests to https://api.buildkite.com using the token you generated in the `Authorization` header:

    ```bash
    curl -H "Authorization: Bearer $TOKEN" https://api.buildkite.com/v2/user
    ```

Learn more about Buildkite's REST API in the [REST API overview](/docs/apis/rest-api).

## GraphQL

The Buildkite GraphQL API provides an alternative to the REST API. The GraphQL API allows for more efficient retrieval of data by enabling you to fetch multiple, nested resources in a single request.

You can access the GraphQL API through the _GraphQL console_ (see the [GraphQL overview](/docs/apis/graphql-api) page > [Getting started](/docs/apis/graphql-api#getting-started) section for more information), as well as at the command line (see the [Console and CLI tutorial](/docs/apis/graphql/graphql-tutorial) page for more information). For command line access, you'll need a Buildkite [API access token](https://buildkite.com/user/api-access-tokens) with the **Enable GraphQL API Access** permission selected.

## Webhooks

Buildkite's webhooks allow your third-party applications and systems to monitor and respond to events within your Buildkite organization, providing a real time view of activity and allowing you to extend and integrate Buildkite into these systems.

For Pipelines, webhooks can be [added and configured](/docs/apis/webhooks/pipelines#add-a-webhook) on your Buildkite organization's [**Notification Services** settings](https://buildkite.com/organizations/-/services) page.

For Test Engine and Package Registries, webhooks can be configured through their specific [test suites](/docs/apis/webhooks/test-engine#add-a-webhook) and [registries](/docs/apis/webhooks/package-registries#add-a-webhook), respectively.

Learn more about Buildkite's webhooks from the [Webhooks overview](/docs/apis/webhooks) page.

## Portals

Buildkite's portals provide an alternative to the REST and GraphQL APIs, which allow Buildkite organization administrators to define custom GraphQL operations that can then be accessed through an authenticated URL endpoint by other members of the Buildkite organization.

Learn more about the portals feature in [Portals](/docs/apis/portals).
