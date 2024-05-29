<!--yml
category: 未分类
date: 2024-05-27 15:01:25
-->

# AWS EC2 Instances Benchmark

> 来源：[https://runs-on.com/reference/benchmarks-ec2-instances/](https://runs-on.com/reference/benchmarks-ec2-instances/)

Want to know which EC2 instance type has the fastest single-threaded CPU performance? You’ve come to the right place.

AWS sometimes uses custom CPU types that you won’t find listed on sites like Passmark, etc. So it’s hard to know which one to pick, if you’re looking for the fastest CPU around.

On this page we provide up-to-date benchmarks for most EC2 instance types, across the following regions:

*   North Virginia (us-east-1)
*   Oregon (us-west-2)
*   Ireland (eu-west-1)
*   Frankfurt (eu-central-1)

This can be used to make an informed decision on the best instance to select for your workflows, based on the processor model, architecture, and single-thread CPU speed.

And since speed isn’t everything, the benchmark also lists the on-demand and spot pricing for each instance type, with the trend over time.

Finally, reliability is key when running your workflows, therefore we provide data on the percentage of interruption for each instance type, giving you a clear picture of what to expect in terms of stability.

As always, it’s best to perform your own benchmarks yourself, but this can at least give you some hints in the right direction.

Note: All instance types are launched using the 2 vCPU variant.

## Results

### x64 EC2 instances

 <details open=""><summary>#### Region: us-east-1</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| r7iz | Intel Xeon Gold 6455B (x86_64) | 3152 | 0.1860 | 0.0688 | 63% | 5-10% |
| m7i-flex | Intel Xeon Platinum 8488C (x86_64) | 2988 | 0.0958 | 0.0440 | 54% | <5% |
| m7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1159 | 0.0475 | 59% | 10-15% |
| c7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1026 | 0.0452 | 56% | 10-15% |
| r7a | AMD EPYC 9R14 (x86_64) | 2897 | 0.1522 | 0.0685 | 55% | 10-15% |
| m6a | AMD EPYC 7R13 Processor (x86_64) | 2615 | 0.0864 | 0.0354 | 59% | <5% |
| c6a | AMD EPYC 7R13 Processor (x86_64) | 2611 | 0.0765 | 0.0367 | 52% | <5% |
| m5zn | Intel Xeon Platinum 8252C CPU @ 3.80GHz (x86_64) | 2481 | 0.1652 | 0.0694 | 58% | <5% |
| m6i | Intel Xeon Platinum 8375C CPU @ 2.90GHz (x86_64) | 2340 | 0.0960 | 0.0422 | 56% | 10-15% |
| z1d | Intel Xeon Platinum 8151 CPU @ 3.40GHz (x86_64) | 2287 | 0.1860 | 0.0707 | 62% | >20% |
| m3 | Intel Xeon CPU E5-2670 v2 @ 2.50GHz (x86_64) | 1431 | 0.1330 | 0.0519 | 61% | 5-10% |
| m5a | AMD EPYC 7571 (x86_64) | 1418 | 0.1030 | 0.0515 | 50% | <5% |
| m4 | Intel Xeon CPU E5-2686 v4 @ 2.30GHz (x86_64) | 1394 | 0.1000 | 0.0500 | 50% | <5% |
| t3a | AMD EPYC 7571 (x86_64) | 1379 | 0.0376 | 0.0154 | 59% | 5-10% |</details>  <details open=""><summary>#### Region: us-west-2</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| r7iz | Intel Xeon Gold 6455B (x86_64) | 3152 | 0.1860 | 0.0465 | 75% | 10-15% |
| m7i-flex | Intel Xeon Platinum 8488C (x86_64) | 2988 | 0.0958 | 0.0306 | 68% | <5% |
| m7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1159 | 0.0406 | 65% | <5% |
| c7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1026 | 0.0308 | 70% | <5% |
| r7a | AMD EPYC 9R14 (x86_64) | 2897 | 0.1522 | 0.0441 | 71% | >20% |
| m6a | AMD EPYC 7R13 Processor (x86_64) | 2615 | 0.0864 | 0.0302 | 65% | 5-10% |
| c6a | AMD EPYC 7R13 Processor (x86_64) | 2611 | 0.0765 | 0.0283 | 63% | <5% |
| m5zn | Intel Xeon Platinum 8252C CPU @ 3.80GHz (x86_64) | 2481 | 0.1652 | 0.0628 | 62% | <5% |
| m6i | Intel Xeon Platinum 8375C CPU @ 2.90GHz (x86_64) | 2340 | 0.0960 | 0.0394 | 59% | 5-10% |
| z1d | Intel Xeon Platinum 8151 CPU @ 3.40GHz (x86_64) | 2287 | 0.1860 | 0.0465 | 75% | >20% |
| m3 | Intel Xeon CPU E5-2670 v2 @ 2.50GHz (x86_64) | 1431 | 0.1330 | 0.0439 | 67% | 5-10% |
| m5a | AMD EPYC 7571 (x86_64) | 1418 | 0.1030 | 0.0340 | 67% | <5% |
| m4 | Intel Xeon CPU E5-2686 v4 @ 2.30GHz (x86_64) | 1394 | 0.1000 | 0.0460 | 54% | <5% |
| t3a | AMD EPYC 7571 (x86_64) | 1379 | 0.0376 | 0.0124 | 67% | 5-10% |</details>  <details open=""><summary>#### Region: eu-west-1</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| r7iz | Intel Xeon Gold 6455B (x86_64) | 3152 | 0.2080 | 0.0749 | 64% | 5-10% |
| m7i-flex | Intel Xeon Platinum 8488C (x86_64) | 2988 | 0.1067 | 0.0491 | 54% | <5% |
| m7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1292 | 0.0620 | 52% | >20% |
| c7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1101 | 0.0462 | 58% | <5% |
| r7a | AMD EPYC 9R14 (x86_64) | 2897 | 0.1703 | 0.0698 | 59% | 10-15% |
| m6a | AMD EPYC 7R13 Processor (x86_64) | 2615 | 0.0963 | 0.0414 | 57% | <5% |
| c6a | AMD EPYC 7R13 Processor (x86_64) | 2611 | 0.0821 | 0.0402 | 51% | <5% |
| m5zn | Intel Xeon Platinum 8252C CPU @ 3.80GHz (x86_64) | 2481 | 0.1841 | 0.0847 | 54% | <5% |
| m6i | Intel Xeon Platinum 8375C CPU @ 2.90GHz (x86_64) | 2340 | 0.1070 | 0.0460 | 57% | <5% |
| z1d | Intel Xeon Platinum 8151 CPU @ 3.40GHz (x86_64) | 2287 | 0.2080 | 0.0749 | 64% | >20% |
| m3 | Intel Xeon CPU E5-2670 v2 @ 2.50GHz (x86_64) | 1431 | 0.1460 | 0.0569 | 61% | <5% |
| m5a | AMD EPYC 7571 (x86_64) | 1418 | 0.1150 | 0.0563 | 51% | <5% |
| m4 | Intel Xeon CPU E5-2686 v4 @ 2.30GHz (x86_64) | 1394 | 0.1110 | 0.0544 | 51% | <5% |
| t3a | AMD EPYC 7571 (x86_64) | 1379 | 0.0408 | 0.0163 | 60% | <5% |</details>  <details open=""><summary>#### Region: eu-central-1</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| r7iz | Intel Xeon Gold 6455B (x86_64) | 3152 | 0.2250 | 0.0607 | 73% | <5% |
| m7i-flex | Intel Xeon Platinum 8488C (x86_64) | 2988 | 0.1147 | 0.0436 | 62% | <5% |
| m7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1389 | 0.0181 | 87% | 10-15% |
| c7a | AMD EPYC 9R14 (x86_64) | 2901 | 0.1171 | 0.0387 | 67% | 5-10% |
| r7a | AMD EPYC 9R14 (x86_64) | 2897 | 0.1835 | 0.0606 | 67% | <5% |
| m6a | AMD EPYC 7R13 Processor (x86_64) | 2615 | 0.1035 | 0.0393 | 62% | <5% |
| c6a | AMD EPYC 7R13 Processor (x86_64) | 2611 | 0.0873 | 0.0349 | 60% | 5-10% |
| m5zn | Intel Xeon Platinum 8252C CPU @ 3.80GHz (x86_64) | 2481 | 0.1979 | 0.0594 | 70% | <5% |
| m6i | Intel Xeon Platinum 8375C CPU @ 2.90GHz (x86_64) | 2340 | 0.1150 | 0.0425 | 63% | 5-10% |
| z1d | Intel Xeon Platinum 8151 CPU @ 3.40GHz (x86_64) | 2287 | 0.2250 | 0.0653 | 71% | >20% |
| m3 | Intel Xeon CPU E5-2670 v2 @ 2.50GHz (x86_64) | 1431 | 0.1580 | 0.0506 | 68% | 5-10% |
| m5a | AMD EPYC 7571 (x86_64) | 1418 | 0.1250 | 0.0537 | 57% | <5% |
| m4 | Intel Xeon CPU E5-2686 v4 @ 2.30GHz (x86_64) | 1394 | 0.1200 | 0.0564 | 53% | <5% |
| t3a | AMD EPYC 7571 (x86_64) | 1379 | 0.0432 | 0.0173 | 60% | 5-10% |</details> 

### arm64 EC2 instances

 <details open=""><summary>#### Region: us-east-1</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| m7g | AWS Graviton3 | 1548 | 0.0816 | 0.0367 | 55% | <5% |
| m6g | Neoverse-N1 (aarch64) | 1101 | 0.0770 | 0.0323 | 58% | 5-10% |
| t4g | Neoverse-N1 (aarch64) | 1097 | 0.0084 | 0.0039 | 54% | <5% |</details>  <details open=""><summary>#### Region: us-west-2</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| m7g | AWS Graviton3 | 1548 | 0.0816 | 0.0228 | 72% | <5% |
| m6g | Neoverse-N1 (aarch64) | 1101 | 0.0770 | 0.0246 | 68% | 5-10% |
| t4g | Neoverse-N1 (aarch64) | 1097 | 0.0084 | 0.0031 | 63% | >20% |</details>  <details open=""><summary>#### Region: eu-west-1</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| m7g | AWS Graviton3 | 1548 | 0.0910 | 0.0346 | 62% | <5% |
| m6g | Neoverse-N1 (aarch64) | 1101 | 0.0860 | 0.0335 | 61% | <5% |
| t4g | Neoverse-N1 (aarch64) | 1097 | 0.0092 | 0.0042 | 54% | <5% |</details>  <details open=""><summary>#### Region: eu-central-1</summary> 

| Instance family | Processor | CPU speed (avg) | $/hour on-demand | $/hour spot (avg) | Spot savings over on-demand | Spot % interruption |
| m7g | AWS Graviton3 | 1548 | 0.0978 | 0.0362 | 63% | <5% |
| m6g | Neoverse-N1 (aarch64) | 1101 | 0.0920 | 0.0285 | 69% | 5-10% |
| t4g | Neoverse-N1 (aarch64) | 1097 | 0.0096 | 0.0024 | 75% | <5% |</details> 

## Analysis

### What's the fastest AWS EC2 instance for x86_64?

The fastest EC2 instance for x86_64 architectures varies based on the specific workload and AWS region. However, instances from the Compute Optimized (C7a, C7i, m7a, m7i, r7a, r7iz) families come with the fastest CPUs.

### What's the fastest AWS EC2 instance for ARM64?

For ARM64 architectures, the instances from the AWS Graviton series (e.g., C7g, M7g) are designed to provide the best performance. These instances are optimized for a variety of workloads and offer a significant price-performance advantage for applications built for ARM64.

## About those benchmarks

Benchmarks are performed using the [Passmark benchmarking tool ↗](https://www.passmark.com/products/performancetest/download.php), using the CPU Single Threaded metric.

Last updated: May 21, 2024