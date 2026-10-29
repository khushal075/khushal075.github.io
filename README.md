# Khushal's Systems Playground

A responsive engineering portfolio with an animated architecture diagram and interactive presence simulation. All interactions run in the browser; no API keys or build tools are required.

## Publish with GitHub Pages

1. Create a public repository named `khushal075.github.io` under `khushal075` (or reuse it if it already exists).
2. Upload `index.html` and `.nojekyll` to the repository root on the `main` branch.
3. In **Settings → Pages**, choose **Deploy from a branch**, then **main** and **/ (root)**, and save.
4. GitHub will show the deployed URL after publication: https://khushal075.github.io/

A user-site repository with that name should be checked for existing content before uploading anything. Keep the separate `khushal075` profile repository for the profile README.

## Local preview

Open `index.html` in a browser. There are no dependencies, backend services, or remote scripts.

## Demo behavior

The client slider changes the simulated session count. Disconnecting Node A marks it offline immediately, then expires its sessions after three seconds. Reconnect restores its sessions. Reset restores the initial 120-client state.

The graph is simulated session history, not live production telemetry. AI chat is not implemented.
