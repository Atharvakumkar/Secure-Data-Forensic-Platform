# Secure Data Forensic Platform --- Technology Stack & Software Specification

> Linux-first platform for secure data sanitization, forensic file
> recovery, intelligent file carving, verification, and audit-ready
> reporting.

## 1. Project Purpose

The platform combines two major capabilities:

1.  **Advanced File Recovery** --- analyze authorized forensic disk
    images/media, recover deleted files using filesystem metadata and
    raw carving, validate recovered artifacts, reconstruct fragmented
    files, and assign explainable confidence scores.
2.  **Secure Data Sanitization** --- identify storage media, select
    media-appropriate sanitization methods, execute controlled
    sanitization, verify the result, and produce an auditable
    certificate.

Core positioning:

**Recover it. Verify it. Sanitize it. Prove what happened.**

The recovery side should operate on forensic copies/read-only evidence
wherever possible. Destructive sanitization operations must be isolated
behind authorization, device identification, safety checks, predefined
strategies, and verification. Do not claim universal permanent deletion,
especially for SSD/NVMe.

------------------------------------------------------------------------

## 2. High-Level Architecture

``` text
                         USER
                          |
              +-----------+-----------+
              |                       |
        Web Dashboard               CLI
              |                       |
              +-----------+-----------+
                          |
                       FastAPI
                          |
                  +-------+-------+
                  | Core Backend  |
                  +-------+-------+
                          |
        +-----------------+------------------+
        |                 |                  |
        v                 v                  v
 Evidence Manager   Recovery Engine   Sanitization Engine
        |                 |                  |
        |          +------+------+           |
        |          |      |      |           |
        |          v      v      v           v
        |         TSK   Carver  ML      Media-Aware
        |                |      |        Sanitization
        |                v      v             |
        |            Validation <--------------+
        |                |
        |          Fragment Recovery
        |                |
        +----------------+----------------+
                         |
                         v
                 Verification Engine
                         |
                         v
                 Report / Audit Engine
                         |
                         v
                    SQLite DB
```

------------------------------------------------------------------------

## 3. Technology Stack

  -----------------------------------------------------------------------------
  Layer                   Recommended Technology  Purpose
  ----------------------- ----------------------- -----------------------------
  OS                      Ubuntu Linux LTS        Development/deployment

  Primary language        Python 3.11+            Core application

  Forensic engine         The Sleuth Kit (TSK)    Filesystem/metadata analysis

  Recovery                Custom Python engine    Carving and orchestration

  Validation              Python + format parsers File integrity/structure

  ML                      scikit-learn            Classification/confidence

  Numerical processing    NumPy                   Feature processing

  Data analysis           pandas                  Dataset/benchmark analysis

  Backend                 FastAPI                 REST API

  ASGI server             Uvicorn                 API serving

  Database                SQLite                  MVP metadata/audit storage

  Future database         PostgreSQL              Multi-user production

  CLI                     Typer                   Command-line interface

  Frontend                React + TypeScript +    Web dashboard
                          Vite                    

  Testing                 pytest                  Unit/integration/regression
                                                  tests

  Containers              Docker + Compose        Reproducible environments

  CI/CD                   Jenkins Pipeline        Automated build/test/deploy

  Version control         Git + GitHub            Source control

  Hashing                 Python hashlib /        Evidence/artifact integrity
                          SHA-256                 

  Reports                 JSON + HTML + PDF       Machine-readable and
                                                  human-readable output

  Sanitization reference  NIST SP 800-88 Rev. 2   Sanitization guidance

  Other references        ISO/IEC 27040:2024,     Storage/security guidance
                          IEEE 2883 concepts      
  -----------------------------------------------------------------------------

------------------------------------------------------------------------

# 4. Repository Structure

