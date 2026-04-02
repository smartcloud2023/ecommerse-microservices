# # Copyright The OpenTelemetry Authors
# # SPDX-License-Identifier: Apache-2.0


# FROM docker.io/library/python:3.14-alpine3.23 AS build-venv

# RUN apk update && \
#     apk add gcc g++ linux-headers
# WORKDIR /app/
# #COPY /src/recommendation/ /app/
# COPY /src/recommendation/requirements.txt requirements.txt


# COPY /src/recommendation/demo_pb2_grpc.py demo_pb2_grpc.py
# COPY /src/recommendation/demo_pb2.py demo_pb2.py
# COPY /src/recommendation/logger.py logger.py
# COPY /src/recommendation/metrics.py metrics.py
# COPY /src/recommendation/recommendation_server.py recommendation_server.py

# RUN python -m venv venv && \
#     venv/bin/pip install --no-cache-dir -r requirements.txt

# RUN venv/bin/opentelemetry-bootstrap -a install

# FROM docker.io/library/python:3.14-alpine3.23

# COPY --from=build-venv /venv/ /venv/

# WORKDIR /app


# EXPOSE ${RECOMMENDATION_PORT}
# ENTRYPOINT [ "/venv/bin/opentelemetry-instrument", "/venv/bin/python", "recommendation_server.py" ]

# --- Stage 1: Build Stage ---
FROM docker.io/library/python:3.14-alpine3.23 AS build-venv

# Install system dependencies required for building some Python packages
RUN apk update && \
    apk add --no-cache gcc g++ linux-headers musl-dev

# Set the working directory for the build context
WORKDIR /app

# 1. Create a virtual environment in a fixed location
RUN python -m venv /venv

# 2. Copy only requirements first to leverage Docker cache
# Use relative paths (no leading slash) since build context is the repo root
COPY ./file1/requirements.txt .

# 3. Install dependencies and OpenTelemetry instrumentation
RUN /venv/bin/pip install --no-cache-dir --upgrade pip && \
    /venv/bin/pip install --no-cache-dir -r requirements.txt && \
    /venv/bin/opentelemetry-bootstrap -a install

# 4. Copy the application source code into the build stage
COPY ./file1 .

# --- Stage 2: Final Runtime Stage ---
FROM docker.io/library/python:3.14-alpine3.23

# Set environment variables for Python
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PATH="/venv/bin:$PATH"

WORKDIR /app

# 1. Copy the prepared virtual environment from the builder
COPY --from=build-venv /venv /venv

# 2. Copy the application code from the builder
COPY --from=build-venv /app /app

# 3. Define build arguments for ports (matches your .env strategy)
ARG RECOMMENDATION_PORT=8080
ENV RECOMMENDATION_PORT=${RECOMMENDATION_PORT}
EXPOSE ${RECOMMENDATION_PORT}

# 4. Run the application using OpenTelemetry instrumentation
ENTRYPOINT [ "opentelemetry-instrument", "python", "recommendation_server.py" ]