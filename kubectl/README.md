# GitHub Actions for Amazon EKS Authentication with kubectl

This Action for [Amazon Elastic Container Service for Kubernetes (Amazon EKS)](https://aws.amazon.com/eks/) that saves a [kubectl config](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) with AWS credentials and wraps the [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) command.

## Usage

A partial example workflow applying a configuration and verifying the deployment status.

```hcl
workflow "Build and Deploy" {
  on = "push"
  resolves = ["Verify EKS Deployment"]
}
action "Deploy to EKS" {
  needs = ["Build and Push Containers"]
  uses = "actions/aws/kubectl@master"
  args = ["apply -f config.yml"]
  secrets = ["KUBE_CONFIG_DATA"]
}

action "Verify EKS Deployment" {
  needs = "Deploy to EKS"
  uses = "actions/aws/kubectl@master"
  args = ["rollout status deployment/aws-example-octodex"]
  secrets = ["KUBE_CONFIG_DATA"]
}
```

### Secrets

- `KUBE_CONFIG_DATA` – **Required** A base64-encoded `kubectl config` file with credentials for Kubernetes to access the cluster. Example encoding from terminal : `cat $HOME/.kube/config | base64`. **Note** Do not use `kubectl config view` as this will hide the `certificate-authority-data`.


## License

The Dockerfile and associated scripts and documentation in this project are released under the [MIT License](LICENSE).

Container images built with this project include third party materials. See [THIRD_PARTY_NOTICE.md](THIRD_PARTY_NOTICE.md) for details.
