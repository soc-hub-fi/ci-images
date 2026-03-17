# syntax=docker/dockerfile:1

FROM debian:bookworm-slim

# Avoid interactive prompts
ENV DEBIAN_FRONTEND=noninteractive
ENV RUST_VERSION=1.94.0
ENV USER=builder
ENV UID=1000

# ---- System deps ----
RUN apt-get update && apt-get install -y \
    ca-certificates \
    curl \
    git \
    build-essential \
    pkg-config \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# ---- Non-root user ----
RUN useradd -m -u ${UID} -s /bin/bash ${USER}

USER ${USER}
WORKDIR /home/${USER}

# ---- Rust (pinned) ----
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --profile minimal --default-toolchain ${RUST_VERSION}

ENV PATH="/home/${USER}/.cargo/bin:${PATH}"

# ---- Cargo cache dirs ----
# These make cache mounts cleaner & predictable
ENV CARGO_HOME=/home/${USER}/.cargo
ENV RUSTUP_HOME=/home/${USER}/.rustup

# ---- Install Bender with cache mounts ----
RUN --mount=type=cache,target=/home/${USER}/.cargo/registry \
    --mount=type=cache,target=/home/${USER}/.cargo/git \
    cargo install bender

CMD ["/bin/bash"]
