# AWS Bedrock

Technical learnings about AWS Bedrock setup via Teleport.

## Teleport Setup

### Prerequisites

1. Teleport CLI (`tsh`) installed
2. Access to N26 Teleport cluster
3. `bedrock-developer-user` role assignment

### Login Flow

```bash
# 1. Login to Teleport cluster
tsh login --proxy=teleport.access26.de:443 access26

# 2. Login to AWS app with Bedrock role
tsh apps login n26-dev-eu --aws-role bedrock-developer-user

# 3. Start proxy (keep running in background)
tsh proxy aws --app n26-dev-eu
```

The proxy outputs environment variables:
- `AWS_ACCESS_KEY_ID` - Temporary credentials
- `AWS_SECRET_ACCESS_KEY` - Temporary credentials
- `AWS_CA_BUNDLE` - CA certificate path
- `HTTPS_PROXY` - Local proxy URL

### Automated Setup

Use `bedrock-setup.sh` to automate:

```bash
./bedrock-setup.sh
# Creates tests/.env with all credentials
# Keeps proxy running - don't close terminal
```

## Inference Profiles

AWS Bedrock uses inference profiles (ARN-based) instead of model names.

### Profile ARNs

```
# Claude via Bedrock
arn:aws:bedrock:eu-central-1:538639307912:application-inference-profile/k375tnm6nr0t

# Titan Embed V2
arn:aws:bedrock:eu-central-1:538639307912:application-inference-profile/sbrfo5s0t4ev
```

### Provider Detection Issue

langchaingo cannot auto-detect model provider from ARN. Must specify:

```go
// For LLM
bedrock.New(
    bedrock.WithModel("arn:aws:bedrock:..."),
    bedrock.WithModelProvider("anthropic"), // Required!
)

// For embeddings
bedrockembed.NewBedrock(
    bedrockembed.WithModel("arn:aws:bedrock:..."),
    // Bedrock embeddings package auto-detects better
)
```

## Environment Variables

AWS SDK automatically uses:

```bash
AWS_ACCESS_KEY_ID=...        # From Teleport proxy
AWS_SECRET_ACCESS_KEY=...    # From Teleport proxy
AWS_CA_BUNDLE=...            # CA cert for Teleport
HTTPS_PROXY=http://...       # Local Teleport proxy
AWS_REGION=eu-central-1      # Must set explicitly
```

## Credential Expiry

Teleport credentials expire after ~1 hour. Signs of expiry:
- `ExpiredTokenException`
- `AccessDenied` errors
- Connection timeouts

**Fix**: Re-run `./bedrock-setup.sh` to refresh.

## Troubleshooting

### "Unable to locate credentials"

```bash
# Verify proxy is running
ps aux | grep "tsh proxy"

# Check env vars are set
env | grep AWS_
```

### "AccessDenied"

```bash
# Re-login and restart proxy
tsh apps login n26-dev-eu --aws-role bedrock-developer-user
tsh proxy aws --app n26-dev-eu
```

### TLS/Certificate Errors

Ensure `AWS_CA_BUNDLE` points to valid certificate:

```bash
ls -la $AWS_CA_BUNDLE
# Should exist and be readable
```

## Cost Monitoring

Track token usage for cost estimation:

```go
// Input/output tokens from response
info := response.Choices[0].GenerationInfo
inputTokens := info["InputTokens"]
outputTokens := info["OutputTokens"]
```

Claude via Bedrock pricing (approximate):
- Input: $0.003 / 1K tokens
- Output: $0.015 / 1K tokens
