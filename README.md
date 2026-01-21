# install Wazuh (SIEM + EDR) on ubuntu 24.04

Wazuh, an open-source SIEM and EDR platform, installs easily on Ubuntu 24.04 using the official all-in-one installation assistant for a single-node setup including the server, indexer, and dashboard. This method deploys everything on one host, suitable for small to medium environments up to 100 endpoints. Follow the steps below as root or with sudo.

## Requirements

Ubuntu 24.04 is fully supported, with minimum hardware of 4 vCPUs, 8 GB RAM, and 50 GB disk for basic use (scale up for more endpoints). Ensure internet access for package downloads and a static IP for the dashboard.[^1_1][^1_2]

## Installation Steps

Update your system first.

- Run `sudo apt update && sudo apt upgrade -y`.
- Download the installer: `curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh`.
- Execute for all-in-one: `sudo bash ./wazuh-install.sh -a`.
The process takes 5-15 minutes and outputs credentials like admin user and a generated password.[^1_2][^1_1]


## Verify and Access

Check services: `systemctl status wazuh-manager wazuh-indexer wazuh-dashboard`. Access the dashboard at `https://<your-server-ip>` (accept self-signed cert warning), logging in with admin and the printed password. Test with `filebeat test output` for indexer connectivity.[^1_1][^1_2]

## Post-Installation

Disable repo updates to avoid breaks: `sudo sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list && sudo apt update`. View all passwords: `sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt`. Deploy agents next via the dashboard.[^1_3][^1_2]
<span style="display:none">[^1_10][^1_4][^1_5][^1_6][^1_7][^1_8][^1_9]</span>

<div align="center">⁂</div>

[^1_1]: https://documentation.wazuh.com/current/installation-guide/wazuh-server/index.html

[^1_2]: https://www.youtube.com/watch?v=fnoksQlenBo

[^1_3]: https://www.quape.com/how-to-install-wazuh-on-vps-or-cloud/

[^1_4]: https://echoxec.com/wazuh-installation-configuration-1

[^1_5]: https://oneuptime.com/blog/post/2026-01-15-setup-wazuh-security-platform-ubuntu/view

[^1_6]: https://syncbricks.com/installing-wazuh-on-ubuntu-server-24-04/

[^1_7]: https://documentation.wazuh.com/current/installation-guide/index.html

[^1_8]: https://dev.to/samueladeduntan/wazuh-on-prem-installation-for-a-single-node-all-in-one-made-easy-complete-siem-setup-walkthrough-53nb

[^1_9]: https://www.youtube.com/watch?v=8VfMgIv0t9Q

[^1_10]: https://www.youtube.com/watch?v=gGSPwKORsaQ


---

# Install Wazuh all-in-one on a single Ubuntu 24.04 VM

Wazuh all-in-one deployment on a single Ubuntu 24.04 VM combines the manager, indexer, and dashboard for quick SIEM+EDR setup. This is ideal for testing or small-scale monitoring. Use the official script for reliability.

## VM Prerequisites

Provision a VM with Ubuntu 24.04 LTS, at least 4 vCPUs, 8 GB RAM, and 50 GB disk. Assign a static IP and open ports 1514/UDP (agents), 1515/TCP (registration), 443/TCP (dashboard).[^2_1]

## Step-by-Step Install

Log in as root or use sudo, then execute these commands sequentially.

- `apt update && apt upgrade -y && apt install curl apt-transport-https unzip lsb-release gnupg2 -y`
- `curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg`
- `echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list && apt update`
- `curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && bash ./wazuh-install.sh -a`
Note the admin password from the output.[^2_1]


## Access Dashboard

Services start automatically; verify with `systemctl status wazuh-*`. Browse to `https://<VM-IP>` and log in as admin/<printed-password>. Ignore SSL warnings initially.[^2_2][^2_1]

## Next Actions

Secure passwords: `cat wazuh-install-files/wazuh-passwords.txt`. Add agents from the dashboard under Management > Deploy new agent. For production, consider Elasticsearch/OpenSearch tuning.[^2_3][^2_2]

<div align="center">⁂</div>

[^2_1]: https://documentation.wazuh.com/current/installation-guide/wazuh-server/index.html

[^2_2]: https://www.youtube.com/watch?v=fnoksQlenBo

[^2_3]: https://echoxec.com/wazuh-installation-configuration-1


---

# Minimum hardware and storage for Wazuh production on 24.04

For production Wazuh all-in-one deployments on Ubuntu 24.04, minimum hardware starts at 4 GB RAM and 2 CPU cores per component, but recommended specs are 16 GB RAM and 8 cores for the indexer node to handle typical loads. Scale based on agents and alerts per second (APS); storage grows with data retention. Ubuntu 24.04 is officially supported.

