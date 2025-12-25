# Week 6: Performance Evaluation and Analysis

[← Previous Week](week5.md) | [Home](README.md) | [Next Week →](week7.md)

## Overview

This week I installed the applications selected in Week 3 and performed performance testing on my Ubuntu Server. I tested CPU, memory, disk I/O, and network performance under different workloads, collected baseline and load data, and implemented optimizations.

## 1. Testing Approach

All performance testing was conducted remotely via SSH from my Ubuntu Desktop workstation. For each application, I:

1. Recorded baseline performance metrics (system at idle)
2. Ran the application under load
3. Monitored resource usage during the load
4. Collected performance data
5. Analyzed results to identify bottlenecks


## 2. Application Installation

I installed all the applications selected in Week 3:
```bash
# Install stress-ng (CPU testing)
sudo apt update
sudo apt install stress-ng -y

# Install Redis (RAM testing)
sudo apt install redis-server -y
sudo systemctl start redis-server
sudo systemctl status redis-server

# Install Apache Benchmark (Network testing)
sudo apt install apache2-utils -y

# Install Nginx (Server application)
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl status nginx

# dd command is built-in (I/O testing)
which dd
```

![Application installations](images/week6-installations.png)


## 3. Performance Data Table

### Summary of All Testing Results

| Application | Metric | Baseline | Under Load | Change | Notes |
|-------------|--------|----------|------------|--------|-------|
| **System Idle** | CPU Idle % | 82.6% | - | - | Normal operation |
| **System Idle** | Memory Used | 474.8 MB | - | - | Low baseline usage |
| **System Idle** | Load Average | 0.12 | - | - | Very light load |
| **Redis** | CPU Usage | 0.0% (idle) | 0.7% | +0.7% | Efficient memory ops |
| **Redis** | Memory Used | 474.8 MB | 473.8 MB | -1 MB | Data in memory |
| **Redis** | Load Average | 0.12 | 0.01 | -0.11 | System stable |
| **Top Process** | amuser (top) | 15.4% CPU | - | - | Monitoring overhead |
| **sshd** | CPU Usage | 7.7% | - | - | SSH connection active |


## 4. Detailed Performance Testing

### Test 1: CPU Performance (stress-ng)

**Baseline Measurement:**

Before running stress test, I checked CPU usage:
```bash
top -bn1 | head -20
```

**Baseline Results:**
- CPU idle: 82.6%
- Load average: 0.12
- Top CPU process: amuser (top command) at 15.4%

![CPU baseline](images/week6-cpu-baseline.png)


**Load Test:**

I ran a CPU stress test using 2 cores for 60 seconds:
```bash
stress-ng --cpu 2 --timeout 60s
```

While monitoring with:
```bash
top
mpstat 1 5
```

**Load Test Results:**
- CPU usage increased significantly
- System remained responsive
- Load average increased during test

![CPU under load](images/week6-cpu-under-load.png)


### Test 2: Memory Performance (Redis)

**Baseline Measurement:**
```bash
free -h
```

**Baseline Results:**
- Total Memory: 4709 MB
- Used: 474.8 MB
- Free: 3508.7 MB
- Available: 962.4 MB

![Memory baseline](images/week6-memory-baseline.png)


**Load Test:**

I loaded data into Redis:
```bash
redis-benchmark -t set -n 100000 -d 1000
```

While monitoring:
```bash
free -h
top | grep redis
```

**Load Test Results:**
- Redis process: 0.7% CPU, 0.3% memory
- System remained stable
- Memory operations were efficient

![Redis running](images/week6-redis-running.png)

**Redis Memory Usage:**
```bash
redis-cli INFO memory | grep used_memory_human
```

![Redis memory info](images/week6-redis-memory.png)


### Test 3: Disk I/O Performance (dd command)

**Baseline Measurement:**
```bash
iostat -x 1 3
```

![I/O baseline](images/week6-io-baseline.png)


**Write Test:**

Created a 1GB test file:
```bash
dd if=/dev/zero of=~/testfile bs=1M count=1024
```

**Write Test Results:**
- Write speed measured in MB/s
- System handled large file creation efficiently

![Disk write test](images/week6-disk-write.png)


**Read Test:**

Read the test file back:
```bash
dd if=~/testfile of=/dev/null bs=1M
```

**Read Test Results:**
- Read speed measured in MB/s
- Disk read performance documented

![Disk read test](images/week6-disk-read.png)

Cleaned up test file:
```bash
rm ~/testfile
```


### Test 4: Network Performance (Nginx + Apache Benchmark)

**Baseline Measurement:**

Checked network connections before load:
```bash
ss -s
netstat -an | grep :80
```

![Network baseline](images/week6-network-baseline.png)


**Load Test:**

Created test file:
```bash
echo "Hello from CMPN202 Server" | sudo tee /var/www/html/test.html
```

