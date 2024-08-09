# vim: set et:ts=4:sw=4:
#####
# Dockerfile for the Intel MPI benchmarks
# https://www.intel.com/content/www/us/en/develop/documentation/imb-user-guide/top.html
#####

FROM ubuntu:22.04 AS build

ENV DEBIAN_FRONTEND=noninteractive

ARG MPITESTS_VERSION=IMB-v2021.8

RUN apt update && apt install -y --no-install-recommends \
    git openssh-server openssh-client rdma-core ucx-utils libopenmpi-dev \
    ca-certificates build-essential && \
    git clone https://github.com/intel/mpi-benchmarks.git /tmp/mpi-benchmarks && \
    cd /tmp/mpi-benchmarks/src_c && git checkout ${MPITESTS_VERSION} && \
    CC=mpicc make -j $(nproc)

FROM ubuntu:22.04

RUN apt update && apt install -y --no-install-recommends \
    openssh-server openssh-client rdma-core ucx-utils \
    openmpi-bin libopenmpi-dev

COPY --from=build /tmp/mpi-benchmarks/src_c/IMB-MPI1 /IMB-MPI1

# Install helper scripts
COPY ./scripts/* /usr/local/bin
