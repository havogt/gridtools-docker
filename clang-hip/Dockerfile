ARG REPOSITORY
ARG CLANG_VERSION=10
FROM ${REPOSITORY}:clang-${CLANG_VERSION}
LABEL maintainer="Felix Thaler <thaler@cscs.ch>"

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        clang-tools-${CLANG_VERSION} \
        curl \
        gnupg \
        libnuma-dev \
        lld-${CLANG_VERSION} \
        zlib1g-dev && \
    curl -sL http://repo.radeon.com/rocm/apt/debian/rocm.gpg.key | apt-key add - && \
    printf "deb [arch=amd64] http://repo.radeon.com/rocm/apt/debian/ xenial main" >> /etc/apt/sources.list.d/rocm.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends rocm-dev && \
    rm -rf /var/lib/apt/lists/*

# Note we use a frozen commit until there is a stable release!
RUN git clone https://github.com/RadeonOpenCompute/ROCm-Device-Libs.git && \
    cd ROCm-Device-Libs && \
    git checkout 87f047619ce9ef1a7f8194ac4e86795e7a423cb7 && \
    mkdir -p build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=1 .. && \
    make -j $(nproc) && \
    make install && \
    cd ../.. && rm -rf ROCm-Device-Libs

# Use a frozen commit (from master-next branch) for now until there is a stable release
RUN git clone https://github.com/ROCm-Developer-Tools/HIP.git && \
    cd HIP && \
    git checkout ffcbd7e63395f8a4d3ccb7e4d5133f8d2dde793e && \
    mkdir -p build && cd build && \
    cmake -DHIP_COMPILER=clang -DCMAKE_BUILD_TYPE=Release .. && \
    make -j $(nproc) && \
    make install && \
    cd ../.. && rm -rf HIP

# Patch for hipcc script, fixing Clang include path
RUN sed -i 's#$HIP_CLANG_INCLUDE_PATH = abs_path("$HIP_CLANG_PATH/../lib/clang/$HIP_CLANG_VERSION/include");#$HIP_CLANG_INCLUDE_PATH = "'/usr/include/clang/${CLANG_VERSION}/include'";#' /opt/rocm/bin/hipcc

# Adding all llvm binaries to path as several of them are required by hipcc
ENV PATH=/usr/lib/llvm-${CLANG_VERSION}/bin:${PATH} LD_LIBRARY_PATH=/usr/lib/llvm-${CLANG_VERSION}/lib:${LD_LIBRARY_PATH}
# Setting HIP_CLANG_PATH seems to be required, too
ENV HIP_CLANG_PATH=/usr/lib/llvm-${CLANG_VERSION}/bin
# Default set of GPU targets, otherwise hipcc just crashes when run without corresponding flags (and kills cmake config step)
ENV HCC_AMDGPU_TARGET=gfx900,gfx906
# Finally add all rocm binaries and libraries to paths and set CXX compiler
ENV PATH=${PATH}:/opt/rocm/bin LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/opt/rocm/lib CXX=/opt/rocm/bin/hipcc
