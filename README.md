> [!IMPORTANT]
>
> The `main` branch hosts live code with latest changes. It is unstable and is used for development.
> It is suitable for contribution and inspection of the latest code. The `release-*` branches are
> stable releases that can be used to build and deploy the system.

---

> FLEDGE has been renamed to Protected Audience API. To learn more about the name change, see the
> [blog post](https://privacysandbox.com/intl/en_us/news/protected-audience-api-our-new-name-for-fledge).

---

# ![Privacy Sandbox Logo](docs/assets/privacy_sandbox_logo.png) Protected Auction Key/Value service

# State of the project

The current codebase represents the implementation of the TEE-based Key/Value service by Privacy
Sandbox.

For
[Protected Audience](https://developers.google.com/privacy-sandbox/private-advertising/protected-audience),
the service can be used as a BYOS KV server. Soon it can be used to communicate with Chrome and the
Bidding and Auction services using
[V2 protocol](https://github.com/WICG/turtledove/blob/main/FLEDGE_Key_Value_Server_API.md).

For
[Protected App Signals](https://developers.google.com/privacy-sandbox/private-advertising/protected-audience/android/protected-app-signals),
the service should be used as the ad retrieval server.

It can be integrated with Chrome and Android with the
[Privacy Sandbox unified origin trial](https://developer.chrome.com/blog/expanding-privacy-sandbox-testing/)
and
[Privacy Sandbox on Android Developer Preview](https://developer.android.com/design-for-safety/privacy-sandbox/program-overview).

## Current features

### Build and deployment

-   Source code is available on Github
-   Releases are done on a regular basis
-   Binaries can be built from source code
    -   C++ binary
    -   [AWS & GCP] Docker container image
    -   [AWS]: Nitro EIF
    -   [AWS]: Reference AMI
    -   Other tools
-   Server can run as a standalone local process for testing without any cloud dependency or
    TEE-related functionality
-   Server can be deployed to AWS Nitro enclave
-   Server can be deployed to GCP Confidential Space
-   Reference terraform available for a clean and comprehensive deployment to AWS or GCP
    -   Clean: assumes the cloud environment has no preset infrastructure
    -   Comprehensive: deploys all dependencies and some recommended (but not necessarily required)
        configuration
    -   Many server behaviors can be configured by parameter flags without having to rebuild

### Data loading

-   Server loads key/value data from cloud file storage
-   Server loads key/value data from cloud pub/sub services
-   Server loads data into an in-RAM representation for query serving
-   Server continuously monitors for new data and incrementally updates ("delta files") the in-RAM
    representation
-   Support independent data ingestion pipelining by monitoring directories in cloud file storage
    independently
-   Supports Flatbuffers as the data event format
-   Supports Avro and Riegeli as the data file format
-   Supports snapshot files for faster server start up
-   Users can perform compactions of delta files into snapshot files in an offline path

### Read request processing

-   Support Protected Audience Key Value Server query spec: can be used as a BYOS server to serve
    requests from Chrome
-   Support simple key value lookups for queries
-   Users can write "user defined functions" to execute custom logic to process queries
-   User defined functions can be written in JavaScript or WASM
-   User defined functions can call "GetValues" to look up key value from the dataset

### Advanced features

-   Set-as-a-value is supported
    -   A key "value" pair in the dataset can be a key and a set of values
-   UDF can call "RunQuery" API to run set operations on sets (intersection, union, difference)
-   For GCP, Terraform supports deploying into an existing VPC, such as used by the Bidding and
    Auction services Non-prod Server logs are persisted after server shutdown
-   Data can be sharded and different servers may load and serve different shards (subset) of the
    dataset.
-   Sharding supports data locality, where the operator specifies "sharding key" for key value pairs
    so different key value pairs can have the same sharding key.

## **Timeline and roadmap**

The following sections include the timelines for the Trusted Key Value Server for Protected
Auctions. Protected Auctions refer to Protected Audiences and Protected App Signals ad targeting
products.

### **Timelines**

<!-- markdownlint-disable no-inline-html -->
<!-- markdownlint-disable line-length -->

<table>
  <tr>
   <td>
   </td>
   <td><strong>Beta testing</strong>
   </td>
   <td><strong>General availability</strong>
   </td>
   <td><strong>Enforcement</strong>
   </td>
  </tr>
  <tr>
   <td>For Protected Audience
<p>
(web browser on desktop)
   </td>
   <td>July 2024
<p>
The Privacy-Sandbox-provided Key Value Server implementation can<ul>

<li>run as a BYOS KV server
<li>support production scale traffic and common functionalities</li></ul>

   </td>
   <td>Q4 2024
<p>
Opt-in TEE mode will be available to the Adtechs. Opt-in guidance will be published in early Q4 2024.
   </td>
   <td><a href="https://developers.google.com/privacy-sandbox/relevance/protected-audience-api/feature-status#feature_availability_timeline">No sooner than Q3 2025</a>
   </td>
  </tr>
</table>

<table>
  <tr>
   <td>
   </td>
   <td><strong>Beta testing</strong>
   </td>
   <td><strong>General availability</strong>
   </td>
  </tr>
  <tr>
   <td>For Protected Audience
<p>
(With Bidding & Auction services for Chrome or Android)
   </td>
   <td>July 2024
<p>
The Privacy-Sandbox-provided Key Value Server implementation can be used with the Bidding and Auction services and<ul>

<li>run as a BYOS KV server
<li>support production scale traffic and common functionalities</li></ul>

   </td>
   <td>Dec 2024
<p>
The Privacy-Sandbox-provided Key Value Server implementation can be used with the Bidding and Auction services and adtechs can opt-in TEE mode
   </td>
  </tr>
  <tr>
   <td>For Protected App Signals
   </td>
   <td>June 2024
<p>
The Privacy-Sandbox-provided Key Value Server implementation supports Ad retrieval server functionality and protected communication for live traffic testing
   </td>
   <td>Dec 2024
<p>
The implementation supports live traffic at scale
   </td>
  </tr>
</table>

<!-- markdownlint-enable no-inline-html -->
<!-- markdownlint-enable line-length -->

### **Roadmap**

#### June 2024 Beta release

##### Deployment and Setup

-   For AWS, Terraform supports deploying into an existing VPC, such as the one that is used by the
    Bidding and Auction services
-   Internal load balancer is used for servers to send metrics to OpenTelemetry collector
    -   In v0.16, the communication goes through a public load balancer

##### Integration with the Bidding & Auction services

-   The Bidding and Auction services can send encrypted requests to the Key Value Server for
    Protected App Signals

##### Debugging support

-   [Consented Debugging](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/debugging_protected_audience_api_services.md#adtech-consented-debugging)
    is supported
-   Diagnose tool to check the cloud environment to warn for potential setup errors before the
    system is deployed
-   Operational playbook
-   Introduction of unsafe metrics
    -   Unsafe metrics have privacy protections such as differential privacy noises
    -   More metrics for comprehensive SLO monitoring

##### Runtime features

-   Data loading error handling
    -   The system can be configured to use different error handling strategy for different dataset

##### Performance/Cost

-   Benchmarking tools
-   Cost explainer
-   Sets-as-values will switch to using bitsets to represent sets for faster RunQuery performance.

##### Support process

-   Commitment to support window for active releases

#### Q4 2024 Chrome-PA GA

##### Chrome integration

-   Update to V2 protocol to support the hybrid mode of BYOS & Opt-in TEE
-   Chrome and Key Value server can communicate in the updated V2 protocol
-   Chrome can send full publisher URL to TEE KV server under V2 protocol

#### H2 2024 Android-PA GA, PAS GA

##### User Defined Functions

-   UDF can perform Key/Value lookup asynchronously
-   Flags can be passed from the server parameters into UDF
-   One Key Value Server system can be used for multiple use cases. Multiple UDFs can be loaded.
    Different UDF can be selected based on the request type.
-   Canaring support for UDF: canary version UDF can be staged in machines with specific tags.

##### Customization support

-   First class support for customization of the system (without violating the trust model)

##### Debugging support

-   Diagnose tool to collect standard and necessary debug information for troubleshooting requests

##### Documentation

-   Complete end to end example as a template to set up the service

## Breaking changes

Backward-incompatible changes are expected to be rare and will result in a major version change.

# Key documents

-   [FLEDGE services overview](https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md)
-   [FLEDGE K/V server API explainer](https://github.com/WICG/turtledove/blob/main/FLEDGE_Key_Value_Server_API.md)
-   [FLEDGE K/V server trust model](https://github.com/privacysandbox/fledge-docs/blob/main/key_value_service_trust_model.md)
-   [Local server quickstart guide](/docs/developing_the_server.md)
-   [AWS server user deployment documentation](/docs/deployment/deploying_on_aws.md)
-   [GCP server user deployment documentation](/docs/deployment/deploying_on_gcp.md)
-   [Integrating the K/V server with FLEDGE](/docs/protected_audience/integrating_with_fledge.md)
-   [FLEDGE K/V server sharding explainer](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/key_value_service_sharding.md)
-   Operating documentation
    -   [Data loading API and operations](/docs/data_loading/loading_data.md)
    -   [Generating and loading UDF files](/docs/generating_udf_files.md)
    -   Error handling explainer (_to be published_)
-   Developer guide
    -   [Codebase structure](/docs/repo_layout.md)
    -   [Working with Terraform](/docs/deployment/working_with_terraform.md)
    -   [Contributing to the codebase](/docs/contributing.md)
-   [Code of conduct](/docs/CODE_OF_CONDUCT.md)
-   [Change log](/CHANGELOG.md)

# Contribution

Contributions are welcome, and we will publish more detailed guidelines soon. In the meantime, if
you are interested,
[open a new Issue](https://github.com/privacysandbox/protected-auction-key-value-service/issues) in
the GitHub repository.

# Feedback

The FLEDGE K/V feature set, API and system design are under active discussion and subject to change
in the future. If you try this API and have feedback, we'd love to hear it:

-   **GitHub**: For questions or feedback that are relevant to this proposal, open a new Issue in
    this repository. For questions that are purely relevant to browser-side logic,
    [open an Issue in the Turtledove repository](https://github.com/WICG/turtledove/issues).
-   **W3C:** You are welcome to
    [attend the FLEDGE WICG meeting](https://github.com/WICG/turtledove/issues/88).
-   **Developer support**: Ask questions and join discussions on
    -   [Privacy Sandbox for the Web Developer Support repo](https://github.com/GoogleChromeLabs/privacy-sandbox-dev-support)
    -   [Privacy Sandbox on Android issue tracker](https://issuetracker.google.com/issues/new?component=1116743&template=1642575)

# Announcements

## General announcements

General announcements will be made on
[the FLEDGE mailing list](https://groups.google.com/a/chromium.org/g/fledge-api-announce/) on
chromium.org and the
[Privacy Sandbox progress update page](https://developer.android.com/design-for-safety/privacy-sandbox/progress-updates/latest)
on the Android developer site, if needed, an accompanying article will be published on the
[Chrome Developers blog](https://developer.chrome.com/tags/privacy/) and
[Android Developers blog](https://android-developers.googleblog.com/).

---

[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/privacysandbox/fledge-key-value-service/badge)](https://securityscorecards.dev/viewer/?uri=github.com/privacysandbox/fledge-key-value-service)

# Key Management System Architecture

This repository contains comprehensive documentation and implementation details for a secure Key Management System (KMS) architecture. The system is designed to manage the entire lifecycle of cryptographic keys within the Protected Auction Key/Value Service.

## Overview

The Key Management System provides:
- Secure key generation and storage
- Safe key distribution
- Access control management
- Key rotation and retirement
- Security monitoring and compliance

## Documentation Structure

- [Key Management Architecture](docs/key_management_architecture.md)
  - System Components
  - Key Hierarchy
  - Security Implementation
  - End-to-End Flows
  - Operational Procedures
  - Monitoring and Maintenance
  - Use Cases and Examples
  - Architecture Diagrams

## Features

- Zero trust architecture
- Defense in depth
- Principle of least privilege
- Complete audit trail
- Automated key rotation
- Hardware Security Module (HSM) integration
- Trusted Execution Environment (TEE) support

## Getting Started

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/key-management-system.git
   cd key-management-system
   ```

2. Review the documentation:
   - Start with the [Key Management Architecture](docs/key_management_architecture.md)
   - Explore the diagrams and use cases
   - Check implementation details and examples

## Security Considerations

This system implements multiple security layers:
- Hardware Security Modules (HSM)
- Trusted Execution Environment (TEE)
- Secure key storage and distribution
- Access control and audit logging
- Real-time monitoring and alerts

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contact

For questions and support, please contact:
- Security Team: security-ops@example.com
- System Administrators: kms-admin@example.com
- Emergency Response: +1-xxx-xxx-xxxx
