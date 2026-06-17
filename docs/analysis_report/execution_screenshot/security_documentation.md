# Authentication & Secrets Handling Guide

This guide walks through how credentials are protected and how authentication to **Azure Cognitive Services** is handled, across three escalating levels of security maturity: local development, centralized secret storage, and fully keyless authentication.

---

## 🧪 1. Keeping Secrets Out of Source Code (Local Dev)

A core rule during development is that no API key or service endpoint should ever be written directly into a source file — doing so risks leaking credentials the moment code is pushed to a public repository.

**How this is handled here:**

- **Reading from the environment.** `src/sentiment_analyzer.py` pulls its endpoint and key at runtime via `os.environ.get("AZURE_LANGUAGE_KEY")`, rather than embedding either value in the script itself.
- **Keeping secret files out of version control.** Any local secret stores — `.env` files, JSON config files holding keys, etc. — should be listed in `.gitignore` so they're never staged or committed.
- **A no-credentials test path.** A built-in Simulation Mode lets anyone run and evaluate the script without needing access to a live Azure key at all, which is useful for grading or onboarding new contributors.

---

## 🔑 2. Centralized Secret Storage with Azure Key Vault

Once an application moves to production, credentials shouldn't live in environment variables scattered across machines — they belong in a dedicated secrets management service such as **Azure Key Vault**.

This is already wired up in `src/sentiment_analyzer.py`: when either the `--vault-url` flag or the `AZURE_KEY_VAULT_URL` environment variable is supplied, the script switches to `DefaultAzureCredential` to authenticate against the vault, pulls the API key from there, and uses it to spin up the Language Service client.

**Example — fetching the key from Key Vault:**

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient
from azure.core.credentials import AzureKeyCredential

# Step 1: Authenticate to the vault via Managed Identity
vault_url = "https://kv-sentiment-prod.vault.azure.net/"
credential = DefaultAzureCredential()
secret_client = SecretClient(vault_url=vault_url, credential=credential)

# Step 2: Pull the secret value out of the vault
api_key = secret_client.get_secret("LanguageServiceApiKey").value
endpoint = "https://lang-sentiment-learning.cognitiveservices.azure.com/"

# Step 3: Spin up the Text Analytics client
client = TextAnalyticsClient(endpoint=endpoint, credential=AzureKeyCredential(api_key))
```

---

## 🪪 3. Going Keyless: Managed Identity Authentication

The most secure pattern available is removing API keys from the picture entirely. With **Microsoft Entra ID** (the renamed Azure Active Directory) **Managed Identities**, Azure compute resources — VMs, App Services, Functions, and so on — can authenticate to the Language Service with no stored secret of any kind.

**Setting it up:**

1. **Turn on Managed Identity.** Enable either a System-Assigned or User-Assigned identity on whichever Azure resource is hosting the application.
2. **Grant access via RBAC.** Assign that identity the **"Cognitive Services User"** role (or an equivalent, more granular Language Service role) on the target resource.
3. **Swap in `DefaultAzureCredential`.** Update the client code to authenticate through `DefaultAzureCredential` from the `azure-identity` package instead of a static key.

**Example — keyless client setup:**

```python
from azure.identity import DefaultAzureCredential
from azure.ai.textanalytics import TextAnalyticsClient

# DefaultAzureCredential auto-detects the right auth source —
# a Managed Identity, Azure CLI login, VS Code session, or env config.
credential = DefaultAzureCredential()
endpoint = "https://lang-sentiment-learning.cognitiveservices.azure.com/"

# No key required anywhere in this setup
client = TextAnalyticsClient(endpoint=endpoint, credential=credential)
```

With this in place, there's no key sitting in a config file, an environment variable, or in memory — which also means no rotation schedule to maintain and one less way for a credential to leak.