Ran Apache Benchmark:
```bash
ab -n 1000 -c 10 http://localhost/test.html
```

**Network Test Results:**
- Requests per second measured
- Average response time recorded
- Connection handling monitored

![Network load test](images/week6-network-test.png)


## 5. Performance Bottleneck Analysis

From my testing, I identified these key findings:

**CPU Performance:**
- System has plenty of CPU capacity (82.6% idle at baseline)
- CPU stress tests show the system can handle intensive workloads
- No CPU bottlenecks detected

**Memory Performance:**
- Low memory usage at baseline (474.8 MB used of 4709 MB total)
- Redis runs efficiently with minimal memory overhead
- Plenty of memory available for applications

**Disk I/O:**
- Disk read/write speeds are adequate for the workloads tested
- No disk I/O bottlenecks observed

**Network Performance:**
- Nginx handles HTTP requests efficiently
- Network not a limiting factor in current configuration


## 6. Optimization Implementation

I implemented two optimizations to improve system performance:

### Optimization 1: Nginx Worker Process Configuration

**Before Optimization:**

Default Nginx configuration with limited worker processes.

Tested performance:
```bash
ab -n 1000 -c 50 http://localhost/test.html
```

![Nginx before optimization](images/week6-nginx-before.png)

**Optimization Applied:**

Modified Nginx configuration:
```bash
sudo nano /etc/nginx/nginx.conf
```

Changed:
```
worker_processes auto;
worker_connections 1024;
```

Restarted Nginx:
```bash
sudo systemctl restart nginx
```

**After Optimization:**

Re-tested performance:
```bash
ab -n 1000 -c 50 http://localhost/test.html
```

![Nginx after optimization](images/week6-nginx-after.png)

**Results:**
- Requests per second improved
- Response time decreased
- Better concurrent connection handling


### Optimization 2: System Swappiness Tuning

**Before Optimization:**

Checked default swappiness:
```bash
cat /proc/sys/vm/swappiness
```

Default value: 60

Tested Redis performance:
```bash
redis-benchmark -t set -n 200000 -d 1000
```

![Before swappiness change](images/week6-swappiness-before.png)

**Optimization Applied:**

Reduced swappiness to prioritize RAM:
```bash
sudo sysctl vm.swappiness=10
```

Made permanent:
```bash
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```

**After Optimization:**

Re-tested Redis:
```bash
redis-benchmark -t set -n 200000 -d 1000
```

![After swappiness change](images/week6-swappiness-after.png)

**Results:**
- Operations per second improved
- System uses RAM more efficiently
- Reduced swap usage


## 7. Performance Visualization

I created graphs to visualize the performance data collected during testing.

![Performance graphs](images/week6-performance-graphs.png)

The graphs show:
- CPU usage baseline vs under load
- Memory usage comparison
- Nginx performance before and after optimization
- Disk I/O read vs write speeds


## 8. Network Performance Analysis

**Latency Testing:**

Tested response times to the server:
```bash
ping -c 10 localhost
```

**Results:**
- Average latency measured
- No packet loss
- Stable network performance

**Throughput Testing:**

Apache Benchmark results showed:
- Requests per second measured
- Average response time documented
- Network handled concurrent connections efficiently

![Network performance analysis](images/week6-network-analysis.png)


## 9. Testing Evidence Summary

All testing was performed via SSH from my workstation. Evidence includes:

- Baseline measurements for all resources
- Load testing screenshots
- Before and after optimization comparisons
- Performance data collection
- Resource monitoring during tests

---

## 10. Key Findings

**System Performance:**
- CPU: Plenty of capacity, 82.6% idle at baseline
- Memory: Well-utilized with 3508.7 MB free
- Disk I/O: Adequate for tested workloads
- Network: Efficient request handling

**Optimization Results:**
1. Nginx worker process tuning improved request handling
2. Swappiness adjustment improved memory management
3. Both optimizations showed measurable improvements

**Bottlenecks:**
- No significant bottlenecks identified
- System is well-balanced for the tested workloads
- Room for growth as application demands increase


## Week 6 Reflection

This week I performed comprehensive performance testing on my server. I tested CPU, memory, disk I/O, and network performance using the applications I selected in Week 3.

**What I learned:**
- How to use performance monitoring tools effectively
- Different applications stress different system resources
- System optimization can improve performance measurably
- Baseline measurements are essential for comparison

**Challenges:**
- Coordinating multiple terminal windows for simultaneous monitoring
- Interpreting performance metrics correctly
- Ensuring tests were repeatable and consistent

**Next Steps:** In Week 7, I will conduct a comprehensive security audit using Lynis and nmap to evaluate the overall security posture of my server.

---

[← Previous Week](week5.md) | [Home](README.md) | [Next Week →](week7.md)
