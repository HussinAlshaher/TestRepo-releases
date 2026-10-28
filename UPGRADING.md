# Upgrading TestWebApp

The footer and `/Home/Updates` show the installed version. The Updates page
links to a newer stable GitHub Release when one exists. Customers choose when
to install it; opening the release page does not upgrade the site.

## Before every upgrade

1. Read the target release notes and all skipped versions for changes to
   configuration, data, and runtime requirements. Test the upgrade on a staging
   site using the same hosting setup when possible.
2. Download the **attached** `TestWebApp-vX.Y.Z.zip` and matching `.sha256` from
   the [public Releases page](https://github.com/HussinAlshaher/TestRepo-releases/releases).
   GitHub's automatically generated source archive is not a deployable app.
   Check the hash with `sha256sum -c TestWebApp-vX.Y.Z.zip.sha256` on Linux, or
   compare the checksum file with PowerShell's
   `Get-FileHash .\TestWebApp-vX.Y.Z.zip -Algorithm SHA256` on Windows.
3. Confirm that the host can run **ASP.NET Core 10**. The attached ZIP contains
   framework-dependent `dotnet publish` output, so the server needs the
   matching ASP.NET Core runtime. It is a file deployment package, not a
   container image or a universal cloud deployment bundle. Current releases
   attach only the ZIP and checksum; for image-based hosts, obtain a tested
   image or build one from the published files before deployment.
4. Back up the running site, host settings, uploads, and persistent data. Keep
   customer data outside the deployment directory; files absent from a new
   package may remain after a file copy or be removed by a platform deploy.
   Keep a copy of the previous release and any compatible data for rollback.
5. Deploy with the matching procedure below. Open `/Home/Updates` and the
   affected pages to confirm the installed version and behavior. If needed,
   restore the previous package and its compatible data.

Set `ReleaseUpdates__GitHubRepository=HussinAlshaher/TestRepo-releases` in the
host's environment settings if overriding the packaged default. Keep secrets
and customer-specific settings in the host's configuration, not in a ZIP that
will be replaced on the next upgrade.

## On-premises Windows (IIS)

Install the [.NET 10 Hosting Bundle and configure IIS](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/?view=aspnetcore-10.0)
before the first deployment. For an upgrade, stop or take the site offline,
back up its physical directory, then extract the ZIP **contents** into that
directory so `TestWebApp.dll`, `web.config`, and `wwwroot/` are at its root.
Preserve host settings and data, start the site or recycle its app pool, and
verify the URL. Coordinate any database migration with the release notes.

## On-premises Linux (systemd and reverse proxy)

Install the ASP.NET Core 10 runtime and configure a service and reverse proxy
using [Microsoft's Linux hosting guide](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-10.0).
Extract the ZIP into a new versioned directory, keep the service's working
directory at the application root, and run `dotnet TestWebApp.dll` through
systemd. Point the service at the new directory (or switch a release symlink),
restart it, and verify the proxied URL. Retain the previous directory for a
quick rollback. Ensure the service account can read the files and host settings.

## MonsterASP.NET and similar ASP.NET shared hosting

Confirm the plan supports ASP.NET Core 10. In MonsterASP's control panel, open
**Websites → Files**, upload the attached ZIP, and [unzip it into `/wwwroot`](https://help.monsterasp.net/websites/deploy/zip-file).
For an existing running .NET site, select **Overwrite existing files** and
**Restart application pool before extracting**. Check that `TestWebApp.dll`
is directly under `/wwwroot`, not in an extra folder. Back up the site first;
review any customer files in `/wwwroot` before overwriting. For another shared
host, follow its ASP.NET Core deployment instructions and check runtime support.

## Azure

- **App Service:** Select a Windows or Linux .NET 10 stack supported by the
  plan. [Deploy the attached ZIP](https://learn.microsoft.com/en-us/azure/app-service/deploy-zip)
  as published files, for example with
  `az webapp deploy --resource-group <group> --name <app> --src-path TestWebApp-vX.Y.Z.zip`.
  Put customer settings in App Service configuration. If available, deploy to
  a [staging slot](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots),
  verify it, then swap. Check the startup command on Linux if the app is not
  started automatically.
- **Azure VM:** Use the Windows or Linux procedure above. **Azure Container
  Apps / AKS:** Build and publish a container image for the target platform;
  the attached ZIP alone is not a deployable image.

## AWS

- **EC2:** Use the Windows or Linux procedure above, according to the instance
  OS. Keep application settings in the instance's environment or secret store.
- **Elastic Beanstalk:** Check that the chosen [.NET platform](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/dotnet-linux-platform.html)
  supports this runtime. Prepare an Elastic Beanstalk application bundle from
  the published files, adding a `Procfile` or other platform configuration if
  needed; test it in a separate environment before updating production.
- **ECS / EKS / App Runner:** Build and publish an appropriate container image,
  then deploy a new service revision or task definition. The release ZIP is
  input to that build, not an image to upload directly.

## Google Cloud (GCP)

- **Compute Engine:** Use the Windows or Linux procedure above for the VM OS.
- **Cloud Run / GKE:** Build and publish a Linux container image, then deploy
  that image. [Cloud Run revisions](https://docs.cloud.google.com/run/docs/deploying)
  let you verify and route traffic to a new image; the attached ZIP cannot be
  deployed directly as a Cloud Run service.

## Other common providers

- **DigitalOcean Droplets, VPS providers, and private VMs:** Use the matching
  Windows or Linux procedure. Confirm the runtime and reverse proxy setup.
- **DigitalOcean App Platform and Render:** Deploy a tested container image
  through their [image deployment](https://docs.digitalocean.com/products/app-platform/how-to/deploy-from-container-images/)
  or [prebuilt image](https://render.com/docs/deploying-an-image) workflow.
  These services do not take the release ZIP as a ready-to-run image.
- **cPanel, Plesk, and other control panels:** Check whether the specific plan
  runs ASP.NET Core 10 and how it starts the app. Some plans only serve static
  or PHP files; a file upload alone cannot make those plans run TestWebApp.

For package and runtime details, see [Microsoft's .NET deployment guide](https://learn.microsoft.com/en-us/dotnet/core/deploying/).
