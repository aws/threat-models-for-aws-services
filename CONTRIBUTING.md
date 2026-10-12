# Contributing Guidelines

Thank you for your interest in contributing to our project. Whether it's a suggested refinement, additional coverage, a new service, or clearer documentation, we greatly value feedback and contributions from our community.

We are unable to accept pull requests for this repository as its content is generated from other sources. Please read through this document before submitting issues to ensure we have all the necessary
information to effectively respond to your contribution.

If you have a question about how to use an AWS service or need further guidance on how to implement the mitigations included in the threat models, you can ask for help on [AWS re:Post](https://repost.aws/), [Stack Overflow](https://stackoverflow.com/questions/tagged/amazon-web-services), or the [AWS Support Center](https://console.aws.amazon.com/support/home#/).


## Submitting feedback and requests

The GitHub issue tracker is the way to suggest improvements to the published threat models and to request additions to them. Because the content is generated, a well-specified issue is more useful to us than a patch would be: tell us which element your suggestion applies to and what it is based on, and we can update it at the source so the change carries into every future release.

Threat models are generated against a specific service model version and reflect service behavior at release time. As services evolve and add features, the applicable mitigations and threats change with them, and a mitigation or threat that applies to most architectures may not fit every one. Suggestions from people using these models in their own environments are how we keep the content current and precise.

### Before you file

* **Check the scope.** Review [threat-models-for-aws-services/reports/mitigation-goals.md](threat-models-for-aws-services/reports/mitigation-goals.md) and [threat-models-for-aws-services/reports/potential-threats.md](threat-models-for-aws-services/reports/potential-threats.md) to confirm the objective or threat class you have in mind is in scope. Workload-specific and application-level threats - application vulnerabilities, business logic flaws, and threats arising from your own code running on AWS - are outside the scope of these models by design. See the [Scope of the threat models](README.md#scope-of-the-threat-models) section of the README for details.
* **Check for duplicates.** Search open and recently closed issues before filing.
* **Check the version.** Compare against the latest release. The `version` value in [threat-models-for-aws-services/manifest.json](threat-models-for-aws-services/manifest.json) identifies what you are looking at, and the `serviceModelUrl` in each threat model identifies the service model version it was generated from.
* **Do not file security vulnerabilities here.** See [Security issue notifications](#security-issue-notifications) below.

### Choosing a template

Issue templates are enabled and blank issues are turned off, so pick the template that matches your suggestion. Each one asks for the specific details we need to act on it.

| Template | Use it when |
|---|---|
| **Threat model content improvement** | You are suggesting a refinement to content that is already published - an update to threats, mitigations, implementations within a published service threat model. |
| **Threat model coverage improvement** | You are suggesting coverage we could add to a published service threat model - an additional threat or mitigation that maps to an existing potential threat and mitigation goal, or another preventive, proactive, detective, or directive implementation. |
| **New service threat model request** | You want a threat model for an AWS service that is not yet listed under `services` in `threat-models-for-aws-services/manifest.json`. |
| **Threat model scope increase request** | You want an additional control objective or threat class brought into scope for *all* services. |
| **Documentation improvement** | The README, design docs, or structure would benefit from clarification or expansion. |

### Writing a useful issue

Two details do the most to speed up a response:

* **Element IDs.** Include the identifiers of the threats, mitigations, or implementations your suggestion applies to - for example `Threat.LAMBDA.Function.12` or `Mitigation.LAMBDA.Function.2`. They are stable across releases and let us locate the exact element.
* **What your suggestion is based on.** Tell us how you established the current service behavior: an API call and its response, a console test, or a link to the relevant AWS documentation, service API model, or condition key reference. Suggestions we can verify directly move fastest.

When proposing a threat, describing it in the same form the threat models use - source, prerequisites, and threat action - makes it directly comparable to existing entries. When proposing a mitigation, state it as a configuration of a service resource and name the mitigation goal it serves.

### Labels

Issues are triaged with the default GitHub labels (`enhancement`, `documentation`, `duplicate`, `question`, `wontfix`) plus a few specific to this repository:

* `threat-model-content` - the content of a published threat model
* `new-service` - a request for coverage of an additional service
* `scope-expansion` - a request affecting the mitigation goals or potential threats assessed across all services


## Code of Conduct
This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct).
For more information see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact
opensource-codeofconduct@amazon.com with any additional questions or comments.


## Security issue notifications
If you discover a potential security issue in this project we ask that you notify AWS/Amazon Security via our [vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/). Please do **not** create a public github issue.


## Licensing

See the [LICENSE](LICENSE) file for our project's licensing.
