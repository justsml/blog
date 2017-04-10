# DRAFT
> DRAFT

# General System Monitors
1. htop





# Get HDD+CPU Baseline Stats
##### Side Note: See also: ioping


```sh
BENCH_ROOT=$HOME/benchmarks
touch $BENCH_ROOT/bench-library.sh
touch $BENCH_ROOT/run-bench.sh
chmod +x $BENCH_ROOT/*.sh

cat << EOT >> tee $BENCH_ROOT/bench-library.sh
#!/bin/bash
set -e

# Install some deps
if [ "$(which sysbench)" == "" -o "$(which inxi)" == "" -o "$(which tcpdump)" == "" ]; then
  apt-get update && apt-get install -y sysbench inxi iotop tcpdump hddtemp
fi
# Variables
DATE_TAG=`date %+F` #YYYY-MM-DD
CPU_CORES="$([ -e /proc/cpuinfo ] && grep -sc ^processor /proc/cpuinfo || sysctl -n hw.ncpu)"
BENCH_DIR=$HOME/benchmarks/results/

mkdir -p $BENCH_DIR

function benchCpu() {
  thread_limit=${1:-1}
  prime_limit=${2:-20000}

  if [ "$CPU_CORES" -lt `expr 1 + $thread_limit` ]; then
    printf "\n\n${yellow}ALERT: Skipping tests limited by \"${thread_limit} thread test\"\n${cyan}Not enough CPU Cores ($CPU_CORES)  ${reset}\n\n"
  else
    printf "\n\n${yellow}ALERT: Skipping tests limited by \"${thread_limit} thread test\"\n${cyan}
    sysbench --test=cpu \
        --cpu-max-prime=${prime_limit} \
        --num-threads=${thread_limit} \
        run | tee -a $BENCH_DIR/cpu-test.log
  fi
}

function benchDisk() {
  # tests=${1:rndrw,seqrd,seqwr}
  freeSpace=`df -kh . | tail -1 | awk '{print $4}'`
  freeSpace=${freeSpace//[A-Za-z]/}
  # Get 80% of available space (from current dir)
  testSize=$(awk "BEGIN {print $freeSpace * 0.8; exit}")
  testSize=${testSize}G
  
  printf "Writing $testSize test data to ${PWD}..."

  sysbench --test=fileio \
    --num-threads=${CPU_CORES} --file-total-size=${testSize} \
    prepare
  # do Rand R+W, Sequential Read AND Seq. Write
  sysbench --test=fileio \
    --file-test-mode=rndrw \
    --file-block-size=64K \
    --num-threads=${CPU_CORES} --init-rng=on --max-time=900 \
    --max-requests=0 run | tee -a ~/sysbench-results-test-fileio.log
  sysbench --test=fileio \
    --file-test-mode=seqrd \
    --file-block-size=64K \
    --num-threads=${CPU_CORES} --init-rng=on --max-time=900 \
    --max-requests=0 run | tee -a ~/sysbench-results-test-fileio.log
  sysbench --test=fileio \
    --file-test-mode=seqwr \
    --file-block-size=64K \
    --num-threads=${CPU_CORES} --init-rng=on --max-time=900 \
    --max-requests=0 run | tee -a ~/sysbench-results-test-fileio.log
  sysbench --test=fileio cleanup
}

EOT

```

## Usage Examples:

Make sure to `source ~/benchmarks/bench-library.sh` before running the following commands manually.


## Get CPU + SSD/HDD Bandwidth (and NFS, SSHFS, etc)

```sh
cat << EOT >> tee $BENCH_ROOT/run-bench.sh
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

```


# I/O - Live Monitor
1. iotop
1. dtrace/ltrace/strace

# NETWORK

* Terminal
  1. massscan
  1. nmap
  1. tcpdump
  1. httping

* Web-tools & Scanners
  1. https://urlscan.io/
  1. https://dnsstuff.com/







