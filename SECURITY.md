# Security / No-Secrets Policy

This repo is public. DO NOT commit:
- API keys, tokens, passwords
- AWS credentials (~/.aws), kubeconfig (~/.kube/config)
- .env files (any variant)
- SSH private keys (id_rsa, id_ed25519, *.pem, *.key)

## Before every push (checklist)
- git status
- git diff
- grep for secrets if unsure: "token", "password", "secret", "AKIA"

If something sensitive was committed by mistake: rotate/revoke it immediately and rewrite history.
