# Tinfoil Containers Template

This is a GitHub template repository for deploying [Tinfoil Containers](https://docs.tinfoil.sh/containers/overview) — Docker containers that run in secure enclaves.

## Getting Started

1. Click **[Use this template](https://github.com/tinfoilsh/tinfoil-containers-template/generate)** → **Create a new repository**
2. Edit `tinfoil-config.yml` in your new repo — set your container image, ports, and paths
3. Commit and push your config:
   ```bash
   git add tinfoil-config.yml
   git commit -m "chore: configure deployment"
   git push origin main
   ```
4. Release a version by running the **Tinfoil Release** workflow:
   - **CLI:** `gh workflow run tinfoil-release.yml -f version=v0.0.1`
   - **UI:** Go to the **Actions** tab → **Tinfoil Release** → **Run workflow**, then enter the version (e.g. `v0.0.1`)

   The workflow tags the release and publishes it.
5. Go to the [Tinfoil Dashboard](https://dash.tinfoil.sh), navigate to **Containers** → **Deploy**, select your repo and tag, and click **Deploy**

Your container will be live at `https://<container-name>.<org>.containers.tinfoil.dev` once the deployment completes.

For a fully working example, see [tinfoil-containers-hello-world](https://github.com/tinfoilsh/tinfoil-containers-hello-world).

## Updating

Edit `tinfoil-config.yml`, commit, then release a new version by running the **Tinfoil Release** workflow (`gh workflow run tinfoil-release.yml -f version=v0.0.2`, or via the **Actions** tab). Then click **Update** in the dashboard. Each release creates an auditable record in the Sigstore transparency log.

## Documentation

For the full configuration reference, secrets management, debug mode, and more:

**[docs.tinfoil.sh/containers](https://docs.tinfoil.sh/containers/overview)**

## Support

- [Documentation](https://docs.tinfoil.sh)
- [Email Support](mailto:contact@tinfoil.sh)
