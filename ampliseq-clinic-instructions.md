# nf-core/ampliseq Clinic — Participant Guide

**Goal of today:** get your own paired-end Illumina amplicon data running through nf-core/ampliseq, using the *official documentation* as your main tool — not this sheet. This guide is a map to the docs, not a replacement for them. Whenever there's a link, open it and look for the answer there first.

---

## 0. Before you touch anything: bookmark these three pages

1. **nf-core/ampliseq pipeline page** — https://nf-co.re/ampliseq
   Every nf-core pipeline has a page like this with tabs: *Introduction, Usage, Parameters, Output, Results*. This is your home base for the whole day.
2. **nf-core general docs** — https://nf-co.re/docs
   Covers everything that's true for *all* nf-core pipelines (installation, running, profiles, troubleshooting). If a question isn't ampliseq-specific, it's answered here.
3. **nf-core/configs institutional profiles** — https://nf-co.re/configs
   The list of `-profile` names for specific HPCs/institutes, including `uppmax` and `pdc_kth`.

---

## 1. Find out what nf-core/ampliseq actually needs from you

Before running anything, open the **ampliseq → Usage** tab (https://nf-co.re/ampliseq/docs/usage) and answer these for yourself:

- What does my **samplesheet** need to look like? (Look for the "Samplesheet input" section.)
- Do I have my **primer sequences** (`--FW_primer` / `--RV_primer`)? If you genuinely have no primers in your reads, note the `--skip_cutadapt` option — but only use it if you're sure.
- Do I want to attach **metadata** (`--metadata`)? Optional, but improves downstream plots.

Then check the **Parameters** tab (https://nf-co.re/ampliseq/parameters) — this is auto-generated from the pipeline schema and is the authoritative list of every flag. Searching this page is faster than searching old blog posts or forum threads.

**Exercise for participants:** before running the pipeline, write down the exact command you *think* you'll need, using only what you found on these two pages.

### Example samplesheet, for reference

The pipeline ships its own example samplesheet — worth opening once so you've seen a real one: https://github.com/nf-core/ampliseq/blob/master/docs/usage.md (see "Samplesheet input").

Here's a minimal one for our clinic's case (paired-end Illumina, two sequencing runs), `samplesheet.tsv`:

```
sampleID	forwardReads	reverseReads	run
sample1	./data/S1_R1_001.fastq.gz	./data/S1_R2_001.fastq.gz	run1
sample2	./data/S2_R1_001.fastq.gz	./data/S2_R2_001.fastq.gz	run1
sample3	./data/S3_R1_001.fastq.gz	./data/S3_R2_001.fastq.gz	run2
sample4	./data/S4_R1_001.fastq.gz	./data/S4_R2_001.fastq.gz	run2
```

A few rules straight from the schema (confirm against the Usage tab, these can change between versions):
- Columns are **tab-separated** (`.tsv`); comma-separated (`.csv`) and YAML (`.yml`/`.yaml`) are also accepted.
- `sampleID` and `forwardReads` are **required**; `reverseReads` and `run` are optional (`reverseReads` is required for paired-end data, which is our case).
- `sampleID` must be unique, start with a letter, and contain only letters, numbers, or underscores.
- FastQ files must be gzip-compressed (`.fastq.gz` / `.fq.gz`).
- Only fill in `run` if your samples came from more than one sequencing run — it lets the pipeline model each run's error profile separately.

**Exercise:** build your own samplesheet for your actual files using this as a template, then check it against the Usage tab's rules before you launch anything.

---

## 2. Everyone: get the basics of running any nf-core pipeline

Read (or skim) the **"Run your first pipeline"** guide: https://nf-co.re/docs/usage/introduction and https://nf-co.re/docs/get_started/run-your-first-pipeline

Key ideas you should be able to explain afterward:
- Nextflow pulls the pipeline code and containers automatically — you don't clone a repo by hand.
- `-profile` (single dash) picks *how software is provided and where jobs run* (docker, singularity, conda, or an institutional profile like `uppmax`).
- `--` (double dash) flags are pipeline parameters (`--input`, `--outdir`, `--FW_primer`, etc.); `-` (single dash) flags are Nextflow options (`-profile`, `-resume`, `-r`).
- Always pin a pipeline version with `-r <version>` (e.g. `-r 2.15.0`) for reproducibility — check the current stable release on the ampliseq page.
- `-resume` lets you re-launch after a crash without redoing finished steps.

A minimal shape (fill in the blanks yourself from what you read in Section 1):

```
nextflow run nf-core/ampliseq \
  -r <version> \
  -profile <docker/singularity/uppmax/pdc_kth/...> \
  --input "samplesheet.tsv" \
  --FW_primer <your primer> \
  --RV_primer <your primer> \
  --outdir "./results"
```

Don't just copy this — go confirm each part against the docs pages above.

---

## 3. Choose your track: where will you run it?

### Track A — Your own laptop

- Docs: https://nf-co.re/docs/usage/introduction and https://nf-co.re/docs/get_started/environment_setup/overview
- You'll need Nextflow + one of Docker / Singularity / Conda installed. The Environment Setup docs walk through installing each.
- Use `-profile docker` (or `singularity`/`conda` depending what you installed).
- **Before running on your real data**, run the pipeline's built-in test profile first to confirm your setup works:
  ```
  nextflow run nf-core/ampliseq -profile test,docker --outdir test_results
  ```
  This is the standard nf-core "does my install work" sanity check — always do this on a new machine.

### Track B — UPPMAX (Rackham / Bianca / Pelle / Miarka)

- Docs: https://nf-co.re/configs/uppmax and UPPMAX's own page https://docs.uppmax.uu.se/software/nextflow/
- Log in to the cluster, then load the modules:
  ```
  module load bioinfo-tools Nextflow nf-core nf-core-pipelines
  ```
- Use `-profile uppmax` and pass your UPPMAX project id with `--project <your-id>` (two dashes — this is a pipeline-adjacent config parameter, not a Nextflow core flag).
- Find your project id: run `groups` on the cluster, or check SUPR.
- Run Nextflow on the **login node** inside `tmux` or `screen` (not directly, and not requesting a compute job for the Nextflow driver itself) — see the UPPMAX page for why.

### Track C — Dardel (PDC / KTH)

- Docs: https://nf-co.re/configs/pdc_kth (also mirrored at https://github.com/nf-core/configs/blob/master/docs/pdc_kth.md)
- Dardel doesn't preload Nextflow — check the docs page for how to fetch it if it isn't already available to you.
- Use `-profile pdc_kth` and pass your Dardel/PDC allocation with `--project <your-allocation>` (e.g. `naiss-XXXX-XX-XXX`).
- Note from the docs: Dardel's partitions lack long-runtime, high-memory nodes, so some heavier steps may need extra tuning — read the "known issues" notes on that page if a job seems stuck or killed.
- `/tmp` on Dardel is RAM-backed, not disk — worth knowing before you write large temp files there.

### Track D (rare) — PacBio long reads + Seqera Platform

If this is you: first confirm with an instructor that ampliseq's current long-read support fits your data — check the **Introduction** tab on the ampliseq page for which sequencing types are currently supported, since this can change between versions.

- Seqera Platform docs: https://docs.seqera.io/platform
- Launching a pipeline: https://docs.seqera.io/platform/latest/getting-started/quickstart-demo/launch-pipelines
- In Seqera Platform you don't type the `nextflow run` command yourself — you pick the pipeline (`nf-core/ampliseq`), pick a compute environment, and fill in the same parameters (`--input`, `--FW_primer`, etc.) through a generated web form. The form is built from the same parameter schema you looked at in Section 1, so what you learned there still applies directly.
- You'll need a configured **compute environment** in Seqera (e.g. AWS Batch) before you can launch — ask an instructor if none exists yet for the clinic.

---

## 4. When something goes wrong

Don't guess — go back to the docs in this order:

1. **Troubleshooting docs**: https://nf-co.re/docs/usage/troubleshooting — covers common Nextflow/nf-core error patterns (exit codes, memory errors, container pull failures).
2. **The specific error's exit code** — nf-core docs explain common exit codes like 137 (out of memory) and 140 (timeout), and how to raise resource limits with a small custom config.
3. **The pipeline's GitHub issues**: https://github.com/nf-core/ampliseq/issues — search before opening a new one; most errors have been hit before.
4. **nf-core Slack**: join at https://nf-co.re/join/slack, then use the `#ampliseq` channel for pipeline-specific questions, or `#helpdesk-hpc-sweden` for UPPMAX/Dardel-specific issues.

---

## 5. Wrap-up checklist

By the end of the clinic you should be able to point to, in the actual docs, where you'd find:

- [ ] The exact samplesheet format required
- [ ] The full parameter list and what each one does
- [ ] The right `-profile` for your machine
- [ ] How to pin a pipeline version
- [ ] Where to look first when a run fails
- [ ] Where the pipeline's output files land and what they mean (Output tab: https://nf-co.re/ampliseq/docs/output)

If you can find all six on your own by the end of the session, the clinic did its job.
