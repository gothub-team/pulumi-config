# pulumi-config

CLI tool to manage Pulumi stack configs (`Pulumi.<stack>.yaml`) in AWS SSM Parameter Store, so they survive gitignore and machine resets.

## SSM parameter layout

```
/pulumi/{repo-name}/{stack-name}/config   SecretString   full Pulumi.<stack>.yaml content
/pulumi/{repo-name}/backend-bucket        String         S3 bucket for Pulumi state
/pulumi/{repo-name}/passphrase            SecretString   PULUMI_CONFIG_PASSPHRASE
```

## Setup for a new project

1. `pulumi-config init-backend <repo>` — creates the S3 state bucket, stores bucket name and passphrase in SSM
2. `eval $(pulumi-config env <repo>)` — exports `PULUMI_BACKEND_URL` and `PULUMI_CONFIG_PASSPHRASE`
3. `pulumi-config push <repo> <stack> Pulumi.<stack>.yaml` — backs up stack config
4. `pulumi-config pull <repo> <stack> Pulumi.<stack>.yaml` — restores it on a fresh machine

## Restoring a full project from scratch

```sh
git clone <repo-url>
cd <repo>
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
- `s3:CreateBucket`, `s3:PutBucketVersioning`, `s3:PutPublicAccessBlock` (for init-backend)
