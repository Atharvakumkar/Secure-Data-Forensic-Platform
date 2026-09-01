# Secure Data Sanitization & Forensic Recovery Platform

> **SIH Project Planning README --- Draft**
>
> A Linux-first unified platform for secure storage sanitization,
> selective data erasure, forensic file carving/recovery, verification,
> auditability, and reporting.

## 1. Project Agenda

The project solves two opposite digital-storage problems:

1.  **Securely destroy sensitive data** so unauthorized recovery becomes
    infeasible under the selected sanitization assurance level.
2.  **Recover deleted or damaged digital evidence** during legitimate
    forensic investigations without modifying the original evidence.

The platform unifies:

``` text
Storage / Forensic Image
        |
   +----+----+
   |         |
   v         v
Sanitize   Forensics
   |         |
Drive/File  Carving/Recovery
   |         |
   +----+----+
        |
        v
Verification Engine
        |
        v
Audit + Evidence Log
        |
        v
Reports / Web Dashboard
```

## 2. Problem in One Paragraph

Organizations need to dispose of storage containing confidential
information, while forensic teams need to recover deleted evidence from
formatted, corrupted, or damaged media. Existing tools generally
specialize in one side: sanitization or forensics. This project combines
secure drive erasure, secure file/folder erasure, advanced file carving
and recovery, verification, audit trails, and forensic reporting in one
Linux-first environment.

## 3. Key Use Cases

### Corporate Hardware Disposal

A company replaces hundreds of laptops containing customer records,
source code, financial documents, and internal data.

**Workflow:**

``` text
Device -> Identify Media -> Select Sanitization Path
       -> Execute -> Validate -> Generate Certificate
```

### Forensic Investigation

Investigators receive a disk image where files were deleted or a
partition was formatted.

``` text
Evidence Registration -> Hash -> Filesystem Analysis
-> Raw Scan -> Signature Detection -> Carving
-> Reconstruction -> Validation -> Report
```

### Incident Response

An organization suspects that sensitive files were deleted before an
employee left. Investigators analyze an acquired image, recover relevant
artifacts, and preserve hashes and analysis metadata.

### Enterprise Decommissioning

Future versions can support batch jobs, asset IDs, centralized reports,
JSON/CSV exports, and integration with asset-management systems.

## 4. Current Situation

The ecosystem is fragmented.

**Sanitization:** nwipe, DBAN-derived workflows, Linux utilities, and
vendor-specific storage utilities.

**Forensics:** Autopsy, The Sleuth Kit, PhotoRec, TestDisk, and
commercial forensic suites.

The opportunity is not to replace every mature tool. It is to provide:

``` text
Sanitization + Forensic Recovery + Verification
+ Auditability + Unified UX + Machine-readable Reports
```

in one platform.

## 5. Traditional Tools and Their Gaps

  ------------------------------------------------------------------------------
  Tool / Method     Primary purpose        Strength          Opportunity for us
  ----------------- ---------------------- ----------------- -------------------
  DBAN              Drive wiping           Well-known        Limited
                                           historical        modern-media and
                                           workflow          unified-forensics
                                                             scope

  nwipe             Drive sanitization     Linux, open       Primarily
                                           source,           sanitization
                                           CLI/ncurses,      
                                           reporting         

  `shred`           File/block overwrite   Simple Linux      Filesystem/media
                                           utility           behavior limits
                                                             guarantees

  BleachBit         Cleanup                User-friendly     Not a forensic
                                                             recovery platform

  TestDisk          Partition/filesystem   Strong recovery   Not a sanitization
                    recovery               capabilities      platform

  PhotoRec          File carving           Useful on damaged Often loses
                                           filesystems       original
                                                             names/structure

  Autopsy           Digital forensics      Case management   Not a secure
                                           and analysis      sanitization
                                                             platform

  The Sleuth Kit    Forensic framework     Strong            Requires assembly
                                           open-source       into a larger
                                           primitives        workflow

  Commercial        Full investigations    Mature and broad  Expensive and not
  forensic suites                                            focused on secure
                                                             disposal

  Vendor            Media-specific         Device            Fragmented across
  secure-erase      sanitization           capabilities      vendors
  tools                                                      
  ------------------------------------------------------------------------------

