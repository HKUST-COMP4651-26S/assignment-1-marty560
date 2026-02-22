[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/aQEb8Lmc)
# COMP4651 Assignment-1: EC2 Measurement (2 questions, 4 marks)

### Deadline: 11:59PM, Feb, 28, Saturday

---

### Name: 
### Student Id: 
### Email: 

---

## Question 1: Measure the EC2 CPU and Memory performance

1. (1 mark) Report the name of measurement tool used in your measurements (you are free to choose *any* open source measurement software as long as it can measure CPU and memory performance). Please describe your configuration of the measurement tool, and explain why you set such a value for each parameter. Explain what the values obtained from measurement results represent (e.g., the value of your measurement result can be the execution time for a scientific computing task, a score given by the measurement tools or something else).

    > I used **sysbench** (version 1.0.20), an open-source multi-purpose benchmarking tool, to measure both CPU and memory performance of each EC2 instance.
    > 
    > **CPU Performance Test:**
    > ```
    > sysbench cpu --cpu-max-prime=20000 run
    > ```
    > - `--cpu-max-prime=20000`: This sets the upper limit for the prime number generator used as the CPU workload. I chose 20000 (larger than the default of 10000) to increase computational stress and make performance differences between instance types more distinguishable.
    > - The metric reported is **events per second** — i.e., how many prime-number computation rounds the CPU completes per second. A higher value indicates better CPU performance.
    > 
    > **Memory Performance Test:**
    > ```
    > sysbench memory --memory-block-size=1M --memory-total-size=10G run
    > ```
    > - `--memory-block-size=1M`: Memory operations are performed in 1MB chunks, which is a common and efficient block size for sequential access tests.
    > - `--memory-total-size=10G`: The total data written/read is 10GB, ensuring a sustained workload large enough to reveal differences in memory bandwidth.
    > - The metric reported is **MB/s** (megabytes per second) — the memory read/write throughput. A higher value indicates better memory performance.

2. (1 mark) Run your measurement tool on general purpose `t2.micro`, `t2.medium`, and `c5d.large` Linux instances, respectively, and find the performance differences among these instances. Launch all the instances in the **US East (N. Virginia)** region. Does the performance of EC2 instances increase commensurate with the increase of the number of vCPUs and memory resource?

    In order to answer this question, you need to complete the following table by filling out blanks with the measurement results corresponding to each instance type.

    | Size        | CPU performance (events/sec) | Memory performance (MB/s) |
    | ----------- | ---------------------------- | ------------------------- |
    | `t2.micro`  | 891.91                       | 19246.57                  |
    | `t2.medium` | 887.08                       | 18969.89                  |
    | `c5d.large` | 449.74                       | 19559.65                  |

    > Region: US East (N. Virginia). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI.

    > **Analysis:**
    >
    > For **CPU performance**, t2.micro and t2.medium show nearly identical results (~891 vs ~887 events/sec), meaning that doubling the vCPU count from 1 to 2 does not improve single-threaded CPU performance. The c5d.large shows a lower single-threaded score (~449 events/sec), likely due to differences in CPU architecture and clock speed between Intel Broadwell (t2) and Intel Cascade Lake (c5d) under sysbench's single-threaded workload.
    > 
    > For **memory performance**, all three instances yield similar bandwidth (~19 GB/s), with c5d.large slightly ahead. The difference is not substantial, indicating that increasing vCPU count or memory capacity does not proportionally improve memory bandwidth under this test.
    > 
    > **Conclusion:** EC2 instance performance does **not** increase strictly in proportion to vCPU count or memory size. CPU performance is dominated by single-core clock speed and architecture, and memory bandwidth is largely similar across instance types.

## Question 2: Measure the EC2 Network performance

1. (1 mark) The metrics of network performance include **TCP bandwidth** and **round-trip time (RTT)**. Within the same region, what network performance is experienced between instances of the same type and different types? In order to answer this question, you need to complete the following table.

    | Type                      | TCP b/w (Mbps) | RTT (ms) |
    | ------------------------- | -------------- | -------- |
    | `t3.medium` - `t3.medium` | 4520           | 0.202    |
    | `m5.large` - `m5.large`   | 4960           | 0.294    |
    | `c5n.large` - `c5n.large` | 4950           | 0.162    |
    | `t3.medium` - `c5n.large` | 4680           | 0.649    |
    | `m5.large` - `c5n.large`  | 4950           | 0.618    |
    | `m5.large` - `t3.medium`  | 4650           | 0.277    |

    > Region: US East (N. Virginia). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI. Note: Use private IP address when using iPerf within the same region. You'll need iPerf for measuring TCP bandwidth and Ping for measuring Round-Trip time.

    > **Analysis:**
    >
    > - **TCP Bandwidth:** All instance pairs achieve roughly 4.5–5.0 Gbps. m5.large and c5n.large pairs reach near the 5 Gbps cap, while t3.medium pairs are slightly lower (~4.52 Gbps). When t3.medium is involved in a cross-type pair, bandwidth is bottlenecked by the weaker instance, suggesting bandwidth is limited by the lower-capability endpoint.
    > 
    > - **RTT:** Same-type instance pairs have notably lower RTT (0.162–0.294 ms) compared to cross-type pairs (0.277–0.825 ms). This suggests AWS likely places same-type instances in closer physical proximity, reducing latency. Cross-type pairs, especially those involving c5n.large, have higher RTT due to different placement groups or network configurations.

2. (1 mark) What about the network performance for instances deployed in different regions? In order to answer this question, you need to complete the following table.

    | Connection                | TCP b/w (Mbps) | RTT (ms) |
    | ------------------------- | -------------- | -------- |
    | N. Virginia - Oregon      | 381            | 59.562   |
    | N. Virginia - N. Virginia | 4770           | 0.315    |
    | Oregon - Oregon           | 4770           | 0.219    |

    > Region: US East (N. Virginia), US West (Oregon). Use `Ubuntu Server 22.04 LTS (HVM)` as AMI. All instances are `c5.large`. Note: Use public IP address when using iPerf within the same region.

    > **Analysis:**
    >
    > - **Same-region connections** (N. Virginia–N. Virginia and Oregon–Oregon) achieve ~4.77 Gbps bandwidth and sub-0.32 ms RTT, reflecting AWS's high-performance internal network.
    > 
    > - **Cross-region connection** (N. Virginia–Oregon) shows a dramatic drop: bandwidth falls to ~381 Mbps (about 12× lower) and RTT rises to ~59.6 ms (about 200× higher). This is expected, as cross-region traffic must traverse long geographical distances (~4,500 km) passing through multiple network hops and exchange points.
    > 
    > **Conclusion:** Network performance degrades significantly across regions compared to within a single region, in both bandwidth and latency.