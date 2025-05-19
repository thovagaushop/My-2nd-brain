
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

##### Step 3: Deploy runners scale sets
* Using Helm chart and cli set variable:
```
	

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