**Positioning:** integrate appropriate techniques, validation, evidence
tracking, API access, and structured reporting rather than claiming to
replace every specialist tool.

## 6. Media-Aware Sanitization

A core principle is:

> **One overwrite technique should not be assumed to work identically
> for every storage technology.**

HDDs and flash-based SSD/NVMe devices behave differently because of
mechanisms such as wear leveling, flash translation layers, garbage
collection, and over-provisioning.

The system should identify the storage context and select an appropriate
sanitization path. Current NIST SP 800-88 Rev. 2 emphasizes sanitization
programs, validation, current standards, and appropriate techniques for
the media.

## 7. Proposed Architecture

``` text
                    Web Dashboard
                         |
                    REST API/Auth
                         |
                    FastAPI Layer
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
 Sanitization       Forensic          Reporting
   Engine            Engine             Engine
        |                |
        |          +-----+------+
        |          | Filesystem |
        |          |  Analysis  |
        |          +-----+------+
        |                |
        |          +-----v------+
        |          |  Carving   |
        |          |   Engine   |
        |          +-----+------+
        |                |
        |          +-----v------+
        |          | ML Classifier|
        |          | + Confidence |
        |          +-------------+
        |                |
        +--------+-------+
                 |
             Audit Store
```

## 8. Core Modules

### Module 1 --- Device & Evidence Manager

-   identify device/image
-   collect non-destructive metadata
-   calculate cryptographic hashes
-   assign case/evidence IDs
-   track acquisition information
-   prevent analysis of the wrong target

Example metadata:

``` text
Evidence ID
Device identifier
Media type
Capacity
Filesystem
Acquisition timestamp
SHA-256
Operator
Case ID
```

### Module 2 --- Secure Drive Eraser

-   identify media type
-   select an appropriate sanitization path
-   execute the selected operation
-   monitor progress
-   validate the result
-   record method and outcome
-   generate an audit record

Destructive operations should require explicit confirmation and be
developed primarily against controlled images or dedicated test media.

### Module 3 --- Secure File & Folder Eraser

-   select files/folders
-   perform supported deletion/sanitization workflow
-   handle filesystem-visible traces where technically supported
-   verify expected results
-   record the operation

**Important:** file-level secure deletion cannot be universally
guaranteed across all modern storage technologies. The platform should
label operations as verified, best-effort, media-specific, or
unsupported.

### Module 4 --- Advanced File Carving

``` text
Raw Storage
   -> Sector/Block Scanner
   -> Signature Detection
   -> Structure Validation
   -> Fragment Detection
   -> Reconstruction
   -> Recovered Artifact
```

Initial file types:

-   JPEG
-   PNG
-   PDF
-   DOCX
-   ZIP
-   MP4

### Module 5 --- Fragment Reconstruction

The engine can score candidate fragment orders using file-structure
constraints.

``` text
A -> B -> C -> D    94%
A -> C -> B -> D    22%
A -> D -> C -> B     4%
```

### Module 6 --- AI / ML

Initial ML scope:

-   recovered-file classification
-   recovery confidence scoring

Example:

``` text
Header validity       ✓
Structure validity    ✓
Footer validity       ✓
Fragment continuity   ✓
Metadata consistency  ✓

Recovery Confidence: 92%
```

Future extensions can include intelligent fragment ordering and anomaly
detection.

## 9. Verification Engine

### Recovery verification

``` text
Recovered artifact
       -> Validate structure
       -> Calculate SHA-256
       -> Store metadata
```

### Sanitization verification

``` text
Sanitization completed
       -> Validation checks
       -> Controlled recovery attempt
       -> Verification result
```

