
### Deploy ARC (Github runner) to k3s
Visit the [documents of Github actions about ARC](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/quickstart-for-actions-runner-controller): 

##### Step 1: Installing Actions runner controller:

* Using Helm chart
```
NAMESPACE="arc-systems"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

##### Step 2: Authenticating to github API
Authenticate ARC with a Github APP
> Note: Detail at [Authenticate with Github app](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/authenticating-to-the-github-api)

1. Craete a github app
2. Get API Id: 
3. Generate Private Key
4. Install the app with Organization and get installation_id
- Using kubectl
```
kubectl create secret generic secret_name \
   --namespace=arc-runners \
   --from-literal=github_app_id=123456 \
   --from-literal=github_app_installation_id=654321 \
   --from-literal=github_app_private_key='-----BEGIN RSA PRIVATE KEY-----********'
```

IceteaSoftware: 
APP_ID: 1269788
INSTALLATION_ID: 66693139
Private key: blc-main-runner.pem

kubectl create secret generic secret_name \
   --namespace=arc-runners \
   --from-literal=github_app_id=1269788 \
   --from-literal=github_app_installation_id=66693139 \
   --from-literal=github_app_private_key='********'
##### Step 3: Deploy runners scale sets
* Using Helm chart and cli set variable:
```
INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="https://github.com/<your_org>"
GITHUB_PAT="<PAT>" OR <the secret_name in the Authenticate step>
helm install "${INSTALLATION_NAME}" \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

* Using values.yaml file
```
githubConfigUrl: "https://github.com/txvitdev-org"

githubConfigSecret: pre-defined-secret

minRunners: 1

maxRunners: 5

template:

  spec:

    initContainers:

    - name: init-dind-externals

      image: ghcr.io/txvitdev/its-action-runner:latest

      command: ["cp", "-r", "/home/runner/externals/.", "/home/runner/tmpDir/"]

      volumeMounts:

        - name: dind-externals

          mountPath: /home/runner/tmpDir

    containers:

    - name: runner

      image: ghcr.io/txvitdev/its-action-runner:latest

      command: ["/home/runner/run.sh"]

      env: 

        - name: DOCKER_HOST

          value: unix:///var/run/docker.sock

      volumeMounts:

        - name: setup-ruby

          mountPath: /opt/hostedtoolcache

        - name: work

          mountPath: /home/runner/_work

        - name: dind-sock

          mountPath: /var/run

  

    - name: dind

      image: docker:dind

      args:

        - dockerd

        - --host=unix:///var/run/docker.sock

        - --group=$(DOCKER_GROUP_GID)

      env:

        - name: DOCKER_GROUP_GID

          value: "123"

      securityContext:

        privileged: true

      volumeMounts:

        - name: work

          mountPath: /home/runner/_work

        - name: dind-sock

          mountPath: /var/run

        - name: dind-externals

          mountPath: /home/runner/externals

  

    volumes:

    - name: setup-ruby

      emptyDir: {}

    - name: work

      emptyDir: {}

    - name: dind-sock

      emptyDir: {}

    - name: dind-externals

      emptyDir: {}
```
The above values.yaml file describe that:
- The runner pod will listen job from your org
- Secret to authenticate
- Auto scaling from 1 to 5 instance
- Using image of txvitdev: Have docker install and install GCC for runby setup
The Dockerfile of ghcr.io/txvitdev/its-action-runner :

```
FROM ghcr.io/actions/actions-runner:latest

USER root

RUN apt-get update && apt-get install -y \

      unzip git curl autoconf bison build-essential \

      libssl-dev libyaml-dev libreadline-dev \

      zlib1g-dev libncurses5-dev libffi-dev libgdbm-dev && rm -rf /var/lib/apt/lists/*

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip" \

  && unzip /tmp/awscliv2.zip -d /tmp \

  && /tmp/aws/install \

  && rm -rf /tmp/aws /tmp/awscliv2.zip

USER runner

```

### Using OIDC and Aws Role to setup credential of AWS

Explain: 
1. **Create identity providers**
	Go to IAM and create Identity Provider
	- Provider URL: https://token.actions.githubusercontent.com
	- Audience: sts.amazonaws.com
2. **Setup Assume Roles**
	Go to IAM and create new role
	- Choose Web identity
	- Choose identity provider: token.actions.githubusercontent.com
	- Input Github organization url: your_org_url_name
3. 