# vim: set et:ts=4:sw=4:
#####
# Dockerfile for the Intel MPI benchmarks
# https://www.intel.com/content/www/us/en/develop/documentation/imb-user-guide/top.html
#####

FROM ubuntu:22.04 AS build_mpi

ENV DEBIAN_FRONTEND=noninteractive

ARG MPITESTS_VERSION=IMB-v2021.8

RUN apt update && apt install -y --no-install-recommends \
    git openssh-server openssh-client rdma-core ucx-utils libopenmpi-dev \
    ca-certificates build-essential && \
    git clone https://github.com/intel/mpi-benchmarks.git /tmp/mpi-benchmarks && \
    cd /tmp/mpi-benchmarks/src_c && git checkout ${MPITESTS_VERSION} && \
    CC=mpicc make -j $(nproc)


FROM ubuntu:22.04 AS build_rocelib

ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y --no-install-recommends \
    curl ca-certificates automake gcc-12 libtool build-essential  \
    ibverbs-utils libibverbs1 rdmacm-utils librdmacm1 librdmacm-dev \
    infiniband-diags perftest hwloc ibutils numactl linux-generic && \
    curl -s -o /tmp/broadcom.tar.gz https://docs.broadcom.com/docs-and-downloads/ethernet-network-adapters/NXE/BRCM_230.1.116.0/bcm_230.1.116.0c.tar.gz && \
    cd /tmp && tar zxvf broadcom.tar.gz && \
    tar zxvf bcm_230.1.116.0c/drivers_linux/bnxt_rocelib/libbnxt_re-230.0.132.0.tar.gz -C /tmp/ && \
    cd /tmp/libbnxt_re-230.0.132.0 && sh ./autogen.sh && \
    ./configure --sysconfdir=/etc && make -j $(nproc) && \
    make all


FROM ubuntu:22.04

RUN apt update && apt install -y --no-install-recommends \
    openssh-server openssh-client rdma-core ucx-utils \
    openmpi-bin libopenmpi-dev ibverbs-utils strace && \
    echo "/usr/local/lib" >> /etc/ld.so.conf && \
    rm /usr/lib/x86_64-linux-gnu/libibverbs/libbnxt_re-rdmav34.so

COPY --from=build_rocelib /tmp/libbnxt_re-230.0.132.0/src/.libs/libbnxt_re-rdmav34.so \
                          /tmp/libbnxt_re-230.0.132.0/src/.libs/libbnxt_re.la \
                          /tmp/libbnxt_re-230.0.132.0/src/.libs/libbnxt_re.so \
                          /tmp/libbnxt_re-230.0.132.0/src/.libs/libbnxt_re.a \
                          /usr/local/lib/
COPY --from=build_rocelib /tmp/libbnxt_re-230.0.132.0/bnxt_re.driver /etc/libibverbs.d/

COPY --from=build_mpi /tmp/mpi-benchmarks/src_c/IMB-MPI1 /IMB-MPI1

# Install helper scripts
COPY ./scripts/* /usr/local/bin