The report must state the scope and limitations of the verification
rather than making unsupported universal guarantees.

## 10. Audit & Chain of Custody

Every important action should record:

``` text
Timestamp
Operator
Case ID
Evidence ID
Action
Input hash
Output hash
Tool/version
Result
Confidence / validation
```

The system should answer:

> Who did what, when, to which evidence, using which software version,
> and what was the result?

## 11. Reporting

Export:

-   PDF
-   JSON
-   CSV

### Forensic report

Include case information, evidence information, acquisition details,
hashes, analysis actions, recovered artifacts, artifact hashes,
confidence scores, validation results, timestamps, software version, and
operator.

### Sanitization certificate

Include device/evidence identifier, media type, sanitization path,
timestamps, validation outcome, device metadata, software version,
operator, and report hash.

## 12. Recommended Technology Stack

### OS

**Linux**, preferably Ubuntu LTS.

### Backend

-   Python 3.11+
-   FastAPI
-   Pydantic
-   SQLAlchemy
-   Alembic
-   Typer
-   Celery/RQ or background workers

### Forensic/storage layer

-   The Sleuth Kit
-   pytsk3
-   libmagic / python-magic
-   libewf/EWF tooling where appropriate
-   smartmontools
-   hdparm
-   nvme-cli
-   blkdiscard

Low-level utilities should be wrapped behind a controlled abstraction
rather than exposed directly through the web UI.

### Recovery

-   Python
-   custom signature definitions
-   The Sleuth Kit
-   custom carving engine
-   file-format validators
-   hashing libraries

### ML

Start with:

-   scikit-learn
-   NumPy
-   pandas

Use PyTorch/ONNX later only if measurable benefits justify it.

### Database

**PostgreSQL**

Suggested entities:

``` text
cases
evidence
devices
operations
sanitization_jobs
recovery_jobs
recovered_files
hashes
audit_events
reports
users
```

### Dashboard

-   React
-   TypeScript
-   Vite or Next.js
-   Tailwind CSS
-   charting library

### DevOps

-   Docker
-   Docker Compose
-   GitHub Actions
-   pytest
-   Ruff
-   mypy
-   pre-commit
-   OpenAPI via FastAPI

## 13. Suggested Repository

``` text
secure-forensic-platform/
├── backend/
│   ├── api/
│   ├── core/
│   ├── models/
│   ├── services/
│   └── workers/
├── cli/
├── sanitization/
│   ├── device_detection/
│   ├── strategies/
│   └── verification/
├── forensics/
│   ├── acquisition/
│   ├── filesystem/
│   ├── carving/
│   ├── reconstruction/
│   └── validation/
├── ml/
│   ├── classification/
│   ├── confidence/
│   └── datasets/
├── reporting/
├── dashboard/
├── tests/
├── docs/
├── docker/
└── README.md
```

## 14. MVP Scope

### Storage

-   HDD
-   USB/SD
-   disk images

### Filesystems

-   NTFS
-   FAT32/exFAT
-   ext4

### Recovery

-   JPEG
-   PNG
-   PDF
-   DOCX
-   ZIP
-   MP4

### Sanitization

-   drive-level workflow
-   controlled test-media validation
-   media-aware decision logic
-   audit reporting

### Interface

-   CLI backend
-   REST API
-   web dashboard

### Reports

-   JSON
-   CSV
-   PDF

### ML

-   file classification
-   recovery confidence score

## 15. Strong SIH Demonstration

### Stage 1 --- Prepare controlled evidence

Create a test image containing:

``` text
photo.jpg
report.pdf
document.docx
archive.zip
video.mp4
```

Delete selected files.

### Stage 2 --- Recover

``` text
Register evidence
 -> Hash
 -> Scan
 -> Carve
 -> Validate
 -> Recover
```

Dashboard example:

``` text
Deleted candidates:     12
Carved objects:           9
Validated files:          7
High-confidence files:   6
```

### Stage 3 --- Report

