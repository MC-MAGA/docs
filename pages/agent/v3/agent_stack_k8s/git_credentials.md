# Git credentials

Similarly to a standalone [Buildkite Agent installation](/docs/agent/v3/installation), to access and clone private Git repositories, Git credentials must be made available for the Agent to use. These credentials can be in the form of an SSH key for cloning over `ssh://`, or using a `.git-credentials` file for cloning over `https://`.

## Cloning repositories using SSH keys

To use SSH to clone your private Git repositories, you'll need to create a Kubernetes Secret containing an authorized SSH private key and configure the Buildkite Agent Stack for Kubernetes to mount this Secret into the `checkout` container that is used to perform the Git repository cloning.

### Create a Kubernetes Secret using an SSH private key

> 🚧 Warning!
> Support for DSA keys was removed from OpenSSH in early 2025. This removal affects `buildkite/agent` version `v3.88.0` and newer. Please migrate to `RSA`, `ECDSA`, or `EDDSA` keys.

After creating an SSH key pair and registering its public key with the remote repository provider (for example, [GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)), you can create a Kubernetes Secret using the SSH private key file.

Ensure that the environment variable name matches the recognized names (`SSH_PRIVATE_*_KEY`) in [`docker-ssh-env-config`](https://github.com/buildkite/docker-ssh-env-config).

- `SSH_PRIVATE_ECDSA_KEY`
- `SSH_PRIVATE_ED25519_KEY`
- `SSH_PRIVATE_RSA_KEY`

A script from this project is included in the default entry point of the default [`buildkite/agent`](https://hub.docker.com/r/buildkite/agent) Docker image. It will process the value of the Kubernetes Secret and write out a private key to the `~/.ssh` directory of the checkout container.

To create a Kubernetes Secret named `my-git-ssh-credentials` containing the contents of the SSH private key file `$HOME/.ssh/id_rsa`:

```bash
kubectl create secret generic my-git-ssh-credentials --from-file=SSH_PRIVATE_RSA_KEY="$HOME/.ssh/id_rsa" -n buildkite
```

This Kubernetes Secret can be referenced by the Buildkite Agent Stack for Kubernetes controller using [EnvFrom](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#configure-all-key-value-pairs-in-a-secret-as-container-environment-variables) from within the controller configuration or via the `gitEnvFrom` config of the `kubernetes` plugin.

### Provide a Kubernetes Secret through a configuration file

Using [`pod-spec-patch`](/docs/agent/v3/agent-stack-k8s/container-resource-limits#using-the-podspec-patch-in-the-controller-values-yaml-configuration-file), you can specify the Kubernetes Secret containing your SSH private key in your configuration values YAML file using `envFrom`:

```yaml
# values.yaml
...
config:
  ...
  pod-spec-patch:
    containers:
    - name: checkout                    # <---- this is needed for the secret to only be mounted on the checkout container
      envFrom:
      - secretRef:
          name: my-git-ssh-credentials  # <---- this is an example name of the Kubernetes Secret
```

### Provide a Kubernetes Secret using the Kubernetes plugin

Under the `kubernetes` plugin, specify the name of the Kubernetes Secret via the `gitEnvFrom` config:

```yaml
# pipeline.yaml
steps:
- label: "\:kubernetes\: Hello World!"
  command: echo Hello World!
  agents:
    queue: kubernetes
  plugins:
  - kubernetes:
      gitEnvFrom:
      - secretRef:
          name: my-git-ssh-credentials  # <---- this is an example name of the Kubernetes Secret
```

> 📘
> If you are using the `kubernetes` plugin to provide the Kubernetes Secret's SSH private key, you need to _define this configuration for every step_ that requires access to the private Git repository.
> If you are defining the Kubernetes Secret using the Buildkite Agent Stack for Kubernetes controller configuration, you'll only needs to configure it once.

### Provide an SSH private key to non-checkout containers

The configurations above only provide the SSH private key as a Kubernetes Secret to the `checkout` container. If Git SSH credentials are required in user-defined job containers, you have these options instead:

- Use a container image based on the default `buildkite/agent` Docker image, which preserves the default entry point by not overriding the command in the job spec.
- Include or reproduce the functionality of the [`ssh-env-config.sh`](https://github.com/buildkite/docker-ssh-env-config/blob/-/ssh-env-config.sh) script in the entry point for your job container image to source from recognized environment variable names.

The following example shows how to set up an SSH private key in `container-0`.

```yaml
# values.yaml
...
config:
  ...
  pod-spec-patch:
    containers:
    ...
    - name: container-0
      env:
        - name: SSH_PRIVATE_RSA_KEY
          valueFrom:
            secretKeyRef:
              name: my-git-ssh-credentials # <---- this is the name of the Kubernetes Secret to use for container-0 (setup on the same way as the command above)
              key: SSH_PRIVATE_RSA_KEY

```

After setting the SSH key to use on `container-0`, you need to set up `container-0` for `git` operations. The recommended approach is to implement a `pre-command` hook to run `ssh-keyscan` and generate the `known_hosts` files.

```bash
#!/bin/bash
set -eufo pipefail

eval $(ssh-agent -s)
mkdir -p ~/.ssh
chmod 700 ~/.ssh

touch ~/.ssh/config
chmod 600 ~/.ssh/config
touch ~/.ssh/known_hosts
chmod 600 ~/.ssh/known_hosts

ssh-keyscan github.com >> ~/.ssh/known_hosts
echo "$SSH_PRIVATE_RSA_KEY" | tr -d '\r' | ssh-add -
```

In your pipeline YAML, you can now add git operations such as `git clone` in the command step.

```yaml
# pipeline.yaml
steps:
  - label: "\:kubernetes\: Hello World!"
    command:  git clone -v git@github.com:{repo_name}.git
    plugins:
      - kubernetes:
          podSpec:
            containers:
                - image: buildkite/agent:alpine-k8s
```

## Cloning repositories using Git credentials

To use HTTPS to clone private Git repositories, you can use a `.git-credentials` file stored in a secret, and refer to this secret using the `gitCredentialsSecret` checkout parameter.

### Create a Kubernetes Secret from a Git credentials file

Create a `.git-credentials` file formatted in the manner expected by the `store` [Git credential helper](https://git-scm.com/docs/git-credential-store). After this file has been created, you can create a Kubernetes Secret containing the contents of this file:

```bash
kubectl create secret generic my-git-https-credentials --from-file='.git-credentials'="$HOME/.git-credentials" -n buildkite
```

### Provide a Kubernetes Secret through a configuration file

Using `default-checkout-params`, you can define your Kubernetes Secret as follows through the configuration yaml file:

```yaml
# values.yaml
...
config:
  ...
  default-checkout-params:
    gitCredentialsSecret:
      secretName: my-git-https-credentials
```

If you wish to use a different key within the Kubernetes Secret than `.git-credentials`, you can [project it](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#project-secret-keys-to-specific-file-paths)
to `.git-credentials` by using `items` within `gitCredentialsSecret`.

```yaml
# values.yaml
...
default-checkout-params:
  gitCredentialsSecret:
    secretName: my-git-https-credentials
    items:
    - key: funky-creds
      path: .git-credentials
```

### Provide a Kubernetes Secret using the Kubernetes plugin

Under the `kubernetes` plugin, specify the name of the Kubernetes Secret using the `checkout.gitCredentialsSecret` config:

```yaml
# pipeline.yaml
steps:
- label: "\:kubernetes\: Hello World!"
  command: echo Hello World!
  agents:
    queue: kubernetes
  plugins:
  - kubernetes:
      checkout:
        gitCredentialsSecret:
          secretName: my-git-https-credentials  # <---- this is the name of the Kubernetes Secret (ex. my-git-https-credentials, from command above)
```

### Provide Git credentials to non-checkout containers

The configurations above only provide Git credentials as a Kubernetes Secret to the `checkout` container. If the `.git-credentials` file is required in user-defined job containers, you can add a volume mount for the `git-credentials` volume, and configure Git to use the file within it (for example, with `git config credential.helper 'store --file ...'`).

## Skipping checkout (v0.13.0 and later)

> 📘 Version requirement
> To implement the following configuration options, `v0.13.0` or newer of the Agent Stack for Kubernetes controller is required.

For some steps, you may wish to avoid checkout (cloning a source repository). This can be done with the `checkout` block under the `kubernetes` plugin:

```yaml
steps:
- label: "\:kubernetes\: Hello World!"
  agents:
    queue: kubernetes
  plugins:
  - kubernetes:
      checkout:
        skip: true # prevents scheduling the checkout container
```

## Using default-checkout-params

Using `default-checkout-params`, `envFrom` can be added to all checkout, command, and sidecar containers separately, either per-step in the pipeline or for all jobs in `values.yaml`.

Pipeline example (note that the blocks are `checkout`, `commandParams`, and `sidecarParams`):

```yaml
# pipeline.yml
...
  kubernetes:
    checkout:
      envFrom:
      - prefix: GITHUB_
        secretRef:
          name: github-secrets
    commandParams:
      interposer: vector
      envFrom:
      - prefix: DEPLOY_
        secretRef:
          name: deploy-secrets
    sidecarParams:
      envFrom:
      - prefix: LOGGING_
        configMapRef:
          name: logging-config
```

An example of how this would be done using a `values.yml` configuration file:

```yaml
# values.yml
config:
  default-checkout-params:
    envFrom:
    - prefix: GITHUB_
      secretRef:
        name: github-secrets
  default-command-params:
    interposer: vector
    envFrom:
    - prefix: DEPLOY_
      secretRef:
        name: deploy-secrets
  default-sidecar-params:
    envFrom:
    - prefix: LOGGING_
      configMapRef:
        name: logging-config
```
