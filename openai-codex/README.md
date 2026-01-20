# OpenAI Codex

## Build Instructions

Use the top level `Makefile` to build this. It injects things in to the build
so the container works correctly for your user under `podman`.

```bash
make openai-codex
```

You can add more local tools to the container to be installed via `apt-get` by
extending the `LOCAL_TOOLS` list in the top-level `Makefile`.

## First time setup

You only need to do this once. It helps persist your configuration between
sessions.

There are no sensitive secrets in the configuration files here so they should
be fine to check in to your dotfiles repository.

Configuration goes in the `config.yaml` file and custom instructions go in
`instructions.md`. See [the
docs](https://github.com/openai/codex?tab=readme-ov-file#configuration) for
more details.

```bash
mkdir -p ~/.config/codex
touch ~/.config/codex/config.yaml
touch ~/.config/codex/instructions.md
```

The local codex configuration path is mounted in at `~/.codex` if you launch
the container as below.

## Run Instructions

Note: If you're running rootless `podman` you'll need to add `--userns=keep-id`
to these instructions.

Note: See the repo [README](../README.md) for some nice shell functions for
launching these containers.

```bash
#!/usr/bin/env bash

PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
NAME="openai-codex-${PROJ}"

# Remove any existing container with this name
docker rm -f "$NAME" 2>/dev/null || true

docker run -it --rm \
  --name "$NAME" \
  -v /etc/localtime:/etc/localtime:ro \
  -e OPENAI_API_KEY \
  -v ${HOME}/.codex:/home/node/.codex:rw \
  -v "$(pwd)":/app:rw \
  openai-codex
```

## References

* [Github](https://github.com/openai/codex)
* You may prefer [this method for running codex containered](https://github.com/openai/codex/blob/main/codex-cli/scripts/run_in_container.sh). I personally don't like a persistent codex instance like that.
