# pulumi-config

CLI tool to manage Pulumi stack configs (`Pulumi.<stack>.yaml`) in AWS SSM Parameter Store, so they survive gitignore and machine resets.

## SSM parameter layout

```
/pulumi/{repo-name}/{stack-name}/config   SecretString   full Pulumi.<stack>.yaml content
/pulumi/{repo-name}/backend-bucket        String         S3 bucket for Pulumi state
/pulumi/{repo-name}/passphrase            SecretString   PULUMI_CONFIG_PASSPHRASE
```

## Setup for a new project

1. `pulumi-config init-backend` — creates the S3 state bucket, stores bucket name and passphrase in SSM
2. `eval $(pulumi-config env)` — exports `PULUMI_BACKEND_URL` and `PULUMI_CONFIG_PASSPHRASE`
3. `pulumi-config push <stack>` — backs up `Pulumi.<stack>.yaml` to SSM
4. `pulumi-config pull <stack>` — restores `Pulumi.<stack>.yaml` from SSM

Repo is auto-detected from the current git folder. Use `--repo <name>` to override on any command.

## Restoring a full project from scratch

```sh
git clone <repo-url>
cd <repo>
pulumi-config pull prod
eval $(pulumi-config env)
cd infra && npm install && pulumi up
```

## Commands

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
- `s3:CreateBucket`, `s3:PutBucketVersioning`, `s3:PutPublicAccessBlock` (for init-backend)