``` text
secure-data-forensic-platform/
│
├── README.md
├── LICENSE
├── pyproject.toml
├── requirements.txt
├── Jenkinsfile
├── Dockerfile
├── docker-compose.yml
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   │
│   │   ├── api/
│   │   │   ├── cases.py
│   │   │   ├── evidence.py
│   │   │   ├── recovery.py
│   │   │   ├── sanitization.py
│   │   │   └── reports.py
│   │   │
│   │   ├── evidence/
│   │   │   ├── manager.py
│   │   │   ├── hashing.py
│   │   │   └── image_reader.py
│   │   │
│   │   ├── recovery/
│   │   │   ├── tsk/
│   │   │   │   ├── runner.py
│   │   │   │   ├── filesystem.py
│   │   │   │   └── metadata.py
│   │   │   ├── signatures.py
│   │   │   ├── scanner.py
│   │   │   ├── carver.py
│   │   │   ├── validator.py
│   │   │   ├── fragments.py
│   │   │   └── confidence.py
│   │   │
│   │   ├── sanitization/
│   │   │   ├── manager.py
│   │   │   ├── device.py
│   │   │   ├── strategies.py
│   │   │   └── verification.py
│   │   │
│   │   ├── reporting/
│   │   │   ├── recovery_report.py
│   │   │   └── sanitization_certificate.py
│   │   │
│   │   └── models/
│   │       ├── case.py
│   │       ├── evidence.py
│   │       ├── recovered_file.py
│   │       └── sanitization_job.py
│   │
│   └── tests/
│
├── cli/
│   └── main.py
│
├── ml/
│   ├── features.py
│   ├── classifier.py
│   ├── confidence.py
│   └── models/
│
├── frontend/
│   ├── src/
│   └── public/
│
├── test-data/
│   ├── files/
│   ├── images/
│   ├── fragmented/
│   └── expected/
│
├── scripts/
│   ├── create_test_image.sh
│   └── generate_test_data.py
│
└── docs/
    ├── architecture/
    ├── recovery/
    ├── sanitization/
    ├── filesystem/
    └── research/
```

------------------------------------------------------------------------

# 5. Evidence Manager

The Evidence Manager is the foundation of forensic analysis.

### Responsibilities

-   Register evidence.
-   Assign a unique evidence ID.
-   Record source path and size.
-   Calculate SHA-256.
-   Identify image/partition/filesystem information.
-   Record timestamps.
-   Preserve read-only analysis state.
-   Record operator/actions.
-   Maintain chain-of-custody information.

Example:

``` text
Evidence ID: EVD-00001
Source: test-disk.img
Size: 500 MB
SHA-256: <hash>
Filesystem: NTFS
Status: READ ONLY
```

------------------------------------------------------------------------

# 6. Disk Image Support

### MVP

-   RAW `.dd`
-   `.img`

### Future

-   E01/EWF
-   AFF
-   Additional forensic image formats

Development should use controlled images rather than real evidence.

------------------------------------------------------------------------

# 7. Sleuth Kit Integration

The Sleuth Kit should provide the filesystem-forensics foundation.

Official documentation:

https://www.sleuthkit.org/sleuthkit/docs.php

Useful tools:

  TSK tool        Role
  --------------- -----------------------------------
  `img_stat`      Image information
  `mmls`          Partition layout
  `fsstat`        Filesystem information
  `fls`           Files/directories/deleted entries
  `istat`         Metadata/inode/MFT information
  `icat`          Extract file contents
  `ifind`         Map data to metadata
  `blkls`         Extract filesystem data units
  `tsk_recover`   Recover/export files
  `sigfind`       Signature searching

### Integration strategy

Start by invoking TSK command-line tools from Python through a
controlled wrapper. Consider direct C/C++ library integration later if
performance or deeper control requires it.

Conceptual wrapper:

``` python
class TSKRunner:
    def image_info(self, image_path):
        ...

    def partition_table(self, image_path):
        ...

    def filesystem_info(self, image_path, offset):
        ...

    def list_files(self, image_path, offset):
        ...

    def list_deleted_files(self, image_path, offset):
        ...

    def metadata(self, image_path, offset, inode):
        ...

    def extract_file(self, image_path, offset, inode):
        ...
```

Resources:

-   https://www.sleuthkit.org/sleuthkit/docs.php
-   https://sleuthkit.org/sleuthkit/docs/api-docs/4.15.0-develop/
-   https://github.com/sleuthkit/sleuthkit
-   https://wiki.sleuthkit.org/

------------------------------------------------------------------------

# 8. Recovery Engine

The Recovery Engine should have two complementary paths.

