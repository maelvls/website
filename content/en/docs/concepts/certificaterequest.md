---
title: "CertificateRequest"
linkTitle: "CertificateRequest"
weight: 300
type: "docs"
---

The `CertificateRequest` is a namespaced resource in cert-manager that is used
to request X.509 certificates from an [`Issuer`](../issuer/). The resource
contains a base64 encoded string of a PEM encoded certificate request which is
sent to the referenced issuer. A successful issuance will return a signed
certificate, based on the certificate signing request. `CertificateRequests` are
typically consumed and managed by controllers or other systems and should not be
used by humans - unless specifically needed.

A simple `CertificateRequest` looks like the following:

```yaml
apiVersion: cert-manager.io/v1
kind: CertificateRequest
metadata:
  name: my-ca-cr
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQzNqQ0NBY1lDQVFBd2daZ3hDekFKQmdOVkJBWVRBbHBhTVE4d0RRWURWUVFJREFaQmNHOXNiRzh4RFRBTApCZ05WQkFjTUJFMXZiMjR4RVRBUEJnTlZCQW9NQ0VwbGRITjBZV05yTVJVd0V3WURWUVFMREF4alpYSjBMVzFoCmJtRm5aWEl4RVRBUEJnTlZCQU1NQ0dwdmMyaDJZVzVzTVN3d0tnWUpLb1pJaHZjTkFRa0JGaDFxYjNOb2RXRXUKZG1GdWJHVmxkWGRsYmtCcVpYUnpkR0ZqYXk1cGJ6Q0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQwpBUW9DZ2dFQkFLd01tTFhuQkNiRStZdTIvMlFtRGsxalRWQ3BvbHU3TlZmQlVFUWl1bDhFMHI2NFBLcDRZQ0c5Cmx2N2kwOHdFMEdJQUgydnJRQmxVd3p6ZW1SUWZ4YmQvYVNybzRHNUFBYTJsY2NMaFpqUlh2NEVMaER0aVg4N3IKaTQ0MWJ2Y01OM0ZPTlRuczJhRkJYcllLWGxpNG4rc0RzTEVuZmpWdXRiV01Zeis3M3ptaGZzclRJUjRzTXo3cQpmSzM2WFM4UkRjNW5oVVcyYU9BZ3lnbFZSOVVXRkxXNjNXYXVhcHg2QUpBR1RoZnJYdVVHZXlZUUVBSENxZmZmCjhyOEt3YTFYK1NwYm9YK1ppSVE0Nk5jQ043OFZnL2dQVHNLZmphZURoNWcyNlk1dEVidHd3MWdRbWlhK0MyRHIKWHpYNU13RzJGNHN0cG5kUnRQckZrU1VnMW1zd0xuc0NBd0VBQWFBQU1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQgpBUUFXR0JuRnhaZ0gzd0N3TG5IQ0xjb0l5RHJrMUVvYkRjN3BJK1VVWEJIS2JBWk9IWEFhaGJ5RFFLL2RuTHN3CjJkZ0J3bmlJR3kxNElwQlNxaDBJUE03eHk5WjI4VW9oR3piN0FVakRJWHlNdmkvYTJyTVhjWjI1d1NVQmxGc28Kd005dE1QU2JwcEVvRERsa3NsOUIwT1BPdkFyQ0NKNnZGaU1UbS9wMUJIUWJSOExNQW53U0lUYVVNSFByRzJVMgpjTjEvRGNMWjZ2enEyeENjYVoxemh2bzBpY1VIUm9UWmV1ZEp6MkxmR0VHM1VOb2ppbXpBNUZHd0RhS3BySWp3ClVkd1JmZWZ1T29MT1dNVnFNbGRBcTlyT24wNHJaT3Jnak1HSE9tTWxleVdPS1AySllhaDNrVDdKU01zTHhYcFYKV0ExQjRsLzFFQkhWeGlKQi9Zby9JQWVsCi0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  isCA: false
  usages:
    - signing
    - digital signature
    - server auth
  duration: 90d
  issuerRef:
    name: ca-issuer
    # We can reference ClusterIssuers by changing the kind here.
    # The default value is Issuer (i.e. a locally namespaced Issuer)
    kind: Issuer
    group: cert-manager.io
```

