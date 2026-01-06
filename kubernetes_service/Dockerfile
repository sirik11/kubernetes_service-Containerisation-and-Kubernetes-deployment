# syntax=docker/dockerfile:1

# Multi‑stage Dockerfile to build the inference engine and produce a small runtime image.

# Stage 1: build dependencies and compile inference binary
FROM ubuntu:22.04 AS build

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential cmake git curl ca-certificates \
        # optional: install ONNX Runtime development files
        && rm -rf /var/lib/apt/lists/*

WORKDIR /build
# Copy the inference engine source code
COPY ../onnx_inference_engine /build/onnx_inference_engine

# Configure and build
RUN mkdir -p onnx_inference_engine/build && \
    cd onnx_inference_engine/build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release && \
    make -j$(nproc)

# Stage 2: runtime image
FROM ubuntu:22.04 AS runtime
RUN apt-get update && \
    apt-get install -y --no-install-recommends libgomp1 && \
    rm -rf /var/lib/apt/lists/*

# Copy the inference binary from the build stage
COPY --from=build /build/onnx_inference_engine/build/inference /usr/local/bin/inference

# Copy model file into the container (this should be replaced with your model)
COPY model.onnx /models/model.onnx

EXPOSE 8080
# Use dumb-init to handle signals gracefully (optional)
CMD ["/usr/local/bin/inference", "/models/model.onnx", "5"]