## A. Filesystem-Based Recovery

``` text
Disk Image
    |
Filesystem
    |
MFT / inode / directory entries
    |
Deleted entry
    |
Data blocks
    |
Recovered File
```

This path can preserve useful metadata such as original filename/path,
timestamps, and filesystem references.

TSK should handle much of this layer.

## B. Raw File Carving

When metadata is missing/unusable:

``` text
Raw Disk Bytes
      |
Signature Scanner
      |
Candidate
      |
Carver
      |
Validator
      |
Recovered File
```

This path is primarily our own implementation.

------------------------------------------------------------------------

# 9. File Signature Scanner

Initial formats:

-   JPEG
-   PNG
-   PDF
-   ZIP/DOCX
-   MP4

Example signatures:

``` text
JPEG: FF D8 FF
PNG:  89 50 4E 47 0D 0A 1A 0A
PDF:  25 50 44 46
ZIP:  50 4B 03 04
```

A signature match creates a candidate; it does not prove that a valid
file exists.

Candidate example:

``` json
{
  "offset": 2389472,
  "type": "JPEG",
  "signature": "FF D8 FF",
  "source": "raw_scan"
}
```

------------------------------------------------------------------------

# 10. Chunked Evidence Reading

Never load a multi-gigabyte image completely into memory.

Use:

``` text
Disk Image
   |
Read chunk
   |
Search signatures
   |
Record offsets
   |
Next chunk
```

The scanner must handle signatures that cross chunk boundaries.

------------------------------------------------------------------------

# 11. Basic Carving

First implement contiguous-file recovery.

``` text
Signature
   |
Read candidate bytes
   |
Find format end condition
   |
Extract range
   |
Save candidate
```

Each result should record:

-   Evidence ID
-   Source offset
-   Start/end offsets
-   File type
-   Recovered size
-   Hash
-   Validation status
-   Recovery method

------------------------------------------------------------------------

# 12. File Validation

Validation should occur before final recovery classification.

Levels:

1.  Signature validation.
2.  End-marker validation.
3.  Internal structure validation.
4.  Parser validation.

Pipeline:

``` text
Candidate
   |
Header
   |
Structure
   |
Footer/end
   |
Trusted parser
   |
PASS / PARTIAL / FAIL
```

A valid signature alone should never be treated as proof of
recoverability.

------------------------------------------------------------------------

# 13. Fragment Reconstruction

Fragmented files are a major advanced feature.

Example:

``` text
Fragment 1: blocks 100-101
Fragment 2: blocks 500-501
Fragment 3: blocks 900-901
```

Possible reconstruction signals:

-   Filesystem allocation information.
-   Block continuity.
-   File-format structure.
-   Expected file size.
-   Internal offsets.
-   Compression structure.
-   Entropy.
-   Fragment adjacency.
-   Header/footer relationships.
-   Known format constraints.

Pipeline:

``` text
Candidate
   |
Possible fragments
   |
Feature extraction
   |
Fragment ranking
   |
Reassembly
   |
Validation
   |
Recovered file
```

------------------------------------------------------------------------

# 14. ML Component

ML should be added after deterministic recovery works.

### ML responsibilities

-   File-type classification when signatures are ambiguous.
-   Candidate ranking.
-   Fragment compatibility scoring.
-   Recovery confidence estimation.

Example:

``` text
Recovered Candidate
        |
Feature Extraction
        |
+--------------------------+
| Header valid             |
| Footer valid             |
| Structure valid          |
| Size plausible           |
| Fragment continuity      |
| Entropy                  |
| Filesystem evidence      |
+--------------------------+
        |
        v
ML Model
        |
        v
Confidence Score
```

Recommended first stack:

-   scikit-learn
-   NumPy
-   pandas
-   joblib

Start with interpretable models such as Random Forest, Gradient Boosting
or Logistic Regression.

Avoid making deep learning a core dependency of the MVP.

------------------------------------------------------------------------

# 15. Recovery Confidence

Use an explainable score rather than a black-box claim.

Example:

``` text
Confidence: 94%

Reasons:
+ Valid header
+ Valid internal structure
+ Valid footer
+ Parser successful
+ Contiguous data
```

