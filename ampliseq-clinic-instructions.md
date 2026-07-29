# nf-core/ampliseq Clinic — Participant Guide

**Goal of today:** get your own paired-end Illumina amplicon data running through nf-core/ampliseq, using the *official documentation* as your main tool — not this sheet.
This guide is a map to the docs, not a replacement for them.
Whenever there's a link, open it and look for the answer there first.

---

## 0. Before you touch anything: bookmark these three pages

1. **nf-core/ampliseq pipeline page** — https://nf-co.re/ampliseq
   Every nf-core pipeline has a page like this with tabs: *Introduction, Usage, Parameters, Output, Results*.
   This is your home base for the whole day.
2. **nf-core general docs** — https://nf-co.re/docs
   Covers everything that's true for *all* nf-core pipelines (installation, running, profiles, troubleshooting).
   If a question isn't ampliseq-specific, it's answered here.
3. **nf-core/configs institutional profiles** — https://nf-co.re/configs
   The list of `-profile` names for specific HPCs/institutes, including `uppmax` and `pdc_kth`.

---

## 1. Find out what nf-core/ampliseq actually needs from you

Before running anything, open the **ampliseq → Usage** tab (https://nf-co.re/ampliseq/docs/usage) and answer these for yourself:

- What does my **samplesheet** need to look like?
  (Look for the "Samplesheet input" section.)
- Do I have my **primer sequences** (`--FW_primer` / `--RV_primer`)?
  If you genuinely have no primers in your reads, note the `--skip_cutadapt` option — but only use it if you're sure.
- Do I want to attach **metadata** (`--metadata`)?
  Optional, but improves downstream plots.

Then check the **Parameters** tab (https://nf-co.re/ampliseq/parameters) — this is auto-generated from the pipeline schema and is the authoritative list of every flag.
Searching this page is faster than searching old blog posts or forum threads.

**Exercise for participants:** before running the pipeline, write down the exact command you *think* you'll need, using only what you found on these two pages.

### Example samplesheet, for reference

The pipeline ships its own example samplesheet — worth opening once so you've seen a real one: https://github.com/nf-core/ampliseq/blob/master/docs/usage.md (see "Samplesheet input").

Here's a minimal one for our clinic's case (paired-end Illumina, two sequencing runs), `samplesheet.tsv`, assuming your fastq files live in a `reads/` subdirectory of your working directory (see the tip on organizing your working directory below):

```
sampleID	forwardReads	reverseReads	run
sample1	./reads/S1_R1_001.fastq.gz	./reads/S1_R2_001.fastq.gz	run1
sample2	./reads/S2_R1_001.fastq.gz	./reads/S2_R2_001.fastq.gz	run1
sample3	./reads/S3_R1_001.fastq.gz	./reads/S3_R2_001.fastq.gz	run2
sample4	./reads/S4_R1_001.fastq.gz	./reads/S4_R2_001.fastq.gz	run2
```

A few rules straight from the schema (confirm against the Usage tab, these can change between versions):

- Columns are **tab-separated** (`.tsv`); comma-separated (`.csv`) and YAML (`.yml`/`.yaml`) are also accepted.
- `sampleID` and `forwardReads` are **required**; `reverseReads` and `run` are optional (`reverseReads` is required for paired-end data, which is our case).
- `sampleID` must be unique, start with a letter, and contain only letters, numbers, or underscores.
- FastQ files must be gzip-compressed (`.fastq.gz` / `.fq.gz`).
- Only fill in `run` if your samples came from more than one sequencing run — it lets the pipeline model each run's error profile separately.

**Exercise:** build your own samplesheet for your actual files using this as a template, then check it against the Usage tab's rules before you launch anything.

**Tip — easiest way to actually build this file:** for most people, the simplest approach is to fill it in as a spreadsheet in Excel (or Google Sheets/Numbers) on your own laptop, save/export it as a tab-separated `.tsv`, then transfer that one small file over to your working directory alongside your data.
This avoids fiddling with columns and tabs directly in a terminal.
If you need to make a small edit once it's already on the cluster or your laptop's terminal — fixing a typo, adding a row — the simplest text editor to do that with is `nano`: run `nano samplesheet.tsv`, edit directly, then save and exit with `Ctrl-O` (write out) followed by `Ctrl-X` (exit).
No need to learn `vim` or `emacs` for this.

**Tip — save time if you don't need the QIIME2 plots:** the downstream QIIME2-based steps (barplots, diversity analysis, differential abundance testing) are often the slowest part of a run.
If you're mainly after ASVs/taxonomy and don't need those plots, add `--skip_qiime` to skip them — worth deciding on up front, especially if compute time is tight during the clinic.
Check the Parameters tab for the exact scope of what this skips versus related flags like `--skip_taxonomy`.

---

## 2. Everyone: get the basics of running any nf-core pipeline

Read (or skim) the **"Run your first pipeline"** guide: https://nf-co.re/docs/usage/introduction and https://nf-co.re/docs/get_started/run-your-first-pipeline

Key ideas you should be able to explain afterward:

- Nextflow pulls the pipeline code and containers automatically — you don't clone a repo by hand.
- `-profile` (single dash) picks *how software is provided and where jobs run* (docker, singularity, conda, or an institutional profile like `uppmax`).
- `--` (double dash) flags are pipeline parameters (`--input`, `--outdir`, `--FW_primer`, etc.); `-` (single dash) flags are Nextflow options (`-profile`, `-resume`, `-r`).
- Always pin a pipeline version with `-r <version>` (e.g. `-r 2.15.0`) for reproducibility — check the current stable release on the ampliseq page.
- `-resume` lets you re-launch after a crash without redoing finished steps.

**Tip — installing your own Nextflow:** if a cluster's preloaded Nextflow module is older than what ampliseq requires (check the `nextflow_schema.json` / pipeline manifest for the minimum version), you don't need admin help — Nextflow installs into your own home directory with one command.
See the official install docs: https://nf-co.re/docs/get_started/environment_setup/nextflow (or https://www.nextflow.io/docs/latest/install.html).
This is often the fastest fix when `nextflow -v` on a cluster is behind what the pipeline expects.
This is also exactly what you'll do if you're on **Track A (your own laptop)** — there's no cluster module to load, so a self-install is the normal, expected route rather than a workaround.

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

## 3. New to the terminal? The minimum you need for today

If you've never really used a command-line terminal before, this is genuinely the whole toolkit you need for Tracks A–C below — not a full Linux course, just the handful of moves that come up in this guide.

Everything for today happens from a terminal prompt: on your laptop that means opening a terminal app; on UPPMAX or Dardel it's what you land in right after logging in over SSH.
From there, four commands cover almost everything:

- `pwd` — prints where you currently are.
  Run this whenever you're unsure of your location; it's harmless and always safe to run.
- `ls` — lists what's in your current directory.
  Add `-lh` (`ls -lh`) for a fuller listing with human-readable file sizes — useful for checking how big your fastq files are before a transfer.
- `cd <name>` — moves into a directory named `<name>`.
  `cd ..` moves up one level; typing `cd` alone (no argument) takes you back to your home directory.
- `mkdir <name>` — creates a new, empty directory named `<name>`.

Put together, this is the pattern you'll use before running anything today — make a working folder, move into it, and confirm where you are:

```
mkdir ampliseq_clinic
cd ampliseq_clinic
pwd
ls
```

The `pwd` output here gives you the **full path** to this folder — worth noting down, since you'll want it later when writing paths into your samplesheet or pointing `--outdir` somewhere specific.
`ls` will show nothing yet, until you add or transfer files into it.

**Worth setting up now: a `reads` subdirectory.** Inside this working folder, make one more directory to keep your sequence files separate from everything else:

```
mkdir reads
```

Your fastq files (transferred in, or copied locally on a laptop) should end up inside this `reads/` folder.
This isn't strictly required by ampliseq, but it keeps your working directory tidy, and it's exactly the layout the example samplesheet above assumes (`./reads/S1_R1_001.fastq.gz`, etc.) — so the paths you write into your samplesheet will line up directly with where your files actually are.

One habit worth having from the start: press **Tab** while typing a filename or path to auto-complete it.
This avoids retyping long fastq filenames by hand, and sidesteps the typos that samplesheet validation will otherwise catch you on later.

---

## 4. Choose your track: where will you run it?

### Track A — Your own laptop

- Docs: https://nf-co.re/docs/usage/introduction and https://nf-co.re/docs/get_started/environment_setup/overview
- You'll need Nextflow + one of Docker / Singularity / Conda installed.
  The Environment Setup docs walk through installing each.
- Use `-profile docker` (or `singularity`/`conda` depending what you installed).
- Before doing anything else, open a terminal and set up a working folder for today using the `mkdir`/`cd` pattern from Section 3 above — this is where you'll keep your samplesheet, your data (or a shortcut to it), and your results.
- Even without a network transfer, it's worth putting your fastq files into a `reads/` subdirectory of that working folder (same convention as the other tracks) — either copy them there, or if they're large, a symbolic link (`ln -s /original/path/to/data reads`) avoids duplicating the files while still giving you the tidy `./reads/...` paths for your samplesheet.
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
- Right after logging in, you'll land in your home directory — use `cd` to move into your project's directory (ask your PI or check SUPR for the path if you don't already know it), then `mkdir` a working folder for today, same pattern as in Section 3.
- Use `-profile uppmax` and pass your UPPMAX project id with `--project <your-id>` (two dashes — this is a pipeline-adjacent config parameter, not a Nextflow core flag).
- Find your project id: run `groups` on the cluster, or check SUPR.
- Run Nextflow on the **login node** inside `tmux` or `screen` (not directly, and not requesting a compute job for the Nextflow driver itself) — see the UPPMAX page for why.
- **Get your files uploaded to the HPC:** use `rsync` or `scp` from your own machine to copy files to the cluster (or the reverse, to fetch results back).
  Transfer your fastq files into the `reads/` subdirectory of the working folder you just created — that keeps them in exactly the location the example samplesheet's paths (`./reads/...`) expect.
  UPPMAX's transfer docs, with copy-pasteable examples: https://docs.uppmax.uu.se/cluster_guides/transfer_rackham/ (Rackham) — other clusters have their own equivalent pages linked from the same docs site.
  If you're working with sensitive data on Bianca, transfers go through the separate Transit service instead — see https://docs.uppmax.uu.se/cluster_guides/transfer_bianca/.
  Once your files have landed, `cd` into `reads/` and `ls` to confirm what actually arrived before building your samplesheet around it.

### Track C — Dardel (PDC / KTH)

- Docs: https://nf-co.re/configs/pdc_kth (also mirrored at https://github.com/nf-core/configs/blob/master/docs/pdc_kth.md)
- Dardel doesn't preload Nextflow — check the docs page for how to fetch it if it isn't already available to you.
- After logging in, the same `pwd`/`ls`/`cd`/`mkdir` pattern from Section 3 applies — use it to find your project directory and set up a working folder before doing anything else.
- **Create this working (run) directory in permanent storage — your home directory or project directory (`/cfs/klemming/projects/...`) — not in scratch.** Scratch is only for Nextflow's `work/` directory later on (see the tip below); your samplesheet, your `reads/` data, and where `--outdir` points should all live under this permanent directory, since scratch gets auto-cleaned after about a month and you don't want your actual run set up to disappear.

**Where to actually run Nextflow — this is different from UPPMAX, don't just reuse that tip.** On UPPMAX, running Nextflow directly on the login node inside `tmux`/`screen` is the expected, supported approach.
Dardel is stricter: the official route is to request an **interactive allocation** first, then run Nextflow on the allocated node itself, not the login node.

```
salloc --nodes=1 -t 04:00:00 -A <your-allocation> -p shared
```

Once `salloc` reports the allocation is granted, log in to the compute node it gave you (`ssh <node-name>`, e.g. `ssh nid001234`), and run Nextflow there.
See PDC's own docs on this: https://support.pdc.kth.se/doc/run_jobs/run_interactively/

Because this can run for hours, run it inside `screen` or `tmux`. These are usually described as "terminal multiplexers," but that's not really why you need one here — the feature that matters is that they let you **walk away and come back**. Start Nextflow inside one of these, detach, close your laptop lid or lose your wifi, and when you reconnect later you reattach to the exact same running session, with Nextflow still going. Without this, closing your terminal or dropping the SSH connection kills the process — which, on an hours-long run, is exactly what you don't want.

- `screen`: `screen -S ampliseq`, detach with `Ctrl-a d`, reattach later with `screen -r ampliseq`.
  New to `screen`? A quick, friendly walkthrough: https://opensource.com/article/21/4/gnu-screen-cheat-sheet — or the official manual for full reference: https://www.gnu.org/software/screen/manual/screen.html
- `tmux` (same idea, different keys, use whichever your instructor demonstrates): start a named session with `tmux new -s ampliseq`; detach with `Ctrl-b d` (press Ctrl+b, release, then press d); reattach later with `tmux attach -t ampliseq`; list running sessions with `tmux ls`.

**Unofficial reality, worth knowing about but not relying on:** in practice, running Nextflow directly on the Dardel login node (skipping `salloc` entirely) often works fine, since Nextflow itself is lightweight and just submits Slurm jobs.
The risk is that this isn't the sanctioned use of the login node — you may get warning emails about it, or in stricter periods the process can simply get killed.
For a one-off clinic run this may be an acceptable shortcut, but the `salloc` + compute-node approach above is the one to teach as the correct default.

**Loading modules on Dardel — this part is genuinely hard to piece together from the docs, so here it is explicitly.** A working module load line:

```
module load bioinfo-tools PDC apptainer java/17.0.4 nextflow
```

A few things worth knowing about why it looks like this:

- `bioinfo-tools` and `PDC` are prerequisite module collections — load them first, or the others may not resolve.
- Dardel offers **both** a `singularity` module and an `apptainer` module — load **apptainer**, not singularity.
  Apptainer is the actively maintained fork and what nf-core pipelines expect on this cluster; the two are not fully interchangeable in practice.
- Nextflow needs a reasonably recent Java — `java/17.0.4` is known to work.
- This gives a modern Nextflow (currently 26.04.0 via this module) — check `nextflow -v` after loading to confirm what you got, and compare against what ampliseq requires.

- Use `-profile pdc_kth` and pass your Dardel/PDC allocation with `--project <your-allocation>` (e.g. `naiss-XXXX-XX-XXX`).
- Note from the docs: Dardel's partitions lack long-runtime, high-memory nodes, so some heavier steps may need extra tuning — read the "known issues" notes on that page if a job seems stuck or killed.
- `/tmp` on Dardel is RAM-backed, not disk — worth knowing before you write large temp files there.

**Tip — use the scratch area for Nextflow's work directory:** Dardel has a global scratch space at `/cfs/klemming/scratch/<u>/<user>` (where `<u>` is the first letter of your username and `<user>` your full username).
Point Nextflow's work directory there with `-w`:

```
nextflow run nf-core/ampliseq \
  -profile pdc_kth --project <your-allocation> \
  -w /cfs/klemming/scratch/<u>/<user>/work \
  ...
```

This keeps the (often large, temporary) intermediate files out of your storage-project quota.
The trade-off: **scratch is auto-cleaned — files untouched for about a month get deleted** — which is exactly why only `-w` (Nextflow's `work/` directory) should point here, and why your run directory itself should stay in permanent storage as set up above.
Make sure final results are written to durable storage with the `--outdir` parameter.
See PDC's own storage docs for the current details: https://support.pdc.kth.se/doc/data_management/klemming/

**Get your files uploaded to the HPC:** use `scp` or `rsync` from your own machine to copy files to Dardel (or back, for results).
PDC's file transfer docs, with worked examples: https://support.pdc.kth.se/doc/data_management/file_transfer/.
Send your fastq files into the `reads/` subdirectory of your working folder (in permanent storage, as set up above).
A minimal example, from your own laptop:

```
scp your_local_file <username>@dardel.pdc.kth.se:/path/to/ampliseq_clinic/reads/
```

`cd` into `reads/` and `ls` afterwards to confirm what actually landed before you build your samplesheet.

### Track D (rare) — PacBio long reads + Seqera Platform

If this is you: first confirm with an instructor that ampliseq's current long-read support fits your data — check the **Introduction** tab on the ampliseq page for which sequencing types are currently supported, since this can change between versions.

- Seqera Platform docs: https://docs.seqera.io/platform
- Launching a pipeline: https://docs.seqera.io/platform/latest/getting-started/quickstart-demo/launch-pipelines
- In Seqera Platform you don't type the `nextflow run` command yourself — you pick the pipeline (`nf-core/ampliseq`), pick a compute environment, and fill in the same parameters (`--input`, `--FW_primer`, etc.) through a generated web form.
  The form is built from the same parameter schema you looked at in Section 1, so what you learned there still applies directly.
- You'll need a configured **compute environment** in Seqera (e.g. AWS Batch) before you can launch — ask an instructor if none exists yet for the clinic.

---

## 5. When something goes wrong

Don't guess — go back to the docs in this order:

1. **Troubleshooting docs**: https://nf-co.re/docs/usage/troubleshooting — covers common Nextflow/nf-core error patterns (exit codes, memory errors, container pull failures).
2. **The specific error's exit code** — nf-core docs explain common exit codes like 137 (out of memory) and 140 (timeout), and how to raise resource limits with a small custom config.
3. **The pipeline's GitHub issues**: https://github.com/nf-core/ampliseq/issues — search before opening a new one; most errors have been hit before.
4. **nf-core Slack**: join at https://nf-co.re/join/slack, then use the `#ampliseq` channel for pipeline-specific questions, or `#helpdesk-hpc-sweden` for UPPMAX/Dardel-specific issues.

---

## 6. Wrap-up checklist

By the end of the clinic you should be able to point to, in the actual docs, where you'd find:

- [ ] The exact samplesheet format required
- [ ] The full parameter list and what each one does
- [ ] The right `-profile` for your machine
- [ ] How to pin a pipeline version
- [ ] Where to look first when a run fails
- [ ] Where the pipeline's output files land and what they mean (Output tab: https://nf-co.re/ampliseq/docs/output)

If you can find all six on your own by the end of the session, the clinic did its job.
