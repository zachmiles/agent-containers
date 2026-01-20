# Build stage
FROM agent-base AS builder

# Install build dependencies and Claude Code
RUN npm install -g @anthropic-ai/claude-code && \
  echo "Installed Claude Code version: $(claude --version)"

# Runtime stage
FROM agent-base

# Copy the installed Claude Code from builder
# We must use --chown to ensure the node user owns the copied files
COPY --from=builder --chown=node:node /home/node/.npm-global /home/node/.npm-global

# Create placeholder file for .claude.json so Docker binds it as a file, not a folder
RUN touch /home/node/.claude.json && chown node:node /home/node/.claude.json

# Create .local/bin directory and symlink claude to satisfy native install method check
RUN mkdir -p /home/node/.local/bin && \
    chown -R node:node /home/node/.local && \
    ln -s /home/node/.npm-global/bin/claude /home/node/.local/bin/claude

# Add .local/bin to PATH for native install method
ENV PATH=/home/node/.local/bin:$PATH

# Display version information
RUN echo "" && \
  echo "\e[34m=================================================================\e[0m" && \
  echo "\e[34m                   CLAUDE CODE VERSION                           \e[0m" && \
  echo "\e[34m=================================================================\e[0m" && \
  echo "\e[34m" && claude --version && \
  echo "\e[0m" && \
  echo ""

CMD ["claude", "--dangerously-skip-permissions"]