Possible bands:

``` text
0.00 - 0.39  LOW
0.40 - 0.69  MEDIUM
0.70 - 0.89  HIGH
0.90 - 1.00  VERY HIGH
```

Confidence is an estimate, not proof.

------------------------------------------------------------------------

# 16. Recovery Result Model

Every recovered artifact should have structured metadata.

``` json
{
  "recovery_id": "REC-00042",
  "evidence_id": "EVD-00001",
  "file_type": "JPEG",
  "start_offset": 2389472,
  "end_offset": 2401024,
  "size": 11552,
  "fragments": 1,
  "method": "carving",
  "validation": "PASS",
  "confidence": 0.987,
  "sha256": "<hash>"
}
```

------------------------------------------------------------------------

# 17. Sanitization Engine

The second major subsystem:

``` text
Device Identification
       |
Media Classification
       |
Method Selection
       |
Controlled Execution
       |
Verification
       |
Certificate
```

## HDD

Use an appropriate device/media sanitization method and verify the
result.

## SSD/NVMe

Do not assume generic overwriting is equivalent to complete physical
sanitization. Account for:

-   Flash Translation Layer (FTL)
-   Wear leveling
-   Garbage collection
-   Over-provisioning

Use appropriate device-supported sanitization mechanisms where
applicable, such as device sanitize/secure erase or cryptographic erase.

------------------------------------------------------------------------

# 18. Sanitization Status

Use explicit statuses:

``` text
SUPPORTED
BEST_EFFORT
VERIFIED
PARTIALLY_VERIFIED
UNSUPPORTED
FAILED
```

Example:

``` text
Media: NVMe SSD
Method: Device Sanitize
Verification: PASS
Status: VERIFIED
```

------------------------------------------------------------------------

# 19. NIST Alignment

Primary sanitization reference:

**NIST SP 800-88 Rev. 2**

Official publication:

https://csrc.nist.gov/pubs/sp/800/88/r2/final

The final Rev. 2 publication was released in September 2025 and
supersedes Rev. 1.

Use it to guide:

-   Media identification.
-   Sanitization method selection.
-   Verification.
-   Documentation.
-   Risk-based decisions.
-   Cryptographic erase where appropriate.
-   Media-specific controls.

Do not describe the application itself as universally "NIST certified."

------------------------------------------------------------------------

# 20. Forensic Integrity and Audit Trail

Record:

``` text
Case ID
Evidence ID
Operator
Start time
End time
Evidence SHA-256
Tool/version information
Filesystem
Recovery method
Recovered artifacts
Artifact hashes
Actions performed
Validation results
```

Every significant action should create an audit event.

------------------------------------------------------------------------

# 21. Reporting

## Recovery Report

Include:

-   Case information.
-   Evidence information.
-   Evidence hash.
-   Filesystem.
-   Deleted entries.
-   Carved candidates.
-   Recovered artifacts.
-   Offsets.
-   File types.
-   Fragment information.
-   Validation results.
-   Confidence scores.
-   Artifact hashes.
-   Timeline.
-   Tool versions.

## Sanitization Certificate

Include:

-   Media/device information.
-   Sanitization method.
-   Method parameters.
-   Start/end times.
-   Verification method.
-   Verification result.
-   Operator.
-   Evidence/audit hashes.
-   Final status.

Preferred formats:

-   JSON --- machine-readable source of truth.
-   HTML --- human-readable.
-   PDF --- formal report/certificate.

------------------------------------------------------------------------

# 22. Database

### MVP: SQLite

Suggested tables:

``` text
cases
evidence
recovery_jobs
recovered_files
file_fragments
validation_results
audit_events
sanitization_jobs
reports
```

### Future

PostgreSQL for multi-user/distributed deployments.

------------------------------------------------------------------------

# 23. REST API

Use FastAPI.

Official documentation:

https://fastapi.tiangolo.com/

Suggested endpoints:

``` text
POST /cases
GET  /cases/{case_id}

POST /evidence
GET  /evidence/{evidence_id}

POST /recovery/analyze
POST /recovery/scan
POST /recovery/carve
GET  /recovery/jobs/{job_id}
GET  /recovery/results/{job_id}

POST /reports/recovery
GET  /reports/{report_id}

POST /sanitization/plan
POST /sanitization/execute
GET  /sanitization/jobs/{job_id}
POST /sanitization/verify
POST /reports/sanitization
```

