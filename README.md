# SOC-Style Login Anomaly Detection (UEBA) with ATO Validation

📓 **Notebook:** [`ueba_login_anomaly_detection.ipynb`](notebooks/ueba_login_anomaly_detection.ipynb)

## Overview
This project builds a SOC-style behavioral anomaly detection pipeline on authentication logs.
Instead of classifying each login independently, it aggregates activity into user-day behavior profiles,
learns per-user baselines, and flags deviations as ranked alerts. Alerts are explained in human-readable
terms and validated using confirmed account takeover (ATO) labels.
This mirrors how modern SOCs prioritize analyst review using behavioral risk ranking.

## Dataset
Kaggle: Login Data Set for Risk-Based Authentication (RBA)  
Source: https://www.kaggle.com/datasets/dasgroup/rba-dataset

- ~31M login events, 16 fields (timestamp, user, IP, geo, device/browser/OS, success, attack IP, ATO label)

Key columns used:
- Identity/time: User ID, Login Timestamp
- Network/geo: IP Address, Country, ASN
- Device: Device Type, Browser/OS
- Labels (evaluation only): Is Account Takeover, Is Attack IP

## Threat Model
Detect suspicious login behavior consistent with:
- credential stuffing / brute force (login spikes, many IPs)
- compromised credentials (new device + geo churn)
- malicious infrastructure (attack IP enrichment)

## Method
1. **User-day aggregation**: logins/day, success rate, unique IPs, countries, ASNs, devices, night ratio  
2. **Per-user baselines (UEBA)**: mean/std behavior per user  
3. **Deviation features**: z-score logins, ratios of IP/country/device churn, night deviation  
4. **Unsupervised detection**: Isolation Forest to rank anomalous user-days  
5. **Alert explanations**: rule-based reasons (e.g., “login volume far above baseline”, “multiple new IPs”)  
6. **Validation**: compare ATO enrichment in top-ranked alerts vs baseline rate  

## Results
- Baseline ATO rate (all user-days): ~0.00115%
- ATO rate in top 0.1% anomalous user-days: ~0.0248%
- Lift: ~21.6× enrichment of account takeovers in the top 0.1% alerts relative to baseline

## Example Alert Explanation
- “login volume far above baseline; multiple new IP addresses; new devices observed”

## Limitations & Future Work
- Production systems would integrate additional threat intelligence feeds (IP reputation, VPN/datacenter ASN tags)
- Time-aware “impossible travel” and sequence-based models could further improve prioritization
- Labels can be delayed or incomplete in real SOC settings; ranking-based evaluation is preferred over accuracy

## How to Run
Open the notebook in `notebooks/` and run cells in order.
