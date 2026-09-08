<!-- llm-readme-management spec=1 commit=988a669eecad4232892ec6baf255ba244474e285 template=helm model=qwen3.6-35b-a3b digest=5b6cfbe96d9f generated=2026-09-08T20:57:16Z -->
<a href="https://hauke.cloud" target="_blank"><img src="https://img.shields.io/badge/home-hauke.cloud-brightgreen" alt="hauke.cloud" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud" target="_blank"><img src="https://img.shields.io/badge/github-hauke.cloud-blue" alt="hauke.cloud Github Organisation" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud/llm-readme-management" target="_blank"><img src="https://img.shields.io/badge/template-helm-orange" alt="Repository type - helm" style="display: block;" /></a>


# Keycloak


<img src="https://raw.githubusercontent.com/hauke-cloud/.github/main/resources/img/organisation-logo-small.png" alt="hauke.cloud logo" width="109" height="123" align="right">


<llm header hint="Name the chart and what it deploys.">

The keycloak Helm chart deploys a production-ready Keycloak identity and access management instance on Kubernetes via the OLM operator, with an optional cloudnative-pg managed PostgreSQL backend. It targets Kubernetes operators managing authentication infrastructure within the hauke.cloud ecosystem. Review the configuration options to provision hostnames, TLS certificates, and database backups before installation.

</llm>


## :book: Description

<llm description>

This Helm chart deploys a production-ready Keycloak identity and access management instance on Kubernetes for the hauke.cloud ecosystem. You use it to provision authentication and authorization infrastructure without managing the underlying operators yourself. The chart assumes the Red Hat Keycloak operator, cloudnative-pg, and cert-manager are already installed in your cluster, and focuses on generating the necessary custom resources, ingress rules, and TLS certificates to run Keycloak securely.

You can optionally enable a highly available PostgreSQL backend managed by cloudnative-pg, complete with automated S3-compatible backups via Barman. The chart also handles external DNS records through external-dns annotations and supports custom theme injection via an init container.

- Creates a `Keycloak` custom resource configured by the OLM operator
- Provisions an optional cloudnative-pg `Cluster` with configurable replicas and persistent storage
- Generates cert-manager `Certificate` resources for TLS on main and admin hostnames
- Renders Kubernetes `Ingress` resources annotated for external-dns integration
- Deploys an optional theme-downloader init container to mount custom Keycloak themes

</llm>


## :clipboard: Requirements

<llm requirements hint="Give the Kubernetes version constraint from Chart.yaml, the Helm version, and any dependency charts or CRDs that must already be present.">

Before you deploy this chart, ensure the following are installed and configured:
- A Kubernetes cluster with Helm >= 3.x supporting OCI registry pushes.
- The Red Hat Keycloak operator (providing `k8s.keycloak.org/v2alpha1`).
- The cloudnative-pg operator (providing `postgresql.cnpg.io/v1`).
- cert-manager (providing `cert-manager.io/v1`).
- A cert-manager `ClusterIssuer` named `issuer` when TLS is enabled.
- An external-dns controller and DNS provider credentials if ingress routing is required.
- S3-compatible storage and a Kubernetes secret containing Barman credentials when provisioning the optional database.

</llm>


## 🚀 Getting started

<llm getting_started hint="helm repo add, helm install and helm upgrade with the real repository URL and chart name. Show a values override only if the chart needs one to start.">

</llm>


## :airplane: Usage

<llm usage hint="Show installing with a values file, and how to reach or verify the deployed workload.">

To deploy Keycloak, create a custom values file that overrides the defaults for your environment. The chart requires the Keycloak operator, cloudnative-pg operator, and cert-manager to be present on your cluster beforehand. Configure the hostnames, enable the optional database, and point to your cert-manager issuer:

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
  backup:
    barmanObjectStore:
      destinationPath: s3://bucket/path/to
```

Install the chart using the OCI registry URL and your values file:

```bash
helm install keycloak oci://ghcr.io/hauke-cloud/charts/keycloak --version 0

</llm>


## :wrench: Configuration

<llm configuration hint="A table of the top-level values from values.yaml: key, default, description. Point at values.yaml for the full set.">

You configure this Helm chart by overriding values in your deployment commands or by providing a custom values file. The chart exposes a structured configuration surface split across Keycloak instance settings, optional database provisioning, and cluster-level scheduling constraints.

| Name | Default | Description |
|---|---|---|
| `keycloak.replicaCount` | `1` | Number of Keycloak instances per the OLM operator. |
| `keycloak.hostname.hostname` | `id.example.com` | Main Keycloak service hostname. |
| `keycloak.hostname.admin` | `admin.id.example.com` | Admin UI hostname. |
| `keycloak.ingress.enabled` | `true` | Whether to create Ingress resources. |
| `keycloak.certificate.enabled` | `true` | Create a cert-manager Certificate resource. |
| `database.enabled` | `false` | Whether to provision a cloudnative-pg Cluster. |
| `database.storage.size` | `10Gi` | Persistent storage size for the database. |
| `nodeSelector` | `` | Node selection constraints for pods. |

The table covers the primary inputs you will adjust during deployment. You can find the complete configuration surface, including nested certificate parameters, backup schedules, resource limits, and advanced operator overrides, in `values.yaml`. All defaults are applied automatically when you do not specify them.

</llm>


## 📄 License

This Project is licensed under the GNU General Public License v3.0

- see the [LICENSE](LICENSE) file for details.


## :coffee: Contributing

To become a contributor, please check out the [CONTRIBUTING](CONTRIBUTING.md) file.


## :email: Contact

For any inquiries or support requests, please open an issue in this
repository or contact us at [contact@hauke.cloud](mailto:contact@hauke.cloud).
