FROM ubuntu:20.04

RUN DEBIAN_FRONTEND=noninteractive apt update -y --fix-missing && apt upgrade -y
RUN DEBIAN_FRONTEND=noninteractive apt install -y software-properties-common
RUN DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:ubuntu-toolchain-r/test
RUN DEBIAN_FRONTEND=noninteractive apt update -y --fix-missing && apt upgrade -y
RUN DEBIAN_FRONTEND=noninteractive apt install -y build-essential git graphviz clang-12 llvm-12 llvm-12-dev llvm-12-tools openssl libssl-dev graphviz-dev libcap-dev lsof wget gpg \
                                                  libsqlite3-dev libelf-dev libc6-dbg gettext libgnutls28-dev apt-utils libtool gettext-base cmake-curses-gui \
                                                  libcurl4-openssl-dev libjson-c-dev libpcre2-dev \
                                                  cmake

# We use llvm-17 now
RUN wget https://apt.llvm.org/llvm-snapshot.gpg.key
RUN gpg --dearmor -o /usr/share/keyrings/llvm-archive-keyring.gpg llvm-snapshot.gpg.key
RUN echo "deb [signed-by=/usr/share/keyrings/llvm-archive-keyring.gpg] http://apt.llvm.org/$(lsb_release -cs)/ llvm-toolchain-$(lsb_release -cs)-17 main" | tee /etc/apt/sources.list.d/llvm.list
RUN DEBIAN_FRONTEND=noninteractive apt update && apt upgrade -y && apt install -y llvm-17 llvm-17-dev llvm-17-tools clang-17 lld-17

ENV LLVM_CONFIG=/usr/bin/llvm-config-17
RUN ln -s /usr/bin/clang-17 /usr/bin/clang && ln -s /usr/bin/clang-17 /usr/bin/clang++

# install fuzzer
WORKDIR /tmp
RUN git clone https://github.com/QUICTester/QUIC-Fuzz.git quic-fuzz
WORKDIR /tmp/quic-fuzz/aflnet
RUN wget https://www.openssl.org/source/openssl-3.0.2.tar.gz
RUN tar xzvf openssl-3.0.2.tar.gz
WORKDIR /tmp/quic-fuzz/aflnet/openssl-3.0.2
RUN ./Configure linux-x86_64 no-shared
RUN make -j
WORKDIR /tmp/quic-fuzz/aflnet
RUN make clean all
WORKDIR /tmp/quic-fuzz/aflnet/llvm_mode
RUN make
WORKDIR /tmp/quic-fuzz/aflnet/SnapFuzz/SaBRe/plugins
RUN ln -sf ../../snapfuzz snapfuzz
WORKDIR /tmp/quic-fuzz/aflnet/SnapFuzz/SaBRe/
RUN mkdir build
WORKDIR /tmp/quic-fuzz/aflnet/SnapFuzz/SaBRe/build
RUN cmake -DCMAKE_BUILD_TYPE=RELEASE -DSF_MEMFS=OFF -DSF_STDIO=ON -DSF_SLEEP=ON -DSF_SMARTDEFER=OFF .. && make -j && mv plugins/snapfuzz/libsnapfuzz.so  plugins/snapfuzz/libsnapfuzz_no_snap.so
RUN cmake -DCMAKE_BUILD_TYPE=RELEASE -DSF_MEMFS=OFF -DSF_STDIO=ON -DSF_SLEEP=ON -DSF_SMARTDEFER=ON .. && make -j
WORKDIR /tmp/quic-fuzz/aflnet/
RUN ln -sf SnapFuzz/SaBRe/build/sabre sabre && ln -sf SnapFuzz/SaBRe/build/plugins/snapfuzz/libsnapfuzz.so libsnapfuzz.so && ln -sf SnapFuzz/SaBRe/build/plugins/snapfuzz/libsnapfuzz_no_snap.so libsnapfuzz_no_snap.so

# install ChatAFL
WORKDIR /tmp
RUN git clone https://github.com/ChatAFLndss/ChatAFL.git chatafl
WORKDIR /tmp/chatafl/ChatAFL
RUN git checkout 1ea603eb && \
    cp /tmp/quic-fuzz/patches/chat_afl.patch chat_afl.patch && \
    cp /tmp/quic-fuzz/patches/chat_afl_quic.c quic.c && \
    cp /tmp/quic-fuzz/patches/chat_afl_quic.h quic.h
ARG OPENAI_API_KEY
RUN sed -i "s/#define OPENAI_TOKEN \".*\"/#define OPENAI_TOKEN \"$OPENAI_API_KEY\"/" chat-llm.h && \
    git apply chat_afl.patch && \
    make clean all && \
    cp /tmp/quic-fuzz/aflnet/aflnet-replay aflnet-replay

# start install the server
WORKDIR /tmp
RUN git clone https://github.com/h2o/quicly.git
WORKDIR /tmp/quicly
RUN git checkout 6a90372
RUN git submodule update --init --recursive
COPY quicly.patch .
RUN git apply quicly.patch
WORKDIR /tmp/quicly/deps/picotls
COPY picotls.patch .
RUN git apply picotls.patch
WORKDIR /tmp/quicly/
RUN cmake -DCMAKE_C_COMPILER=/tmp/quic-fuzz/aflnet/afl-clang-fast -DCMAKE_CXX_COMPILER=/tmp/quic-fuzz/aflnet/afl-clang-fast++ -DWITH_FUSION=OFF .
RUN AFL_USE_ASAN=1 CFLAGS="-fsanitize=address" CXXFLAGS="-fsanitize=address" LDFLAGS="-fsanitize=address" make clean all -j

# start install the server (with cov patch)
WORKDIR /tmp
RUN git clone https://github.com/h2o/quicly.git quicly_cov
WORKDIR /tmp/quicly_cov
RUN git checkout 6a90372
RUN git submodule update --init --recursive
COPY quicly_cov.patch .
RUN git apply quicly_cov.patch
WORKDIR /tmp/quicly_cov/deps/picotls
COPY picotls.patch .
RUN git apply picotls.patch
WORKDIR /tmp/quicly_cov/
RUN cmake -DCMAKE_C_COMPILER=/tmp/quic-fuzz/aflnet/afl-clang-fast -DCMAKE_C_FLAGS="-fprofile-instr-generate -fcoverage-mapping" -DWITH_FUSION=OFF .
RUN make -j

# copy script/file to be run in container
WORKDIR /tmp
COPY run.sh run
RUN chmod +x run
COPY cov_script.sh cov_script
RUN chmod +x cov_script
COPY server-cert.pem server-cert.pem
COPY server-key.pem server-key.pem
COPY quicly_seed quicly_seed
ENV WORKDIR="/tmp"