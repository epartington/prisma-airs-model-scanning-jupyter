# Prisma AIRS Model Security - Demo & Examples

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10-3.12](https://img.shields.io/badge/python-3.10--3.12-blue.svg)](https://www.python.org/downloads/)

Demonstration repository for scanning machine learning models for security vulnerabilities using Prisma AIRS Model Security.

## Overview

This repository provides working examples and a comprehensive Jupyter notebook demonstrating how to:

- Scan HuggingFace models for security threats
- Detect malicious code, backdoors, and supply chain attacks
- Integrate model scanning into CI/CD pipelines
- Analyze scan results and take action
- Configure security policies for different model sources

## Quick Start

> **Important:** The required packages (`model-security-client` and `airs-schemas`) are **proprietary Palo Alto Networks packages** not available on public PyPI. You need Prisma AIRS credentials to access them via a private package repository.

### Prerequisites

- Python 3.10, 3.11, or 3.12 (3.12 recommended)
- Prisma AIRS Model Security account
- Service account credentials (Client ID, Client Secret, TSG ID)

### Installation

**Automated Setup (Recommended)**

```bash
# 1. Clone the repo
git clone https://github.com/scthornton/prisma-airs-model-scanning-jupyter.git
cd prisma-airs-model-scanning-jupyter

# 2. Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# 3. Copy credentials template and fill in your values
cp .env.template .env
nano .env  # Get credentials from https://strata.paloaltonetworks.com

# 4. Run automated setup (authenticates + installs SDK)
./setup-sdk.sh
```

**Manual Installation**

If you prefer manual setup:

```bash
# Set credentials as environment variables
export MODEL_SECURITY_CLIENT_ID="AIRS@your-tsg-id.iam.panserviceaccount.com"
export MODEL_SECURITY_CLIENT_SECRET="your-client-secret-uuid"
export TSG_ID="your-tsg-id"

# Get PyPI URL and install
pip install model-security-client --extra-index-url $(./get-pypi-url.sh)
```

### Getting Your Credentials

1. **Log in to Strata Cloud Manager:** https://strata.paloaltonetworks.com
2. **Create Service Account:**
   - Navigate to: **Settings → Identity & Access → Service Accounts**
   - Click **Add Service Account**
   - Name: `Prisma AIRS Model Security`
   - Role: Select appropriate permissions (minimum: `ai_ms_pypi_auth`, `ai_ms.scans`, `ai_ms.security_groups`)
   - Save and copy the **Client ID** and **Client Secret** (shown once!)
3. **Get TSG ID:**
   - Navigate to: **Tenant Management**
   - Copy your **Tenant Service Group (TSG) ID**

### Run the Jupyter Notebook

```bash
jupyter notebook

# Open: notebooks/prisma-airs-interactive-model-security.ipynb
# Run all cells or step through individually
```

### Run Example Scripts

```bash
# List available security groups
python examples/list_security_groups.py

# Scan HuggingFace models
python examples/scan_huggingface_model.py
```

## Repository Structure

```
prisma-airs-model-scanning-jupyter/
├── notebooks/
│   ├── prisma-airs-interactive-model-security.ipynb  # Interactive demo (widgets)
│   ├── model_security_demo.ipynb                     # Step-by-step demo
│   └── README.md                                     # Notebook documentation
├── examples/
│   ├── list_security_groups.py                       # List security groups
│   └── scan_huggingface_model.py                     # Scan HuggingFace models
├── .env.template                                     # Credential template
├── get-pypi-url.sh                                   # PyPI authentication script
├── setup-sdk.sh                                      # Automated SDK installer
├── requirements.txt                                  # Python dependencies
├── QUICK-START.md                                    # Quick start guide
├── SDK-TLDR.md                                       # SDK installation FAQ
├── OVERVIEW.md                                       # Product overview
├── SECURITY.md                                       # Security policy
├── CONTRIBUTING.md                                   # Contribution guidelines
├── CODE_OF_CONDUCT.md                                # Code of conduct
├── LICENSE                                           # MIT License
└── ai-model-security.pdf                             # Official PANW documentation
```

## What Gets Scanned?

Prisma AIRS Model Security detects:

**Critical Threats**
- **Malicious Code Execution** - Pickle deserialization attacks, arbitrary code execution
- **Supply Chain Attacks** - Compromised dependencies, poisoned models
- **Neural Backdoors** - Hidden triggers that cause misclassification
- **Data Exfiltration** - Models designed to leak training data

**Policy Violations**
- **Unapproved Licenses** - GPL, AGPL, or custom licenses violating policy
- **Unsafe Formats** - Pickle, Keras (H5) files that allow code execution
- **Unverified Publishers** - Models from untrusted organizations

## Usage Examples

### Basic Model Scan

```python
from model_security_client.api import ModelSecurityAPIClient
from uuid import UUID

# Initialize client
client = ModelSecurityAPIClient(
    base_url="https://api.sase.paloaltonetworks.com/aims"
)

# Scan a model
result = client.scan(
    security_group_uuid=UUID("your-security-group-uuid"),
    model_uri="https://huggingface.co/microsoft/DialoGPT-medium"
)

# Check result
if str(result.eval_outcome) == "EvalOutcome.ALLOWED":
    print("Model is safe to deploy")
else:
    print(f"Model blocked: {result.eval_summary.rules_failed} rules failed")
```

## Viewing Detailed Results

The SDK returns summary data. For detailed findings:

1. Go to: https://strata.paloaltonetworks.com
2. Navigate to: **Insights → Prisma AIRS → Model Security → Scans**
3. Click on your Scan ID
4. View specific rule failures, threat descriptions, remediation steps, and file-level findings

## Security

**Never commit credentials to version control.**

- Use environment variables for credentials
- Add `.env` files to `.gitignore`
- Use service accounts with minimum required permissions
- Rotate credentials regularly
- See [SECURITY.md](SECURITY.md) for detailed security guidelines

## Troubleshooting

### ValidationError: Model URI format

HuggingFace URLs must include the author/organization:
```python
# Wrong
model_uri = "https://huggingface.co/gpt2"

# Correct
model_uri = "https://huggingface.co/openai-community/gpt2"
```

### Module Not Found / Package Installation Errors

The SDK is proprietary and hosted on Palo Alto's private PyPI, not public PyPI:
```bash
# Ensure virtual environment is activated
source .venv/bin/activate

# Use the automated setup script
./setup-sdk.sh

# OR install manually with authentication
pip install model-security-client --extra-index-url $(./get-pypi-url.sh)
```

See [SDK-TLDR.md](SDK-TLDR.md) for detailed troubleshooting.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

- **Documentation:** https://docs.paloaltonetworks.com/ai-runtime-security/ai-model-security
- **GitHub Issues:** [Create an issue](https://github.com/scthornton/prisma-airs-model-scanning-jupyter/issues)

---

**Note:** This is a demonstration repository. Always follow your organization's security policies when handling ML models and credentials.

---

## Contact

**Scott Thornton** — AI Security Researcher

- Website: [perfecxion.ai](https://perfecxion.ai/)
- Email: [scott@perfecxion.ai](mailto:scott@perfecxion.ai)
- LinkedIn: [linkedin.com/in/scthornton](https://www.linkedin.com/in/scthornton)
- ORCID: [0009-0008-0491-0032](https://orcid.org/0009-0008-0491-0032)
- GitHub: [@scthornton](https://github.com/scthornton)

**Security Issues**: Please report via [SECURITY.md](SECURITY.md)
