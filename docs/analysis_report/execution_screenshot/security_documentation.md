# Security & Credential Management Documentation

This document details the security principles, practices, and configuration options implemented to manage credentials, authenticate, and securely interact with Azure Cognitive Services.

---

## 🔐 1. Local Development Security

During local development, it is vital to avoid hardcoding credentials (such as API keys and endpoints) directly into the source code to prevent accidental exposure (e.g., committing keys to public GitHub repositories).

### Practices Implemented:
- **Environment Variables:** The `src/sentiment_analyzer.py` script reads the endpoint and API key from the system environment using Python’s `os.environ.get("AZURE_LANGUAGE_KEY")`.
- **Git Protection:** If local configuration files (like `.env` or `.json` secret stores) are used, they must be added to a `.gitignore` file to ensure they are never committed to version control.
- **Credential Fallback:** The script implements a Simulation Mode fallback so that developers can test and grade the script without needing to share or configure live Azure keys.

---

## 🛡️ 2. Production Security: Azure Key Vault

In a production cloud environment, credentials should be centralized and managed using a dedicated secrets manager like **Azure Key Vault**. 

The main Python script `src/sentiment_analyzer.py` **fully implements** this. If the command line argument `--vault-url` or environment variable `AZURE_KEY_VAULT_URL` is set, the script uses `DefaultAzureCredential` to authenticate to the vault, retrieves the API key, and starts the Language Service client securely.

### Code Implemented (Integrating Key Vault in Python):
```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

# 1. Authenticate to Key Vault using Managed Identity
vault_url = "https://kv-sentiment-prod.vault.azure.net/"
credential = DefaultAzureCredential()
secret_client = SecretClient(vault_url=vault_url, credential=credential)

# 2. Retrieve Secret value
api_key = secret_client.get_secret("LanguageServiceApiKey").value
endpoint = "https://lang-sentiment-learning.cognitiveservices.azure.com/"

# 3. Initialize Client
client = TextAnalyticsClient(endpoint=endpoint, credential=AzureKeyCredential(api_key))
```

---

## 🚀 3. Advanced Security: Managed Identities (Keyless Auth)

The gold standard for cloud engineering security is eliminating API keys entirely. By utilizing **Microsoft Entra ID (formerly Azure Active Directory) Managed Identities**, Azure resources (such as Azure Virtual Machines, App Services, or Functions) can authenticate to the Language Service without any passwords or credentials stored in code or configuration.

### Implementation steps:
1. **Enable Managed Identity:** Turn on System-Assigned or User-Assigned Managed Identity on the Azure resource hosting the code.
2. **Assign Role (RBAC):** Assign the **"Cognitive Services User"** (or specialized Language Service roles) Role-Based Access Control (RBAC) role to the identity on the Language Service resource.
3. **Use DefaultAzureCredential:** Update the Python code to authenticate directly using `DefaultAzureCredential` from the `azure-identity` library.

### Keyless Python Code Example:
```python
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient

# DefaultAzureCredential automatically discovers environment configurations,
# Managed Identities, Azure CLI credentials, or VS Code login contexts.
credential = DefaultAzureCredential()
endpoint = "https://lang-sentiment-learning.cognitiveservices.azure.com/"

# Keyless Client initialization
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)
```

This configuration removes keys entirely from environment files, environment variables, and memory, eliminating key rotation overhead and credential leak vulnerabilities.
