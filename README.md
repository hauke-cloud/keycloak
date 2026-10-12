<!-- llm-readme-management spec=1 commit=c1ddf99d58270bd044f0a8c33a5d5aaf149cbf05 template=helm model=qwen3.8-27b-q4 digest=5b6cfbe96d9f generated=2026-10-12T07:35:52Z -->
<a href="https://hauke.cloud" target="_blank"><img src="https://img.shields.io/badge/home-hauke.cloud-brightgreen" alt="hauke.cloud" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud" target="_blank"><img src="https://img.shields.io/badge/github-hauke.cloud-blue" alt="hauke.cloud Github Organisation" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud/llm-readme-management" target="_blank"><img src="https://img.shields.io/badge/template-helm-orange" alt="Repository type - helm" style="display: block;" /></a>


# Keycloak


<img src="https://raw.githubusercontent.com/hauke-cloud/.github/main/resources/img/organisation-logo-small.png" alt="hauke.cloud logo" width="109" height="123" align="right">


<llm header hint="Name the chart and what it deploys.">

The `keycloak` Helm chart deploys a Keycloak identity provider on Kubernetes via the Red Hat Keycloak operator, with an optional cloudnative-pg PostgreSQL backend. It is for Kubernetes operators who already run the Keycloak operator, cloudnative-pg, and cert-manager and want one chart to manage the Keycloak resource, TLS, ingress, and database.

</llm>


## :book: Description

<llm description>

This Helm chart deploys a Keycloak identity and access management instance on Kubernetes by rendering the custom resources consumed by the Red Hat Keycloak operator (installed via OLM). It does not install the operator; your cluster must already have the Keycloak operator, and optionally cloudnative-pg and cert-manager, before you install this chart. The chart is published to `oci://ghcr.io/hauke-cloud/charts` as part of the `hauke-cloud` organisation.

When you enable the bundled database, the chart provisions a three-replica cloudnative-pg PostgreSQL cluster with S3-compatible Barman backups. It also renders cert-manager `Certificate` resources for TLS, `Ingress` resources annotated for external-dns reconciliation, and an optional init container that downloads custom Keycloak themes into the pod.

- Renders a `Keycloak` custom resource (`k8s.keycloak.org/v2alpha1`) with configurable replicas, hostnames, and passthrough options
- Provisions an optional cloudnative-pg `Cluster` with hostname-affinity spread and scheduled S3 backups
- Generates a cert-manager `Certificate` covering the main and admin hostnames
- Renders `Ingress` resources (nginx class) annotated for external-dns
- Injects an optional theme-downloader init container via the operator's `unsupported.podTemplate`

</llm>


## :clipboard: Requirements

<llm requirements hint="Give the Kubernetes version constraint from Chart.yaml, the Helm version, and any dependency charts or CRDs that must already be present.">

- Helm 3 (the chart declares `apiVersion: v2`; no specific Helm version is pinned in the repository).
- No Kubernetes version constraint is set in `Chart.yaml`.
- The Red Hat Keycloak operator (OLM-based), which provides the `k8s.keycloak.org/v2alpha1` CRD.
- The cloudnative-pg operator, which provides the `postgresql.cnpg.io/v1` CRD (required when `database.enabled: true`).
- cert-manager, which provides the `cert-manager.io/v1` CRD (required when `keycloak.certificate.enabled: true`).
- A cert-manager `ClusterIssuer` named `issuer` (or a custom `issuerRef`), when TLS is enabled.
- An ingress controller registered for class `nginx`.
- An external-dns controller (ingress resources carry the `external-dns.alpha.kubernetes.io/hostname` annotation).
- An S3-compatible bucket and a Kubernetes secret `cnpg-s3-creds` (keys `credentials`, `config`; profile `cnpg`) when database backups are enabled.
- `pre-commit` for running the repository's lint and documentation hooks.

No dependency charts are declared in `Chart.yaml` and no `Chart.lock` is present.

</llm>


## 🚀 Getting started

<llm getting_started hint="helm repo add, helm install and helm upgrade with the real repository URL and chart name. Show a values override only if the chart needs one to start.">

1. Clone the repository.

```bash
git clone https://github.com/hauke-cloud/keycloak.git
cd keycloak
```

2. Install the chart into a cluster that already runs the Red Hat Keycloak operator (OLM), cert-manager with a `ClusterIssuer` named `issuer`, an nginx ingress controller, and external-dns.

