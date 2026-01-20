# Aider

## Installation Instructions

```bash
docker pull paulgauthier/aider
```

## Run Instructions

```bash
#!/usr/bin/env bash

PROJ="$(basename "$(pwd)")"
NAME="aider-${PROJ}"

# Remove any existing container with this name
docker rm -f "$NAME" 2>/dev/null || true

docker run -it --rm \
  --name "$NAME" \
  --user $(id -u):$(id -g) \
  -e OPENAI_API_KEY \
  -v $(pwd):/app:rw \
  -e ANTHROPIC_API_KEY \
  paulgauthier/aider
```

## References

* [Aider](https://aider.chat)
* [Docs](https://aider.chat/docs/)
* [Github](https://github.com/Aider-AI/aider)
