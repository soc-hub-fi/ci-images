# Use a small, stable base
FROM debian:bookworm-slim

# Avoid interactive prompts during install
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    git \
    build-essential \
    pkg-config \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Rust (needed because Bender is distributed via Cargo)
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --profile minimal
ENV PATH="/root/.cargo/bin:${PATH}"

# Install Bender
RUN cargo install bender

# Default shell
CMD ["/bin/bash"]