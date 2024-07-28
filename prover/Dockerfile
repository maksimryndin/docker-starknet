FROM ubuntu:22.04 AS stone
RUN apt-get update && apt-get install -y git wget gnupg2 elfutils libdw-dev libgmp3-dev unzip wget python3 libncurses5
RUN apt-get install -y clang-12 clang-format-12 clang-tidy libclang-12-dev llvm-12 && \
    ln -sf /usr/bin/clang++-12 /usr/bin/clang++ && \
    ln -sf /usr/bin/clang-12 /usr/bin/clang
RUN wget -O bazelisk "https://github.com/bazelbuild/bazelisk/releases/download/v1.20.0/bazelisk-linux-$(dpkg --print-architecture)" && \
    chmod +x bazelisk && mv bazelisk /usr/local/bin/bazel
RUN git clone --depth 1 https://github.com/starkware-libs/stone-prover && \
    echo "5.4.0" > /stone-prover/.bazelversion
ENV CC="/usr/bin/clang"

# ARM-specific
FROM stone AS stone-arm64
RUN git clone --depth 1 https://github.com/BLAKE2/BLAKE2 && \
    mv /BLAKE2/ref/blake2s-ref.c /stone-prover/src/third_party/blake2_ref/blake2s.c
WORKDIR stone-prover
# Preload required bazel version as a separate docker cache layer
RUN bazel --version
# Switch to portable reference implementations (turn off x86-64 specifics)
RUN sed -i 's/"prime_field_element.S",//g' src/starkware/algebra/fields/BUILD && \
    mv src/third_party/XKCP/KeccakP-1600-OptimizedAVX2/BUILD src/third_party/XKCP/CompactFIPS202 && \
    sed -i 's@"KeccakP-1600-AVX2.s"@"Keccak-readable-and-compact.c"@g' src/third_party/XKCP/CompactFIPS202/BUILD && \
    sed -i 's@"//src/third_party/XKCP/KeccakP-1600-OptimizedAVX2:Keccak1600F"@"//src/third_party/XKCP/CompactFIPS202:Keccak1600F"@g' src/starkware/crypt_tools/BUILD && \
    sed -i 's/"-DASSEMBLY"//g' src/third_party/poseidon/BUILD && \
    sed -i 's/"f251_asm.s",//g' src/third_party/poseidon/BUILD && \
    find src/starkware -type f -name 'BUILD' -exec sed -i 's@"//src/third_party/blake2:blake2.h"@"//src/third_party/blake2_ref:blake2.h"@g' {} \; && \
    find src/starkware -type f -name 'BUILD' -exec sed -i 's@"//src/third_party/blake2:blake2-impl.h"@"//src/third_party/blake2_ref:blake2-impl.h"@g' {} \; && \
    find src/starkware -type f -name 'BUILD' -exec sed -i 's@"//src/third_party/blake2:blake2s"@"//src/third_party/blake2_ref:blake2s"@g' {} \; && \
    sed -i 's@third_party/blake2/@third_party/blake2_ref/@g' src/starkware/crypt_tools/blake2s.inl && \
    mv src/third_party/blake2/BUILD src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2-config.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2s-load-sse2.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2b-load-sse2.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2s-round.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2b-round.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2s-load-sse41.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2b-load-sse41.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/"blake2s-load-xop.h",//g' src/third_party/blake2_ref/BUILD && \
    sed -i 's/static void blake2s_compress/void blake2s_compress/g' src/third_party/blake2_ref/blake2s.c
RUN sed -i "s/x86_64/$(uname -m)/g" .bazelrc
RUN bazel build --copt="-DNO_AVX" //...

# x86_64-specific
FROM stone AS stone-x86_64
WORKDIR stone-prover
RUN bazel build //...

FROM stone-${TARGETARCH} AS built
RUN mv $(bazel info bazel-bin)/src/starkware/main/cpu/cpu_air_prover /usr/local/bin/prover && \
    mv $(bazel info bazel-bin)/src/starkware/main/cpu/cpu_air_verifier /usr/local/bin/verifier && \
    mkdir -p /shared_libs && \
    cp $(find / -iname "libdw.so.*") /shared_libs/ && \
    cp $(find / -iname "libelf.so.*") /shared_libs/

FROM ubuntu:22.04
COPY --from=built /usr/local/bin/prover /usr/local/bin/prover
COPY --from=built /usr/local/bin/verifier /usr/local/bin/verifier
COPY --from=built /shared_libs/ /shared_libs
RUN mv /shared_libs/* /usr/lib/$(uname -m)-linux-gnu