Show evidence ID, SHA-256, recovered artifacts, artifact hashes,
confidence, timeline, tool version, and operator.

### Stage 4 --- Sanitize

Use a dedicated test device/image and execute the sanitization workflow.

### Stage 5 --- Verify

Run the appropriate controlled verification/recovery checks.

``` text
Sanitization: COMPLETE
Validation:   PASS
Targeted recovery after sanitization: NONE
```

The result should explicitly state the validated test conditions.

## 16. How We Stay Ahead of Competition

### 1. Evidence-first architecture

Every operation has a case, evidence ID, hash, operator, timestamp, tool
version, action, and result.

### 2. Explainable recovery confidence

Instead of "AI recovered it":

``` text
Confidence: 93%

Why?
+ Valid header
+ Valid structure
+ Valid footer
+ Fragment continuity
+ Metadata consistency
```

### 3. Media-aware sanitization

Use storage context to select an appropriate sanitization strategy
instead of blindly overwriting everything.

### 4. Machine-readable certificates

Provide JSON in addition to human-readable PDF reports.

### 5. Recovery-vs-sanitization proof loop

``` text
Deleted data
    -> Recovered
    -> Sanitized
    -> Verification
    -> No targeted recovery
```

### 6. Plugin architecture

``` text
Carving:
JPEG | PNG | PDF | DOCX | MP4 | ...

Sanitization:
HDD | SSD/NVMe | USB | Future media
```

This makes the platform extensible.

## 17. Testing Strategy

### Unit tests

Test parsers, signature detection, hashing, confidence scoring, report
generation, and metadata extraction.

### Integration tests

``` text
Image
 -> acquisition
 -> analysis
 -> carving
 -> validation
 -> reporting
```

### Recovery benchmark

Create a known dataset such as:

``` text
100 deleted files
20 JPEG
20 PNG
20 PDF
15 DOCX
15 ZIP
10 MP4
```

Measure:

-   detection rate
-   recovery rate
-   validation rate
-   false positives
-   processing time
-   confidence calibration

### Sanitization benchmark

Use dedicated non-production test media and measure completion,
validation, time, failure handling, and audit completeness.

## 18. Security Requirements

The platform itself is security-sensitive.

Implement:

-   role-based access control
-   authentication
-   explicit destructive-operation confirmation
-   audit logging
-   append-only audit design where practical
-   input validation
-   least privilege
-   isolated workers
-   secure report storage
-   API authentication
-   rate limiting
-   secrets management
-   no arbitrary command execution

Critical architecture:

``` text
Dashboard
   -> Validated job
   -> Authorization
   -> Safety checks
   -> Privileged worker
   -> Controlled operation
```

## 19. Limitations

Do **not** claim:

-   recovery of every deleted file
-   perfect fragmented-file reconstruction
-   universal filesystem support
-   universal secure file deletion
-   guaranteed SSD file-level sanitization through simple overwriting
-   legal admissibility in every jurisdiction
-   replacement of mature commercial forensic suites

Do claim:

-   validated support for a defined scope
-   reproducible forensic workflows
-   explainable recovery scoring
-   auditable operations
-   media-aware sanitization strategy
-   extensible architecture

## 20. Roadmap

### Phase 1 --- SIH MVP

``` text
Linux + Disk images + HDD/USB/SD
+ 3–6 file formats
+ Basic carving
+ ML classification
+ Sanitization workflow
+ Verification
+ Dashboard
+ Reports
```

### Phase 2 --- Advanced Forensics

-   fragmented-file reconstruction
-   filesystem-aware carving
-   timeline analysis
-   deleted-directory recovery
-   artifact correlation
-   larger file-format library

### Phase 3 --- Enterprise Sanitization

-   fleet management
-   batch sanitization
-   asset inventory integration
-   centralized audit logs
-   API integrations
-   certificate management
-   policy engine

### Phase 4 --- Advanced Intelligence

