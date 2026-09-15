# Processing-pharmacogenomic-samples
Pipeline for extracting the CYP2C19 genomic region from 1000 Genomes Phase 3 BAM files using SAMtools. Supports parallel processing, sample-level output organization, BAM indexing, execution logs, Excel progress reports, and resuming previous runs.
## CYP2C19 target-region extraction

### Overview

The `scripts/cyp2c19_pipeline.py` script extracts a predefined genomic interval from remotely hosted 1000 Genomes Phase 3 BAM files using SAMtools. Extracted files are organized by sample and alignment category, with processing outcomes recorded in logs and an Excel report.

The configured target interval is `10:96517438-96617309`, annotated in the script as GRCh37/hg19. The script does not verify the reference assembly or retrieve gene coordinates from NCBI.

### Processing steps

1. List sample directories in the configured 1000 Genomes Phase 3 archive.
2. Search each sample for `alignment/` and `high_coverage_alignment/` directories.
3. Select a BAM with a corresponding `.bam.bai` entry, preferring filenames containing `mapped`.
4. Extract alignments overlapping the target interval using `samtools view`.
5. Run `samtools index` on each extracted BAM.
6. Record source URLs, output paths, file sizes, processing status, errors, and timestamps.

Samples can be processed in parallel. Resume mode skips samples previously marked as successful in the Excel report.

### Requirements

- Python 3.9 or later
- SAMtools available on the system `PATH`, with support for remote HTTPS access
- Python packages: `requests`, `beautifulsoup4`, `lxml`, and `openpyxl`
- Internet access to the source archive

Install the Python dependencies:

```bash
python -m pip install requests beautifulsoup4 lxml openpyxl
```

### Usage

Process a small subset first:

```bash
python scripts/cyp2c19_pipeline.py --output results/cyp2c19 --workers 2 --limit 2
```

Process all discovered samples:

```bash
python scripts/cyp2c19_pipeline.py --output results/cyp2c19 --workers 4
```

Resume processing:

```bash
python scripts/cyp2c19_pipeline.py --output results/cyp2c19 --workers 4 --resume
```

### Outputs

```text
results/cyp2c19/
├── pipeline.log
├── progresso_cyp2c19.xlsx
├── logs/
│   └── <SAMPLE>.log
└── <SAMPLE>/
    ├── alignment/
    │   ├── <SAMPLE>.alignment.cyp2c19.bam
    │   └── <SAMPLE>.alignment.cyp2c19.bam.bai
    └── high_coverage/
        ├── <SAMPLE>.high_coverage.cyp2c19.bam
        └── <SAMPLE>.high_coverage.cyp2c19.bam.bai
```

Alignment categories depend on source availability. Index files are expected when indexing succeeds.

### Scope and current limitations

- This script performs BAM region extraction and attempts indexing. Variant calling, BCFtools processing, and downstream format conversion are outside its implementation.
- Reference assembly compatibility and target coordinates must be verified separately before analysis.
- Extraction success is assessed using the SAMtools exit code and a minimum output file size. The indexing exit code is not checked.
- Resume mode relies on the Excel status and does not revalidate existing BAM files or indexes.
- The summary sheet describes samples processed during the current invocation.
