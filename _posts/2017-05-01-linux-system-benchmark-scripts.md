---
layout: post
title:  "Linux Server Benchmarking Scripts"
date:   2017-05-01
categories: servers
tags: [benchmarks, servers, performance]
image:
  feature: abstract-2.jpg
  credit:
---



# Get HDD+CPU Baseline Stats

```sh
# COPY + PASTE THE FOLLOWING TO CREATE FOLDER & MAIN SCRIPT(S)
# Create folder for results & scripts
export BENCH_DIR=$HOME/benchmarks
mkdir -p $BENCH_DIR/results

touch $BENCH_DIR/bench-library.sh
touch $BENCH_DIR/run-bench.sh
chmod +x $BENCH_DIR/*.sh

cat << 'EOT' >> $BENCH_DIR/bench-library.sh
#!/bin/bash
set -e

# Install some deps
if [ "$(which sysbench)" == "" -o "$(which inxi)" == "" -o "$(which tcpdump)" == "" ]; then
  apt-get update && apt-get install -y sysbench inxi htop iotop tcpdump hddtemp
fi
# Variables
export DATE_TAG=`date +%F` #YYYY-MM-DD
export CPU_CORES="$([ -e /proc/cpuinfo ] && grep -sc ^processor /proc/cpuinfo || sysctl -n hw.ncpu)"
export BENCH_DIR=$HOME/benchmarks/

mkdir -p $BENCH_DIR

function benchCpu() {
  thread_limit=${1:$CPU_CORES}
  prime_limit=${2:-20000}

  if [ $CPU_CORES -lt `expr 1 + $thread_limit` ]; then
    printf "\n\n${yellow}ALERT: Skipping tests limited by \"${thread_limit} thread test\"\n${cyan}Not enough CPU Cores ($CPU_CORES)  ${reset}\n\n"
  else
    printf "\n\n${yellow}ALERT: Skipping tests limited by \"${thread_limit} thread test\"\n${reset}"
sysbench --test=cpu \
  --cpu-max-prime=1000 \
  --num-threads=1 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=1000 \
  --num-threads=4 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=1000 \
  --num-threads=8 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=1000 \
  --num-threads=16 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=20000 \
  --num-threads=24 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=100000 \
  --num-threads=32 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=200000 \
  --num-threads=48 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=200000 \
  --num-threads=64 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=2000000 \
  --num-threads=96 \
  run | tee -a $BENCH_DIR/results/cpu-test.log

sysbench --test=cpu \
  --cpu-max-prime=500000 \
  --num-threads=96 \
  run | tee -a $BENCH_DIR/results/cpu-test.log




sysbench --test=cpu \
  --cpu-max-prime=${prime_limit} \
  --num-threads=${CPU_CORES} \
  run | tee -a $BENCH_DIR/results/cpu-test.log
  fi
}

# benchDisk - tests random read & write, and sequential r, and sequential write, before final cleanup.
function benchDisk() {
  #   Generates test files - up to 80% of your free space - in local dir, then runs the 3 tests (up to 20 minutes each)
  # tests=${1:rndrw,seqrd,seqwr}
  freeSpace=`df -kh . | tail -1 | awk '{print $4}'`
  freeSpace="${freeSpace//G/}"
  # Get 80% of available space (from current dir)
  testSize=$(awk "BEGIN {print $freeSpace * 0.8; exit}")
  testSize=${testSize}G

  printf "####>>> \nWriting $testSize test data to ${PWD}...\n"

  # echo 'Starting' | tee -a $BENCH_DIR/results/sysbench-debug.log

  # sysbench --test=fileio cleanup
sysbench --test=fileio \
  --num-threads=${CPU_CORES} --file-total-size=60G \
  prepare
# do Rand R+W, Sequential Read AND Seq. Write
sysbench --test=fileio --init-rng=on \
  --file-test-mode=seqrd --file-block-size=64K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=40G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=seqrd --file-block-size=8K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=40G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=seqwr --file-block-size=64K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=seqwr --file-block-size=8K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=rndrd --file-block-size=64K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=rndrw --file-block-size=64K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log


sysbench --test=fileio --init-rng=on \
  --file-test-mode=rndwr --file-block-size=64K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=rndrd --file-block-size=8K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=rndwr --file-block-size=8K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

sysbench --test=fileio --init-rng=on \
  --file-test-mode=rndrw --file-block-size=8K \
  --num-threads=${CPU_CORES} --max-time=120 --file-total-size=60G \
  --max-requests=0 run | tee -a $BENCH_DIR/results/sysbench-fileio.log

  # sysbench --test=fileio cleanup

  printf "\n\n####>>> \nCOMPLETED TESTS! Great Success!!! \n\n\n"
}

EOT

###### CREATE RUN SCRIPT
cat << 'EOT' >> tee $BENCH_DIR/run-bench.sh
#!/bin/bash
set -e

source ./bench-library.sh

# Benchmark HDD Speed (in Current Directory)
###########
benchDisk

# Benchmark CPU - trying different thread counts (and work sizes)
# It'll automatically skip test if we don't have enough cores (to have an impact)
# NB: results comparable between different hardware - up to their same CPU CORE #.
###########
benchCpu 1
benchCpu 4
benchCpu 8  50000
benchCpu 12 100000
benchCpu 16 100000
benchCpu 32 250000
benchCpu 48 500000
benchCpu 64 2000000

EOT

chmod +x $BENCH_DIR/*.sh





```

## Usage Examples:

Make sure to `source ~/benchmarks/bench-library.sh` before running the following commands manually.




# I/O - Live Monitor
1. System: iotop
1. Per command: dtrace/ltrace/strace