This `CertificateRequest` will make cert-manager attempt to request the `Issuer`
`ca-issuer` in the default issuer group `cert-manager.io`, return a
certificate based upon the certificate signing request. Other groups can be
specified inside the `issuerRef` which will change the targeted issuer to other
external, third party issuers you may have installed.

The resource also exposes the option for stating the certificate as CA, Key
Usages, and requested validity duration.

All fields within the `spec` of the `CertificateRequest`, as well as any managed
cert-manager annotations, are immutable and cannot be modified after creation.

A successful issuance of the certificate signing request will cause an update to
the resource, setting the status with the signed certificate, the CA of the
certificate (if available), and setting the `Ready` condition to `True`.

Whether issuance of the certificate signing request was successful or not, a retry of the
issuance will _not_ happen. It is the responsibility of some other controller to
manage the logic and life cycle of `CertificateRequests`.

## Output of `kubectl get`

The `kubectl get certificaterequest` displays information about the
approval:

```bash
% kubectl get certificaterequest
NAMESPACE      NAME                   APPROVED  DENIED  READY  ISSUER            REQUESTOR
cert-manager   service-1-a45bc1       True              True   letsencrypt-prod  system:serviceaccount:cert-manager:letsencrypt-prod
istio-system   service-mesh-ca-whh5b  True              True   mesh-ca           system:serviceaccount:istio-system:istiod
istio-system   my-app-fj9sa                     True           mesh-ca           system:serviceaccount:my-app:my-app
```

The columns have the following meaning:

| Column                       | Description                                       |
| ---------------------------- | ------------------------------------------------- |
| `APPROVED`                   | Status of the `Approved` condition                |
| `DENIED`                     | Status of the `Denied` condition                  |
| `READY`                      | Status of the `Ready` condition                   |
| `ISSUER`                     | Name of the issuer given in `spec.issuerRef.name` |
| `REQUESTOR`                  | [User name][subject] given in `spec.username`     |
| `STATUS` (requires `-owide`) | Message contained in the `Ready` condition        |

## Conditions

`CertificateRequests` have a set of strongly defined conditions that should be
used and relied upon by controllers or services to make decisions on what
actions to take next on the resource.

### Ready

Each ready condition consists of the pair `Ready` - a boolean value, and
`Reason` - a string. The set of values and meanings are as follows:

| Ready | Reason  | Condition Meaning                                                                                                                                                                                                                               |
| ----- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| False | Pending | The `CertificateRequest` is currently pending, waiting for some other operation to take place. This could be that the `Issuer` does not exist yet or the `Issuer` is in the process of issuing a certificate.                                   |
| False | Failed  | The certificate has failed to be issued - either the returned certificate failed to be decoded or an instance of the referenced issuer used for signing failed. No further action will be taken on the `CertificateRequest` by it's controller. |
| True  | Issued  | A signed certificate has been successfully issued by the referenced `Issuer`.                                                                                                                                                                   |

## Information about the requestor

Similarly as for the Kubernetes [CSR][csr-ref], CertificateRequests also
include information about the requestor. of `UserInfo` fields as part of
the spec, namely: `username`, `groups`, `uid`, and `extra`. These values
contain the user who created the `CertificateRequest`. This user will be
cert-manager itself in the case that the `CertificateRequest` was created
by a [`Certificate`](../certificate/) resource, or instead the user who
created the `CertificateRequest` directly.

```yaml
kind: CertificateRequest
spec:
  uid: 71a1025e-e820-4331-9de0-a1de1bdee249
  username: system:serviceaccount:cert-manager:cert-manager
  groups:
    - system:serviceaccounts
    - system:serviceaccounts:cert-manager
    - system:authenticated
  extra:
    some-property: some-value
```

The four filds are populated by the cert-manager webhook when the
CertificateRequest is created and their immutability is enforced by the
webhook.

```yaml
kind: CertificateRequest
spec:
  uid: 71a1025e-e820-4331-9de0-a1de1bdee249
  username: system:serviceaccount:cert-manager:cert-manager
  groups:
    - system:serviceaccounts
    - system:serviceaccounts:cert-manager
    - system:authenticated
  extra:
    some-property: some-value
```

|            |                                                                 |
| ---------- | --------------------------------------------------------------- |
| `uid`      | UID of the user who created the CertificateRequest              |
| `username` | Name of the user who created the CertificateRequest             |
| `groups`   | Group membership of the user who created the CertificateRequest |
| `extra`    | Extra attributes of the user who created the CertificateRequest |

