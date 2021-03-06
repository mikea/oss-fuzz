# Copyright 2016 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
################################################################################

# Docker image with head clang installed.

FROM ossfuzz/base
MAINTAINER mike.aizatsky@gmail.com

ENV LLVM_DEP_PACKAGES "build-essential make cmake ninja-build git python2.7"
RUN apt-get install -y $LLVM_DEP_PACKAGES

RUN mkdir /src /work

# Checkout
RUN cd /src && git clone --depth 1 http://llvm.org/git/llvm.git
RUN cd /src/llvm/tools && git clone --depth 1 http://llvm.org/git/clang.git
RUN cd /src/llvm/projects && git clone --depth 1 http://llvm.org/git/compiler-rt.git
RUN cd /src/llvm/projects && git clone --depth 1 http://llvm.org/git/libcxx.git
RUN cd /src/llvm/projects && git clone --depth 1 http://llvm.org/git/libcxxabi.git
RUN cd /src/llvm/projects && git clone --depth 1 http://llvm.org/git/lld.git

# Build & Install
RUN mkdir -p /work/llvm
WORKDIR /work/llvm
RUN cmake -G "Ninja" \
    -DLIBCXX_ENABLE_SHARED=OFF -DLIBCXX_ENABLE_STATIC_ABI_LIBRARY=ON \
    -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="X86" \
    /src/llvm
RUN ninja
RUN ninja install
RUN rm -rf /work/llvm

# Copy libfuzzer sources
RUN mkdir /src/libfuzzer
RUN cp -r /src/llvm/lib/Fuzzer/* /src/libfuzzer/

# Setup the environment.
ENV CC "clang"
ENV CXX "clang++"
ENV CCC "clang++"

ENV CFLAGS "-g"
ENV CXXFLAGS_EXTRA "-stdlib=libc++"
ENV CXXFLAGS "$CFLAGS $CXXFLAGS_EXTRA"

# Build MSan libraries
RUN mkdir -p /work/llvm-msan
WORKDIR /work/llvm-msan
RUN cmake -G "Ninja" \
    -DLIBCXX_ENABLE_SHARED=OFF -DLIBCXX_ENABLE_STATIC_ABI_LIBRARY=ON \
    -DCMAKE_BUILD_TYPE=Release -DLLVM_USE_SANITIZER=Memory \
    /src/llvm
# RUN ninja cxx cxxabi
# RUN mkdir /usr/local/lib/msan
# RUN cp lib/*.so /usr/local/lib/msan
# RUN cp lib/*.a /usr/local/lib/msan
RUN rm -rf /work/llvm-msan

# Cleanup
RUN rm -rf /src/llvm
RUN apt-get remove --purge -y $LLVM_DEP_PACKAGES
RUN apt-get autoremove -y