## Hardware Minimums

These apply to single-node or distributed setups.


| Component | Min RAM | Min CPU | Rec RAM | Rec CPU |
| :-- | :-- | :-- | :-- | :-- |
| Wazuh Server | 2 GB | 2 cores | 4 GB | 8 cores [^3_1] |
| Wazuh Indexer | 4 GB | 2 cores | 16 GB | 8 cores [^3_2] |
| Wazuh Dashboard | 2 GB | 2 cores | 4 GB | 4 cores [^3_2] |

## Storage Sizing

Disk needs SSD for performance; calculate for 90-day retention per agent type.


| Endpoint Type | APS | Indexer Storage (GB/90 days/agent) | Server Storage (GB/90 days/agent) |
| :-- | :-- | :-- | :-- |
| Servers | 0.25 | 3.7 | 0.1 |
| Workstations | 0.1 | 1.5 | 0.04 |
| Network Devices | 0.5 | 7.4 | 0.2 |

Example: 100 mixed agents (80 workstations, 10 servers, 10 networks) requires ~230 GB indexer + 6 GB server storage.[^3_3][^3_1][^3_2]

## Scaling Tips

Monitor `/var/ossec/var/run/wazuh-analysisd.state` for dropped events in production; add cluster nodes if non-zero. For 100+ agents, separate components across VMs with load balancers.[^3_4][^3_1]
<span style="display:none">[^3_10][^3_5][^3_6][^3_7][^3_8][^3_9]</span>

<div align="center">⁂</div>

[^3_1]: https://www.quape.com/how-to-install-wazuh-on-vps-or-cloud/

[^3_2]: https://oneuptime.com/blog/post/2026-01-15-setup-wazuh-security-platform-ubuntu/view

[^3_3]: https://documentation.wazuh.com/current/installation-guide/wazuh-server/index.html

[^3_4]: https://www.reddit.com/r/Wazuh/comments/10gdgcz/hardware_requirements_for_production_environment/

[^3_5]: https://documentation.wazuh.com/current/quickstart.html

[^3_6]: https://groups.google.com/g/wazuh/c/ItDiTwjy85Y

[^3_7]: https://certbar.com/technical-blogs/topic-wazuh-deployment-guide

[^3_8]: https://www.reddit.com/r/Wazuh/comments/1lx722k/wazuh_sizing_formula/

[^3_9]: https://github.com/wazuh/wazuh-documentation/issues/2125

[^3_10]: https://groups.google.com/g/wazuh/c/5vFednZa_fU


---

# How to configure Wazuh indexer and Elasticsearch alternatives

Wazuh indexer is a customized OpenSearch fork serving as the search and analytics engine for alerts. Configuration focuses on networking, security certificates, and cluster settings via `/etc/wazuh-indexer/opensearch.yml`. Elasticsearch is no longer supported; OpenSearch/Wazuh indexer is the standard, with no direct alternatives in official deployments.

## Indexer Configuration

Edit `/etc/wazuh-indexer/opensearch.yml` post-install for production tweaks. Key settings include `network.host` (bind IP), `node.name` (unique per node), `discovery.seed_hosts` (cluster peers for multi-node), and `plugins.security.nodes_dn` (certificate DNs).[^4_1][^4_2]

- Generate certs first with `wazuh-certs-tool.sh` using `config.yml` defining node IPs/names.
- Deploy certs: Extract to `/etc/wazuh-indexer/certs/`, set permissions `chmod 400`, own by `wazuh-indexer`.
- Init security: Run `/usr/share/wazuh-indexer/bin/indexer-security-init.sh` once per cluster.

