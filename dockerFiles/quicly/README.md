# Fuzz Quicly server (commit 6a90372)
Follow these instructions to fuzz Quicly server using the Dockerfile provided.

### Build docker image
```bash
# build the docker image
docker build --build-arg OPENAI_API_KEY="$(cat ~/.openai_key)" -t quicly .
```

### Run the fuzzing
Once the docker image is built, you can run the fuzzing in a docker container with **2 options**.

**Option 1**: Run the fuzzing using the ```quic-fuzz/dockerFiles/run_common.sh``` script.<br/>
This script will automaticallty assign a free CPU core, run docker and extract all the fuzzing results.
For example to run the fuzzing for ``10``` runs, output the results to ```../results/```, using ```quic-fuzz/aflnet``` fuzzer with the fuzzer options: ```-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz.so -m none -y -P QUIC -q 3 -s 3 -E -K``` for ```86400``` seconds (24 hours), run coverage for every 5 testcases and delete the container once it is done will be:
```bash
cd..
# the command below run the fuzzing in the background, see the log file to track the process

# with encryption module + Synchronisation + Snapshot
setsid ./run_common.sh quicly 10 ../results/ quic-fuzz/aflnet out-quicly-quic-fuzz '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz.so -m none -p -1 -y -P QUIC -q 3 -s 3 -E -K' 86400 5 1 > quicly_quic_snap_aflnet.log 2>&1 &

# with encryption module + Synchronisation
setsid ./run_common.sh quicly 10 ../results/ quic-fuzz/aflnet out-quicly-quic-fuzz-nosnap '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz_no_snap.so -m none -p -1 -y -P QUIC -q 3 -s 3 -E -K' 86400 5 1 > quicly_quic_nosnap_aflnet.log 2>&1 &

# with encryption module
setsid ./run_common.sh quicly 10 ../results/ quic-fuzz/aflnet out-quicly-quic-aflnet '-m none -y -P QUIC -D 1000 -q 3 -s 3 -E -K' 86400 5 1 > quicly_quic_aflnet.log 2>&1 &

# baseline
setsid ./run_common.sh quicly 10 ../results/ quic-fuzz/aflnet out-quicly-aflnet '-m none -P QUIC -D 1000 -q 3 -s 3 -E -K' 86400 5 1 > quicly_aflnet.log 2>&1 &

# chatafl
setsid ./run_common.sh quicly 10 ../results/ chatafl/ChatAFL out-quicly-chatafl '-m none -P QUIC -D 1000 -q 3 -s 3 -E -K' 86400 5 1 > quicly_chatafl.log 2>&1 &
```

<br/>

**Option 2**: Run the fuzzing in the docker images directly using docker.<br/>
By using this method, the user will need to extract the fuzzing results manually.
For example to run the fuzzing on CPU_ID=1 for 24 hours will be:
```bash
# with encryption module + Synchronisation + Snapshot
docker run --cpus=1 --cpuset-cpus 1 -d -it quicly /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-quicly-quic-fuzz '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz.so -m none -p -1 -y -b 1 -P QUIC -q 3 -s 3 -E -K' 86400 5"

# with encryption module + Synchronisation
docker run --cpus=1 --cpuset-cpus 1 -d -it quicly /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-quicly-quic-fuzz-nosnap '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz_no_snap.so -m none -p -1 -y -b 1 -P QUIC -q 3 -s 3 -E -K' 86400 5"

# with encryption module
docker run --cpus=1 --cpuset-cpus 1 -d -it quicly /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-quicly-quic-aflnet '-m none -y -b 1 -P QUIC -D 1000 -q 3 -s 3 -E -K' 86400 5"

# baseline
docker run --cpus=1 --cpuset-cpus 1 -d -it quicly /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-quicly-aflnet '-m none -b 1 -P QUIC -D 1000 -q 3 -s 3 -E -K' 86400 5"

# chatafl
docker run --cpus=1 --cpuset-cpus 1 -d -it quicly /bin/bash -c "cd /tmp && ./run chatafl/ChatAFL out-quicly-chatafl '-m none -b 1 -P QUIC -D 1000 -q 3 -s 3 -E -K' 86400 5"
```

