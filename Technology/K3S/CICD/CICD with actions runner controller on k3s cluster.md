
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

kubectl create secret generic pre-defined-secret \
   --namespace=arc-runners \
   --from-literal=github_app_id=1269788 \
   --from-literal=github_app_installation_id=66693139 \
   --from-literal=github_app_private_key='-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA03JZ+EQVVlFFoVkk7nmAAH5xliaQ0FI8zKW5E2M1MU7G+uUo
FZkMYUPNjlP7HW3X5HDMSjTiZRl6VCaEvVeH+y+mtJVM2qINWfGJQYfbWArkclJu
tY0HQ9qZYEUp5k0siE1GhlO2pa/k7mduvLVKwTexyQvtBJPElrjKjWghr5BeQKgq
PjfKxa0MfaQbBRce5TDvZGwiZ420OITXZhI2oZD2yRbc1v+eJx8SzzKiC9YOKB5c
gnoBNH7I80SCwowZK608i9pzviCAB6MazvPhJy01fA7/CaD7T7/ZvWQN6NoqK6j9
nqpS/MrCmWqNXzPeb4J7VKUz0zXZBkGInemhTwIDAQABAoIBABLdVeePc3PjBlpR
0LiyAFiK72ldAXiEjcYYVv3C0SEYDSqfPUdIA1G+Md2r4nCKt0M7SQ6pzzUD4+UU
Fq2p3IjthGsCEvxCjvxiT0OYclpVhnIoppXuAiLsothy8z0Hz5xNgKhFWbtIiR03
/yo45nV8mZD9TmUlJdnonLGnA7Jc/l6nbOS0hA/ijEIk/RRbpqBAW2UP9m5iguE7
IQK2/GC9OjL54731JG8tTYUaF29oAwnNPHNvNWr5WhL0P+X/IYc4dkiamrVXaO0i
7b5SExVXQvupKF2EnT+Vj7mH9HV6rW+gYsEx68rWWm1MGJk8llpwV0AMnueylFO9
24dywaECgYEA+UNw7dh6vGNv0Kre+HfoLEYpVtMmGoQo8A/6s8w0iuskVg1tp02r
8uB85jzkPbbgfRoIjleUui08Wn15RjQF4CbwIiZBnSNfiygf3c3NIvdYa/5WOWRH
N1ut87BE8SbH1z3mjBmdl3iSdDbfFy+3yCpogktc7rbKHmUHEV5jfJ8CgYEA2SlF
RC9iveSODROO6/BR+ZZGFwnhZUMxknnM/D8w8TVeceBNtvdQrLTvSht+7IT4Qv9z
GJ0CZlNG8GUkYq5addyXCJMBSotsqYrMcu81ThV1acjY85avJx0PK6mSnNk6FS5k
KWIrNMM3roEVHtLU2nYFRujNBb4tpLljeo/w7VECgYEA8shOgUu+WCneKfeUT5yy
5hS8hRYKYf9hxFk8Dc4TS0+2x54ytKcBmQIwhSy//qBWTWOC++mwMhqHU3gtHETl
iCtE724lsIFYuTiuuSKP8MPMOvuyThovB2tjphyFOgFU2oAvQzxb88H7m/gqGPJg
ZjVwL6Bp9xTRDwPF+5PdAO8CgYEAhK2zdUJiWNTQeWrOspaE7zICJsdRn9Xa7rxe
ImvVUjoiNv8tXDFkZ/CwFp4QASAIsp5ySsJ7Gdudhvi0r1oJSON6n7F5Y3sl67wy
x7Ig5lE0CGq/KqyQ3RvjDfEv84bA9vn+Fk61SSpQ1dxl9AwqIkNjW/yWbwDP0Em4
XsSdFdECgYB1lS4ukeG69KLaN0ahQh2vVLrD2PuPn+ZB4JVE32jeYfwYOoOP5SUn
oZ0Cl2lPele/q9HyywSIQr/FlZum5UojdJ8tol7NDlrL5LudLPczUaY8RpDHjflG
1+qIwRVOf0eykvLL2pY+U+tfP4X9/nSdXN+3isjA4ftzdnolWjniNQ==
-----END RSA PRIVATE KEY-----'
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