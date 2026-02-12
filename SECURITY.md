# Security Guidelines

## QNLLM v3.1 Security Guidelines

**Date**: February 2026  
**Version**: 1.0

---

## Overview

This document provides security guidelines for using QNLLM in research and development environments.

---

## 1. Installation & Environment Setup

### Safe Installation

```bash
# Use a virtual environment
python -m venv qnllm_env
source qnllm_env/bin/activate  # On Windows: qnllm_env\Scripts\activate

# Install from source
pip install -e .
```

### Dependency Verification

```bash
# Check installed packages
pip freeze > requirements_locked.txt

# Verify integrity
pip check
```

### Avoid
- Installing as root/administrator
- Using `--skip-installed` flag
- Installing from untrusted sources

---

## 2. Data Security

### Protecting Snapshots

QNLLM snapshots contain the learned state. Protect them:

```bash
# Set restrictive permissions
chmod 600 data/snapshot_*.qnllm
chmod 600 data/embeddings/*.bin
```

### Sensitive Data

If handling sensitive information:

1. Use encrypted filesystems
2. Enable full-disk encryption (BitLocker, LUKS)
3. Secure deletion of old snapshots
4. Audit access logs

### Backup Security

```bash
# Secure backup
tar czf - data/ | gpg --symmetric > snapshot_backup.tar.gz.gpg

# Verify backup (without decrypting)
gpg --list-only snapshot_backup.tar.gz.gpg
```

---

## 3. Credential Management

### Environment Variables

```bash
# Create .env for API credentials (never commit!)
cat > .env << EOF
NVIDIA_API_KEY=your_key_here
EOF

# Set permissions
chmod 600 .env
```

### .gitignore Configuration

```
# Ensure credentials are not committed
.env
.env.local
*.key
*.pem
credentials/
secrets/
```

### API Security

- Rotate API keys regularly
- Use minimal-scope tokens
- Monitor API usage
- Implement rate limiting

---

## 4. Code Security

### Dependencies

```bash
# Check for known vulnerabilities
pip install safety
safety check

# Update packages securely
pip list --outdated
pip install --upgrade package_name
```

### Input Validation

For production use:

```python
# Validate inputs
def safe_query(question: str) -> str:
    # Max length check
    if len(question) > 10000:
        raise ValueError("Query too long")
    
    # Character validation
    if not all(ord(c) < 128 for c in question):
        return "Non-ASCII characters not allowed"
    
    return engine.query(question)
```

### Code Review

- Review all changes before deployment
- Use version control (git) for tracking changes
- Enable branch protection on main

---

## 5. Deployment Security

### Development vs. Production

**Development (Unsafe)**
```bash
python -c "import qnllm; engine = qnllm.QNLLMEngine(debug=True)"
```

**Production (Safe)**
```bash
# Run in isolated environment
docker run --rm -v /data:/data qnllm:3.1
```

### Resource Limits

```bash
# Limit memory usage
ulimit -v 4000000  # 4GB max

# Limit CPU usage
taskset -c 0-3 python qnllm_chat.py  # Use 4 cores
```

### Monitoring

```bash
# Monitor running process
ps aux | grep qnllm
top -p $(pidof python)

# Check system resources
df -h  # Disk space
free -h  # Memory
```

---

## 6. Network Security (When Applicable)

### API Server Hardening

```python
# If exposing via FastAPI (future versions)
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["localhost"],  # Restrict origins
    allow_methods=["POST"],  # Only POST for queries
    allow_credentials=True,
)
```

### Firewalls

- White-list only necessary ports
- Disable unused network interfaces
- Use VPN for remote access
- Enable rate limiting

---

## 7. Audit & Logging

### Enable Audit Logs

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('qnllm_audit.log'),
        logging.StreamHandler()
    ]
)
```

### Log Rotation

```bash
# Use logrotate for log files
cat > /etc/logrotate.d/qnllm << EOF
/var/log/qnllm/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
}
EOF
```

### Monitor Logs

```bash
# Check for suspicious activity
grep -i error qnllm_audit.log
grep -i warning qnllm_audit.log
```

---

## 8. Access Control

### File Permissions

```bash
# Restrict access to configuration
chmod 640 configs/*.yaml
chown qnllm:qnllm configs/

# Restrict access to data
chmod 700 data/
```

### User Isolation

```bash
# Create dedicated user
useradd -m -s /bin/bash qnllm_user
su - qnllm_user
```

---

## 9. Regular Maintenance

### Security Updates

1. Subscribe to GitHub releases
2. Review release notes
3. Test updates in staging
4. Deploy to production

### Cleanup

```bash
# Remove temporary files
rm -f temp/*.tmp
rm -rf __pycache__ .pytest_cache

# Clear old logs
find logs -mtime +30 -delete
```

---

## 10. Incident Response

### If Compromised

1. **Isolate**: Disconnect from network
2. **Preserve**: Preserve evidence/logs
3. **Assess**: Determine scope of breach
4. **Notify**: Report to appropriate parties
5. **Remediate**: Fix vulnerabilities
6. **Review**: Post-mortem analysis

### Reporting Security Issues

For security vulnerabilities:

1. Do NOT report publicly
2. Email security contact
3. Include detailed description
4. Allow 90 days for fix
5. Coordinate disclosure

---

## Quick Security Checklist

- [ ] Using virtual environment
- [ ] .env file created and .gitignore configured
- [ ] File permissions set correctly (chmod 600 for sensitive files)
- [ ] Dependencies checked for vulnerabilities
- [ ] Input validation implemented
- [ ] Audit logging enabled
- [ ] Backups encrypted and tested
- [ ] Access controls configured
- [ ] Monitoring and alerting set up
- [ ] Incident response plan documented

---

## References

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE/SANS Top 25](https://cwe.mitre.org/top25/)
- [Python Security](https://docs.python.org/3/library/security_warnings.html)

---

**For security questions or concerns, contact the project maintainers.**
