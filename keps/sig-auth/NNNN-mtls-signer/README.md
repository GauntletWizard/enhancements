<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [ ] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [ ] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [ ] **Fill out as much of the kep.yaml file as you can.**
  At minimum, you should fill in the "Title", "Authors", "Owning-sig",
  "Status", and date-related fields.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary" and "Motivation" sections.
  These should be easy if you've preflighted the idea of the KEP with the
  appropriate SIG(s).
- [ ] **Create a PR for this KEP.**
  Assign it to people in the SIG who are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the KEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a KEP is merged does not mean it is complete or approved. Any KEP
marked as `provisional` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing KEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

One KEP corresponds to one "feature" or "enhancement" for its whole lifecycle.
You do not need a new KEP to move from beta to GA, for example. If
new details emerge that belong in the KEP, edit the KEP. Once a feature has become
"implemented", major changes should get new KEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/keps/NNNN-kep-template/README.md).

**Note:** Any PRs to move a KEP to `implementable`, or significant changes once
it is marked `implementable`, must be approved by each of the KEP approvers.
If none of those approvers are still appropriate, then changes to that list
should be approved by the remaining approvers and/or the owning SIG (or
SIG Architecture for cross-cutting KEPs).
-->
# KEP-NNNN: MTLS Certificate issuer


