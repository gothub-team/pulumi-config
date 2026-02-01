# pulumi-config

CLI tool to manage Pulumi stack configs (`Pulumi.<stack>.yaml`) in AWS SSM Parameter Store, so they survive `.gitignore` and machine resets.

## Install

```sh
# clone and add to PATH
git clone https://github.com/gothub-team/pulumi-config.git
export PATH="$PATH:$(pwd)/pulumi-config/bin"

# optional: add a shorter alias
alias puc='pulumi-config'
```

## SSM parameter layout

```
/pulumi/{repo}/{stack}/config     SecretString   full Pulumi.<stack>.yaml content
/pulumi/{repo}/backend-bucket     String         S3 bucket for Pulumi state
/pulumi/{repo}/passphrase         SecretString   PULUMI_CONFIG_PASSPHRASE
```

## Quick start

### New project setup

```sh
pulumi-config init-backend <repo>                     # create S3 bucket + passphrase
eval $(pulumi-config env <repo>)                      # export backend URL + passphrase
pulumi-config push <repo> <stack> Pulumi.<stack>.yaml  # back up stack config
```

### Restore on a fresh machine

```sh
git clone <repo-url> && cd <repo>
pulumi-config pull <repo> prod infra/Pulumi.prod.yaml
eval $(pulumi-config env <repo>)
cd infra && npm install && pulumi up
```

## Commands

| Command | Description |
|---|---|
| `list [--repo name]` | List all stacks, or filter by repo |
| `get <repo> <stack>` | Print config to stdout |
| `push <repo> <stack> [file]` | Upload config (file or stdin) |
| `pull <repo> <stack> [file]` | Download config (to file or stdout) |
| `delete <repo> <stack>` | Remove config from SSM |
| `init-backend <repo>` | Create S3 bucket + passphrase |
| `env <repo>` | Print shell export statements |

## Requirements

- AWS CLI configured with appropriate IAM permissions
- `ssm:GetParameter`, `ssm:GetParametersByPath`, `ssm:PutParameter`, `ssm:DeleteParameter`
- `s3:CreateBucket`, `s3:PutBucketVersioning`, `s3:PutPublicAccessBlock` (for `init-backend`)
