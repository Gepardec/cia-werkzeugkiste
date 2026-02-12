## Debug Images

Each folder in this directory contains a Containerfile for an image that can be used for debugging and troubleshooting in customer environments. 
The images are built and kept up-to-date automatically. Every week a rebuild of the images is triggered, this ensures packages inside the image are up to date.
Dependabot is configured to watch for updates of the base images and will create a pull request in case of an update.

## Image "base-debug"

This image can be used as a base image for other debug images. It also has enough utilities installed to be used as a standalone image for debugging and troubleshooting.

## Creating a new debug image

To create a new debug image, simply create a new folder in this directory and add a Containerfile describing the image.
Make sure to also add the image to the build-and-push workflow in the .github/workflows/build-and-push.yml file, so that the image is built and pushed to the container registry automatically.
More details about the workflow configuration can be found in the workflow file comments.

## Image signing 

the images are signed using cosign and the signatures are stored in the same container registry as the images.
To verify the signature of an image, you can use the following command:

```bash
podman run --rm gcr.io/projectsigstore/cosign:v2.2.3 verify \
  --certificate-identity-regexp "https://github.com/Gepardec/cia-werkzeugkiste" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  ghcr.io/gepardec/debug-images/base-debug@sha256:<digest here>
```

If you want the output to be more readable use this (credits to my LLM)
```bash
podman run --rm gcr.io/projectsigstore/cosign:v2.2.3 verify \
  --certificate-identity-regexp "https://github.com/Gepardec/cia-werkzeugkiste" \
  --certificate-oidc-issuer "https://token.actions.githubusercontent.com" \
  ghcr.io/gepardec/debug-images/base-debug@sha256:<digest here> \ 
  | jq -r '.[0] | "✅ VALID SIGNATURE FOUND\n------------------------\nSIGNED BY: \(.optional.Subject)\nISSUER:    \(.optional.Issuer)\nDIGEST:    \(.critical.image."docker-manifest-digest")\nWORKFLOW:  \(.optional.githubWorkflowName)\nREF:       \(.optional.githubWorkflowRef)"'
```
