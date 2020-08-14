# GitHub Actions for AWS

This Action for [AWS](https://aws.amazon.com/) enables arbitrary actions for interacting with AWS services via [the `aws` command-line client](https://docs.aws.amazon.com/cli/index.html).

## Usage

An example workflow for creating and publishing to Simple Notification Service ([SNS](https://aws.amazon.com/sns/)) topic follows.

The example illustrates a pattern for consuming a previous action's output using [`jq`](https://stedolan.github.io/jq/), made possible since each `aws` Action's output is captured by default as JSON in `$GITHUB_HOME/$GITHUB_ACTION.json`:

```hcl
workflow "Publish to SNS topic" {
  on = "push"
  resolves = ["Publish"]
}

action "Topic" {
  uses = "actions/aws/cli@master"
  args = "sns create-topic --name my-topic"
  secrets = ["AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY"]
}

action "Publish" {
  needs = ["Topic"]
  uses = "actions/aws/cli@master"
  args = "sns publish --topic-arn `jq .TopicArn /github/home/Topic.json --raw-output` --subject \"[$GITHUB_REPOSITORY] Code was pushed to $GITHUB_REF\" --message file://$GITHUB_EVENT_PATH"
  secrets = ["AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY"]
}
```

### Secrets

- `AWS_ACCESS_KEY_ID` – **Required** The AWS access key part of your credentials ([more info](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys))
- `AWS_SECRET_ACCESS_KEY` – **Required** The AWS secret access key part of your credentials ([more info](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys))

### Environment variables

All environment variables listed in [the official documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html) are supported.

The most common environment variables are:

- `AWS_DEFAULT_REGION`- **Optional** The AWS region name, defaults to `us-east-1` ([more info](https://docs.aws.amazon.com/general/latest/gr/rande.html))
- `AWS_DEFAULT_OUTPUT`- **Optional** The CLI's output output format, defaults to `json` ([more info](https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html))
- `AWS_CONFIG_FILE` - **Optional** A path to an AWS config file, combined with [`$GITHUB_WORKSPACE`](https://developer.github.com/actions/creating-github-actions/accessing-the-runtime-environment/#environment-variables) this can reference files within the repo. For example `${GITHUB_WORKSPACE}/.aws/config` will use the file at `.aws/config` within the repository as the AWS config file. **Do not commit .aws/credentials to your repo**, keep your secrets in the secrets api. No default is provided ([more info](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html))


## License

The Dockerfile and associated scripts and documentation in this project are released under the [MIT License](LICENSE).

Container images built with this project include third party materials. See [THIRD_PARTY_NOTICE.md](THIRD_PARTY_NOTICE.md) for details.
