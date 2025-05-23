# Fuzz Google_quiche server (commit 149b7e62)
Follow these instructions to fuzz Google_quiche server using the Dockerfile provided.

### Build docker image
```bash
# build the docker image
docker build --build-arg OPENAI_API_KEY="$(cat ~/.openai_key)" -t google_quiche .
```

### Run the fuzzing
Once the docker image is built, you can run the fuzzing in a docker container with **2 options**.

**Option 1**: Run the fuzzing using the ```quic-fuzz/dockerFiles/run_common.sh``` script.<br/>
This script will automaticallty assign a free CPU core, run docker and extract all the fuzzing results.
For example to run the fuzzing for ```10``` runs, output the results to ```../results/```, using ```quic-fuzz/aflnet``` fuzzer with the fuzzer options: ```-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz.so -p 0 -y -P QUIC -m none -q 3 -s 3 -E -K``` for ```86400``` seconds (24 hours), run coverage for every 5 testcases and delete the container once it is done will be:
```bash
cd ..
# the command below run the fuzzing in the background, see the log file to track the process

# with encryption module + Synchronisation + Snapshot
setsid ./run_common.sh google_quiche 10 ../results/ quic-fuzz/aflnet out-google_quiche-quic-fuzz '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz.so -p 0 -y -P QUIC -m none -q 3 -s 3 -E -K' 86400 5 1 > google_quiche_quic_snap_aflnet.log 2>&1 &

# with encryption module + Synchronisation
setsid ./run_common.sh google_quiche 10 ../results/ quic-fuzz/aflnet out-google_quiche-quic-fuzz-nosnap '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz_no_snap.so -p 0 -y -P QUIC -m none -q 3 -s 3 -E -K' 86400 5 1 > google_quiche_quic_nosnap_aflnet.log 2>&1 &

# with encryption module
setsid ./run_common.sh google_quiche 10 ../results/ quic-fuzz/aflnet out-google_quiche-quic-aflnet '-y -P QUIC -m none -D 10000 -q 3 -s 3 -E -K' 86400 5 1 > google_quiche_quic_aflnet.log 2>&1 &

# baseline
setsid ./run_common.sh google_quiche 10 ../results/ quic-fuzz/aflnet out-google_quiche-aflnet '-P QUIC -m none -D 10000 -q 3 -s 3 -E -K' 86400 5 1 > google_quiche_aflnet.log 2>&1 &

# chatafl
setsid ./run_common.sh google_quiche 10 ../results/ chatafl/ChatAFL out-google_quiche-chatafl '-P QUIC -m none -D 10000 -q 3 -s 3 -E -K' 86400 5 1 > google_quiche_chatafl.log 2>&1 &
```

<br/>

**Option 2**: Run the fuzzing in the docker images directly using docker.<br/>
By using this method, the user will need to extract the fuzzing results manually.
For example to run the fuzzing on CPU_ID=1 for 24 hours will be:
```bash
# with encryption module + Synchronisation + Snapshot
docker run --cpus=1 --cpuset-cpus 1 -d -it google_quiche /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-google_quiche-quic-fuzz '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz.so -p 0 -y -b 1 -P QUIC -m none -q 3 -s 3 -E -K' 86400 5"

# with encryption module + Synchronisation
docker run --cpus=1 --cpuset-cpus 1 -d -it google_quiche /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-google_quiche-quic-fuzz-nosnap '-a /tmp/quic-fuzz/aflnet/sabre -A /tmp/quic-fuzz/aflnet/libsnapfuzz_no_snap.so -p 0 -y -b 1 -P QUIC -m none -q 3 -s 3 -E -K' 86400 5"

# with encryption module
docker run --cpus=1 --cpuset-cpus 1 -d -it google_quiche /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-google_quiche-quic-aflnet '-y -b 1 -P QUIC -m none -D 10000 -q 3 -s 3 -E -K' 86400 5"

# baseline
docker run --cpus=1 --cpuset-cpus 1 -d -it google_quiche /bin/bash -c "cd /tmp && ./run quic-fuzz/aflnet out-google_quiche-aflnet '-b 1 -P QUIC -m none -D 10000 -q 3 -s 3 -E -K' 86400 5"

# chatafl
docker run --cpus=1 --cpuset-cpus 1 -d -it google_quiche /bin/bash -c "cd /tmp && ./run chatafl/ChatAFL out-google_quiche-chatafl '-b 1 -P QUIC -m none -D 10000 -q 3 -s 3 -E -K' 86400 5"
```