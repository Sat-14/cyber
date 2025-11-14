# Phase 1: Detection and Quarantine

## Overview

Phase 1 implements comprehensive malware detection using multiple open-source tools integrated into a unified scanning pipeline. The system combines YARA rule-based detection, PE file analysis, and optional VirusTotal cloud scanning to identify malicious files.

**Integration**: Phase 1 seamlessly integrates with [Phase 2 (Metadata Extraction)](../Phase2-Metadata) for comprehensive forensic analysis. See [Phase 1+2 Integration Guide](#integration-with-phase-2) below.

## Key Features

- **Multi-Engine Detection**: YARA rules + PE analysis + VirusTotal integration
- **1000+ YARA Rules**: Comprehensive malware signature database from Yara-Rules repository
- **Advanced PE Analysis**: Import analysis, entropy calculation, security feature detection
- **Suspicious Scoring**: 0-100 risk score based on multiple indicators
- **Threat Detection**:
  - Dangerous API imports (VirtualAlloc, WriteProcessMemory, etc.)
  - High entropy sections (packed/encrypted malware)
  - Missing security features (ASLR, DEP, SEH)
  - Known malware patterns (via YARA)
- **Audit Trail**: Tamper-proof cryptographic audit logs
- **Automated Quarantine**: Suspicious files automatically isolated

## Installation

### Prerequisites

- **Python 3.8+** (tested with Python 3.13)
- **Git** (for cloning YARA rules and ML models)
- **pip** (Python package manager)
- **Windows/Linux/macOS**

### Step 1: Install Python Dependencies

```bash
cd Phase1-Detection
pip install -r requirements.txt
```

This installs:
- `yara-python` - YARA pattern matching engine
- `pefile` - PE file parser and analyzer
- `requests` - HTTP client for VirusTotal API
- `audittrail` - Cryptographic audit logging
- `vt-py` - VirusTotal Python SDK

### Step 2: Download Open-Source Tools

Run the automated setup script to download YARA rules and ML detection models:

```bash
python scripts/setup_phase1_tools.py
```

This will:
1. Clone **Yara-Rules** repository (1000+ malware signatures)
2. Clone **Malware-Detection-ML-Model** (Random Forest detector)
3. Clone **deep-malware-detection** (PyTorch neural networks)
4. Clone **MDAML** (ML + VirusTotal hybrid)
5. Organize rules into `yara-rules/` directory
6. Download models into `downloads/` directory

**Expected time**: 2-5 minutes (downloads ~500MB)

### Step 3: Configure VirusTotal (Optional)

To enable VirusTotal cloud scanning:

1. Get free API key from https://www.virustotal.com/gui/join-us
2. Edit `config.json`:

```json
{
  "virustotal": {
    "api_key": "your_api_key_here",
    "enabled": true
  }
}
```

**Note**: Free tier allows 4 requests/minute

## Directory Structure

```
Phase1-Detection/
├── scripts/
│   ├── integrated_detector.py      # Main detection engine
│   ├── setup_phase1_tools.py       # Tool installation script
│   ├── malware_detector.py         # Legacy multi-engine scanner
│   ├── create_test_samples.py      # Test file generator
│   └── run_full_pipeline_test.py   # Automated testing
├── yara-rules/
│   └── yara-rules-repo/            # 1000+ YARA rules by category
│       ├── malware/                # Malware signatures
│       ├── antidebug_antivm/       # Anti-analysis techniques
│       ├── crypto/                 # Cryptography patterns
│       ├── packers/                # Packer detection
│       └── webshells/              # Web shell patterns
├── downloads/                      # ML detection models
├── quarantine/                     # Quarantined malicious files
├── audit-logs/                     # Tamper-proof audit trail
├── config.json                     # Configuration file
├── requirements.txt                # Python dependencies
└── README.md                       # This file
```

## Usage

### Basic File Scanning

Scan a single file with text output:

```bash
cd Phase1-Detection
python scripts/integrated_detector.py --file path/to/suspicious_file.exe
```

**Example output**:
```
================================================================================
SCAN RESULTS: suspicious_file.exe
================================================================================
Status: [!] MALWARE DETECTED
Risk Level: HIGH
File Size: 45.2 KB

Threat Indicators:
  - YARA rules matched (3 detections)
  - High PE suspicion score (85)
  - VirusTotal: 42/70 engines detected

YARA Detections:
  [malware] Generic_Trojan
  [packer] UPX_Packed
  [antidebug] Anti_VM_Techniques

PE Analysis:
  Suspicious Score: 85/100
  Dangerous Imports: VirtualAlloc, WriteProcessMemory, CreateRemoteThread
  High Entropy Section: .text (7.9)
  Missing Security: DEP, ASLR
```

### JSON Output

Get machine-readable JSON results:

```bash
python scripts/integrated_detector.py --file malware.exe --format json
```

**Example JSON**:
```json
{
  "file_path": "malware.exe",
  "file_name": "malware.exe",
  "file_size": 46234,
  "hashes": {
    "md5": "5d41402abc4b2a76b9719d911017c592",
    "sha1": "aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d",
    "sha256": "2c26b46b68ffc68ff99b453c1d30413413422d706..."
  },
  "scanners": {
    "yara_advanced": {
      "matches": [
        {"rule": "Generic_Trojan", "category": "malware"},
        {"rule": "UPX_Packed", "category": "packer"}
      ],
      "malware_detected": true
    },
    "pe_advanced": {
      "suspicious_score": 85,
      "threats": ["VirtualAlloc", "High entropy"],
      "is_pe": true
    },
    "virustotal": {
      "malicious": 42,
      "total": 70
    }
  },
  "malware_detected": true,
  "risk_level": "high"
}
```

### Batch Scanning

Scan entire directory:

```bash
# Scan all .exe files
for file in /path/to/directory/*.exe; do
    python scripts/integrated_detector.py --file "$file" --format json >> results.jsonl
done
```

### Understanding Risk Levels

- **HIGH**: 2+ threat indicators detected
- **MEDIUM**: 1 threat indicator detected
- **LOW**: No threats detected (clean)

### Threat Indicators

1. **YARA Matches**: Signature-based detection of known malware patterns
2. **High PE Score (≥60)**: Suspicious characteristics in executable file
3. **VirusTotal Detections**: Multiple AV engines flagged as malicious

## Configuration

### config.json Structure

```json
{
  "phase1_config": {
    "clamav": {
      "enabled": true,
      "database_path": "/usr/local/share/clamav",
      "scan_timeout": 300
    },
    "ml_detection": {
      "enabled": true,
      "model_path": "ml-detection/models/malware_model.pkl",
      "threshold": 0.85
    },
    "yara": {
      "enabled": true,
      "rules_directory": "yara-rules/yara-rules-repo",
      "fast_mode": false,
      "max_rules_per_category": 15
    },
    "virustotal": {
      "enabled": false,
      "api_key": "",
      "rate_limit": 4
    },
    "quarantine": {
      "enabled": true,
      "directory": "quarantine",
      "auto_quarantine": true,
      "encryption": true
    }
  }
}
```

### Customization Options

- **YARA Rules**: Add custom `.yar` files to `yara-rules/yara-rules-repo/malware/`
- **Quarantine**: Change `quarantine.directory` to store suspicious files elsewhere
- **Detection Threshold**: Adjust `ml_detection.threshold` (0.0-1.0) for ML sensitivity
- **Rate Limiting**: Modify `virustotal.rate_limit` for API quota management

## How It Works

### Detection Pipeline

```
File Input
    ↓
1. File Validation & Hash Calculation (MD5, SHA1, SHA256)
    ↓
2. YARA Scanning (1000+ rules across 5 categories)
    ↓
3. PE Analysis (if executable)
    ├─ Import Analysis (dangerous API detection)
    ├─ Entropy Calculation (packer detection)
    ├─ Security Features (ASLR, DEP, SEH, CFG)
    └─ Suspicious Score (0-100)
    ↓
4. VirusTotal Lookup (optional, hash-based)
    ↓
5. Threat Assessment
    ├─ Risk Level: HIGH / MEDIUM / LOW
    ├─ Malware Detected: TRUE / FALSE
    └─ Threat Indicators: [list]
    ↓
6. Output Results (JSON / Text)
```

### YARA Detection

- **Categories Scanned**: malware, antidebug_antivm, crypto, packer, webshell
- **Rules per Category**: 15 (prioritizes common + EICAR rules)
- **Total Rules Checked**: ~365 rules per scan
- **Match Details**: Rule name, category, matched strings

### PE Analysis Scoring

Suspicious score calculated from:

| Indicator | Points | Description |
|-----------|--------|-------------|
| Dangerous imports | +20 each | VirtualAlloc, WriteProcessMemory, CreateRemoteThread, etc. |
| High entropy section | +30 | Entropy > 7.0 (indicates encryption/packing) |
| Missing ASLR | +10 | No Address Space Layout Randomization |
| Missing DEP | +10 | No Data Execution Prevention |
| TLS callbacks | +15 | Often used for anti-debugging |

**Score Interpretation**:
- 0-30: Clean
- 31-59: Suspicious
- 60-100: Malware

### Audit Trail

Every scan is logged with:
- Timestamp
- File hash (SHA256)
- Scan results
- User/system context
- Cryptographic chain linking (tamper-proof)

Logs stored in: `audit-logs/audit_trail.json`

## Testing

### Generate Test Samples

```bash
python scripts/create_test_samples.py
```

Creates 8 test files:
- 3 benign (text, Python script, fake PE)
- 4 suspicious (malicious patterns, dangerous imports)
- 1 EICAR test virus (may be quarantined by antivirus)

### Run Full Pipeline Test

```bash
python scripts/run_full_pipeline_test.py
```

Runs integrated detector on all test samples and generates summary report.

## Troubleshooting

### YARA Module Not Found

```bash
pip install yara-python
```

### pefile Module Not Found

```bash
pip install pefile
```

### YARA Rules Not Found

Run setup script:
```bash
python scripts/setup_phase1_tools.py
```

### Windows Defender Quarantines Test Files

This is expected behavior for EICAR test file. To prevent:
1. Add `Phase1-Detection/` to Windows Defender exclusions
2. **Warning**: Only do this for testing, not production

### Git Clone Fails

Ensure Git is installed:
```bash
git --version
```

If not installed, download from: https://git-scm.com/downloads

### Permission Denied Errors

Run terminal as Administrator (Windows) or use sudo (Linux/macOS)

## Performance

- **Single File Scan**: 1-3 seconds
- **YARA Rules Loaded**: 365 rules (cached after first load)
- **Memory Usage**: ~200MB (YARA rules + PE analysis)
- **VirusTotal API**: 4 requests/minute (free tier)

## Security Considerations

1. **Isolation**: Scan suspicious files in isolated VM/sandbox
2. **Quarantine Access**: Restrict access to quarantine directory
3. **Audit Logs**: Review regularly for security incidents
4. **API Keys**: Keep VirusTotal API key confidential
5. **Updates**: Update YARA rules monthly

## Integration with Phase 2

Phase 1 detection seamlessly integrates with Phase 2 (Metadata Extraction) for comprehensive forensic analysis.

### Automated Integration (Recommended)

Use the integrated analysis script that combines both phases:

```bash
# Single command for Phase 1 + Phase 2 analysis
cd ../Phase2-Metadata
python scripts/phase1_phase2_integration.py --file malware.exe --output report.txt

# JSON output for further processing
python scripts/phase1_phase2_integration.py --file suspicious.dll --format json --output analysis.json
```

**What it does**:
1. Runs Phase 1 malware detection (YARA + PE + VirusTotal)
2. Extracts Phase 2 metadata (Zone.Identifier, hashes, PE details)
3. Correlates results from both phases
4. Generates unified threat score (0-100)
5. Provides recommended actions based on findings

### Manual Integration

For manual workflow control:

```bash
# Step 1: Phase 1 Detection
python scripts/integrated_detector.py --file malware.exe --format json > detection.json

# Step 2: Phase 2 Metadata Extraction (if malware detected)
if grep -q '"malware_detected": true' detection.json; then
    python ../Phase2-Metadata/scripts/metadata_extractor.py --file malware.exe --assess-risk --output metadata.json
fi
```

### Integration Benefits

- **Download Source Tracing**: Zone.Identifier reveals where malware was downloaded from
- **Timeline Correlation**: Match file timestamps with detection events
- **Attribution**: Link malware to threat actors via metadata
- **Evidence Collection**: Combine detection + metadata for forensic reports
- **Risk Scoring**: Unified threat score from multiple indicators

See [Phase 2 README](../Phase2-Metadata/README.md) for detailed metadata extraction capabilities.

## References

- **YARA Rules Repository**: https://github.com/Yara-Rules/rules
- **YARA Documentation**: https://yara.readthedocs.io/
- **pefile Library**: https://github.com/erocarrera/pefile
- **VirusTotal API**: https://developers.virustotal.com/reference
- **Malware Detection ML**: https://github.com/topics/malware-detection

## Project Documentation

- [OPENSOURCE_PROJECTS_BY_PHASE.md](../OPENSOURCE_PROJECTS_BY_PHASE.md) - Complete list of 62+ DFIR tools
- [Phase2-Metadata/README.md](../Phase2-Metadata/README.md) - Metadata extraction phase
- [QUICKSTART.md](../QUICKSTART.md) - Quick start guide for all phases

## Support

For issues, questions, or contributions:
1. Check troubleshooting section above
2. Review GitHub repositories for individual tools
3. Consult ZIWIZ forensic pipeline documentation

## License

This implementation uses multiple open-source tools, each with their own licenses. Refer to individual tool repositories for license details.
# Phase 2: Metadata Extraction & Augmentation

## Overview
Phase 2 implements comprehensive metadata extraction from various file types for forensic analysis. This phase focuses on **must-have** open-source tools to extract actionable intelligence from files detected in Phase 1.

## Architecture

Phase 2 aggregates metadata from multiple sources:
1. **EXIF Metadata** - Images (GPS, timestamps, camera info)
2. **PE Analysis** - Executables (headers, imports, packers)
3. **Zone.Identifier** - Download sources (URLs, referrer)
4. **Browser Forensics** - Chrome/Chromium artifacts (history, downloads, cookies)
5. **Timeline Generation** - Chronological correlation of events

## Tools Implemented (Must-Have Only)

### 1. EXIF Metadata Extraction
- **ExifTool**: Industry-standard for 100+ file formats
- Extracts: GPS coordinates, timestamps, camera make/model, thumbnails
- Script: `scripts/exif_extractor.py`

### 2. PE File Analysis
- **pefile**: Python library for parsing PE files
- Extracts: Headers, imports/exports, resources, imphash, compile time
- Detects: Packers, suspicious APIs, code signing
- Script: `scripts/pe_analyzer.py`

### 3. Zone.Identifier & ADS Parser
- **PowerShell Get-Item**: Native Windows ADS parsing
- Extracts: Zone ID (0-4), download URL, referrer URL
- Detects: Downloaded files from Internet (Zone 3), Restricted Sites (Zone 4)
- Script: `scripts/zone_identifier.py`

### 4. Browser Forensics
- **SQLite-based extraction** for Chrome/Chromium browsers
- Extracts: History, downloads, cookies, autofill, bookmarks
- Multi-platform: Windows, Linux, macOS
- Script: `scripts/browser_forensics.py`

### 5. Metadata Aggregator (Main Module)
- **Orchestrates all extraction modules**
- Generates unified timeline of events
- Risk assessment across all metadata sources
- Script: `scripts/metadata_extractor.py`

## Installation

### Prerequisites
- Python 3.8+
- ExifTool (Windows: download, Linux: apt, macOS: brew)
- PowerShell (Windows only, for Zone.Identifier)

### Quick Start

```bash
# 1. Install Python dependencies
pip install -r requirements.txt

# 2. Install ExifTool
# Windows: Download from https://exiftool.org/
# Linux: sudo apt-get install libimage-exiftool-perl
# macOS: brew install exiftool

# 3. Run setup and verification
python setup.py --install
python setup.py --verify

# 4. Test with a file
python scripts/metadata_extractor.py --file suspicious.exe --assess-risk
```

## Directory Structure

```
Phase2-Metadata/
├── scripts/
│   ├── exif_extractor.py         # EXIF metadata extraction
│   ├── pe_analyzer.py            # PE file analysis
│   ├── zone_identifier.py        # Zone.Identifier parser
│   ├── browser_forensics.py      # Browser artifact extraction
│   └── metadata_extractor.py     # Main aggregator (use this!)
├── tools/                        # External tools (ExifTool)
├── output/                       # Extracted metadata output
├── requirements.txt              # Python dependencies
├── setup.py                      # Installation & verification
└── README.md                     # This file
```

## Usage Examples

### 1. Analyze Single File (Recommended)

```bash
# Comprehensive analysis with risk assessment
python scripts/metadata_extractor.py --file suspicious.exe --assess-risk --output report.json

# Text output with timeline
python scripts/metadata_extractor.py --file malware.exe --assess-risk --timeline --format text
```

### 2. Scan Directory

```bash
# Scan directory recursively
python scripts/metadata_extractor.py --directory C:\Downloads --recursive --assess-risk

# Filter by extension
python scripts/metadata_extractor.py --directory /tmp --recursive --extensions .exe .dll --output scan_results.json
```

### 3. Batch Analysis

```bash
# Create file list (file_list.txt with one path per line)
# Then analyze all files
python scripts/metadata_extractor.py --batch file_list.txt --assess-risk --timeline --output batch_report.json
```

### 4. Individual Module Usage

#### EXIF Extraction
```bash
python scripts/exif_extractor.py --file photo.jpg --output exif_data.json
```

#### PE Analysis
```bash
python scripts/pe_analyzer.py --file suspicious.exe --output pe_analysis.json --check-virustotal
```

#### Zone.Identifier
```bash
# Single file
python scripts/zone_identifier.py --file downloaded.exe --assess-risk

# Directory scan
python scripts/zone_identifier.py --directory C:\Downloads --recursive --output zone_report.json
```

#### Browser Forensics
```bash
# Auto-detect Chrome profiles
python scripts/browser_forensics.py --auto-detect --browser chrome --assess-risk

# Specific profile
python scripts/browser_forensics.py --profile "C:\Users\user\AppData\Local\Google\Chrome\User Data\Default" --output browser_artifacts.json

# All profiles
python scripts/browser_forensics.py --all-profiles --browser chrome --format json
```

## Output Formats

All scripts support multiple output formats:

### JSON (Machine-readable)
```bash
--format json --output results.json
```

### Text (Human-readable)
```bash
--format text
```

### Example JSON Output Structure
```json
{
  "scan_timestamp": "2025-11-14T12:34:56Z",
  "total_files_analyzed": 1,
  "results": [
    {
      "file_path": "C:\\Downloads\\suspicious.exe",
      "file_metadata": {
        "file_size": 102400,
        "file_extension": ".exe",
        "created_time": "2025-11-14T10:00:00Z"
      },
      "hashes": {
        "md5": "d41d8cd98f00b204e9800998ecf8427e",
        "sha256": "..."
      },
      "zone_identifier": {
        "zone_id": 3,
        "zone_name": "Internet",
        "host_url": "https://malicious-site.com/payload.exe"
      },
      "pe_metadata": {
        "pe_type": "PE32",
        "compile_time": "2025-11-01T08:30:00Z",
        "is_packed": true,
        "imphash": "abc123..."
      },
      "risk_assessment": {
        "risk_level": "high",
        "risk_score": 75,
        "indicators": [
          "Downloaded from Internet (Zone 3)",
          "PE file is packed (possible obfuscation)",
          "PE file is not digitally signed"
        ]
      }
    }
  ]
}
```

## Key Features

### Metadata Extraction

#### EXIF (Images)
- GPS coordinates (latitude, longitude, altitude)
- Camera make, model, lens
- Timestamps: DateTimeOriginal, CreateDate, ModifyDate
- Software used, copyright info
- Thumbnail extraction

#### PE (Executables)
- DOS header, PE header, Optional header
- Section analysis (.text, .data, .rsrc)
- Import/Export tables
- Resource analysis
- Imphash, Rich header hash
- Packer detection (UPX, ASPack, etc.)
- Digital signature verification
- Compile timestamp

#### Zone.Identifier (Downloads)
- Zone ID: 0=Local, 1=Intranet, 2=Trusted, 3=Internet, 4=Restricted
- HostUrl: Direct download source
- ReferrerUrl: Website that linked to download
- Alternate Data Streams listing

#### Browser Forensics (Chrome/Chromium)
- Browsing history with visit counts
- Download history with referrer URLs
- Cookies (domain, name, creation, expiry)
- Autofill data (forms, addresses)
- Timeline of browser activity

### Risk Assessment

The `--assess-risk` flag provides security risk scoring:

**Risk Levels**: Low (0-29), Medium (30-49), High (50-69), Critical (70-100)

**Risk Indicators**:
- High-risk file extensions (.exe, .dll, .scr, .bat, .vbs)
- Downloads from Internet (Zone 3) or Restricted Sites (Zone 4)
- Suspicious download URLs (malware, crack, keygen, hack)
- Packed executables (obfuscation)
- Unsigned PE files
- Suspicious API imports (VirtualAlloc, WriteProcessMemory)
- Images with GPS coordinates (privacy risk)

### Timeline Generation

The `--timeline` flag generates chronological event timeline:

**Event Sources**:
- File system (created, modified, accessed)
- EXIF (photo taken, modified)
- PE (compile time)
- Browser (downloads, visits)
- Zone.Identifier (download time)

**Output**: Sorted list of events with timestamps, types, sources

## Integration with Phase 1

Phase 2 seamlessly integrates with [Phase 1 (Detection)](../Phase1-Detection) for comprehensive malware analysis.

### Automated Integration (Recommended)

Use the integrated analysis script that combines both phases:

```bash
# Single command for complete Phase 1 + Phase 2 analysis
python scripts/phase1_phase2_integration.py --file suspicious.exe --output report.txt

# JSON format for automated processing
python scripts/phase1_phase2_integration.py --file malware.dll --format json --output analysis.json
```

**Integrated Workflow**:
1. **Phase 1 Detection**: YARA rules + PE analysis + VirusTotal
2. **Phase 2 Metadata**: Zone.Identifier + Browser + Hashes + Timeline
3. **Correlation Analysis**: Combines findings from both phases
4. **Unified Threat Score**: 0-100 score based on all indicators
5. **Recommended Actions**: QUARANTINE, BLOCK, MONITOR, or ALLOW

### Example Integrated Report

```
================================================================================
PHASE 1 + PHASE 2 INTEGRATED FORENSIC REPORT
================================================================================
File: malware.exe
Threat Level: HIGH
Threat Score: 85/100

PHASE 1: MALWARE DETECTION
--------------------------------------------------------------------------------
Malware Detected: YES
YARA Rule Matches: 3 (ransomware, packer, suspicious_imports)
PE Analysis: SUSPICIOUS (score: 75/100)
VirusTotal Positives: 45/70

PHASE 2: METADATA EXTRACTION
--------------------------------------------------------------------------------
SHA256: abc123...
Zone.Identifier: Zone 3 (Internet)
Download URL: https://malicious-site.example.com/payload.exe
Risk Level: CRITICAL
Risk Score: 90/100

CORRELATION ANALYSIS
--------------------------------------------------------------------------------
Key Findings:
  1. Malware detected AND downloaded from Internet (Zone 3)
  2. YARA detected (3 rules) AND suspicious download URL
  3. PE file flagged as suspicious by Phase 1
  4. VirusTotal: 45 engines detected malware

RECOMMENDED ACTIONS
--------------------------------------------------------------------------------
  1. QUARANTINE file immediately
  2. Block download source URL in firewall
  3. Scan system for additional infections
  4. Report to security team
  5. Preserve forensic evidence
================================================================================
```

### Manual Integration

For step-by-step control:

```bash
# Step 1: Phase 1 - Detect malware
cd ../Phase1-Detection
python scripts/integrated_detector.py --file suspicious.exe --format json > detection.json

# Step 2: Phase 2 - Extract metadata
cd ../Phase2-Metadata
python scripts/metadata_extractor.py --file suspicious.exe --assess-risk --output metadata.json

# Step 3: Combine results manually
# Review both detection.json and metadata.json
```

### Integration Benefits

1. **Download Source Tracing**: Zone.Identifier reveals where malware was downloaded from
2. **Timeline Correlation**: Match file creation time with detection events
3. **Attribution**: PE metadata + download URL links to threat actors
4. **Browser Evidence**: Download history confirms infection vector
5. **Hash Correlation**: SHA256 matches with threat intelligence databases
6. **Unified Threat Scoring**: Combined risk assessment from both phases
7. **Automated Response**: Actions based on correlation findings

See [Phase 1 README](../Phase1-Detection/README.md) for malware detection capabilities.

## Performance Optimization

- **Parallel Processing**: Analyze multiple files concurrently
- **Database Caching**: SQLite copies for browser forensics (avoid locks)
- **Stream Processing**: Handle large datasets efficiently
- **Selective Extraction**: Filter by extension to reduce overhead

## Security Considerations

1. **Data Privacy**: Handle extracted metadata (GPS, personal info) securely
2. **File Locks**: Browser databases copied to temp to avoid locks
3. **Encrypted Data**: Cookie values may be DPAPI-encrypted on Windows
4. **Chain of Custody**: All operations logged with timestamps
5. **Hash Verification**: SHA256 hashes for integrity validation

## Troubleshooting

### ExifTool Not Found
```bash
# Verify installation
exiftool -ver

# Add to PATH (Windows)
set PATH=%PATH%;C:\Users\user\Desktop\Cyber\Phase2-Metadata\tools\exiftool

# Or use full path in scripts
```

### PowerShell Execution Error
```powershell
# Enable scripts (Windows)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Browser Database Locked
- Close browser before analysis
- Scripts create temporary copies to avoid locks
- Check temp directory: `Phase2-Metadata/.browser_forensics_temp/`

### Python Module Not Found
```bash
# Install missing modules
pip install -r requirements.txt

# Verify
python setup.py --verify
```

## Advanced Usage

### Custom Risk Assessment Weights

Modify `metadata_extractor.py` risk scoring:
```python
# Line ~398: Adjust risk scores
if zone_id == 3:  # Internet
    risk_score += 30  # Change this value
```

### Output to Database

```bash
# Extract metadata and save to SQLite
python scripts/metadata_extractor.py --directory /evidence --recursive --output metadata.db --format sqlite
```

### Integration with Phase 3 (Path Reconstruction)

```bash
# Phase 2 output feeds into Phase 3
python scripts/metadata_extractor.py --file malware.exe --output phase2_output.json
python ../Phase3-PathReconstruction/path_reconstructor.py --metadata phase2_output.json
```

## Testing

```bash
# Verify all modules
python setup.py --verify

# Test individual modules
python scripts/zone_identifier.py --help
python scripts/browser_forensics.py --help
python scripts/metadata_extractor.py --help

# Test with sample file
python scripts/metadata_extractor.py --file README.md --assess-risk --format text
```

## References

### Documentation
- ExifTool: https://exiftool.org/
- pefile: https://github.com/erocarrera/pefile
- Chrome Forensics: https://www.digitalforensics.com/blog/chrome-forensics/
- Zone.Identifier: https://docs.microsoft.com/en-us/windows/security/threat-protection/

### Research Papers
- "PE-Miner: Mining Structural Information to Detect Malicious Executables in Realtime" (Shafiq et al.)
- "Detecting Malware in JPEG Files Through EXIF Tag Analysis using Machine Learning" (Majumdar)

### Related Projects
- ZIWIZ Forensic Pipeline (Phase 1: Detection, Phase 3: Path Reconstruction, Phase 4: Reporting)

## Support

For issues, questions, or feature requests:
1. Check troubleshooting section above
2. Verify installation: `python setup.py --verify`
3. Review script help: `python scripts/metadata_extractor.py --help`
4. Check Phase 1 integration for workflow issues

## License

Open-source tools used under their respective licenses:
- ExifTool: Perl Artistic License / GPL
- pefile: MIT License
- Python scripts: Same as parent ZIWIZ project

---

**Quick Reference Commands**

```bash
# Setup
python setup.py --install --verify

# Single file analysis
python scripts/metadata_extractor.py --file malware.exe --assess-risk --timeline

# Directory scan
python scripts/metadata_extractor.py --directory C:\Downloads --recursive --assess-risk --output scan.json

# Browser forensics
python scripts/browser_forensics.py --auto-detect --browser chrome --assess-risk

# Zone.Identifier check
python scripts/zone_identifier.py --directory C:\Downloads --recursive --assess-risk
```

