# syntax=docker/dockerfile:1

FROM ubuntu:24.04

# Avoid interactive prompts
ENV DEBIAN_FRONTEND=noninteractive
ARG VERILATOR_VERSION=5.020-1
ARG RUST_VERSION=1.94.0
ARG BENDER_VERSION=0.29.1
ARG RISCV32_ELF_TOOLCHAIN_URL=https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2023.07.05/riscv32-elf-ubuntu-22.04-gcc-nightly-2023.07.05-nightly.tar.gz
ARG RISCV_PREFIX=/opt/riscv

ENV USER=builder
ENV UID=1000

# ---- System deps ----
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    git \
    python3 \
    tar \
    build-essential \
    pkg-config \
    zlib1g-dev \
    libssl-dev \
    xxd \
    verilator=${VERILATOR_VERSION} \
    && rm -rf /var/lib/apt/lists/*

# ---- riscv32-elf toolchain ----
RUN mkdir -p "${RISCV_PREFIX}" \
    && curl -fsSL "${RISCV32_ELF_TOOLCHAIN_URL}" -o /tmp/riscv32-elf-toolchain.tgz \
    && tar -xzf /tmp/riscv32-elf-toolchain.tgz -C "${RISCV_PREFIX}" --strip-components=1 \
    && rm -f /tmp/riscv32-elf-toolchain.tgz

# Provide common alias names expected by some projects/docs
RUN if [ -x "${RISCV_PREFIX}/bin/riscv32-unknown-elf-gcc" ] && [ ! -e "${RISCV_PREFIX}/bin/riscv32-elf-gcc" ]; then \
        ln -s riscv32-unknown-elf-gcc "${RISCV_PREFIX}/bin/riscv32-elf-gcc"; \
    fi

ENV PATH="${RISCV_PREFIX}/bin:${PATH}"

# ---- Non-root user ----
# Some base images already have UID 1000 allocated (e.g. "ubuntu").
# If so, reuse that UID and rename/move it to our desired user/home.
RUN if getent passwd "${UID}" >/dev/null; then \
        existing_user="$(getent passwd "${UID}" | cut -d: -f1)"; \
        usermod -l "${USER}" "${existing_user}"; \
        usermod -d "/home/${USER}" -m "${USER}"; \
    else \
        useradd -m -u "${UID}" -s /bin/bash "${USER}"; \
    fi

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
RUN --mount=type=cache,target=/home/${USER}/.cargo/registry,uid=${UID},gid=${UID} \
    --mount=type=cache,target=/home/${USER}/.cargo/git,uid=${UID},gid=${UID} \
    cargo install bender --version ${BENDER_VERSION}

CMD ["/bin/bash"]
