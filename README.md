# agent-containers

A bunch of container-ed AI agents with some simple instructions for running
them against your local code in slightly safer ways.

Where possible, I've documented how to persist configuration from session to
session as well. As a lot of these agents store your API credentials _in_ their
configuration files, you should be really cautious about checking in config to
some place like your dotfiles repo.

## Prerequisites

You'll need either [Docker](https://www.docker.com/) (preferably running
rootless) or [podman](https://podman.io/) installed on your system to build and
launch these containers.

## Building

To build all the containers:

```bash
make all
```

To build any specific container use the directory name. For example:

```bash
make claude-code
```

### Architecture

The project uses a multi-stage build approach with a common base image
(`agent-base`) that contains shared dependencies and configuration.
Tool-specific images extend this base with their unique requirements:

```
agent-base
├── claude-code
└── openai-codex
```

Each Dockerfile uses multi-stage builds to:
- Separate build-time dependencies from runtime dependencies
- Minimize final image sizes by ~20-40%
- Improve security by reducing the attack surface
- Speed up builds through better layer caching

### Build Options

The build system supports caching control options:

```bash
# Build all containers (base, claude-code, openai-codex)
make all

# Build just the base image
make base

# Build a specific tool (automatically builds base if needed)
make claude-code

# Build without cache
make claude-code DISABLE_CACHE=1

# Build using a specific image as cache source
make claude-code CACHE_FROM=claude-code:latest
```

### Cleaning

To remove built images:

```bash
# Remove images but preserve build cache
make clean

# Remove images and prune build cache older than 24h
make deep-clean
```

## Running

See the README files in the sub-directories for instructions on initial setup,
configuration persistence and executing the AI agents in their container
environments.

For launching the containers, I recommend a small set of shell functions. These
will work in `zsh` or `bash` and automatically adjust for `podman` and
`docker`.

```bash
function __ai_container_launcher() {
  if type podman >/dev/null; then
    LAUNCHER="podman run --userns=keep-id"
  else
    LAUNCHER="docker run"
  fi
  echo $LAUNCHER
}

function cclaude() {
  local PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
  local NAME="claude-code-${PROJ}"
  docker rm -f "$NAME" 2>/dev/null || true
  local CMD="$(__ai_container_launcher) --rm --tty --interactive \
    --name \"$NAME\" \
    -v /etc/localtime:/etc/localtime:ro \
    -v ${HOME}/.claude.json:/home/node/.claude.json:rw \
    -v ${HOME}/.claude:/home/node/.claude:rw \
    -v \"$(pwd)\":/app:rw \
    claude-code"
  eval "$CMD \"\$@\""
}

function ccodex() {
  local PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
  local NAME="openai-codex-${PROJ}"
  docker rm -f "$NAME" 2>/dev/null || true
  local CMD="$(__ai_container_launcher) --rm --tty --interactive \
    --name \"$NAME\" \
    -v /etc/localtime:/etc/localtime:ro \
    -e OPENAI_API_KEY \
    -v ${HOME}/.codex:/home/node/.codex:rw \
    -v \"$(pwd)\":/app:rw \
    openai-codex"
  eval "$CMD \"\$@\""
}

function copencode() {
  local PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
  local NAME="opencode-${PROJ}"
  docker rm -f "$NAME" 2>/dev/null || true
  local CMD="$(__ai_container_launcher) --rm --tty --interactive \
    --name \"$NAME\" \
    -v /etc/localtime:/etc/localtime:ro \
    --add-host=host.docker.internal:host-gateway \
    -v $HOME/.local/state/opencode:/home/node/.local/state/opencode \
    -v $HOME/.local/share/opencode:/home/node/.local/share/opencode \
    -v $HOME/.config/opencode:/home/node/.config/opencode \
    -v $HOME/.opencode:/home/node/.opencode \
    -v \"$(pwd)\":/app:rw \
    open-code"
  eval "$CMD \"\$@\""
}

function czai() {
  local PROJ="$(basename "$(pwd)" | sed 's/[[:space:]]/-/g')"
  local NAME="claude-zai-${PROJ}"
  docker rm -f "$NAME" 2>/dev/null || true
  local CMD="$(__ai_container_launcher) --rm --tty --interactive \
    --name \"$NAME\" \
    -v /etc/localtime:/etc/localtime:ro \
    -v ${HOME}/.claude.json:/home/node/.claude.json:rw \
    -v ${HOME}/.claude:/home/node/.claude:rw \
    -v \"$(pwd)\":/app:rw \
    -e ANTHROPIC_BASE_URL=https://api.z.ai/api/anthropic \
    -e ANTHROPIC_AUTH_TOKEN=TOKEN \
    -e API_TIMEOUT_MS=3000000 \
    -e CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1 \
    -e ANTHROPIC_DEFAULT_HAIKU_MODEL=glm-4.7-flash \
    -e ANTHROPIC_DEFAULT_SONNET_MODEL=glm-4.7 \
    -e ANTHROPIC_DEFAULT_OPUS_MODEL=glm-4.7 \
    claude-code"
  eval "$CMD \"\$@\""
}
```

Put those some place in your `.zshrc` or `.bashrc` file and you'll be able to
launch the agent in a working directory with a call to `claude` or `codex`. You
can test they work by getting a bash shell in them with `claude bash` or
`claude codex`.

## See Also

* [My dotfiles](https://github.com/ianchesal/dotfiles)