The API must not expose arbitrary destructive shell commands.

------------------------------------------------------------------------

# 24. CLI

The complete recovery workflow should work without the web UI.

Example:

``` bash
sdfp case create

sdfp evidence add test-disk.img
sdfp evidence info EVD-00001

sdfp recovery analyze EVD-00001
sdfp recovery scan EVD-00001
sdfp recovery recover EVD-00001
sdfp recovery validate EVD-00001

sdfp report recovery EVD-00001
```

Sanitization should use additional safety gates:

``` bash
sdfp sanitize identify-device
sdfp sanitize plan
sdfp sanitize execute
sdfp sanitize verify
sdfp report sanitization
```

------------------------------------------------------------------------

# 25. Frontend

Recommended:

-   React
-   TypeScript
-   Vite

Dashboard sections:

``` text
Dashboard
Cases
Evidence
Recovery
Recovered Files
Sanitization
Reports
Audit Logs
```

Recovery result table:

``` text
Type | Size | Fragments | Validation | Confidence
JPG  | 4MB  | 1         | PASS       | 98.7%
PDF  | 2MB  | 3         | PASS       | 91.3%
PNG  | 845K | 1         | PASS       | 99.1%
MP4  | 31MB | 7         | PARTIAL    | 72.4%
```

------------------------------------------------------------------------

# 26. Security Architecture

Do not give the web application unrestricted raw-device access.

Recommended:

``` text
Web UI
  |
FastAPI
  |
Authorization
  |
Job Manager
  |
Privileged Worker
  |
Controlled Device Operation
```

The privileged worker should:

-   Verify device identity.
-   Reject system/root disks by default.
-   Require explicit authorization.
-   Require confirmation for destructive operations.
-   Use predefined strategies.
-   Log every operation.
-   Verify target media before execution.
-   Never execute arbitrary user-supplied shell commands.

Recovery analysis should preferably use read-only access.

------------------------------------------------------------------------

# 27. Docker

Use Docker for:

-   Backend environment.
-   Frontend environment.
-   Development.
-   Testing.
-   Reproducible builds.

Official documentation:

https://docs.docker.com/

Suggested development services:

``` text
Docker Compose
    |
    +--- frontend
    +--- backend
    +--- database
    +--- worker
```

Physical-device access should not be casually exposed to containers.
Device-level operations need an explicitly designed privileged execution
environment.

------------------------------------------------------------------------

# 28. Jenkins CI/CD

Jenkins is appropriate for CI/CD and automated regression testing, but
it should remain outside the runtime recovery path.

Official documentation:

https://www.jenkins.io/doc/book/pipeline/

Pipeline:

``` text
GitHub
   |
   v
Jenkins
   |
   +--- Checkout
   +--- Install dependencies
   +--- Lint
   +--- Unit tests
   +--- TSK tests
   +--- Recovery regression tests
   +--- Security checks
   +--- Docker build
   +--- Container tests
   +--- Deploy test environment
```

Recommended stages:

``` text
Checkout
Environment Setup
Lint
Unit Tests
Create Test Image
TSK Integration Tests
Carving Tests
Validation Tests
Fragment Tests
ML Tests
API Tests
Build
Container Test
Publish/Deploy
```

Store the pipeline as a `Jenkinsfile` in Git.

------------------------------------------------------------------------

# 29. Testing Strategy

Use pytest.

Official documentation:

https://docs.pytest.org/

### Unit tests

``` text
Signature detection
Offset calculation
Hashing
Validation
Confidence scoring
```

### Integration tests

``` text
TSK + disk image
Filesystem analysis
Deleted-file extraction
Carving
Recovery pipeline
```

### Regression tests

Maintain controlled images:

``` text
test-image-ntfs.img
test-image-fat32.img
test-image-exfat.img
test-image-ext4.img
```

Each should have known:

-   Files.
-   Deleted files.
-   Fragmented files.
-   Expected hashes.
-   Expected recovery outcomes.

------------------------------------------------------------------------

