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