-   adaptive recovery prioritization
-   ML-assisted reconstruction
-   anomaly detection
-   automated artifact classification
-   recovery-quality prediction

### Phase 5 --- Research/Productization

-   benchmark datasets
-   reproducible evaluation suite
-   peer-reviewed methodology
-   hardware compatibility matrix
-   enterprise deployment architecture

## 21. Learning Roadmap

### Storage fundamentals

Learn:

1.  HDD vs SSD/NVMe
2.  sectors and blocks
3.  partitions
4.  filesystems
5.  metadata
6.  deleted-file behavior
7.  flash translation layers
8.  wear leveling

### Digital forensics

Learn:

1.  forensic imaging
2.  hashing
3.  chain of custody
4.  filesystem analysis
5.  deleted-file recovery
6.  file carving
7.  evidence preservation
8.  forensic reporting

### Sanitization

Study:

1.  NIST SP 800-88 Rev. 2
2.  ISO/IEC 27040:2024
3.  IEEE 2883 concepts
4.  cryptographic erase
5.  clear/purge/destroy concepts
6.  media-specific sanitization
7.  sanitization validation

### File carving

Learn:

1.  magic numbers/signatures
2.  headers and footers
3.  file structures
4.  sector scanning
5.  contiguous carving
6.  fragmented carving
7.  file validation

### ML

Learn:

1.  feature engineering
2.  classification
3.  confidence calibration
4.  anomaly detection
5.  evaluation metrics

## 22. Official Resources

-   NIST SP 800-88 Rev. 2: https://csrc.nist.gov/pubs/sp/800/88/r2/final
-   ISO/IEC 27040:2024: https://www.iso.org/standard/80194.html
-   The Sleuth Kit: https://www.sleuthkit.org/
-   Autopsy: https://www.sleuthkit.org/autopsy/
-   Autopsy Documentation: https://www.sleuthkit.org/autopsy/docs.php
-   TestDisk / PhotoRec: https://www.cgsecurity.org/
-   PhotoRec Documentation:
    https://www.cgsecurity.org/testdisk_doc/photorec.html
-   nwipe: https://github.com/martijnvanbrummelen/nwipe

## 23. Project Success Criteria

### Recovery

-   register a controlled disk image
-   preserve and verify its hash
-   identify deleted/carvable artifacts
-   recover selected file types
-   validate recovered files
-   produce confidence scores
-   generate a forensic report

### Sanitization

-   identify supported media
-   select an appropriate sanitization strategy
-   operate on controlled test media
-   perform validation
-   generate an auditable report

### Platform

-   CLI + REST API
-   usable web dashboard
-   audit logs
-   JSON/CSV/PDF exports
-   automated tests
-   reproducible benchmark results

## 24. Final Vision

This is not simply:

> **"A disk wiper + file recovery tool."**

It is:

> **An evidence-aware digital storage security platform that manages the
> storage lifecycle from forensic acquisition and recovery to validated
> sanitization and auditable disposal.**

``` text
             DIGITAL STORAGE LIFECYCLE

      Storage / Evidence
              |
       +------+------+
       |             |
       v             v
   FORENSICS     SANITIZATION
       |             |
       v             v
    Analyze        Erase
       |             |
       v             v
    Recover       Validate
       |             |
       +------+------+
              |
              v
       Evidence / Audit
              |
              v
           Reporting
              |
              v
     Enterprise Integration
```

### SIH-winning philosophy

**Don't compete by claiming the most features.**

Compete through:

1.  Technical correctness
2.  Evidence integrity
3.  Media-aware sanitization
4.  Explainable recovery
5.  Reproducible benchmarks
6.  Strong verification
7.  Excellent UX
8.  A complete end-to-end story

> **Recover it. Verify it. Sanitize it. Prove what happened.**

## Disclaimer

Develop and demonstrate only on storage devices, disk images, and
datasets for which the team has explicit authorization. Destructive
sanitization must be isolated from production systems and real evidence
unless performed under an authorized forensic or organizational
procedure.
