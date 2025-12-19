# Week 3: Application Selection for Performance Testing

[← Previous Week](week2.md) | [Home](README.md) | [Next Week →](week4.md)

## 1. Application Selection Matrix

I have selected five applications representing different workload types to test system performance:

| Application | Workload Type | Justification |
|-------------|---------------|---------------|
| **stress-ng** | CPU-intensive | A stress testing tool that can max out CPU cores. Good for testing processor performance and thermal behavior under load. |
| **Redis** | RAM-intensive | An in-memory database that stores all data in RAM. Perfect for testing memory performance and management. |
| **Apache Benchmark (ab)** | Network-intensive | A tool for benchmarking web server performance. Tests network throughput and connection handling. |
| **dd command with large files** | I/O-intensive | Creates and copies large files to test disk read/write performance. Shows how well the system handles disk operations. |
| **Nginx Web Server** | Server application | A lightweight web server that handles HTTP requests. Real-world server application that uses CPU, RAM, and network together. |

## 2. Installation Documentation

All installations will be performed via SSH from my workstation to the server.

### Installing stress-ng (CPU-intensive)
```bash
ssh amuser@192.168.56.11
sudo apt update
sudo apt install stress-ng -y
```

### Installing Redis (RAM-intensive)
```bash
ssh amuser@192.168.56.11
sudo apt update
sudo apt install redis-server -y
sudo systemctl start redis-server
sudo systemctl status redis-server
```

### Installing Apache Benchmark (Network-intensive)
```bash
ssh amuser@192.168.56.11
sudo apt update
sudo apt install apache2-utils -y
```

### Installing dd (I/O-intensive)
```bash
# dd command is built-in, no installation needed
# Verify it's available:
ssh amuser@192.168.56.11
which dd
```

### Installing Nginx (Server application)
```bash
ssh amuser@192.168.56.11
sudo apt update
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl status nginx
```

## 3. Expected Resource Profiles

### stress-ng (CPU-intensive)
**Expected CPU Usage:** 90-100% per core when running stress tests  
**Expected RAM Usage:** Low (< 100MB)  
**Expected Disk I/O:** Minimal  
**Expected Network Usage:** None  

### Redis (RAM-intensive)
**Expected CPU Usage:** Low to moderate (10-30%)  
**Expected RAM Usage:** High - will consume as much RAM as configured (can use GBs)  
**Expected Disk I/O:** Low (data is in memory)  
**Expected Network Usage:** Moderate when serving requests  

### Apache Benchmark (Network-intensive)
**Expected CPU Usage:** Moderate (30-60%)  
**Expected RAM Usage:** Low to moderate  
**Expected Disk I/O:** Low  
**Expected Network Usage:** High - many concurrent connections  

### dd command (I/O-intensive)
**Expected CPU Usage:** Low (10-20%)  
**Expected RAM Usage:** Low  
**Expected Disk I/O:** Very high - constant reading/writing  
**Expected Network Usage:** None  

### Nginx (Server application)
**Expected CPU Usage:** Low when idle, moderate under load (20-50%)  
**Expected RAM Usage:** Low (50-100MB)  
**Expected Disk I/O:** Moderate when serving static files  
**Expected Network Usage:** High when serving many requests  

## 4. Monitoring Strategy

### For stress-ng (CPU-intensive)
**Commands to run:**
- `top` - monitor CPU usage percentage in real-time
- `mpstat 1 10` - detailed CPU statistics every second for 10 seconds
- `uptime` - check system load average

**Measurement approach:**
1. Record baseline CPU usage before running stress-ng
2. Start stress-ng with specific CPU load
3. Monitor CPU percentage and load average during test
4. Compare before and after metrics

### For Redis (RAM-intensive)
**Commands to run:**
- `free -h` - monitor total RAM usage
- `top` - check Redis process memory consumption
- `redis-cli INFO memory` - Redis-specific memory stats

**Measurement approach:**
1. Record baseline memory usage
2. Load data into Redis to fill memory
3. Monitor RAM usage as data is loaded
4. Check if system uses swap space
5. Compare memory before and after loading data

### For Apache Benchmark (Network-intensive)
**Commands to run:**
- `ss -s` - network socket statistics
- `iftop` or `nethogs` - real-time network bandwidth monitoring
- `netstat -an | grep :80` - count active connections
- Apache Benchmark output - requests per second, time per request

**Measurement approach:**
1. Start monitoring network connections
2. Run Apache Benchmark against Nginx server
3. Record requests per second and connection counts
4. Monitor network bandwidth usage
5. Check if network becomes bottleneck

### For dd (I/O-intensive)
**Commands to run:**
- `iostat -x 1 10` - disk I/O statistics every second
- `iotop` - real-time disk I/O by process
- `dd` output - shows transfer speed

**Measurement approach:**
1. Record baseline disk I/O stats
2. Run dd command to create large file (e.g., 1GB)
3. Monitor read/write speeds in MB/s
4. Record time taken to complete
5. Compare disk performance metrics

### For Nginx (Server application)
**Commands to run:**
- `top` - overall resource usage
- `systemctl status nginx` - service status
- `curl -I http://localhost` - test response time
- `ss -s` - network connections
- Nginx access logs - `/var/log/nginx/access.log`

**Measurement approach:**
1. Record baseline resource usage with Nginx idle
2. Send requests to Nginx using Apache Benchmark
3. Monitor CPU, RAM, and network during load
4. Check response times and throughput
5. Compare idle vs. under-load performance

## Week 3 Reflection
This week I selected five applications to test different workload types and planned how to monitor each one. 

**Next Steps:** Configure SSH security in phase 4.

[← Previous Week](week2.md) | [Home](README.md) | [Next Week →](week4.md)
