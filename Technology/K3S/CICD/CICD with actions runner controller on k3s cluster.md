
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