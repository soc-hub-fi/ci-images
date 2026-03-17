# zeroHETI container

```sh
# Build the container
podman build -t zeroheti:0.1 .

## Replay upstream `zeroHETI` CI (with reusable caches)

The upstream project runs its regression tests in
`.github/workflows/rust.yml` (see `https://github.com/ANurmi/zeroHETI/blob/main/.github/workflows/rust.yml`).
The workflow can be replayed by running those same commands locally in this container, while reusing
host caches for faster re-runs.

```sh
# One-time: clone upstream repo (host)
mkdir -p _tmp
git clone --depth 1 https://github.com/ANurmi/zeroHETI.git _tmp/zeroHETI

# Optional but recommended: persistent caches (host)
mkdir -p _cache/zeroheti/{cargo-registry,cargo-git,target}

# Run the workflow steps in the container, with caches mounted
podman run --rm \
  -v "$(pwd)/_tmp/zeroHETI:/work:Z,U" \
  -v "$(pwd)/_cache/zeroheti/cargo-registry:/home/builder/.cargo/registry:Z,U" \
  -v "$(pwd)/_cache/zeroheti/cargo-git:/home/builder/.cargo/git:Z,U" \
  -v "$(pwd)/_cache/zeroheti/target:/work/target:Z,U" \
  -w /work \
  -e CARGO_TERM_COLOR=always \
  -e TEST_BUILD_RUSTFLAGS='-Clinker=riscv32-unknown-elf-gcc -Clink-arg=-Wl,-Tmemory.x -Clink-arg=-Wl,-Tlink.x -Clink-arg=-nostartfiles -Clink-arg=-march=rv32ec -Clink-arg=-mabi=ilp32e' \
  ghcr.io/soc-hub-fi/zeroheti-ci:local \
  bash -lc 'set -euo pipefail

bender update && bender vendor init

cd examples/zeroheti-bsp
cargo check -Frtl-tb -Fintc-hetic --lib --examples
cargo check -Frtl-tb -Fintc-clic --lib --examples
cargo check -Frtl-tb -Fintc-edfic --lib --examples

cd /work
make verilate
cd examples/zeroheti-bsp
RUSTFLAGS="$TEST_BUILD_RUSTFLAGS" cargo run --release -Frtl-tb --example hello

cd /work
make verilate
cd examples/zeroheti-bsp
if RUSTFLAGS="$TEST_BUILD_RUSTFLAGS" cargo run --release -Frtl-tb --example fail; then
  echo "ERROR: expected fail example to fail, but it succeeded" >&2
  exit 1
fi

cd /work
make verilate INTC=CLIC
cd examples/zeroheti-bsp
RUSTFLAGS="$TEST_BUILD_RUSTFLAGS" cargo run --release -Frtl-tb -Fintc-clic --example all_irqs

cd /work
make verilate INTC=HETIC
cd examples/zeroheti-bsp
RUSTFLAGS="$TEST_BUILD_RUSTFLAGS" cargo run --release -Frtl-tb -Fintc-hetic --example all_irqs

cd /work
make verilate INTC=EDFIC
cd examples/zeroheti-bsp
RUSTFLAGS="$TEST_BUILD_RUSTFLAGS" cargo run --release -Frtl-tb -Fintc-edfic --example all_irqs
'
```
