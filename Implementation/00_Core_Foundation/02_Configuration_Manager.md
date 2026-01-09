# Core Foundation 02: Configuration & Secrets Management

> *"Configuration is the DNA of the application. It must be valid, or the organism should not be born."*

## 1. The Requirement
Hardcoding paths (`C:/Vault`) or API keys is a security risk and an operational nightmare.
*   **The Problem:** "It works on my machine" bugs.
*   **The Solution:** Environment Variables + Strict Validation.
*   **The Tool:** `pydantic-settings`. It forces type-checking on your configuration.

---

## 2. Technical Specification

### A. The `.env` File Protocol
We do not use system-level variables. We use a local `.env` file that is **strictly gitignored**.

**Example `.env`:**
```ini
ENV_STATE="production"
VAULT_PATH="./Vault"
LOG_LEVEL="INFO"
ANTHROPIC_API_KEY="sk-ant-..."
GMAIL_CREDENTIALS_PATH="./Vault/System/credentials.json"
MAX_RETRIES=3
```

### B. The `config.py` Module
*This file should be placed in `AI_Employee/src/shared/config.py`.*

**Dependencies:**
```bash
uv add pydantic pydantic-settings
```

**The Code Blueprint:**
```python
from pathlib import Path
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import DirectoryPath, FilePath, SecretStr, Field

class GlobalConfig(BaseSettings):
    # 1. Core Identity
    ENV_STATE: str = Field(default="dev", pattern="^(dev|prod)$")
    
    # 2. File System (Strict Type Checking)
    # DirectoryPath ensures the folder actually exists on startup
    VAULT_PATH: DirectoryPath = Field(default=Path("./Vault"))
    
    # 3. Security (SecretStr hides value in logs)
    ANTHROPIC_API_KEY: SecretStr
    GMAIL_CREDENTIALS_PATH: FilePath | None = None
    
    # 4. Operational Limits
    MAX_RETRIES: int = Field(default=3, ge=1, le=10)
    POLL_INTERVAL_SECONDS: int = Field(default=60, ge=5)

    # Load from .env
    model_config = SettingsConfigDict(
        env_file=".env", 
        env_file_encoding="utf-8",
        extra="ignore"
    )

# Singleton Instance
try:
    settings = GlobalConfig()
except Exception as e:
    print(f"❌ CONFIGURATION ERROR: {e}")
    exit(1) # Fail Fast
```

---

## 3. Usage Pattern
Every script imports the `settings` object. It guarantees that if the script is running, the configuration is valid.

```python
from src.shared.config import settings
from src.shared.logger import setup_logger

logger = setup_logger("bootstrap")

def main():
    # Access without fear of KeyErrors
    logger.info(f"Starting Agent in {settings.ENV_STATE} mode")
    
    # Use the Secret
    api_key = settings.ANTHROPIC_API_KEY.get_secret_value()
    
    # Use the Path (It is already a Path object)
    inbox = settings.VAULT_PATH / "00_Inbox"
    
    logger.info(f"Watching directory: {inbox}")

if __name__ == "__main__":
    main()
```

---

## 4. Why this is Enterprise-Grade
1.  **Fail Fast:** If you forget to create the `Vault` folder, `DirectoryPath` validation will throw an error immediately on startup, preventing runtime crashes later.
2.  **Secret Safety:** `SecretStr` ensures that if you accidentally `print(settings)`, the API key will show as `**********`, not the actual key.
3.  **Type Safety:** You know that `MAX_RETRIES` is always an integer, never a string "3".