# 30. Automated Recovery Regression

A strong CI/CD feature:

``` text
Jenkins
   |
Create controlled test image
   |
Insert known files
   |
Delete selected files
   |
Run recovery engine
   |
Compare recovered artifacts
   |
Calculate metrics
   |
PASS / FAIL
```

Track:

-   Recovery precision.
-   Recovery recall.
-   False-positive rate.
-   Validation rate.
-   Fragment reconstruction success.
-   Processing throughput.
-   Memory usage.
-   Confidence calibration.

------------------------------------------------------------------------

# 31. Development Roadmap

## Phase 0 --- Environment

Install and learn:

``` text
Ubuntu/WSL
Python
Git
Sleuth Kit
pytest
Docker
Jenkins
```

Learn:

-   Linux filesystem basics.
-   Disk images.
-   Binary/hex data.
-   Python binary I/O.

## Phase 1 --- Evidence Manager

Implement:

``` text
Image registration
SHA-256
Metadata
Read-only handling
Case IDs
Audit events
```

## Phase 2 --- TSK Integration

Implement:

``` text
img_stat
mmls
fsstat
fls
istat
icat
```

## Phase 3 --- Signature Scanner

Implement:

``` text
JPEG
PNG
PDF
ZIP/DOCX
MP4
```

## Phase 4 --- Basic Carver

Implement contiguous recovery.

## Phase 5 --- Validation

Implement:

``` text
Header checks
Footer checks
Format structure
Parser validation
Hashing
```

## Phase 6 --- Fragment Reconstruction

Implement:

``` text
Fragment detection
Fragment ranking
Reassembly
Validation
```

## Phase 7 --- ML

Implement:

``` text
Feature extraction
Classifier
Confidence score
Candidate ranking
```

## Phase 8 --- Reporting

Implement:

``` text
JSON
HTML
PDF
Audit log
```

## Phase 9 --- FastAPI

Expose evidence, recovery, results and reports.

## Phase 10 --- Web Dashboard

Build case management, evidence, recovery, results, reports and audit
screens.

## Phase 11 --- Sanitization

Implement carefully:

``` text
Device detection
Media classification
Sanitization planning
Controlled execution
Verification
Certificate
```

## Phase 12 --- Docker + Jenkins

Automate:

``` text
Build
Test
Recovery regression
Security checks
Container build
Deployment
```

------------------------------------------------------------------------

# 32. SIH Demonstration

Use a controlled disk image.

### Step 1

Create:

``` text
test-disk.img
```

### Step 2

Add:

``` text
photo.jpg
report.pdf
document.docx
video.mp4
```

### Step 3

Delete selected files.

### Step 4

Analyze with the platform.

### Step 5

Recover through filesystem metadata.

### Step 6

Run raw carving.

### Step 7

Reconstruct selected fragmented files.

### Step 8

Show:

``` text
photo.jpg       98.7%
report.pdf      94.2%
document.docx   91.6%
```

### Step 9

Generate forensic report.

### Step 10

Run sanitization on dedicated test media.

### Step 11

Verify sanitization.

### Step 12

Run a controlled recovery attempt again and report the result under the
defined verification procedure.

### Step 13

Generate sanitization certificate and audit trail.

------------------------------------------------------------------------

# 33. SIH Differentiators

Do not position the project as merely another recovery tool.

Position it as:

> An integrated platform that can recover deleted data, validate what
> was recovered, quantify recovery confidence, securely sanitize media
> using media-aware methods, verify the result, and generate an
> auditable record.

Main differentiators:

1.  Filesystem recovery + raw carving.
2.  Advanced fragmented-file reconstruction.
3.  Deterministic validation before ML confidence.
4.  Explainable recovery confidence.
5.  Media-aware sanitization.
6.  Sanitization verification.
7.  Evidence hashing and audit trail.
8.  Machine-readable JSON reports.
9.  CLI + web dashboard.
10. Automated forensic regression testing.
11. Jenkins CI/CD.
12. Reproducible Docker development.
13. Clear separation of recovery, sanitization and verification.
14. Explicit `verified`, `best-effort`, `unsupported` and `failed`
    states.

------------------------------------------------------------------------