<!--
A table of contents is helpful for quickly jumping to sections of a KEP and for
highlighting any additional information provided beyond the standard KEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories (Optional)](#user-stories-optional)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
  - [Notes/Constraints/Caveats (Optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Test Plan](#test-plan)
  - [Graduation Criteria](#graduation-criteria)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
  - [Version Skew Strategy](#version-skew-strategy)
- [Production Readiness Review Questionnaire](#production-readiness-review-questionnaire)
  - [Feature Enablement and Rollback](#feature-enablement-and-rollback)
  - [Rollout, Upgrade and Rollback Planning](#rollout-upgrade-and-rollback-planning)
  - [Monitoring Requirements](#monitoring-requirements)
  - [Dependencies](#dependencies)
  - [Scalability](#scalability)
  - [Troubleshooting](#troubleshooting)
- [Implementation History](#implementation-history)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Infrastructure Needed (Optional)](#infrastructure-needed-optional)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [kubernetes/enhancements] referencing this KEP and targeting a release
milestone **before the [Enhancement Freeze](https://git.k8s.io/sig-release/releases)
of the targeted release**.

For enhancements that make changes to code or processes/procedures in core
Kubernetes—i.e., [kubernetes/kubernetes], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [ ] (R) Enhancement issue in release milestone, which links to KEP dir in [kubernetes/enhancements] (not the initial KEP PR)
- [ ] (R) KEP approvers have approved the KEP status as `implementable`
- [ ] (R) Design details are appropriately documented
- [ ] (R) Test plan is in place, giving consideration to SIG Architecture and SIG Testing input (including test refactors)
- [ ] (R) Graduation criteria is in place
- [ ] (R) Production readiness review completed
- [ ] (R) Production readiness review approved
- [ ] "Implementation History" section is up-to-date for milestone
- [ ] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [ ] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary

<!--
This section is incredibly important for producing high-quality, user-focused
documentation such as release notes or a development roadmap. It should be
possible to collect this information before implementation begins, in order to
avoid requiring implementors to split their attention between writing release
notes and implementing the feature itself. KEP editors and SIG Docs
should help to ensure that the tone and content of the `Summary` section is
useful for a wide audience.

A good summary is probably at least a paragraph in length.

Both in this section and below, follow the guidelines of the [documentation
style guide]. In particular, wrap lines to a reasonable length, to make it
easier for reviewers to cite specific portions, and to minimize diff churn on
updates.

[documentation style guide]: https://github.com/kubernetes/community/blob/master/contributors/guide/style-guide.md
-->

This proposal introduces a new signer to `certificates.k8s.io/v1` that
is capable of validating and signing certificate signing requests
(CSRs) that are suitable for securing transport between two entities
trusted by the cluster.

The `certificates.k8s.io/v1/CertificateSigningRequest` object introduced
the concept of signers with the ability to have independent CAs
handling certificates for each signer type, analogously to the
ingress's `ingressClass`. This proposal adds a new signer
`kubernetes.io/mlts` that will sign certificates scoped to pods or
service accounts.

## Motivation

<!--
This section is for explicitly listing the motivation, goals, and non-goals of
this KEP.  Describe why the change is important and the benefits to users. The
motivation section can optionally provide links to [experience reports] to
demonstrate the interest in a KEP within the wider Kubernetes community.

[experience reports]: https://github.com/golang/go/wiki/ExperienceReports
-->

One of the most common extensions to Kubernetes is the ability to
encrypt traffic from pod to pod. This proposal creates a signer that
can be used to create MTLS certificates for use in communications
between pods, as many teams were doing before the certs/v1 changes
from certs/v1beta1.

This signer's CA certificiate would be easily accessible and be used
as a root of trust for communication that happens within the cluster.

### Goals

<!--
List the specific goals of the KEP. What is it trying to achieve? How will we
know that this has succeeded?
-->

  * Sign certificates suitable for MTLS between pods.
  * Compatible with [SPIFFE SVIDs](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/#spiffe-verifiable-identity-document-svid) and other community-driven certificate standards.
  * Scoped widely enough that it can be used with existing TLS1.3 implementations including `["server auth"]` or `["client auth"]`.

### Non-Goals

  * This signer is not intended to mint public facing certificates
  * This signer is not used to define trust relationships between clusters.

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation. What is the desired outcome and how do we measure success?.
The "Design Details" section below is for the real
nitty-gritty.
-->
This proposal adds a complete set of new controllers that operate on Pods and CertificateSigningRequests as well as a Container Storage Interface used for projecting key material, signed certificates, and trust roots to pods.

The additions default workflow of the cluster are as follows: 
  * A new pod is Created. A built-in Workload-Certificate AdmissionController (similar to the `ServiceAccount` AdmissionController) inspects the pod and adds a projected volume. This projected volume is of type x509Certificate, and includes the name of the signer (“kubernetes.io/workload-certificate”) and the details about the pod (SubjectAlternativeNames)
  * The kubelet schedules this pod. The CSI Driver begins the creation of the volume by generating a private key and CSR data-structure. It uses this CSR data-structure to create a certificates.k8s.io/v1 CertificateSigningRequest object.
  * The Workload-Certificate Approval Controller verifies the details on the CSR. It validates that the CSR (both object and data-structure) conforms to the standard laid out in this document, and updates the CSR Object with the approval.
  * The Workload-Certificate Signing Controller repeats the verification; Re-confirming (probably using the same code) that the CSR conforms to this standard. It then issues the certificate, updating the CSR Object with the signed Certificate.

At a high level, this proposal is focused around a new Signer named “kubernetes.io/workload-certificate”. This signer can be used to automatically provision MTLS certificates that are appropriate for pod-to-pod communication. For compatibility reasons, this signer should be disabled by default at first release, but enabled by a flag and should default to on before `kubernetes.io/legacy-unknown` is disabled.
  * `kubernetes.io/workload-certificate`: signs certificates that can be used as TLS server or client certificates by pods within the cluster.
  1. Trust distribution: signed certificates must be verifiable by a CA that is well defined and accessible by pods.
  1. Permitted subjects - subject must be the fully-qualified `serviceAccountName` specified by the pod's specification.
  1. Permitted x509 extensions - constrains subjectAltName and key usage extensions and discards other extensions.
  1. Permitted key usages - Must be `["digital signature", "key encipherment", "server auth", "client auth"]`.
  1. Expiration/certificate lifetime - set by the `--pod-mtls-signing-duration` option for the kube-controller-manager implementation of this signer (*).
CA bit allowed/disallowed - not allowed.

(*) This value will default to `--cluster-signing-duration` if not specified (and for MVP this flag may not be implemented).

Under the provided controller, each pod with an automatically projected MTLS Certificate, a new Volume is automatically mounted at `/var/run/secrets/kubernetes.io/workload-certificate`, containing a matching TLS v1.3 key and a signed certificate in PEM encoded ASN.1 format.

The details of the certificate are as follows:
  * The certificate is signed by a CA corresponding to the signer-name, and the .
  * Common name is the fully-qualified service account that the pod is running as. I.e., `system:serviceaccount:<namespace>:<serviceaccountname>`.
  * SubjectAltName contains
  * An ip address entry with the ip address address of the pod, i.e., `172.17.0.3`.
  * A DNS name entry with the fully quailty name of the pod, i.e., `172-17-0-3.default.pod.cluster-domain.example`*.
  * The DNS names of matching services in multiple commonly used formats matching the pod's DNS policy, i.e., `my-service`, `my-server.my-namespace`, `my-svc.my-namespace.svc.cluster-domain.example`*. Matching services are determined at pod creation time.
  * The DNS names for the pod in commonly used formats, i.e. the value returned by hostname. This matches the pod's hostname and subdomain fields as described in the existing Kubernetes documenation.
  * a SPIFFE ID https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/#spiffe-id) for the pod, in a well-defined name format. This format should be as `spiffe://cluster-domain.example/ns/<namespace>/sa/<serviceaccountname>` for a given pod’s namepsace and effective serviceaccountname. Cluster domain may or may not be exactly the same identifier as the FQDN.

(*) This presents a new but not unique problem to the cluster; That the cluster known it's own name. We should constrain this as much as possible to this subsystem, though core-dns is already using a cluster-specific DNS Suffix.

#### Admission Controller
The Workload-certificate Admission Controller is analogous to and complementary to the ServiceAccount AdmissionController. It inspects the pod and adds a projected volume of type x509Certificate, adding the appropriate volumeAttributes to tell the CSI to generate a CSR in the format defined above.

#### x509Certificate CSI

`x509Certificate` is a new Container Storage Interface Driver added with this change. It is recommended to install it by default, but not required. ‘fsType’ should be omitted and is ignored; The volume always mounts as an read-only overlay over a ramfs (Not tmpfs, as that could be written to swap, and the size of these entries is negligible). readOnly is likewise ignored, as is nodePublishSecretRef. volumeAttributes, however, contains interesting decisions. Please refer to rfc5280 for the full definitions
volumeAttributes must be populated with at least one:
  * distinguishedName: Any valid x509 DistinguishedName in text form. It’s inclusion is encouraged, but it is discouraged from being unaccompanied by one of the other SubjectAlternativeName entries
  * uniformResourceIdentifier, dNSName, iPAddress, etc… as defined in RFC5280 Page 38. All of the available definitions should be available. Each represents a LIST of the appropriate type, json encoded. 

The CSI Volume generated by the workload-certificate Admission Controller should match the format specified above.

The CSI is responsible for mounting the completed tuple of Private Key and Certificate in a well known location.

The CSI is also responsible for some other well-known objects. The CSI should provide to the 

#### Approval Controller

The signer’s Approval-controller is responsible for verifying that the format of the CSR data structure generated by the CSI Plugin is exactly as stated above, and does not contain any additional grants, claims, SANs, or attributes.

Additionally, the Approval Controller should verify that the `spec.username` (the requester's fully-qualified username, controlled by the apiserver) MUST match the node-account of the node that the pod is scheduled to.

Once all verification has passed, the approval controller should update the CSR Object with the approval, passing it to the Signing Controller

#### Signing Controller

The signer’s signing controller is responsible for exactly the same verification logic as above. Additionally, the signer should be implemented by copying the relevant information from the CSR to a new template, rather than blindly signing the existing CSR.

The signer’s signing controller is then responsible for updating the CSR Object with the completed certificate. The completed certificate should include not only the direct leaf certificate, but all intermediate certificates required to verify up to the associated trust root. 

The signing controller should be considered to be pluggable; Cloud providers may implement their own, pursuant to the specification, but handling the CA’s key material as they like. Implementations that use HSMs, third-party trust stores like Hashicorp Vault, or cloud provider specific key management solutions are encouraged. Commercial Kubernetes Providers are further encouraged to make this controller optional or tie it to the other controllers described in this doc.

#### Trust Root Object

The trust root object is a new cluster-scoped Object type under the CertificatesV1 objects. Because it is authentication and authorization sensitive, the APIServer should enforce special care around who can update it and what names are allowed. 

The trust root object exists only to provide a CA bundle to the CSI Driver. The CSI Driver is responsible for taking the trust root object and projecting it into the pod’s volume. The CSI is responsible for verifying that the certificates issued by the Signing Controller chain up to the CA Bundle.


### User Stories (Optional)

<!--
Detail the things that people will be able to do if this KEP is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.
-->


#### Story 1

Our example customer uses GRPC to communicate between two pods on a cluster (regardless of namespace).
Client ensures that server provides a valid certificated signed by `kubernetes.io/mlts` CA certificate.

  * The client generates a key for each pod.
  * Those keys are then signed by teh Kubernetes/mtls signer.
  * The keys and certificates are then placed into a secret of type
    `tls` and mounted to the pods.
  * The certificate on the server pod include `service/servciename` as
    a subject alternative name.
  * The certificate on the server pod uniquely identifies the service
    account that the pod is running as.
  * The server is is able to load and serve a TLS authenciated session
    using the certifiates from the secret using standard application
    cryptographic libraries.
  * The client is is able to validate the server's TLS certificate
    using the `kubernetes.io/mlts` CA certificate which also inclued
    in the the secret, again using standard application cryptographic
    libraries.
  * The server is is able to validate the client's provided client
    certificate by using the service account information provided in
    the certificate.


#### Story 2

A user is running a legacy application on Kubernetes, but still requires trust between the pods. The legacy application’s server is configured to use the Workload-certificates provided private key and certificate, but ignores the CA bundle. The client uses the CA bundle to verify that it is talking to the right server, but ignores it’s provided key, and uses protocol-specific password authentication. MTLS is disabled, and only hostname verification (against the un-suffixed service name) is performed, with the SPIFFE ID Ignored. 

### Notes/Constraints/Caveats (Optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above?
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

This `kubernetes.io/mlts` signer is explictly not using the cluster CA
and is decoupled from other signers.


### Risks and Mitigations

<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubernetes ecosystem.

How will security be reviewed, and by whom?

How will UX be reviewed, and by whom?

Consider including folks who also work outside the SIG or subproject.
-->

How will security be reviewed, and by whom?
Extensive unit testing must be written. This is of interest to many companies security departments, and should be 

Much trust is left in the apiserver and kubelet. The ability to schedule pods for execution is the ability to impersonate or leak secrets. The master and kubelet have always had to be trusted to protect secrets, much like the underlying hardware is assumed to be trustworthy.

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->






### Test Plan

<!--
**Note:** *Not required until targeted at a release.*

Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all of the test cases, just the general strategy. Anything
that would count as tricky in the implementation, and anything particularly
challenging to test, should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations). Please adhere to the [Kubernetes testing guidelines][testing-guidelines]
when drafting this test plan.

[testing-guidelines]: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
-->

End to end testing would involve launching pods intended to communicate with each other, returning errors if the certificates failed to validate.

From the node admission standpoint, the mock API Server would expect to receive a certificateSigningRequest after the node was informed of pod admission.

Both the Approval and Signing Controllers should be unit tested with extensive lists of invalid CSRs, including tricky things like null bytes, international domain names and utf-8 character encodings (including invalid codepoints). Fuzzing should be done on their x509 parsing code.


### Graduation Criteria

<!--
**Note:** *Not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, or as something else. The KEP
should keep this high-level with a focus on what signals will be looked at to
determine graduation.

Consider the following in developing the graduation criteria for this enhancement:
- [Maturity levels (`alpha`, `beta`, `stable`)][maturity-levels]
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc
definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning)
or by redefining what graduation means.

In general we try to use the same stages (alpha, beta, GA), regardless of how the
functionality is accessed.

[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

Below are some examples to consider, in addition to the aforementioned [maturity levels][maturity-levels].

#### Alpha -> Beta Graduation

- Gather feedback from developers and surveys
- Complete features A, B, C
- Tests are in Testgrid and linked in KEP

#### Beta -> GA Graduation

- N examples of real-world usage
- N installs
- More rigorous forms of testing—e.g., downgrade tests and scalability tests
- Allowing time for feedback

**Note:** Generally we also wait at least two releases between beta and
GA/stable, because there's no opportunity for user feedback, or even bug reports,
in back-to-back releases.

#### Removing a Deprecated Flag

- Announce deprecation and support policy of the existing flag
- Two versions passed since introducing the functionality that deprecates the flag (to address version skew)
- Address feedback on usage/changed behavior, provided on GitHub issues
- Deprecate the flag

**For non-optional features moving to GA, the graduation criteria must include 
[conformance tests].**

[conformance tests]: https://git.k8s.io/community/contributors/devel/sig-architecture/conformance-tests.md
-->

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to maintain previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to make use of the enhancement?
-->

### Version Skew Strategy

<!--
If applicable, how will the component handle version skew with other
components? What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- Does this enhancement involve coordinating behavior in the control plane and
  in the kubelet? How does an n-2 kubelet without this feature available behave
  when this feature is used?
- Will any other components on the node change? For example, changes to CSI,
  CRI or CNI may require updating that component before the kubelet.
-->

## Production Readiness Review Questionnaire

<!--

Production readiness reviews are intended to ensure that features merging into
Kubernetes are observable, scalable and supportable; can be safely operated in
production environments, and can be disabled or rolled back in the event they
cause increased failures in production. See more in the PRR KEP at
https://git.k8s.io/enhancements/keps/sig-architecture/1194-prod-readiness.

The production readiness review questionnaire must be completed and approved
for the KEP to move to `implementable` status and be included in the release.

In some cases, the questions below should also have answers in `kep.yaml`. This
is to enable automation to verify the presence of the review, and to reduce review
burden and latency.

The KEP must have a approver from the
[`prod-readiness-approvers`](http://git.k8s.io/enhancements/OWNERS_ALIASES)
team. Please reach out on the
[#prod-readiness](https://kubernetes.slack.com/archives/CPNHUMN74) channel if
you need any help or guidance.
-->

### Feature Enablement and Rollback

<!--
This section must be completed when targeting alpha to a release.
-->

###### How can this feature be enabled / disabled in a live cluster?

<!--
Pick one of these and delete the rest.
-->

- [ ] Feature gate (also fill in values in `kep.yaml`)
  - Feature gate name:
  - Components depending on the feature gate:
- [ ] Other
  - Describe the mechanism:
  - Will enabling / disabling the feature require downtime of the control
    plane?
  - Will enabling / disabling the feature require downtime or reprovisioning
    of a node? (Do not assume `Dynamic Kubelet Config` feature is enabled).

###### Does enabling the feature change any default behavior?

<!--
Any change of default behavior may be surprising to users or break existing
automations, so be extremely careful here.
-->

###### Can the feature be disabled once it has been enabled (i.e. can we roll back the enablement)?

<!--
Describe the consequences on existing workloads (e.g., if this is a runtime
feature, can it break the existing applications?).

NOTE: Also set `disable-supported` to `true` or `false` in `kep.yaml`.
-->

###### What happens if we reenable the feature if it was previously rolled back?

###### Are there any tests for feature enablement/disablement?

<!--
The e2e framework does not currently support enabling or disabling feature
gates. However, unit tests in each component dealing with managing data, created
with and without the feature, are necessary. At the very least, think about
conversion tests if API types are being modified.
-->

### Rollout, Upgrade and Rollback Planning

<!--
This section must be completed when targeting beta to a release.
-->

###### How can a rollout fail? Can it impact already running workloads?

<!--
Try to be as paranoid as possible - e.g., what if some components will restart
mid-rollout?
-->

###### What specific metrics should inform a rollback?

<!--
What signals should users be paying attention to when the feature is young
that might indicate a serious problem?
-->

###### Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?

<!--
Describe manual testing that was done and the outcomes.
Longer term, we may want to require automated upgrade/rollback tests, but we
are missing a bunch of machinery and tooling and can't do that now.
-->

###### Is the rollout accompanied by any deprecations and/or removals of features, APIs, fields of API types, flags, etc.?

<!--
Even if applying deprecation policies, they may still surprise some users.
-->

### Monitoring Requirements

<!--
This section must be completed when targeting beta to a release.
-->

###### How can an operator determine if the feature is in use by workloads?

<!--
Ideally, this should be a metric. Operations against the Kubernetes API (e.g.,
checking if there are objects with field X set) may be a last resort. Avoid
logs or events for this purpose.
-->

###### What are the SLIs (Service Level Indicators) an operator can use to determine the health of the service?

<!--
Pick one more of these and delete the rest.
-->

- [ ] Metrics
  - Metric name:
  - [Optional] Aggregation method:
  - Components exposing the metric:
- [ ] Other (treat as last resort)
  - Details:

###### What are the reasonable SLOs (Service Level Objectives) for the above SLIs?

<!--
At a high level, this usually will be in the form of "high percentile of SLI
per day <= X". It's impossible to provide comprehensive guidance, but at the very
high level (needs more precise definitions) those may be things like:
  - per-day percentage of API calls finishing with 5XX errors <= 1%
  - 99% percentile over day of absolute value from (job creation time minus expected
    job creation time) for cron job <= 10%
  - 99,9% of /health requests per day finish with 200 code
-->

###### Are there any missing metrics that would be useful to have to improve observability of this feature?

<!--
Describe the metrics themselves and the reasons why they weren't added (e.g., cost,
implementation difficulties, etc.).
-->

### Dependencies

<!--
This section must be completed when targeting beta to a release.
-->

###### Does this feature depend on any specific services running in the cluster?

<!--
Think about both cluster-level services (e.g. metrics-server) as well
as node-level agents (e.g. specific version of CRI). Focus on external or
optional services that are needed. For example, if this feature depends on
a cloud provider API, or upon an external software-defined storage or network
control plane.

For each of these, fill in the following—thinking about running existing user workloads
and creating new ones, as well as about cluster-level services (e.g. DNS):
  - [Dependency name]
    - Usage description:
      - Impact of its outage on the feature:
      - Impact of its degraded performance or high-error rates on the feature:
-->

### Scalability

<!--
For alpha, this section is encouraged: reviewers should consider these questions
and attempt to answer them.

For beta, this section is required: reviewers must answer these questions.

For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field.
-->

###### Will enabling / using this feature result in any new API calls?

<!--
Describe them, providing:
  - API call type (e.g. PATCH pods)
  - estimated throughput
  - originating component(s) (e.g. Kubelet, Feature-X-controller)
Focusing mostly on:
  - components listing and/or watching resources they didn't before
  - API calls that may be triggered by changes of some Kubernetes resources
    (e.g. update of object X triggers new updates of object Y)
  - periodic API calls to reconcile state (e.g. periodic fetching state,
    heartbeats, leader election, etc.)
-->

###### Will enabling / using this feature result in introducing new API types?

<!--
Describe them, providing:
  - API type
  - Supported number of objects per cluster
  - Supported number of objects per namespace (for namespace-scoped objects)
-->

###### Will enabling / using this feature result in any new calls to the cloud provider?

<!--
Describe them, providing:
  - Which API(s):
  - Estimated increase:
-->

###### Will enabling / using this feature result in increasing size or count of the existing API objects?

<!--
Describe them, providing:
  - API type(s):
  - Estimated increase in size: (e.g., new annotation of size 32B)
  - Estimated amount of new objects: (e.g., new Object X for every existing Pod)
-->

###### Will enabling / using this feature result in increasing time taken by any operations covered by existing SLIs/SLOs?

<!--
Look at the [existing SLIs/SLOs].

Think about adding additional work or introducing new steps in between
(e.g. need to do X to start a container), etc. Please describe the details.

[existing SLIs/SLOs]: https://git.k8s.io/community/sig-scalability/slos/slos.md#kubernetes-slisslos
-->

###### Will enabling / using this feature result in non-negligible increase of resource usage (CPU, RAM, disk, IO, ...) in any components?

<!--
Things to keep in mind include: additional in-memory state, additional
non-trivial computations, excessive access to disks (including increased log
volume), significant amount of data sent and/or received over network, etc.
This through this both in small and large cases, again with respect to the
[supported limits].

[supported limits]: https://git.k8s.io/community//sig-scalability/configs-and-limits/thresholds.md
-->

### Troubleshooting

<!--
This section must be completed when targeting beta to a release.

The Troubleshooting section currently serves the `Playbook` role. We may consider
splitting it into a dedicated `Playbook` document (potentially with some monitoring
details). For now, we leave it here.
-->

###### How does this feature react if the API server and/or etcd is unavailable?

###### What are other known failure modes?

<!--
For each of them, fill in the following information by copying the below template:
  - [Failure mode brief description]
    - Detection: How can it be detected via metrics? Stated another way:
      how can an operator troubleshoot without logging into a master or worker node?
    - Mitigations: What can be done to stop the bleeding, especially for already
      running user workloads?
    - Diagnostics: What are the useful log messages and their required logging
      levels that could help debug the issue?
      Not required until feature graduated to beta.
    - Testing: Are there any tests for failure mode? If not, describe why.
-->

###### What steps should be taken if SLOs are not being met to determine the problem?

## Implementation History

<!--
Major milestones in the lifecycle of a KEP should be tracked in this section.
Major milestones might include:
- the `Summary` and `Motivation` sections being merged, signaling SIG acceptance
- the `Proposal` section being merged, signaling agreement on a proposed design
- the date implementation started
- the first Kubernetes release where an initial version of the KEP was available
- the version of Kubernetes where the KEP graduated to general availability
- when the KEP was retired or superseded
-->

## Drawbacks

<!--
Why should this KEP _not_ be implemented?
-->

Providing the ability to sign certificates with the cluster's CA is fraught with peril. Imporoperly issued certificates can manifest as a variety of ills, including MITM attacks, improper cluster access, and incorrect identity or identity impersonation. These risks are real but they exist through other threat models as well. (Adding these features to Kubernetes addresses those concerns by having a single source of truth, alleviating these concerns.) 

All of this complexity existed in the v1beta1 version of k8s, including being listed as [a feature](https://web.archive.org/web/20181005085009/https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) in previous versions of kubernetes.

## Alternatives

<!--
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

## Infrastructure Needed (Optional)

<!--
Use this section if you need things from the project/SIG. Examples include a
new subproject, repos requested, or GitHub details. Listing these here allows a
SIG to get the process for these resources started right away.
-->
