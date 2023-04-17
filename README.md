# NGINX + OAuth Proxy 2 Demostration

## Initial values to set up

Unfortunately because the set-up needs to deal with secret values and also OAuth2 sign-up (Azure
AD), we cannot simply push every value as default values and be able to just spin up the
docker-compose configuration without any initialization work.

The guide here will tell exactly which values are required to do the above initialization work:

Before starting, make a copy of `.env.sample` into `.env`. The `.env` is git-ignored by default, and
will be automatically fed into the `docker-compose.yml` later.

### `OAUTH2_PROXY_COOKIE_SECRET`

This can be any reasonably cryptographically secure value, so you can generate
like this on Bash:

```bash
dd if=/dev/urandom bs=32 count=1 2>/dev/null | base64 | tr -d -- '\n' | tr -- '+/' '-_'; echo
```

Ref: <https://oauth2-proxy.github.io/oauth2-proxy/docs/configuration/overview>:

### `OAUTH2_PROXY_CLIENT_ID`, `OAUTH2_PROXY_CLIENT_SECRET`, `OAUTH2_PROXY_AZURE_TENANT`

Unfortunately this section is the hardest. You will need Microsoft Azure (with access to Azure AD),
which does provide free trial for a year.

You will be required to create a new tenant and set up a new OAuth2 application in Azure AD, which
you can follow the guides:

- <https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant>
- <https://learn.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad>

Roughly, you should be:

1. Log in and getting into Azure AD (Active Directory) at
   <https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade>
2. Register an application:
   <https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/CreateApplicationBlade/isMSAApp~/false>
   - Change the Redirect URI to `http://localhost:8080/oauth2/callback`.
3. Copy the following values:
   - `Application (client) ID` as `OAUTH2_PROXY_CLIENT_ID` in `.env`
   - `Directory (tenant) ID` as `OAUTH2_PROXY_AZURE_TENANT` in `.env`
   - For `OAUTH2_PROXY_CLIENT_SECRET`, go to:
     - `Certificates & secrets` on the left pane,
     - `Client secrets`,
     - `+ New client secret` and enter any description text and any reasonable expiration date. It
     should create a new secret entry. Copy the `Value` (not the `Secret ID`) as
     `OAUTH2_PROXY_CLIENT_SECRET` in `.env`.

## Overall

Finally with all the secret values in place, run both services with:

```bash
docker-compose up --build
```

Then use your web browser and go into:
<http://localhost:8080>

If you have not logged in before, you should see a OAuth2 Proxy page with `Sign in with Azure`
button.

If the above doesn't appear, and you instead see the NGINX page `Welcome to nginx!`, it means you
have already logged in. If you prefer to want to retry the log in step to confirm the root page is
being secured by OAuth Proxy 2, you can either:

- Use an incognito mode to start a new session
- Remove all the following cookies in <http://localhost:8080>, in particular:
  - `_oauth2_proxy_0`
  - `_oauth2_proxy_1`
