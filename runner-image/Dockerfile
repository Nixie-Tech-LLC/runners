# Custom ARC runner image — the upstream actions-runner plus the tools our CI installs on every job
# (docker CLI + buildx for dind builds, git/jq/unzip). Pre-baking them trims per-job setup, complementing
# the warm-runner + workflow-cache work. Extend freely (tofu, kubectl, language toolchains) as CI needs grow.
# Base tag unpinned here; Renovate/Dependabot can pin the digest.
FROM ghcr.io/actions/actions-runner:latest

USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates curl gnupg git jq unzip \
 && install -m 0755 -d /etc/apt/keyrings \
 && curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc \
 && chmod a+r /etc/apt/keyrings/docker.asc \
 && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
      > /etc/apt/sources.list.d/docker.list \
 && apt-get update \
 && apt-get install -y --no-install-recommends docker-ce-cli docker-buildx-plugin \
 && rm -rf /var/lib/apt/lists/*
USER runner