# 34. Traditional Tools and Our Position

  Capability                                TSK   PhotoRec   DBAN/nwipe   Our Platform
  ---------------------------- ---------------- ---------- ------------ --------------
  Filesystem analysis                       Yes    Limited           No            Yes
  Deleted metadata recovery                 Yes         No           No            Yes
  Raw carving                     Tool-assisted        Yes           No            Yes
  Fragment reconstruction               Limited    Limited           No       Advanced
  ML confidence                              No         No           No            Yes
  File validation                       Partial      Basic           No            Yes
  Drive sanitization                         No         No          Yes            Yes
  Media-aware workflow                  Limited         No      Limited            Yes
  Sanitization verification             Limited         No      Limited            Yes
  Integrated audit/reporting     Tool dependent      Basic      Limited            Yes
  Web dashboard                              No         No           No            Yes
  REST API                                   No         No           No            Yes
  CI/CD regression                           No         No           No            Yes

The goal is to integrate proven tools and add a differentiated workflow
rather than claim that existing tools are useless.

------------------------------------------------------------------------

# 35. Research Opportunities

Potential research directions:

-   Fragment reconstruction using file-structure constraints.
-   Explainable recovery confidence.
-   Classification from partial/corrupted data.
-   Cross-filesystem recovery benchmarking.
-   Automated recovered-file validation.
-   Media-aware sanitization verification.
-   Recovery-versus-sanitization effectiveness evaluation.
-   Forensic auditability of automated recovery systems.

------------------------------------------------------------------------

# 36. Important Limitations

### Deleted data

A deleted file is not necessarily recoverable.

### Overwritten data

Overwritten data may be unrecoverable.

### SSD/NVMe

Flash-storage behavior makes generic overwrite assumptions unreliable.

### Encryption

Encrypted data may not be meaningfully recoverable without appropriate
access/keys.

### Fragmentation

Highly fragmented files can be difficult or impossible to reconstruct.

### Corruption

A partially recovered file may not be usable.

### Validation

A matching signature does not prove that a valid file exists.

### Sanitization

Never claim universal permanent deletion without an appropriate
media-specific method and defined verification procedure.

------------------------------------------------------------------------

# 37. Learning Resources

## Sleuth Kit

Official docs:

https://www.sleuthkit.org/sleuthkit/docs.php

API docs:

https://sleuthkit.org/sleuthkit/docs/api-docs/4.15.0-develop/

GitHub:

https://github.com/sleuthkit/sleuthkit

Wiki:

https://wiki.sleuthkit.org/

## NIST

SP 800-88 Rev. 2:

https://csrc.nist.gov/pubs/sp/800/88/r2/final

## FastAPI

https://fastapi.tiangolo.com/

## Jenkins

https://www.jenkins.io/doc/

Pipeline:

https://www.jenkins.io/doc/book/pipeline/

## Docker

https://docs.docker.com/

## pytest

https://docs.pytest.org/

## scikit-learn

https://scikit-learn.org/stable/

## Book

**File System Forensic Analysis --- Brian Carrier**

Use it to understand disk structures, partitions, FAT, NTFS, Unix/Linux
filesystems, allocation, metadata and deleted-file recovery.

------------------------------------------------------------------------

# 38. YouTube Learning Searches

### File carving

https://www.youtube.com/results?search_query=file+carving+digital+forensics+tutorial

### Sleuth Kit

https://www.youtube.com/results?search_query=The+Sleuth+Kit+tutorial+digital+forensics

### Autopsy

https://www.youtube.com/results?search_query=Autopsy+digital+forensics+tutorial

### NTFS/MFT

https://www.youtube.com/results?search_query=NTFS+filesystem+internals+MFT+tutorial

### File signatures

https://www.youtube.com/results?search_query=file+signatures+magic+numbers+digital+forensics

### Disk imaging

https://www.youtube.com/results?search_query=disk+imaging+digital+forensics+dd+E01+tutorial

### Jenkins

https://www.youtube.com/results?search_query=Jenkins+Pipeline+tutorial

### FastAPI

https://www.youtube.com/results?search_query=FastAPI+tutorial+Python

### Docker

https://www.youtube.com/results?search_query=Docker+tutorial+for+beginners