```bash
helm install keycloak oci://ghcr.io/hauke-cloud/charts/keycloak --version 0
```

3. The shipped defaults use placeholder hostnames and a TLS secret name that does not match the Certificate; pass a values file (via the conventional `-f` flag) that sets real hostnames and aligns the secret.

```yaml
keycloak:
  hostname:
    hostname: "https://id.example.com"
    admin: "https://admin.id.example.com"
  http:
    tlsSecret: "keycloak"
```

</llm>


## :airplane: Usage

<llm usage hint="Show installing with a values file, and how to reach or verify the deployed workload.">

Before installing, ensure your cluster already has the Red Hat Keycloak operator, cert-manager with a `ClusterIssuer` named `issuer`, and an nginx ingress controller. If you enable the bundled database, the cloudnative-pg operator must also be present.

Create a values file with your environment-specific settings:

```yaml
keycloak:
  hostname:
    hostname: id.example.com
    admin: admin.id.example.com
  certificate:
    enabled: true
    issuerRef:
      name: issuer
      kind: ClusterIssuer
      group: cert-manager.io
database:
  enabled: true
```

Install the chart from the OCI registry:

```bash
helm install keycloak oci://ghcr.io/hauke-cloud/charts/keycloak --version 0 -f values.yaml
```

Verify the deployment by checking the Keycloak custom resource and the rendered Ingresses:

```bash
kubectl get keycloak
kubectl get ingress
```

The main Keycloak instance is reachable at the hostname you configured (for example `https://id.example.com`), and the admin console at the admin hostname (for example `https://admin.id.example.com`). The Ingress resources carry the `external-dns.alpha.kubernetes.io/hostname` annotation, so an external-dns controller reconciles DNS entries for you.

</llm>


## :wrench: Configuration

<llm configuration hint="A table of the top-level values from values.yaml: key, default, description. Point at values.yaml for the full set.">

You configure this chart by overriding values in your `helm install` command or by supplying a custom values file. The table below lists the values that gate major features or that you will most likely override. The full set lives in `charts/keycloak/values.yaml`.

| Key | Default | Description |
|---|---|---|
| `keycloak.replicaCount` | `1` | Number of Keycloak instances (`spec.instances` on the operator CR). |
| `keycloak.hostname.hostname` | `"https://id.example.com"` | Main service hostname; the helper strips any scheme prefix. |
| `keycloak.hostname.admin` | `"https://admin.id.example.com"` | Admin console hostname. |
| `keycloak.ingress.enabled` | `true` | Render Ingress resources for main and admin hostnames. |
| `keycloak.http.tlsSecret` | `"keycloak-tls"` | Secret name the Keycloak operator serves TLS from. |
| `keycloak.certificate.enabled` | `true` | Render a cert-manager `Certificate` for the hostnames. |
| `keycloak.db` | *(unset)* | When omitted the chart wires Keycloak to the in-cluster cloudnative-pg database; set it to override the entire `spec.db` block. |
| `keycloak.additionalOptions` | `[]` | Passed through as `spec.additionalOptions` on the Keycloak CR. |
| `keycloak.themeDownloader.enabled` | `false` | Inject a theme-downloader init container via `unsupported.podTemplate`. |
| `database.enabled` | `false` | Gate for the cloudnative-pg `Cluster` and its `ScheduledBackup` resources. |
| `database.replicaCount` | `3` | Number of cloudnative-pg replicas. |
| `database.storage.size` | `"10Gi"` | Persistent volume size per replica. |

Values not listed here—certificate issuer and duration, backup schedules and S3 settings, the `keycloak.unsupported` passthrough, and the standard `nameOverride`/`fullnameOverride` hooks—are documented in `charts/keycloak/values.yaml`. Note that top-level `nodeSelector`, `tolerations`, and `affinity` keys exist in that file but are not consumed by any template.

</llm>


## 📄 License

This Project is licensed under the GNU General Public License v3.0

- see the [LICENSE](LICENSE) file for details.


## :coffee: Contributing

To become a contributor, please check out the [CONTRIBUTING](CONTRIBUTING.md) file.


## :email: Contact

For any inquiries or support requests, please open an issue in this
repository or contact us at [contact@hauke.cloud](mailto:contact@hauke.cloud).
