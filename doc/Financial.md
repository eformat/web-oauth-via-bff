# How to run the SPA with the Financial-grade Token Handler

## Overview

The token handler uses the following behaviour:

- OAuth work is done for the SPA by an API
- Authorization Code Flow with PKCE is used, along with Mutual TLS client authentication
- [PAR](https://tools.ietf.org/id/draft-lodderstedt-oauth-par-00.html) and [JARM](https://openid.net/specs/openid-financial-api-jarm.html) are also used, as state-of-the-art security features
- Only the strongest `SameSite=strict` cookies are used in the browser
- The code example uses HTTPS for all components

## Configure Development Domains

Add these entries to your /etc/hosts file:

```bash
127.0.0.1 localhost www.example.com api.example.com login.example.com
:1        localhost
```

## Install Prerequisites

Ensure that these tools are installed locally:

- [Node.js](https://nodejs.org/en/download/)
- [Java 11 or later](https://openjdk.java.net/projects/jdk/11/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [jq](https://stedolan.github.io/jq/download/)

Contact Curity to get a trial license with access to financial grade features.\
Then copy the `license.json` file to the folder where you have cloned this repository.

## Build the Code

This will compile projects, build Docker images and generate development certificates:

```bash
./build.sh financial
```

## Configure SSL Trust

Configure the browser to trust the root certificate authority at `./resources/financial/certs/example.ca.pem`.\
For most browsers this can be done by importing it to the system trust store, eg Keychain Access / System / Certificates.

## Deploy the System

Then run this script to spin up all components in a small Docker Compose network:

```bash
./deploy.sh financial
```

## Use the System

Then browse to https://www.example.com and sign in with the following test user name and password:

- **demouser / Password1**

The SPA has an initial unauthenticated view to focus on triggering a login:

![Unauthenticated View](/doc/ui-unauthenticated-financial.png)

The authenticated view demonstrates multi-tab browsing, which works reliably in all browsers:

![Authenticated View](/doc/ui-authenticated-financial.png)

## Deployed System

Once the system is deployed you can also browse to these URLs:

- Sign in to the [Curity Admin UI](https://localhost:6749/admin) with credentials `admin / Password1`
- Browse to the [Identity Server Metadata Endpoint](https://login.example.com:8443/oauth/v2/oauth-anonymous/.well-known/openid-configuration)
- Browse to the SPA's [Token Handler API Base URL](https://api.example.com:3000/tokenhandler), which is the OAuth agent
- Browse to the [Example API Base URL](https://api.example.com:3000/api), which uses the OAuth proxy to forward JWTs to APIs

## Token Handler Details

- To better understand deployment, see the [SPA Deployments](https://github.com/curityio/spa-deployments) repository.
- To better understand how the token handler works, see the [SPA Financial Grade Token Handler](https://github.com/curityio/token-handler-kotlin-spring-fapi).

## Troubleshoot

If you need to troubleshoot then access token handler related logs via the following commands:

```bash
export TOKEN_HANDLER_CONTAINER_ID=$(docker container ls | grep token-handler-api | awk '{print $1}')
docker logs -f $TOKEN_HANDLER_CONTAINER_ID
```

```bash
export REVERSE_PROXY_CONTAINER_ID=$(docker container ls | grep reverse-proxy | awk '{print $1}')
docker logs -f $REVERSE_PROXY_CONTAINER_ID
```

## Free Resources

When finished with your development session, run the following script to free resources:

```bash
./teardown.sh financial
```
