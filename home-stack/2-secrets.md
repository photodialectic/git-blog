# Secrets

Managing secrets in a Git-tracked monorepo presents a classic challenge: how do you store sensitive configuration without exposing it in version control? My solution uses [age](https://github.com/FiloSottile/age) for encryption, allowing me to commit encrypted secrets directly to the repository.

## Why age?

After evaluating several options including git-crypt and sops, I chose age for its simplicity and elegance:

- **Modern cryptography**: Uses X25519, ChaCha20Poly1305, and HMAC
- **Simple key management**: SSH keys work as encryption keys
- **Small footprint**: Single binary with minimal dependencies
- **Future-proof**: Active development with a clean design philosophy

The ability to use my existing SSH keys was the deciding factor - no additional key management infrastructure required.

## Setup and Configuration

My secrets workflow revolves around two files:

- `.env` - The plaintext environment variables (git-ignored)
- `.env.age` - The encrypted version (committed to git)

### Basic Commands

The core operations are straightforward:

``` bash
# Encrypt secrets for commit
curl -s https://github.com/photodialectic.keys | age -R - .env > .env.age

# Decrypt for local use
age -d -i ~/.ssh/github_rsa .env.age > .env
```

The encrypt command fetches my public key from GitHub (where it's automatically available) and pipes it to age for encryption. The decrypt command uses my private SSH key to recover the plaintext.

## CLI Integration

I've built secrets management into my HomeStack CLI tool to streamline the workflow:

``` go
// Encrypt with editor integration
nhdc secrets encrypt

// This command:
// 1. Opens .env in your $EDITOR
// 2. Encrypts the result to .env.age
// 3. Fetches public key from GitHub automatically

// Decrypt
nhdc secrets decrypt --identity ~/.ssh/github_rsa
```

The CLI automatically opens your editor when encrypting, making it easy to add or modify secrets before encryption. This integrated workflow reduces friction and makes secrets management feel natural.

## Docker Compose Integration

Services access decrypted environment variables through Docker Compose's `--env-file` flag:

``` yaml
services:
  chat-gpt:
    environment:
      - NODE_ENV=production
      - MYSQL_PWD=${MYSQL_PWD}
      - AUTH0_SECRET=${AUTH0_SECRET}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
```

## Security Benefits

This approach provides several security advantages:

- **Encrypted at rest**: Secrets in git are always encrypted
- **Audit trail**: Changes to secrets are tracked in git history (encrypted)
- **Key rotation**: Re-encrypt with new keys by updating GitHub keys
- **Access control**: Only holders of the private key can decrypt
- **Backup inherent**: Secrets are backed up with the git repository
