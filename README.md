# Secure Serverless API

Infrastructure-as-code for a secure serverless API on AWS: API Gateway, Cognito,
Lambda, DynamoDB, IAM, WAF, CloudWatch, and API throttling.

## Security

- No AWS credentials are ever stored in this repo. Configure the AWS CLI locally
  with `aws configure` (or a named profile) — credentials live in `~/.aws/`,
  outside the project.
- Local-only secrets go in `.env` (git-ignored) — copy `.env.example` to get started.
- A pre-commit hook (`.githooks/pre-commit`) runs [gitleaks](https://github.com/gitleaks/gitleaks)
  on every commit and blocks it if a likely secret is detected. It's enabled
  automatically via `core.hooksPath` in this repo's git config.
- IAM roles/policies defined here follow least-privilege — no `AdministratorAccess`
  or wildcard `*` actions on production resources.

## Status

`template.yaml` defines a working stack: Lambda + HTTP API Gateway (`GET /hello`
public, `POST /items` / `GET /items/{id}` backed by DynamoDB), Cognito
authentication on the `/items` routes, and stage-level API throttling. Manually
tested end to end — see [TESTING.md](TESTING.md) for what was verified and how.

Not yet added: WAF, CloudWatch alarms.
