# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv target/x86_64-unknown-linux-gnu/release/router /router && \
    mv target/x86_64-unknown-linux-gnu/release/federation /federation && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv target/release/router /router_no_inst && \
    mv target/release/federation /federation_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /router /router_no_inst /federation /federation_no_inst /
RUN mkdir /fuzz
COPY --from=builder /src/fuzz/*graphql /fuzz/
