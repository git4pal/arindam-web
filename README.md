# arindam-web

Static site hosted on Azure Static Web Apps, using built-in Microsoft Entra ID
and GitHub sign-in. Only invited users with the `site_user` role can access the
site.

## Deployment and access

1. Deploy these changes using the existing Azure Static Web Apps GitHub Actions
   workflow. The root `staticwebapp.config.json` must be included in the deployed
   content; the current workflow deploys directly from the repository root.
2. In Azure Portal, open the Static Web App, select **Role management**, and
   create an invitation for each approved user. Select their identity provider
   and assign the custom role `site_user` (exact spelling).
3. Send each user their invitation link privately. They must accept it with the
   intended account and use the same provider when signing in.
4. In the GitHub repository's **Settings > Pages**, unpublish any previously
   deployed Pages site. Its deployment workflow has been removed, but removing
   the workflow does not unpublish an existing deployment.

Unauthenticated requests are redirected to `/login.html`. Signed-in users
without `site_user` receive the login page with HTTP 403 and instructions to
request access. All other site routes, including `/` and `/index.html`, require
the role. The `/.auth/*` endpoints remain public so authentication, logout, and
invitation acceptance work. Azure validates the session before serving protected
content; no password JSON, GUID header, or browser-only authorization is used.
Responses use `Cache-Control: no-store` to avoid caching protected HTML.

## Verify after deployment

- In a private browser window, visit `/` and `/index.html`: both should redirect
  to the login page without returning the retail page's HTML.
- Sign in with an account that has not accepted an invitation: protected routes
  should return HTTP 403 and show the access instructions.
- Accept an invitation granting `site_user`, then sign in: the retail page should
  load. Test each identity provider you use for invitations.
- Select **Sign out**, then request `/index.html` again: login should be required.
- Confirm the former GitHub Pages URL no longer serves the site.

A plain local static file server does not enforce Azure's authentication rules.
Authentication and role enforcement must be verified on Azure (or with the Azure
Static Web Apps CLI emulator, whose local identities are simulated).
CLI 2.0.7 redirects users without the required role to login instead of returning
Azure's documented HTTP 403; verify the 403 experience after deployment.

This protects hosted content, not source files in a public GitHub repository.
Do not commit passwords or confidential content; use a private repository if the
HTML source itself must be private.

See Microsoft's [authentication documentation](https://learn.microsoft.com/azure/static-web-apps/authentication-authorization)
and [routing configuration reference](https://learn.microsoft.com/azure/static-web-apps/configuration).