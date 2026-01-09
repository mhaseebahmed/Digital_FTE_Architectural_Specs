# Core Foundation 03: Error Handling & Resilience Protocol

> *"Failures are inevitable. Crashing is optional."*

## 1. The Requirement
An autonomous agent encounters three types of errors:
1.  **Transient:** Network glitch, API timeout. (Action: Retry).
2.  **Permanent:** Invalid File, 401 Unauthorized. (Action: Alert & Skip).
3.  **Fatal:** Disk full, Memory error. (Action: Crash Safely).

We need a standardized way to handle these without writing `try/except` loops in every single function.

---

## 2. Technical Specification

### A. The Custom Exceptions
Define the vocabulary of failure.

*File: `src/shared/exceptions.py`*
```python
class DigitalFTEError(Exception):
    """Base class for all agent errors."""
    pass

class TransientError(DigitalFTEError):
    """Retryable errors (Network, timeouts)."""
    pass

class ConfigurationError(DigitalFTEError):
    """Fatal setup errors."""
    pass

class SafetyViolation(DigitalFTEError):
    """Agent tried to do something forbidden."""
    pass
```

### B. The Retry Decorator
A reusable tool to make any function resilient.

*File: `src/shared/resilience.py`*
```python
import time
import logging
import functools
from .exceptions import TransientError

logger = logging.getLogger("resilience")

def with_retry(max_attempts: int = 3, base_delay: float = 1.0, backoff_factor: float = 2.0):
    """
    Decorator that retries a function if it raises a TransientError.
    Uses Exponential Backoff: 1s, 2s, 4s...
    """
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            attempt = 1
            delay = base_delay
            
            while attempt <= max_attempts:
                try:
                    return func(*args, **kwargs)
                
                except TransientError as e:
                    if attempt == max_attempts:
                        logger.error(f"❌ Final Failure in {func.__name__}: {e}")
                        raise e # Re-raise after max retries
                    
                    logger.warning(f"⚠️ Attempt {attempt} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
                    
                    attempt += 1
                    delay *= backoff_factor
                
                except Exception as e:
                    # Non-Transient errors crash immediately
                    logger.critical(f"💥 Unhandled Exception in {func.__name__}: {e}")
                    raise e
                    
        return wrapper
    return decorator
```

---

## 3. Usage Pattern

```python
from src.shared.resilience import with_retry
from src.shared.exceptions import TransientError

@with_retry(max_attempts=5)
def call_claude_api(prompt):
    # Simulate Logic
    response = requests.post(...)
    
    if response.status_code == 503:
        # We explicitly raise TransientError to trigger the retry
        raise TransientError("Claude API is overloaded")
    
    if response.status_code == 401:
        # This will NOT retry, it will crash (as it should)
        raise ValueError("Invalid API Key")
        
    return response.json()
```

---

## 4. Why this is Enterprise-Grade
1.  **Exponential Backoff:** We don't hammer the API. We wait 1s, then 2s, then 4s. This is "Good Citizenship" in distributed systems.
2.  **Semantic Errors:** We distinguish between "The server is busy" (Retry) and "Your password is wrong" (Don't Retry).
3.  **Code Cleanliness:** The business logic (`call_claude_api`) is clean. The messy retry logic is hidden in the decorator.
