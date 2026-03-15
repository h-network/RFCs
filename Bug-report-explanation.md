# Bug Report Answers — Pre-emptive Counterarguments

Reference: Bug report filed against `nv-morpheus/Morpheus` regarding `abp-pcap-xgb` model false positive rate.

---

## Anticipated Response #1: "The model is just an example, not production-ready"

The README calls it an example pipeline. It was never intended for arbitrary traffic.

The documentation makes no mention of this limitation. The README presents a complete, working pipeline with no caveats about the model only functioning on its own training data. If the model is purely demonstrative, this should be stated explicitly. Users following the official example will reasonably expect the model to detect anomalies in their traffic, not flag everything as anomalous.

---

## Anticipated Response #2: "You need to retrain the model on your own data"

XGBoost is a supervised model. Of course it needs training on your environment.

Agreed — but there are no retraining instructions, scripts, or guidance provided. The example ships a pre-trained model and a pipeline to run it. The gap between "here's a working example" and "train your own model" is completely undocumented. We're asking for that documentation, not for the model to magically generalise.

---

## Anticipated Response #3: "Your tshark/jq conversion may not match the expected input format"

The custom `tshark_to_morpheus.jq` transform may produce subtly different data than the training set, causing the model to misclassify.

1. The jq transform produces the **exact same JSON schema** as `examples/data/abp_pcap_dump.jsonlines`: `timestamp`, `host_ip`, `data_len`, `data`, `src_mac`, `dest_mac`, `protocol`, `src_ip`, `dest_ip`, `src_port`, `dest_port`, `flags` — all string-typed, matching the sample data format
2. The Morpheus preprocessing stage (`AbpPcapPreprocessingStage`) accepted the input **without any errors or warnings** — it successfully parsed flags, computed flow IDs, performed groupby aggregation, and produced valid tensors for Triton
3. If the format were wrong, the pipeline would error at the preprocessing stage, not silently produce valid-but-wrong classifications
4. The full jq transform is included in the bug report for verification

---

## Anticipated Response #4: "The binary threshold hides the real model performance — try different thresholds"

The `add-class` stage binarises at 0.5. The raw scores might show better separation with threshold tuning.

We extracted the raw probability scores. The results are worse, not better:

| Scenario | Mean Score | Median | Min | Max |
|----------|-----------|--------|-----|-----|
| DDoS attack | 0.958 | 1.0 | 0.0 | 1.0 |
| SCP outbound | 0.997 | 1.0 | 0.0 | 1.0 |
| SCP inbound | 1.000 | 1.0 | 1.0 | 1.0 |

The SCP inbound capture has **min=1.0, max=1.0** — every single packet received the maximum possible anomaly score. The legitimate file transfer has higher confidence scores than the actual DDoS attack. No threshold selection would produce meaningful separation. The model is saturated.

---

## Anticipated Response #5: "Your sample size is too small"

5,000 packets per run isn't enough for statistical significance.

1. We ran 5 consecutive captures of 5,000 packets for the DDoS scenario — **24,548 packets total** with consistent results across all runs (95.7%-100%)
2. Two additional captures of ~5,000 packets for SCP scenarios
3. A separate test with 5 runs of 500 packets (Report #1) — **2,346 packets** showing the same 98.8% rate
4. Total real-world packets analysed: **36,872**
5. The results are consistent across every single run. There is no variance to speak of — the model saturates regardless of sample size
6. We also ran the NVIDIA sample data at 537,241 packets and got 2.6% — proving the pipeline works correctly when the data matches the training distribution

---

## Anticipated Response #6: "Capturing on a shared interface mixes traffic — you should isolate scenarios"

Background LAN traffic (mDNS, DHCP, broadcasts) mixed into the capture makes the results less clean.

1. Real networks have background traffic. A model that can't handle mDNS and DHCP broadcasts — the most common traffic on any LAN — is not useful for network security
2. The background traffic is itself being flagged at 100%, which is the core problem
3. Even if we could isolate pure SCP-only or pure DDoS-only captures, the raw scores show the model outputs 1.0 for both — isolation wouldn't change the saturation

---

## Anticipated Response #7: "The model was trained for ABP (Anomalous Behavior Profiling) on a specific environment"

ABP means profiling anomalies relative to a known baseline. The model defines "normal" as its training data.

This is essentially acknowledging the bug. The model's definition of "normal" is one specific Kubernetes cluster from 2021. Anything else is "anomalous" — including mDNS, DHCP, SSH, HTTPS, PostgreSQL, and NFS. If the model requires a matching baseline to function, this must be documented. Currently it is not.

---

## Anticipated Response #8: "File an issue, don't write a report"

This should be a normal GitHub issue, not a public document.

It is a GitHub bug report. The supporting reports exist because the evidence is extensive and the methodology should be transparent and reproducible.

---

## The Bottom Line

We're not asking for a model that generalises to all traffic. We're asking for documentation that says it doesn't.
