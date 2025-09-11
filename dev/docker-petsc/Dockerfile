# syntax=docker/dockerfile:1
FROM gcc:15 as builder

ARG PETSC_REPO=https://gitlab.com/petsc/petsc.git
ARG SLEPC_REPO=https://gitlab.com/slepc/slepc.git
ARG PETSC_PREFIX=/opt/petsc
ARG SLEPC_PREFIX=/opt/slepc
ARG MAKEFLAGS=-j$(nproc)

ENV DEBIAN_FRONTEND=noninteractive \
    PETSC_DIR=${PETSC_PREFIX} \
    PETSC_ARCH=arch-linux-c-opt \
    SLEPC_DIR=${SLEPC_PREFIX}

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    wget \
    git \
    python3 \
    cmake \
    make \
    m4 \
    perl \
    pkg-config \
    gfortran \
    libopenblas-dev \
    liblapack-dev \
    openmpi-bin \
    libopenmpi-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build

# --- Build PETSc ---
RUN git clone --depth 1 ${PETSC_REPO} petsc-src
WORKDIR /build/petsc-src

RUN ./configure \
    --prefix=${PETSC_PREFIX} \
    --with-cc=mpicc \
    --with-cxx=mpicxx \
    --with-fc=mpifort \
    --with-mpi=1 \
    --with-scalar-type=complex \
    --with-shared-libraries=1 \
    --with-debugging=0 \
    --download-fblaslapack=1 \
    && ${MAKEFLAGS:+make $MAKEFLAGS} all \
    && make install

ENV PATH=${PETSC_PREFIX}/bin:${PATH} \
    LD_LIBRARY_PATH=${PETSC_PREFIX}/lib:${LD_LIBRARY_PATH}

# --- Build SLEPc ---
WORKDIR /build
RUN git clone --depth 1 ${SLEPC_REPO} slepc-src
WORKDIR /build/slepc-src

RUN ./configure \
    --prefix=${SLEPC_PREFIX} \
    --with-petsc-dir=${PETSC_PREFIX} \
    --with-scalar-type=complex \
    --with-shared-libraries=1 \
    --with-debugging=0 \
    CC=mpicc CXX=mpicxx FC=mpifort \
    && ${MAKEFLAGS:+make $MAKEFLAGS} all \
    && make install

ENV PATH=${SLEPC_PREFIX}/bin:${PATH} \
    LD_LIBRARY_PATH=${SLEPC_PREFIX}/lib:${LD_LIBRARY_PATH}

# --- Runtime image ---
FROM debian:stable-slim

ARG PETSC_PREFIX=/opt/petsc
ARG SLEPC_PREFIX=/opt/slepc

RUN apt-get update && apt-get install -y --no-install-recommends \
    libopenblas0 \
    liblapack3 \
    libgfortran5 \
    openmpi-bin \
    libopenmpi3 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder ${PETSC_PREFIX} ${PETSC_PREFIX}
COPY --from=builder ${SLEPC_PREFIX} ${SLEPC_PREFIX}

ENV PETSC_DIR=${PETSC_PREFIX} \
    PETSC_ARCH=arch-linux-c-opt \
    SLEPC_DIR=${SLEPC_PREFIX} \
    PATH=${PETSC_PREFIX}/bin:${SLEPC_PREFIX}/bin:${PATH} \
    LD_LIBRARY_PATH=${PETSC_PREFIX}/lib:${SLEPC_PREFIX}/lib:${LD_LIBRARY_PATH}

WORKDIR /work
CMD ["/bin/bash"]
