---
marp: true
title: nf-core/ampliseq Clinic
---

# nf-core/ampliseq Clinic

Get your own paired-end Illumina amplicon data running through nf-core/ampliseq

**Today's approach:** the docs are the tool. This deck just points you at them.

---

## Bookmark these three

1. **Pipeline page** — https://nf-co.re/ampliseq
   Introduction / Usage / Parameters / Output / Results tabs
2. **General nf-core docs** — https://nf-co.re/docs
   Anything true for *all* nf-core pipelines
3. **Institutional profiles** — https://nf-co.re/configs
   `-profile uppmax`, `-profile pdc_kth`, etc.

---

## Step 1 — Know what ampliseq needs

Open **ampliseq → Usage**: https://nf-co.re/ampliseq/docs/usage

- Samplesheet format
- `--FW_primer` / `--RV_primer`
- Optional `--metadata`

Check **Parameters**: https://nf-co.re/ampliseq/parameters
— the authoritative flag list, generated from the pipeline schema

**Exercise:** write your command *before* you touch a terminal

---

## Example samplesheet (paired-end, two runs)

`samplesheet.tsv` (tab-separated):

```
sampleID	forwardReads	reverseReads	run
sample1	./data/S1_R1_001.fastq.gz	./data/S1_R2_001.fastq.gz	run1
sample2	./data/S2_R1_001.fastq.gz	./data/S2_R2_001.fastq.gz	run1
sample3	./data/S3_R1_001.fastq.gz	./data/S3_R2_001.fastq.gz	run2
sample4	./data/S4_R1_001.fastq.gz	./data/S4_R2_001.fastq.gz	run2
```

- `sampleID` + `forwardReads` required; `reverseReads` required for paired-end
- IDs: unique, start with a letter
- FastQ must be `.gz`
- `run` only needed for multi-run data

Official example: github.com/nf-core/ampliseq → `docs/usage.md`

---

## Step 2 — nf-core basics (applies to any pipeline)

https://nf-co.re/docs/usage/introduction

- `-profile` → software + where jobs run
- `--flag` → pipeline parameter, `-flag` → Nextflow option
- Pin a version: `-r <version>`
- Resume a crashed run: `-resume`

```
nextflow run nf-core/ampliseq \
  -r <version> -profile <docker/uppmax/pdc_kth/...> \
  --input "samplesheet.tsv" \
  --FW_primer <p1> --RV_primer <p2> \
  --outdir "./results"
```

---

## Step 3 — Pick your track

| Track | Profile | Docs |
|---|---|---|
| Laptop | `docker` / `singularity` / `conda` | nf-co.re/docs/get_started/environment_setup |
| UPPMAX | `uppmax` + `--project` | nf-co.re/configs/uppmax |
| Dardel (PDC/KTH) | `pdc_kth` + `--project` | nf-co.re/configs/pdc_kth |
| PacBio + Seqera *(rare)* | Seqera launch form | docs.seqera.io/platform |

Always test first: `-profile test,<yours>`

---

## Step 4 — When it breaks

1. **Troubleshooting docs** — nf-co.re/docs/usage/troubleshooting
2. **Exit code** (137 = OOM, 140 = timeout)
3. **GitHub issues** — github.com/nf-core/ampliseq/issues
4. **Slack** — `#ampliseq`, `#helpdesk-hpc-sweden`

---

## By the end of today, you should be able to find...

- [ ] Samplesheet format
- [ ] Full parameter list
- [ ] Right `-profile` for your machine
- [ ] How to pin a version
- [ ] Where to start when a run fails
- [ ] Where outputs land (Output tab)

**That's the win — not just a finished run.**