`username`, `groups`, `uid`, and `extra`

These four fields are documented in the [API reference
documentation](/docs/reference/api-docs/#cert-manager.io/v1.CertificateRequestSpec).

[csr-ref]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#certificatesigningrequestspec-v1-certificates-k8s-io
[userinfo-ref]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/#userinfo-v1beta1-authentication-k8s-io

> **Warning**: These fields are managed by cert-manager and must not be set
> or modified by anyone else. When the `CertificateRequest` is created,
> these fields will be overridden, and any request attempting to modify
> them will be rejected.

## Approval API

We introduced the Approval API in cert-manager v1.3. The approval API
introduces two mutually-exclusive conditions for CertificateRequests:
`Approved` and `Denied`. This API gates CertificateRequests from being
signed by its signer.

As part of the approval API, we decided to make use of the term "signer"
instead of "issuer" as a way to come closer to the terms used in the
Kubernetes [CSR][csr] API:

| Term   | Meaning                                                                                      |
| ------ | -------------------------------------------------------------------------------------------- |
| Signer | An entity that signs X.509 CSRs                                                              |
| Issuer | A Kubernetes controller that calls to a signer and sets the readiness of CertificateRequests |

[csr]: https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#create-certificatesigningrequest

In other words, an issuer is a cert-manager-specific implementation of a
signer. By "issuer" we refer to both external issuers (such as
[aws-pca-issuer](https://github.com/jniebuhr/aws-pca-issuer)) and internal
issuers (such as the ACME issuer embedded into cert-manager).

The approval API kicks in right before the signer (e.g., the ACME issuer)
gets to sign the X.509 CSR contained in a CertificateRequest. We call
"approver" the Kubernetes controller that takes care of setting the
`Approved` and `Denied` conditions. conditions.

The following diagram shows the two possible scenarios. Note that in both
cases, the issuer is still responsible for setting the `Ready` condition
(either to `True` or to `False`):

```text

 APPROVAL SCENARIO                                               DENIAL SCENARIO
 +------------------------------+        |      +------------------------------+
 | kind: CertificateRequest     |        |      | kind: CertificateRequest     |
 | status:                      |        |      | status:                      |
 |  conditions:                 |        |      |  conditions:                 |
 |   - type: Ready              |        |      |   - type: Ready              |
 |     status: "False"          |        |      |     status: "False"          |
 |     reason: Pending          |        |      |     reason: Pending          |
 +------------------------------+        |      +------------------------------+
                |                        |                     |
                |                        |                     |
                | The approver           |                     | The approver
                | approves the           |                     | denies the
                | request                |                     | request
                |                        |                     |
                |                        |                     |
                v                        |                     v
 +------------------------------+        |      +------------------------------+
 | kind: CertificateRequest     |        |      | kind: CertificateRequest     |
 | status:                      |        |      | status:                      |
 |  conditions:                 |        |      |  conditions:                 |
 |   - type: Ready              |        |      |   - type: Ready              |
 |     status: "False"          |        |      |     status: "False"          |
 |     reason: Pending          |        |      |     reason: Pending          |
 |   - type: Approved           |        |      |   - type: Denied             |
 |     status: "True"           |        |      |     status: "True"           |
 +------------------------------+        |      +------------------------------+
                |                        |                     |
                |                        |                     |
                |- The signer signs      |                     |
                |  the X.509 CSR         |                     |
                |                        |                     |
                |- The issuer sets       |                     |- The issuer sets
                |  Ready=True            |                     |  Ready=False
                |                        |                     |
                |                        |                     |
                v                        |                     v
 +------------------------------+        |      +------------------------------+
 | kind: CertificateRequest     |        |      | kind: CertificateRequest     |
 | status:                      |        |      | status:                      |
 |  conditions:                 |        |      |  conditions:                 |
 |   - type: Ready              |        |      |   - type: Ready              |
 |     status: "True"           |        |      |     status: "False"          |
 |     reason: Issued           |        |      |     reason: Denied           |
 |   - type: Approved           |        |      |   - type: Denied             |
 |     status: "True"           |        |      |     status: "True"           |
 +------------------------------+        |      +------------------------------+
               ✅                                               ❌
```

<!--
Diagram source: https://textik.com/#5739af8ae4cfb124
-->

Note that each CertificateRequests is managed by one issuer. For example,
the following CertificateRequest is managed by the

```yaml
kind: CertificateRequest
spec:
  issuerRef:
    kind: Issuer
    name: example-selfsigned-issuer
```

#### Embedded default approver

cert-manager comes with a default approver.

The consistency of the approval API is guarenteed by the admission webhook.

-
- A signer will sign a managed CertificateRequest with an Approved
  condition.
- A signer will never sign a managed CertificateRequest with a Denied condition

- The conditions `Approved` or `Denied` are terminal: they cannot be
  modified or changed once set.

|            |                                                                           |
| ---------- | ------------------------------------------------------------------------- |
|            | A signer should not sign a managed CertificateRequest without an Approved |
| condition. |

### RBAC requirements for the Approval API

Let us imagine that we want to build an approver that uses OPA (Open Policy
Agent). Our approver takes the form of a Kubernetes controller maned
`opa-approver-controller` that watches the CertificateRequest objects. For
every CertificateRequest, our approver asks the OPA engine whether the
CertificateRequest should be accepted or not:

- If OPA returns true, the approver adds the `Approved=True` condition to the
  CertificateRequest;
- If OPA returns false, the approver adds the `Denied=True` condition to the
  CertificateRequest.
  By default, our approver does not have the permission to add the `Approved=True`
  and `Denied=True` conditions. In order to allow it to add these conditions, we
  use a custom RBAC syntax that cert-manager knows about. Let us imagine that our
  approver needs to approve CertificateRequests that are targeting the internal
  ACME issuer as well as the external issuer for [AWS
  PrivateCA](https://github.com/jniebuhr/aws-pca-issuer):

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-issuer
  namespace: some-namespace
spec:
  acme:
    email: jane.doe@gmail.com
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-issuer-account
    solvers:
      - http01:
          ingress:
            class: traefik
---
apiVersion: awspca.cert-manager.io
kind: AWSPCAClusterIssuer
metadata:
  name: awspca-issuer
  namespace: some-namespace
spec:
  arn: "pca-807911e9"
```

To make our approver work for all CertificateRequests across all namespaces, we
apply the following role to our cluster:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: opa-approver-for-acme-and-aws-pca
rules:
  - apiGroups: ["cert-manager.io"]
    resources: ["signers"]
    verbs: ["approve"]
    resourceNames:
      - "clusterissuers.acme.cert-manager.io/letsencrypt-issuer"
      #  <-------------------------------->  <---------------->
      #    resource type + resource group        issuer name
      - "awspcaclusterissuers.awspca.cert-manager.io/awspca-issuer"
```

Note: the "resource type"
[is](https://kubernetes.io/docs/reference/using-api/api-concepts/#standard-api-terminology)
the lower-case and plural version of the "kind" field.

#### Building your own approver controller

By default, cert-manager will run an internal approval controller which
will automatically approve all CertificateRequests that reference any
internal issuer type in any namespace: `cert-manager.io/Issuer`,
`cert-manager.io/ClusterIssuer`.

To disable this default behavior, add the following argument to the
cert-manager-controller: `--controllers=*,-certificaterequests-approver`.
This can be achieved with helm by appending:

```bash
--set extraArgs="{--controllers='*\,-certificaterequests-approver'}"
```

Alternatively, in order for the internal approver controller to approve
CertificateRequests that reference an external issuer, add the following
RBAC to the cert-manager-controller Service Account. Please replace the
given resource names with the relevant names:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cert-manager-controller-approve:my-issuer-example-com # edit
rules:
  - apiGroups:
      - cert-manager.io
    resources:
      - signers
    verbs:
      - approve
    resourceNames:
      - issuers.my-issuer.example.com/* # edit
      - clusterissuers.my-issuer.example.com/* # edit
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cert-manager-controller-approve:my-issuer-example-com # edit
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cert-manager-controller-approve:my-issuer-example-com # edit
subjects:
  - kind: ServiceAccount
    name: cert-manager
    namespace: cert-manager
```

{{% pageinfo color="info" %}}
The "resource type"
[corresponds to](https://kubernetes.io/docs/reference/using-api/api-concepts/#standard-api-terminology)
the lower-case and plural version of the "kind" field.
{{% /pageinfo %}}

[subject]: https://kubernetes.io/docs/reference/access-authn-authz/rbac/#referring-to-subjects

#### Reason and message of the Approval API

The `reason` and `message` fields on the `Denied` and `Approved` condition
are approver-specific, meaning that each approver will set their own. Here
is the set of guidelines:

| Field     | Guideline                                               |
| --------- | ------------------------------------------------------- |
| `reason`  | Machine-readable string identifying the approver        |
| `message` | Human-readable explanation of why the condition was set |

Here is a few examples of approved CertificateRequests:

```yaml
kind: CertificateRequest
status:
  conditions:
    - type: Approved
      status: "True"
      reason: opa-approver-controller
      message: Certificate request was approved by opa-approver-controller
```

> The `opa-approver-controller` is hypothetical and does not exist yet.

```yaml
kind: CertificateRequest
status:
  conditions:
    - type: Approved
      status: "True"
      reason: cert-manager.io
      message: Certificate request has been approved by cert-manager.io
```

We can also approve the certificate request using the [kubectl
cert-manager](https://cert-manager.io/docs/usage/kubectl-plugin/) plugin:

```sh
kubectl cert-manager approve cert-request-1
```

Here is what the approved CertificateRequest looks like:

```yaml
kind: CertificateRequest
status:
  conditions:
    - type: Approved
      status: "True"
      reason: kubectl-cert-manager/v1.3.0
      message: Certificate request was manually approved by kubectl-cert-manager/v1.3.0
```

Here are the a few examples of denied CertificateRequests:

```yaml
kind: CertificateRequest
status:
  conditions:
    - type: Denied
      status: "True"
      reason: opa-approver-controller
      message: >
        Certificate request has been denied by opa-approver-controller
        due to [key lenght is too short, DNS names must match service name]
```

```yaml
kind: CertificateRequest
status:
  conditions:
    - type: Denied
      status: "True"
      reason: kubectl_cert-manager/v1.3.0
      message: Certificate request was manually denied by kubectl_cert-manager/v1.3.0
```

### RBAC syntax for the Approval API

When a user or controller attempts to approve or deny a CertificateRequest,
the cert-manager webhook will evaluate whether it has sufficient
permissions to do so. These permissions are based upon the request itself-
specifically the request's IssuerRef:

```yaml
apiGroups: ["cert-manager.io"]
resources: ["signers"]
verbs: ["approve"]
resourceNames:
  # namesapced signers
  - "<signer-resource-name>.<signer-group>/<signer-namespace>.<signer-name>"
  # cluster scoped signers
  - "<signer-resource-name>.<signer-group>/<signer-name>"
  # all signers of this resource name
  - "<signer-resource-name>.<signer-group>/*"
```

An example ClusterRole that would grant the permissions to set the Approve and
Denied conditions of CertificateRequests that reference the cluster scoped
`myissuers` external issuer, in the group `my-example.io`, with the name `myapp`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: my-example-io-my-issuer-myapp-approver
rules:
  - apiGroups: ["cert-manager.io"]
    resources: ["signers"]
    verbs: ["approve"]
    resourceNames: ["myissuers.my-example.io/myapp"]
```

If the approver does not have sufficient permissions defined above to set the
Approved or Denied conditions, the request will be rejected by the cert-manager
validating admission webhook.

- The RBAC permissions _must_ be granted at the cluster scope
- Namespaced signers are represented by a namespaced resource using the syntax of `<signer-resource-name>.<signer-group>/<signer-namespace>.<signer-name>`
- Cluster scoped signers are represented using the syntax of `<signer-resource-name>.<signer-group>/<signer-name>`
- An approver can be granted approval for all namespaces via `<signer-resource-name>.<signer-group>/*`
- The apiGroup must _always_ be `cert-manager.io`
- The resource must _always_ be `signers`
- The verb must _always_ be `approve`, which grants the approver the permissions to set _both_ Approved and Denied conditions

An example of signing all `myissuer` signers in all namespaces, and
`clustermyissuers` with the name `myapp`, in the `my-example.io` group:

```yaml
resourceNames:
  ["myissuers.my-example.io/*", "clustermyissuers.my-example.io/myapp"]
```

An example of signing `myissuer` with the name `myapp` in the namespaces `foo`
and `bar`:

```yaml
resourceNames:
  ["myissuers.my-example.io/foo.myapp", "myissuers.my-example.io/bar.myapp"]
```
