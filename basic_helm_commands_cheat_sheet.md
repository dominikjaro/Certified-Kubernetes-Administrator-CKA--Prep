Helm is basically "apt-get" or "yum" for Kubernetes. Instead of writing 10 different YAML files (Deployment, Service, ConfigMap, etc.) manually, you install one "Chart," and Helm creates them all for you.

## 1. The Setup (Repositories)

`helm repo add <repo_name> <repo_url>`
    - Use: Links your cluster to a library of charts.

`helm repo update`
    - Use: Updates your local list of charts from the repositories. Always run this after adding a repo.

`helm search repo <keyword>`
    - Use: Finds the exact name of the chart you want (e.g., bitnami/nginx).

---

## 2. Installation & Management

`helm install <release_name> <chart_name> [--namespace <namespace>] [--set key=value]`
    - Use: Installs a chart with a specific release name. You can customize settings with `--set`. Deploys the application.

`helm upgrade <release_name> <chart_name> [--set key=value]`
    - Use: Updates an existing app (e.g., changing version or config) without deleting it.

`helm uninstall <release_name> [--namespace <namespace>]`
    - Use: Deletes the Deployment, Service, ConfigMaps, and everything else associated with that release.

`helm list  [--namespace <namespace>]` OR `helm list -A`
    - Use: Lists all Helm releases in the specified namespace or across all namespaces with `-A`.

### The "Dry Run" / "Template" (Generating YAML)

`helm template <chart_name> [--set key=value]`
    - Commamd: `helm template my-release bitnami/nginx > output.yaml`
    - Use: It renders the templates into standard Kubernetes YAML and saves it to a file. It does not talk to the cluster
` --dry-run`
    - Command: helm install my-release bitnami/nginx --dry-run
    - Use: Simulates the install to check for errors, but doesn't actually create anything.

---

**Start fresh** -- `helm repo add ...` then `helm repo update`
**Find defaults** -- `helm show values <chart> > default.yaml` e.g., `helm show values bitnami/nginx > nginx-defaults.yaml`
**Install** -- `helm install <name> <chart>`
**Change config** -- `helm install .. --set key=value ` or edit `values.yaml` and use `-f values.yaml`
**Just YAML** -- `helm template ... > file.yaml` or `--dry-run`
**Where is it?** -- `helm list -A` or `helm list -n <namespace> `

---
If you are stuck figuring out the correct --set variable name (e.g., is it image.tag or image.version?), do not guess. Run `helm show values <chart> | grep image` to see exactly what the chart author called that variable.