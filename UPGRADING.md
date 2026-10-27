# Upgrading TestWebApp

The app shows its installed version in the footer and on `/Home/Updates`. That
page links to a newer stable release when one is available. Viewing or
downloading a release does not install it; each customer chooses when to
upgrade.

## Common steps

1. Read the release notes for the target version and any versions skipped.
   Check for changes to runtime requirements, configuration, or data.
2. Confirm the host supports the package. The current ZIP is a
   **framework-dependent ASP.NET Core 10** build and requires a compatible
   ASP.NET Core runtime on the host. If the host cannot support it, request a
   compatible build and verify that the host can run that deployment mode.
3. Download the attached application ZIP and `.sha256` file from the GitHub
   Release. Verify the ZIP's SHA-256 hash against the checksum file.
4. Back up the current application, host-specific settings, uploads, and any
   persistent data. Keep settings and customer data outside files replaced by
   the deployment, or restore them from the backup afterward.
5. Deploy through the hosting provider's normal process, then restart the app
   if that process requires it. Check `/Home/Updates` and the affected pages.
   If the upgrade fails, restore the previous package and its compatible data.

## Choose the host's deployment process

| Hosting setup | Check before deploying |
| --- | --- |
| MonsterASP.NET or other shared hosting | Confirm ASP.NET Core 10 support and use the provider's publishing or file-upload process. |
| Azure App Service | Confirm the App Service runtime and use its configured deployment method. Preserve App Service settings. |
| AWS | Identify the actual service first: EC2, Elastic Beanstalk, and containers use different deployment packages and steps. A container needs an image built from the release, rather than this ZIP alone. |
| On-premise | Have the server administrator confirm the runtime or IIS Hosting Bundle, deploy to the configured site, and restart the service as needed. |

The same version number and release notes apply across providers. Deployment
commands depend on the customer's hosting setup. For runtime and packaging
details, see [Microsoft's .NET publishing guide](https://learn.microsoft.com/en-us/dotnet/core/deploying/)
and [ASP.NET Core hosting guide](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/).