------------------------------------------------------------------------

# 39. Recommended Final Stack

``` text
OS
Ubuntu Linux LTS

LANGUAGE
Python 3.11+

FORENSICS
The Sleuth Kit

FILESYSTEMS
NTFS
FAT32
exFAT
ext4

RECOVERY
Custom Python carving
Custom validation
Custom fragment reconstruction

ML
scikit-learn
NumPy
pandas

BACKEND
FastAPI
Uvicorn

DATABASE
SQLite -> PostgreSQL later

CLI
Typer

FRONTEND
React
TypeScript
Vite

CONTAINERIZATION
Docker
Docker Compose

TESTING
pytest

CI/CD
Jenkins Pipeline

VERSION CONTROL
Git
GitHub

REPORTING
JSON
HTML
PDF

HASHING
SHA-256

SANITIZATION REFERENCE
NIST SP 800-88 Rev. 2
ISO/IEC 27040:2024
IEEE 2883 concepts
```

------------------------------------------------------------------------

# 40. Exact Build Order

Do not build everything simultaneously.

``` text
1. Linux + Python
        |
2. Disk images + binary data
        |
3. Evidence Manager
        |
4. Sleuth Kit
        |
5. Filesystem analysis
        |
6. Deleted-file recovery
        |
7. Signature scanner
        |
8. Basic carving
        |
9. File validation
        |
10. Fragment reconstruction
        |
11. ML confidence
        |
12. Reporting
        |
13. FastAPI
        |
14. CLI refinement
        |
15. Web dashboard
        |
16. Sanitization engine
        |
17. Verification
        |
18. Docker
        |
19. Jenkins CI/CD
        |
20. SIH benchmarking/demo
```

------------------------------------------------------------------------

# 41. Recovery MVP Definition of Done

The Recovery MVP is complete when it can:

-   [ ] Accept a controlled RAW disk image.
-   [ ] Calculate/store SHA-256.
-   [ ] Identify partitions.
-   [ ] Identify NTFS/FAT32/exFAT/ext4.
-   [ ] Use TSK for filesystem metadata.
-   [ ] Find deleted entries where metadata remains.
-   [ ] Recover metadata-backed files.
-   [ ] Scan raw bytes for signatures.
-   [ ] Carve JPEG/PNG/PDF/DOCX/ZIP/MP4 candidates.
-   [ ] Validate recovered files.
-   [ ] Detect basic fragmentation.
-   [ ] Reconstruct selected fragmented test cases.
-   [ ] Generate confidence scores.
-   [ ] Hash recovered artifacts.
-   [ ] Produce JSON recovery reports.
-   [ ] Produce HTML/PDF reports.
-   [ ] Maintain an audit trail.
-   [ ] Pass automated regression tests.

------------------------------------------------------------------------

# 42. Full SIH Product Definition of Done

``` text
                 CASE
                   |
                EVIDENCE
                   |
                HASHING
                   |
          +--------+--------+
          |                 |
       RECOVERY        SANITIZATION
          |                 |
    TSK + CARVING       MEDIA-AWARE
          |                 |
     VALIDATION        VERIFICATION
          |                 |
   RECONSTRUCTION      CERTIFICATE
          |                 |
          +--------+--------+
                   |
                REPORT
                   |
              AUDIT TRAIL
                   |
             WEB + CLI + API
```

Success criteria:

-   Technical correctness.
-   Reproducibility.
-   Recovery accuracy.
-   Validation quality.
-   Fragment reconstruction.
-   Explainability.
-   Sanitization correctness.
-   Verification quality.
-   Evidence integrity.
-   Usability.
-   Security.
-   Demonstration quality.
-   Research potential.

------------------------------------------------------------------------

# 43. Final Architecture Principle

Use established forensic technology such as **The Sleuth Kit** for
filesystem analysis, while building the project's differentiation in:

-   recovery orchestration,
-   advanced carving,
-   file validation,
-   fragmented-file reconstruction,
-   confidence scoring,
-   sanitization verification,
-   auditability,
-   reporting,
-   and the integrated recovery-to-sanitization workflow.

The result should be a serious forensic platform rather than a wrapper
around a single existing utility.
