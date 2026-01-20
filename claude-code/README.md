# Claude Code

## Build Instructions

Use the top level `Makefile` to build this. It injects things in to the build
so the container works correctly for your user under `podman`.

```bash
make claude-code
```

You can add more local tools to the container to be installed via `apt-get` by
extending the `LOCAL_TOOLS` list in the top-level `Makefile`.

## First time setup

You only need to do this once. It helps persist your configuration between
sessions and ensures you don't have to fetch a new Claude Code API key _every_
time you start a new container instance. Because this file will hold an API key
you should be _very_ careful to protect it! DO NOT check this file in to your
dotfiles repo!

```bash
mkdir -p ~/.claude
touch ~/.claude.json
chmod 700 ~/.claude
chmod 600 ~/.claude.json
```

The first time you run `claude` it will complain about the `config.json` file
being invalid configuration JSON. Just select the "Reset to default" setting
from the options it presents and it will not complain again.

Authorize `claude` once and it'll persist your authorization in the
`~/.claude.json` file and not ask again.

You probably want to persist these settings as well:

```
claude config set -g autoUpdaterStatus disabled
```

## Run Instructions

Note: If you're running rootless `podman` you'll need to add `--userns=keep-id`
to these instructions.

Note: See the repo [README](../README.md) for some nice shell functions for
launching these containers.

Run the container:

```bash
#!/usr/bin/env bash

PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
NAME="claude-code-${PROJ}"

# Remove any existing container with this name
docker rm -f "$NAME" 2>/dev/null || true

docker run -it --rm \
  --name "$NAME" \
  -v /etc/localtime:/etc/localtime:ro \
  -v ${HOME}/.claude.json:/home/node/.claude.json:rw \
  -v ${HOME}/.claude:/home/node/.claude:rw \
  -v $(pwd):/app:rw \
  claude-code
```

You can obviously make that a shell alias for ease of use.

If you want an instance of the container with a shell so you can explore inside
or use the `claude` CLI to change and persist settings and what not just run:

```bash
#!/usr/bin/env bash

PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
NAME="claude-code-shell-${PROJ}"

# Remove any existing container with this name
docker rm -f "$NAME" 2>/dev/null || true

docker run -it --rm \
  --name "$NAME" \
  -v /etc/localtime:/etc/localtime:ro \
  -v ${HOME}/.claude.json:/home/node/.claude.json:rw \
  -v ${HOME}/.claude:/home/node/.claude:rw \
  -v "$(pwd)":/app:rw \
  claude-code \
  bash
```

### Z.AI Variant

To run Claude Code with Z.AI as the backend:

```bash
#!/usr/bin/env bash

PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
NAME="claude-zai-${PROJ}"

# Remove any existing container with this name
docker rm -f "$NAME" 2>/dev/null || true

docker run -it --rm \
  --name "$NAME" \
  -v /etc/localtime:/etc/localtime:ro \
  -v ${HOME}/.claude.json:/home/node/.claude.json:rw \
  -v ${HOME}/.claude:/home/node/.claude:rw \
  -v "$(pwd)":/app:rw \
  -e ANTHROPIC_BASE_URL=https://api.z.ai/api/anthropic \
  -e ANTHROPIC_AUTH_TOKEN=TOKEN \
  -e API_TIMEOUT_MS=3000000 \
  -e CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1 \
  -e ANTHROPIC_DEFAULT_HAIKU_MODEL=glm-4.7-flash \
  -e ANTHROPIC_DEFAULT_SONNET_MODEL=glm-4.7 \
  -e ANTHROPIC_DEFAULT_OPUS_MODEL=glm-4.7 \
  claude-code
```

## References

* [Documentation](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
* [Github Repo](https://github.com/anthropics/claude-code)
