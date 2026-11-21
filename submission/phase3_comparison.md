# Phase 3 — Comparison Writeup

## 1. Facts from your runs

| | Value |
|---|---|
| Map tasks in the cloud job (`mapreduce.num_splits`) | <!--answer:num_map_tasks-->12<!--/answer--> |
| Reduce tasks in the cloud job (`mapreduce.num_reducers`) | <!--answer:num_reduce_tasks-->4<!--/answer--> |
| Cloud MapReduce wall time, seconds (`mapreduce.cloud_elapsed_s`) | <!--answer:cloud_elapsed_s-->51.1<!--/answer--> |
| Local single-process wall time, seconds (`mapreduce.local_elapsed_s`) | <!--answer:local_elapsed_s-->0.015<!--/answer--> |
| Spark wall time for the `collect`, seconds (`tfidf.spark_elapsed_s`) | <!--answer:spark_elapsed_s-->1.5<!--/answer--> |
| Number of **stages** in the Spark job (read off notebook 2b / 3a) | <!--answer:num_stages-->4<!--/answer--> |
| Task **retries** you counted in your `--chaos` run (0 if none happened) | <!--answer:chaos_retries-->18<!--/answer--> |

**Explain the timings.**

<!--answer:timing_explanation-->
The local word-count run was fastest (0.015 s), followed by Spark TF-IDF (1.5 s), then cloud MapReduce (51.1 s). The local and cloud MapReduce timings are the like-for-like comparison because both compute the same word count. Spark performs the heavier TF-IDF computation, including two reduceByKey operations and a join, on one machine. The much longer cloud time mainly reflects distributed-system overhead: workflow orchestration, HTTP calls, worker startup, Cloud Storage I/O and retry delays, rather than the word-count computation itself.
<!--/answer-->

**What the counts tell you.**

<!--answer:counts_interpretation-->
The Spark execution had 3 shuffle operations: the two reduceByKey operations and the join. I counted 18 task retries across 16 initial Map and Reduce tasks. This is possible because a task can be retried more than once. The retry count is not expected to equal exactly 30%: --chaos 0.3 applies a random failure probability to worker invocations, and retries can occur multiple times.
<!--/answer-->

---

## 2. Spark: transformations, actions, lineage

**Transformations vs actions.**

<!--answer:transformations_vs_actions-->
The RDD transformations are flatMap, map, reduceByKey and join; they build a computation lineage lazily. Calling build_tfidf_rdd constructs this lineage but does not execute the full computation. collect is an action: it triggers Spark execution of the required stages and brings the resulting TF-IDF rows to the driver. The notebook's count or show operations, when used, are also actions.
<!--/answer-->

**Lineage and recovery.**

<!--answer:lineage_recovery-->
If a partition produced by the first reduceByKey were lost, Spark would recompute that missing shuffle partition from its parent RDD partitions, starting at the reduceByKey shuffle boundary. It can reuse surviving partitions and completed shuffle outputs where available; it does not need to restart the entire application. Downstream stages that depend on the lost partition must then be rerun.
<!--/answer-->

---

## 3. Your cloud MapReduce, mapped onto Hadoop

- **JobTracker** (schedules tasks, waits for the map phase, retries failures):
  <!--answer:role_jobtracker-->Google Cloud Workflows, using the `mr-wordcount` workflow.<!--/answer-->
- **Workers** (run your map and reduce code):
  <!--answer:role_workers-->The `mr-worker` Cloud Run service, executing `main.py`.<!--/answer-->
- **HDFS** (holds inputs, shuffle files, outputs):
  <!--answer:role_hdfs-->Google Cloud Storage, which stores input splits, intermediate shuffle JSON files and reducer output files.<!--/answer-->
- **Partitioner** (decides which reducer owns a key):
  <!--answer:role_partitioner-->The `partition()` function in `mapreduce.py`, using `zlib.crc32(word.encode("utf-8")) % num_reducers`.<!--/answer-->

**The barrier.**

<!--answer:barrier_location-->
The barrier is the end of the `map_phase` parallel step. Cloud Workflows waits for all branches in that parallel block to finish before entering `reduce_phase`.
<!--/answer-->

**Idempotence.**

<!--answer:idempotence-->
Each map task writes its output to deterministic Cloud Storage object names, using `upload_from_string()` through `write_json()`. Repeating the same task overwrites the same objects with the same computed content instead of creating duplicates. If run_map appended pairs instead, a retry could duplicate intermediate records, causing reducers to count words more than once and produce incorrect totals.
<!--/answer-->

**The chaos run.**

<!--answer:chaos_observations-->
With `--chaos 0.3`, the workflow execution log showed simulated worker failures and retry handling. Logs Explorer contained five HTTP 503 request entries from Google Cloud Workflows to `mr-worker`. Despite these failures, the `mr-wordcount` execution finished with state SUCCEEDED. The result reported `matches_local: true`, confirming that the final distributed word counts matched the local reference.
<!--/answer-->
