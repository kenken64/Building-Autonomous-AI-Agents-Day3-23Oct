# Deploy ClawMacdo on Railway

This folder runs the ClawMacdo workshop deployment UI on Railway. The image installs the exact requested npm version and starts the server on port **3456**:

```sh
npm install -g clawmacdo@0.95.1
clawmacdo serve --port 3456
```

ClawMacdo's UI can provision the separate Hermes server in AWS Lightsail. Prepare the credentials using the [workshop setup guide](../README.md).

## Deploy from GitHub

1. In Railway, create a project and choose **Deploy from GitHub repo**.
2. Select `kenken64/Building-Autonomous-AI-Agents-Day3-23Oct` and the `main` branch.
3. Open the service's settings and set **Root Directory** to `/railway`.
4. Set the **Railway Config File** path to `/railway/railway.toml`. This path is relative to the repository, even when a root directory is configured.
5. Leave custom build and start commands empty so Railway uses the Dockerfile.
6. Add the variables below, and attach a volume mounted at `/data` to preserve deployment records, SSH keys, backups, and the database.
7. Deploy. Under **Settings → Networking → Public Networking**, generate a domain with target port **3456**.
8. Open the HTTPS domain and sign in with your configured PIN. Select **Hermes Agent** and enter the workshop credentials in its deployment form.

Railway documents [Dockerfile detection](https://docs.railway.com/builds/dockerfiles), [subfolder configuration](https://docs.railway.com/deployments/monorepo), and [public networking](https://docs.railway.com/networking/public-networking).

## Railway variables

| Variable | Value |
| --- | --- |
| `PORT` | `3456`; set this explicitly because the startup command uses a fixed port. |
| `CLAWMACDO_BIND` | `0.0.0.0`; already set in the image. |
| `CLAWMACDO_STATE_DIR` | `/data`; already set in the image. Mount the volume at this path. |
| `CLAWMACDO_PIN` | Your private six-digit login PIN; set it before exposing the service. |
| `CLAWMACDO_API_KEY` | Optional API key for `/api/*` clients; store it as a Railway variable if used. |

The bind address makes the server reachable through Railway. A `/login` health check is configured in `railway.toml`. The [ClawMacdo v0.95.1 documentation](https://github.com/kenken64/clawmacdo/blob/v0.95.1/README.md) describes the PIN, API key, and state-directory settings.

Keep a single replica with the attached state volume. Without a volume, state stored in `/data` is lost when the container is replaced. See [Railway volumes](https://docs.railway.com/volumes).

## Build and run locally

From the repository root, build for Linux amd64. The pinned package publishes a Linux x64 binary, so use this platform on Apple Silicon too:

```sh
docker build --platform linux/amd64 -t hermes-workshop-clawmacdo:0.95.1 ./railway
```

Create a private, Git-ignored `railway/.env` file with `CLAWMACDO_PIN` set to your chosen six-digit PIN. Then run:

```sh
docker run --rm --platform linux/amd64 \
  --name hermes-workshop-clawmacdo \
  -p 127.0.0.1:3456:3456 \
  --env-file railway/.env \
  -v hermes-workshop-clawmacdo:/data \
  hermes-workshop-clawmacdo:0.95.1
```

Open [http://localhost:3456](http://localhost:3456). In another terminal, check the login endpoint:

```sh
curl --fail http://localhost:3456/login
```

## Runtime notes

- The base image is `node:24-trixie-slim`. The npm Linux binary for `0.95.1` references `GLIBC_2.39`; Debian Trixie provides a compatible runtime. Older Debian Bookworm images are insufficient for this published binary.
- AWS CLI, SSH, TLS libraries, and archive utilities are included for the Lightsail deployment workflow.
- The Docker build runs `clawmacdo --version` to catch missing native binaries or shared libraries before deployment.
- `tini` forwards termination signals to the npm wrapper and its native child process.
- If Railway returns a gateway error, confirm the service's `PORT` and domain target port are both `3456`, and that `CLAWMACDO_BIND` remains `0.0.0.0`.
