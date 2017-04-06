# DRAFT
> DRAFT

# General System Monitors
1. htop

# CPU Baseline Examples

```sh
# CPU Baseline Examples
sysbench --test=cpu --cpu-max-prime=20000 run | tee -a ~/sysbench-results-test-cpu.log
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=8 run | tee -a ~/sysbench-results-test-cpu.log
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=16 run | tee -a ~/sysbench-results-test-cpu.log
sysbench --test=cpu --cpu-max-prime=200000 --num-threads=32 run | tee -a ~/sysbench-results-test-cpu.log
sysbench --test=cpu --cpu-max-prime=500000 --num-threads=48 run | tee -a ~/sysbench-results-test-cpu.log


# SSD/HDD Bandwidth (and NFS, SSHFS, etc)
##### See also: ioping

cd /data
sysbench --test=fileio --num-threads=16 --file-total-size=512G prepare
# do Rand R+W, Sequential Read AND Seq. Write
sysbench --test=fileio --batch --batch-delay=300 \
  --file-test-mode=rndrw \
  --file-block-size=64K \
  --num-threads=16 --init-rng=on --max-time=900 \
  --max-requests=0 run | tee -a ~/sysbench-results-test-fileio.log
sysbench --test=fileio --batch --batch-delay=300 \
  --file-test-mode=seqrd \
  --file-block-size=64K \
  --num-threads=16 --init-rng=on --max-time=900 \
  --max-requests=0 run | tee -a ~/sysbench-results-test-fileio.log
sysbench --test=fileio --batch --batch-delay=300 \
  --file-test-mode=seqwr \
  --file-block-size=64K \
  --num-threads=16 --init-rng=on --max-time=900 \
  --max-requests=0 run | tee -a ~/sysbench-results-test-fileio.log
sysbench --test=fileio cleanup

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







