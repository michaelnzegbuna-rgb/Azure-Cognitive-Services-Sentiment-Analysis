# Azure Sentiment Analysis Learning Project

This repository holds a Python implementation that performs sentiment analysis and aspect-based opinion mining on text using **Azure Cognitive Services Language Service** (previously known as Text Analytics).

The codebase is built to showcase several practical skills: integrating with a cloud API, handling authentication securely, optimizing requests through batching, recovering gracefully from transient errors, and producing structured output reports.

---

## 📋 Table of Contents
1. [Prerequisites](#-prerequisites)
2. [Project Layout](#-project-layout)
3. [Setting Up Azure Resources](#-setting-up-azure-resources)
4. [Environment Configuration](#-environment-configuration)
5. [Running the Project](#-running-the-project)
6. [Core Features](#-core-features)
7. [Output Files](#-output-files)
8. [Credential Security Practices](#-credential-security-practices)
9. [Troubleshooting](#-troubleshooting)

---

## ✅ Prerequisites

Before getting started, make sure you have:
- **Python 3.8+** installed
- An active **Azure subscription** (optional — the project runs fine in Simulation Mode without one)
- The **Azure CLI** installed, if you plan to provision resources from the command line rather than the Portal

---

## 📁 Project Layout

```
├── data/
│   └── sample_reviews.json        # Preprocessed sample review dataset
├── docs/
│   ├── analysis_report.md         # Narrative report & breakdown of results
│   └── security_documentation.md  # Detailed API security practices
├── output/
│   ├── results.json               # Parsed sentiment analysis JSON output
│   └── summary_report.txt         # Plaintext summary analytics report
├── src/
│   └── sentiment_analyzer.py      # Core python script integrating Azure SDK
└── README.md                      # Setup and environment configuration guide
```

---

## ☁️ Setting Up Azure Resources

Before this project can talk to live Azure Cognitive Services, you'll need to set up a Language Service resource. If you'd rather skip this step entirely, see the **Simulation Mode** note below — no Azure resource is required to test the program logic.

### Option A: Through the Azure Portal
1. Log into the [Azure Portal](https://portal.azure.com/).
2. Select **Create a resource** and look up **Language**.
3. Choose **Language Service**, then click **Create**.
4. Fill out the resource details:
   - **Subscription**: The Azure subscription you want billed.
   - **Resource group**: Pick an existing one or spin up a new one (for example, `rg-sentiment-analysis`).
   - **Region**: Choose a region near your location (such as `East US` or `West Europe`).
   - **Name**: Give it a unique name (for example, `lang-sentiment-learning`).
   - **Pricing tier**: Choose `Free F0` for learning purposes, or `Standard S` for production-level usage.
5. Hit **Review + create**, then confirm with **Create**.
6. After deployment finishes, open the resource and go to **Keys and Endpoint** under **Resource Management**.
7. Make a note of **KEY 1** and the **Endpoint URL** — you'll need both in the next section.

### Option B: Through the Azure CLI
With the Azure CLI installed and signed in (`az login`), the following commands will get you set up:

```bash
# 1. Create a Resource Group (if not already existing)
az group create --name rg-sentiment-analysis --location eastus

# 2. Provision the Language Cognitive Service Account
az cognitiveservices account create \
    --name lang-sentiment-learning \
    --resource-group rg-sentiment-analysis \
    --kind TextAnalytics \
    --sku F0 \
    --location eastus \
    --yes

# 3. Retrieve Keys and Endpoint
az cognitiveservices account keys list --name lang-sentiment-learning --resource-group rg-sentiment-analysis
az cognitiveservices account show --name lang-sentiment-learning --resource-group rg-sentiment-analysis --query properties.endpoint -o tsv
```

---

## ⚙️ Environment Configuration

### 1. Install Required Packages
This project relies on the official Azure SDKs, installable through `pip`:

```bash
pip install azure-ai-textanalytics azure-identity azure-keyvault-secrets
```

### 2. Set Up Credentials
Credentials are pulled securely from environment variables rather than stored in code. Set these in your shell:

**Windows PowerShell:**
```powershell
$env:AZURE_LANGUAGE_ENDPOINT="https://<your-resource-name>.cognitiveservices.azure.com/"
$env:AZURE_LANGUAGE_KEY="<your-azure-key>"
```

**Windows CMD:**
```cmd
set AZURE_LANGUAGE_ENDPOINT=https://<your-resource-name>.cognitiveservices.azure.com/
set AZURE_LANGUAGE_KEY=<your-azure-key>
```

**Linux/macOS Bash:**
```bash
export AZURE_LANGUAGE_ENDPOINT="https://<your-resource-name>.cognitiveservices.azure.com/"
export AZURE_LANGUAGE_KEY="<your-azure-key>"
```

> [!TIP]
> **Fallback to Simulation Mode:** Without an active Azure subscription, or if these environment variables aren't set, the script defaults to **Simulation Mode**, which relies on a local mock client. This means the program's logic can be tested and graded right away, with no need to provision any cloud resources.

---

## 🚀 Running the Project

Launch the script with Python. It will try connecting to Azure by default, falling back to Simulation Mode if credentials aren't available:

```bash
python src/sentiment_analyzer.py
```

### Command Line Arguments

| Argument | Purpose | Default |
|---|---|---|
| `--mock` | Forces Simulation Mode even if Azure credentials are set | off |
| `--endpoint` | Overrides the Azure endpoint URL | from `AZURE_LANGUAGE_ENDPOINT` |
| `--vault-url` | Azure Key Vault URL to pull the API key from | none |
| `--secret-name` | Name of the secret in Key Vault holding the API key | none |
| `--input` | Path to the input review dataset | `data/sample_reviews.json` |
| `--output` | Path to write the results JSON | `output/results.json` |
| `--batch-size` | Number of reviews sent per API call | `10` |

Example usages:

```bash
# Force Simulation Mode even if credentials are set
python src/sentiment_analyzer.py --mock

# Run with Key Vault retrieval
python src/sentiment_analyzer.py --endpoint "https://<lang-resource>.cognitiveservices.azure.com/" --vault-url "https://<vault-name>.vault.azure.net/" --secret-name "LanguageServiceApiKey"

# Specify custom inputs, outputs, and batch size
python src/sentiment_analyzer.py --input data/sample_reviews.json --output output/results.json --batch-size 5
```

---

## 🛠️ Core Features

1. **Request Batching:** Input reviews are grouped into batches of `10` (matching the standard API limit) prior to each call, which cuts down on request overhead and keeps usage within free-tier caps.
2. **Opinion Mining:** Aspect-based sentiment extraction can be turned on, pulling out target aspects (such as "food" or "battery life") and linking them to their descriptors (like "delicious" or "fantastic").
3. **Handling Transient Errors:** The script applies exponential backoff when it encounters transient failures (HTTP `429` rate-limit responses or `5xx` server errors), spacing retry attempts further apart each time (`2s`, `4s`, `8s`) before eventually giving up.
4. **Output Structuring:** SDK responses get normalized into clean, flat JSON written to `output/results.json`, capturing:
   - Document ID
   - Overall document sentiment and confidence scores
   - Per-sentence text, sentiment label, and scores
   - Aspect-level target and descriptor pairings

---

## 📄 Output Files

After a run completes, check the `output/` directory for:
- **`results.json`** — the full structured sentiment and opinion-mining output, suitable for further processing or analysis.
- **`summary_report.txt`** — a quick, human-readable summary of overall sentiment trends across the dataset.

For a narrative walkthrough of sample results, see [docs/analysis_report.md](docs/analysis_report.md).

---

## 🔒 Credential Security Practices

*   **No Hardcoded Keys:** Credentials never appear directly in the source code; they're retrieved via `os.environ.get` instead.
*   **Azure Key Vault & Managed Identity:** For details on integrating Azure Key Vault and swapping out API keys for Microsoft Entra ID (Azure Active Directory) Managed Identities in a production setup, see [docs/security_documentation.md](docs/security_documentation.md).

---

## 🧩 Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| Script runs in Simulation Mode unexpectedly | Environment variables not set or not exported in the current shell session | Re-run the `export`/`set`/`$env:` command in the same terminal session you're using to run the script |
| `401` or `403` errors | Incorrect or expired API key | Regenerate the key in the Azure Portal under **Keys and Endpoint** |
| `429` errors persist after retries | Free-tier (`F0`) rate limits exceeded | Reduce `--batch-size`, add delay between runs, or upgrade to `Standard S` tier |
| Key Vault retrieval fails | Missing permissions or incorrect `--vault-url`/`--secret-name` | Confirm your identity has `get` access to secrets in the Key Vault's access policy |
