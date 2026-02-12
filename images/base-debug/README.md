# Base Debugging Image

This is the common base image for all containerized debugging images at Gepardec. 
It's built on **Debian 12 (Bookworm) Slim** and includes a set of essential networking, process, and file management utilities.

Other specialized debugging images (e.g., for Java, k8s ..) can extend this base image to provide additional tools.

## Contents

| Package           | Description                                                                         |
|-------------------|-------------------------------------------------------------------------------------|
| `ca-certificates` | Contains SSL certificates to allow tools like `curl` to access HTTPS sites.         |
| `curl` & `wget`   | Used to make HTTP requests to test endpoints or download files from the web.        |
| `git`             | Allows you to clone repositories or inspect Git-related files.                      |
| `procps`          | Provides `ps` and `top` to view and monitor running processes inside the container. |
| `net-tools`       | Includes classic networking tools like `netstat` and `ifconfig`.                    |
| `iproute2`        | Provides modern networking tools like `ss` and `ip` for inspecting sockets/routes.  |
| `dnsutils`        | Contains `dig` and `nslookup` for diagnosing DNS resolution issues.                 |
| `unzip` & `zip`   | For creating and extracting zip archives.                                           |
| `openssl`         | Certificates, SSH and other cryptographic things                                    |
| `vim-tiny`        | A minimal but powerful text editor for viewing or editing configuration files.      |


## Key Features

Non-Root by Default: The container runs with a non-root user (`debuguser`) to enhance security and comply with best practices.

---

## Usage

To start a shell session inside this container for general-purpose debugging, run the following command:

```bash
podman run -it --rm ghcr.io/gepardec/debug-images/base-debug:latest
```
