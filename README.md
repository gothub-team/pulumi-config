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
cd <repo>
pulumi-config init-backend              # create S3 bucket + passphrase
eval $(pulumi-config env)               # export backend URL + passphrase
pulumi-config push <stack>                     # back up Pulumi.<stack>.yaml
```

### Restore on a fresh machine

```sh
git clone <repo-url> && cd <repo>
pulumi-config pull prod
eval $(pulumi-config env)
cd infra && npm install && pulumi up
```

## Commands

Repo is auto-detected from the current git folder. Use `--repo <name>` to override.

| Command | Description |
|---|---|
| `list [--all] [--repo name]` | List stacks for current repo (or all with `--all`) |
| `get <stack>` | Print config to stdout |
| `push <stack>` | Push `Pulumi.<stack>.yaml` to SSM |
| `pull <stack>` | Pull config to `Pulumi.<stack>.yaml` |
| `delete <stack>` | Remove config from SSM |
| `init-backend [--bucket name] [--passphrase val]` | Create or register S3 bucket + passphrase |
| `env` | Print shell export statements |

## Requirements

- AWS CLI configured with appropriate IAM permissions
- `ssm:GetParameter`, `ssm:GetParametersByPath`, `ssm:PutParameter`, `ssm:DeleteParameter`
- `s3:CreateBucket`, `s3:PutBucketVersioning`, `s3:PutPublicAccessBlock` (for `init-backend`)
