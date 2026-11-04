# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---
# Phase 1 — Comparison Report

## 1. Setup facts

**Your image size**, in MB:

<!--answer:image_size_mb-->
49.99
<!--/answer-->

**The custom VPC and subnet you created:**

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

| Platform | Cold (ms) | Warm median (ms) |
|---|---:|---:|
| IaaS VM | <!--answer:vm_cold_ms-->255.0<!--/answer--> | <!--answer:vm_warm_median_ms-->210.8<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->139.0<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->631.1<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->190.0<!--/answer--> | <!--answer:function_warm_median_ms-->130.8<!--/answer--> |

**Latency plot**

<!--answer:latency_plot-->
```text
Warm latency (ms)

VM:             ████████████████████  209–215
Cloud Run:      █████████ 120–130
                ███████████ 631–668
Function:       █████████████ 122–133
                █ 197
                ██████ 634–642
```
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour**

<!--answer:scaling_behaviour-->
The VM does not scale automatically; it uses one machine (`e2-micro`). Cloud Run creates additional container instances when traffic increases, up to the configured limit of 3 instances (`--max-instances=3`). Cloud Functions also creates instances automatically according to incoming requests, subject to Google Cloud quotas and instance limits.
<!--/answer-->

**Deployment effort**

<!--answer:deployment_effort-->
The VM required machine, network, firewall and application configuration. Cloud Run required a Dockerfile, image build, Artifact Registry and deployment. Cloud Functions required Python source code, a runtime, an entry point and an HTTP trigger. Google manages most infrastructure and scaling for the two managed platforms, making deployment easier than with a VM.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

**Complete output of `gcloud run services list`:**

<!--answer:run_services_list-->
```text
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-992086212292.us-central1.run.app
LAST DEPLOYED BY: service-992086212292@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-11-04T21:37:17.000898Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-992086212292.us-central1.run.app
LAST DEPLOYED BY: Sounsin-Isaac-Tychique.DAKIO@eurecom.fr
LAST DEPLOYED AT: 2026-11-04T21:36:08.006098Z
```
<!--/answer-->

**Convergence explanation**

<!--answer:convergence_explanation-->
Both Cloud Run and Cloud Functions appear as Cloud Run services because second-generation Functions use Cloud Run infrastructure. This explains their similar cold-start times. However, Cloud Run requires deploying a Docker image, while Cloud Functions deploys source code with a specified entry point. Their warm performance differs: Cloud Run has a median of 631.1 ms, compared with 130.8 ms for the Function.
<!--/answer-->