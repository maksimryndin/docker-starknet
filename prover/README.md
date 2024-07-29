# Docker image Stone prover

This docker image wraps [Stone prover](https://github.com/starkware-libs/stone-prover) to prove and verify Cairo programs (only Cairo1).
And it supports multiarch, i.e. you can build it for x86-64 and arm64 at least.

## Docker Hub

[Repository](https://hub.docker.com/r/maksimryndin/starknet-stone)

Run the prover

```sh
docker run --rm -it -v $(pwd):/tmp maksimryndin/starknet-stone:latest prover \
    --out_file=/tmp/proof.json \
    --private_input_file=/tmp/private_input.json \
    --public_input_file=/tmp/public_input.json \
    --prover_config_file=/tmp/cpu_air_prover_config.json \
    --parameter_file=/tmp/cpu_air_params.json
```

## Building image locally

```sh
docker build . -f prover/Dockerfile -t stone
```

Run the prover
```sh
docker run --rm -it -v $(pwd):/tmp stone prover \
    --out_file=/tmp/proof.json \
    --private_input_file=/tmp/private_input.json \
    --public_input_file=/tmp/public_input.json \
    --prover_config_file=/tmp/cpu_air_prover_config.json \
    --parameter_file=/tmp/cpu_air_params.json
```

Run verifier
```sh
docker run --rm -i -t stone verifier
```