```
- Restart: `systemctl restart wazuh-indexer` and verify `curl -k -u admin:<pass> https://<ip>:9200/_cat/nodes?v`.[^4_2][^4_1]
```


## Elasticsearch Alternatives

Wazuh migrated from Elasticsearch to OpenSearch (now Wazuh indexer) due to licensing; it's fully compatible with prior features. No official support for vanilla Elasticsearch or other forks like Elastic Cloud. For custom setups, integrate Logstash with external OpenSearch via `/etc/logstash/conf.d/wazuh-opensearch.conf`, but use bundled indexer for simplicity.[^4_3][^4_4][^4_5]
<span style="display:none">[^4_10][^4_11][^4_12][^4_6][^4_7][^4_8][^4_9]</span>

<div align="center">⁂</div>

[^4_1]: https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/step-by-step.html

[^4_2]: https://documentation.wazuh.com/current/installation-guide/wazuh-server/index.html

[^4_3]: https://hugs4bugs.me/wazuh's-big-move-from-elasticsearch-to-opensearch/

[^4_4]: https://documentation.wazuh.com/current/integrations-guide/opensearch/index.html

[^4_5]: https://www.youtube.com/watch?v=fnoksQlenBo

[^4_6]: https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/index.html

[^4_7]: https://www.initmax.com/wiki/wazuh-installation-and-configuration/

[^4_8]: https://www.reddit.com/r/Wazuh/comments/1b1gaw5/wazuh_indexing_and_index_pattern_usage/

[^4_9]: https://groups.google.com/g/wazuh/c/5Ox7RRFRIaY

[^4_10]: https://wazuh.github.io/wazuh-indexer-plugins/ref/getting-started/installation.html

[^4_11]: https://groups.google.com/g/wazuh/c/zpNZxK6JaEs

[^4_12]: https://wazuh.com/blog/detection-with-opensearch-integration/


---

# Compare OpenSearch vs Elasticsearch for Wazuh indexer

OpenSearch powers the Wazuh indexer as a community-driven fork of Elasticsearch 7.10.2, offering identical core functionality for Wazuh SIEM/EDR indexing and search. Wazuh dropped Elasticsearch support after its SSPL license change, making OpenSearch the only official choice for new installs.

## Key Comparison

| Aspect | OpenSearch (Wazuh Indexer) | Elasticsearch |
| :-- | :-- | :-- |
| License | Apache 2.0 (fully open) [^5_1][^5_2] | SSPL/Elastic License (restrictive) [^5_1] |
| Wazuh Support | Official, bundled in installer [^5_2] | Deprecated since 4.3 [^5_1][^5_3] |
| Compatibility | Full drop-in for ES 7.10 APIs/features | Similar but licensing blocks integration |
| Plugins | Security, alerting, SQL (community-driven) [^5_2] | Advanced ML, enterprise features (paid) |
| Community | Active, AWS-backed | Elastic-controlled |
| Migration | Easy from ES; Wazuh tools provided | Not feasible to OpenSearch in Wazuh |

OpenSearch suits Wazuh production due to licensing freedom and seamless integration via Filebeat/Logstash.[^5_2][^5_1][^5_4]

## Wazuh Implications

Indexer uses OpenSearch by default; config files (`opensearch.yml`) mirror `elasticsearch.yml`. No performance gap reported; use for alert storage/visualization in dashboard. External Elasticsearch integrations possible via Logstash but unsupported long-term.[^5_5][^5_6][^5_2]
<span style="display:none">[^5_10][^5_7][^5_8][^5_9]</span>

<div align="center">⁂</div>

[^5_1]: https://hugs4bugs.me/wazuh's-big-move-from-elasticsearch-to-opensearch/

[^5_2]: https://documentation.wazuh.com/current/integrations-guide/opensearch/index.html

[^5_3]: https://groups.google.com/g/wazuh/c/J0z7C2H3XfA

[^5_4]: https://groups.google.com/g/wazuh/c/8XyHQvbHd5I

[^5_5]: https://wazuh.com/blog/detection-with-opensearch-integration/

[^5_6]: https://www.reddit.com/r/Wazuh/comments/1b8bcz0/difference_between_wazuh_indexer_and_elasticsearch/

[^5_7]: https://www.reddit.com/r/cybersecurity/comments/1ajq0yb/elasticsearch_vs_wazuh/

[^5_8]: https://www.reddit.com/r/Wazuh/comments/1ldloqd/pulling_data_from_elasticsearch_to_wazuh_dashboard/

[^5_9]: https://github.com/wazuh/wazuh-packages/issues/666

[^5_10]: https://www.reddit.com/r/Wazuh/comments/1b8bcz0/difference_between_wazuh_indexer_and_elasticsearch/?tl=pt-br


---

# Performance benchmarks OpenSearch vs Elasticsearch for Wazuh

OpenSearch and Elasticsearch deliver comparable performance for Wazuh workloads like log indexing and alert querying, as Wazuh indexer is OpenSearch-based with optimizations. Benchmarks vary by workload, with Elasticsearch often faster in vector search and sorting (2x-12x), but OpenSearch excelling in some aggregations (up to 1.6x). No Wazuh-specific benchmarks exist; general tests apply since both fork from Lucene.

## Benchmark Summary

| Workload | Elasticsearch Edge | OpenSearch Edge |
| :-- | :-- | :-- |
| Vector Search (KNN) | 2x-12x faster p99 latency [^6_1] | N/A |
| Text Query/Sorting | 40%-140% faster [^6_2] | N/A |
| Big5/Date Histogram | N/A | 1.6x faster [^6_3] |
| Resource Efficiency | 37% less disk, lower CPU [^6_2] | Comparable in some [^6_4] |

Elastic-funded benchmarks favor ES; independent ones (e.g., Trail of Bits) show mixed results with OpenSearch competitive.[^6_3][^6_1][^6_2]

## Wazuh Context

Wazuh indexer (OpenSearch 2.x) handles typical SIEM loads efficiently; tune heap (`jvm.options`) to 50% RAM max. For high APS (>1000), cluster nodes regardless of engine. Licensing makes OpenSearch the production choice.[^6_5][^6_4][^6_3]
<span style="display:none">[^6_10][^6_6][^6_7][^6_8][^6_9]</span>

<div align="center">⁂</div>

[^6_1]: https://www.elastic.co/search-labs/blog/elasticsearch-opensearch-vector-search-performance-comparison

[^6_2]: https://www.elastic.co/blog/elasticsearch-opensearch-performance-gap

[^6_3]: https://blog.trailofbits.com/2025/03/06/benchmarking-opensearch-and-elasticsearch/

[^6_4]: https://www.youtube.com/watch?v=QF0eCS1xr8I

[^6_5]: https://documentation.wazuh.com/current/integrations-guide/opensearch/index.html

[^6_6]: https://bigdataboutique.com/blog/elasticsearch-vs-opensearch-2025-update-5b5c81

[^6_7]: https://redis.io/blog/elasticsearch-vs-opensearch/

[^6_8]: https://documentation.wazuh.com/current/integrations-guide/elastic-stack/index.html

[^6_9]: https://www.reddit.com/r/elasticsearch/comments/1i1035l/is_the_2023_elasticsearch_vs_opensearch_benchmark/

[^6_10]: https://groups.google.com/g/wazuh/c/MOdLCNzemTc


---

# How does indexing impact search latency in both engines

Indexing operations in both OpenSearch (Wazuh indexer) and Elasticsearch create or update Lucene segments, which must merge periodically; high indexing rates increase segment count, degrading search latency until merges complete. Concurrent indexing amplifies this, with benchmarks showing Elasticsearch 2x-8x faster p99 search latency under indexing load, while OpenSearch improves throughput by 25% in recent versions. Factors like JVM heap, disk I/O, and refresh_interval tune the impact.

## Latency Impact Factors

- **Segment Merges**: Frequent indexing builds many small segments; searches scan more, raising latency. Set `index.refresh_interval` higher (e.g., 30s) to batch.[^7_1][^7_2]
- **Concurrent Load**: In read+write tests, ES maintains ~7-16ms p99 KNN latency vs. OpenSearch's 49ms (4x slower).[^7_3]
- **JVM GC**: Heavy indexing triggers GC pauses, spiking p99 tails; limit heap to 30GB max.[^7_2]
- **Bulk Sizing**: Optimal 5-15MB per bulk request minimizes overhead in both.[^7_4]


## Benchmark Comparison

| Scenario | ES Latency (p99) | OS Latency (p99) | Notes [^7_3][^7_5] |
| :-- | :-- | :-- | :-- |
| KNN Search + Indexing | 13.8 ms | 49.3 ms | 4x slower OS |
| Date Histogram (no cache) | ~7s | ~6.7s | Similar |
| High Throughput Indexing | Baseline | +25% faster | OS 2.11 vs ES 7.10 |

OpenSearch matches or beats ES in aggregations but lags in mixed workloads; monitor via `_cat/indices` and adjust for Wazuh alert volumes.[^7_6][^7_5][^7_7]
<span style="display:none">[^7_10][^7_11][^7_8][^7_9]</span>

<div align="center">⁂</div>

[^7_1]: https://discuss.elastic.co/t/what-is-indexing-latency/78125

[^7_2]: https://redis.io/blog/elasticsearch-vs-opensearch/

[^7_3]: https://www.elastic.co/search-labs/blog/elasticsearch-opensearch-vector-search-performance-comparison

[^7_4]: https://drdroid.io/alert-diagnosis-knowledge/opensearch-indexing-latency-high

[^7_5]: https://opensearch.org/blog/opensearch-performance-improvements/

[^7_6]: https://blog.trailofbits.com/2025/03/06/benchmarking-opensearch-and-elasticsearch/

[^7_7]: https://www.elastic.co/blog/elasticsearch-opensearch-performance-gap

[^7_8]: https://blunders.io/posts/es-benchmark-3-latency

[^7_9]: https://forum.opensearch.org/t/having-a-latency-issue-with-opensearch/10087

[^7_10]: https://www.elastic.co/search-labs/blog/elasticsearch-semantic-text-vs-opensearch-semantic-field

[^7_11]: https://opensearch.org/blog/opensearch-project-update-performance-progress-in-opensearch-3-0/


