# DRAFT
> DRAFT

# General System Monitors
1. htop

# CPU Baseline Examples

```sh
sysbench --test=cpu --cpu-max-prime=20000 | tee -a ~/sysbench-results-test-cpu.log
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=8 | tee -a ~/sysbench-results-test-cpu.log
sysbench --test=cpu --cpu-max-prime=20000 --num-threads=16 | tee -a ~/sysbench-results-test-cpu.log
```

# SSD/HDD Bandwidth (and NFS, SSHFS, etc)
1. ioping
1. sysbench

```sh
cd /data
sysbench --test=fileio --num-threads=16 --file-total-size=256G prepare
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







