# UOR Technical Proposal

# Mission

We want a way to make secure supply chain concepts integral to content management in a universal and accessible way.

# Vision

Normalize supply chain security at scale that serves internal
and external stakeholders.

# Summary

Traceability of software artifacts in supply chains is a long-standing, but increasingly serious security concern. Some ecosystems have made an effort to provide solutions to enhance the verifiability of software artifacts, but often these solutions are not universally applicable.

The purpose of the UOR initiative is to create a scalable cross-content correlation framework that enhances the traceability of decentralized software artifacts.

# Workflow and Critical User Journeys

The mock CLI below describes how a user would interact with a registry using the UOR client. This will also describe workflow for package manager plugins. For the purposes of this example, we will use dnf.

## Publishing an Artifact

- From disk input
```bash
uor-client build artifact registry.example.com/test:1.0
uor-client push registry.example.com/test:1.0 --sign
```
- From registry generated manifest
```bash
uor-client store /path/to/manifest registry.example.com/test:1.0
uor-client push registry.example.com/test:1.0
```

## Resolving attribute queries

```bash
export UOR_REGISTRY_CONFIG=registry-config.yaml
uor-client resolve /path/to/attribute-query
```

## Create a deployment record
```bash
# Adds links on manifest as is/ This can be used to
# continuously updated dependencies and create new deployment records.
uor-client create aggregate /path/to/config 

# Resolve linked attribute queries into manifest descriptor.
uor-client create aggregate /path/to/config --freeze

# This will generate an immutable deployment record and a corresponding
# software inventory that can be attached.
uor-client create deployment-record registry.example.com/test:1.0
```

## Generate Software Inventory
```bash
uor-client create inventory registry.example.com/test:1.0 --format spdx
```

## Consuming an artifact or aggregate
```bash
uor-client pull registry.example.com/test:1.0 --follow-links
```

## Through the App Proxy
```bash
# Start app proxy server with CLI
uor-client serve /var/run/uor.sock
# With systemd
systemctl start uor-manager
# With dnf plugin
dnf install httpd
# Find with fuse driver
uor-fuse ls
...
httpd
```

# High-Level Plan

## Phases

### POC (Pre-MVP)
- Schema Validation
- Attestations
- Software Inventory Automation
- Signature Verification
- Manifest/Blob Signing
- Registry Attribute Queries
- Linked Artifacts
- Aggregates
### Crawl (MVP)
- Search Domains
- Deployment Records
- Package Manager Compatability (i.e. app proxy)
- Host Policy Service
- Enable SLSA Level 2 Compliance
- Schema Cataloging
### Walk (Beyond MVP)
- UOR Native Package Managers
- Registry Gateway Policy Service
- Change Management
- Enable SLSA Level 3 Compliance
### Run (Beyond MVP)
- RBAC/ABAC hybrid capability
- Attribute and Blob Encryption
- Enable SLSA Level 4 Compliance

> Note: SLSA compliance levels would be achieved when UOR is integrated with a hosted build service that produces attestable build metadata.

## Steps

1. Create a spec to serve as an extension of the OCI artifact spec. There will be an option to be compatible with the OCI artifact spec as is.
2. Define an initial attribute extension for the distribution spec
3. Fork Quay and add functionality that will support the distribution spec
4. Develop a local solution that can enable compatibility with v2 registries
5. Create a policy engine solution that apply logic to operations based on attributes
6. Clarify extension spec following lessons learned from Quay fork

## Required Components

- Server-side (registry) distribution implementation
- CLI/Client Libraries
- Local Storage Solution with embedded database
- Single tool for service management
- Services:
    - Policy Engine 
      - Host Level Policy Engine
      - Policy Engine as a Registry Gateway
    - Artifact Manager

### Upstreams
- sigstore/cosign
- in-toto
- oras
- anchore syft

## Distribution Spec Changes
The goals for distribution spec changes is to build an extension to the existing OCI distribution spec to allow for cross-repo/cross-namespace attribute querying.

With query optimization and backward compatability in mind, we have the following requirements:
- This must retain the V2 structure with standalone blob storage
- This will need an indexing service to optimize certain types of queries (e.g. attributes)
### RBAC
The multi-tenant design of container registries will not change with the addition of attribute queries. Repository-scoped permissions should still be used and when returning artifact results, the result set should be filtered by what is viewable to the user. In other words, owners will be able to choose to make artifacts public.
### Query scoping
On the client side, scoping queries to namespace and repositories will be supported.
## CLI/Client Libraries
We have existing CLI and library code bases that manage single and linked artifacts. These will continue to be used to publish single artifact and manage local storage. The CLI/libraries will be extended to interface with the proposed API changes.
## Local Storage Solution
- V2 Compatability
- Mimics V3 functionality using an embedded database
- Descriptors are stored in the database and used to create aggregates that can be published as a single unit.
## Service Manager
- Manages running gRPC services
### Service - Policy Engine
> Question: Could this be an OPA Plugin?

Component information or any attestations on content add little to no value if that information is not used for security gating. The policy engine allows users to configure and enforce policy based on the existence of these artifacts and their contents.

### Service - Artifact Manager
The purpose of the artifact manager service is to act as a proxy for existing package managers and installation tooling. This can also be used for clients that do not have knowledge of artifact publishing and manipulation.

## UOR Concepts

### Attribute Schemas

Schema Type is part of an attribute set. Each schema has an id and in order for an artifact to be compliant, it must have schema type and value declarations that matches the validating schema. Referencing schema by the ID in the manifest allows the schema source to come from anywhere (i.e. remote reference, a local file, structs in code).

Schema enables consistency and shared attribute meaning for artifact publishers. Schemas can also be used in the generation of other configuration files such as a dataset configuration file or an attribute query.

Each published schema will be discoverable by its attributes which will identification, description, and category.

#### Core Schema

UOR has a core schema that will be validated during publish time. All artifacts will require this information to be consumable. Part of the core schema include component information.

### Multi-Schema

Option 1: The schema type would be an array of schema IDs as defined by the publisher.
Option 2: The publisher defines core schema and schemas can be registered with the Artifacts Manager to add context to an operation. The schemas would be added to a schema pool along with the core schema.
Option 3: Combination of both.

### Aggregates

The attribute query endpoint can be used to automate building application from existing artifacts. The endpoint will resolve an attribute query to an index manifest of matching descriptors that can be used for artifact publishing.

To enable efficiency in this workflow, it would be best practice to only publish one component or file per artifact. However, if an artifact is published with more than one file, the UOR client will allow attribute filtering on individual artifacts.

### Search Domains

A content query can be sent to multiple OCI registries. A user can specify which registries to query and prioritize registry results.
If an artifact or aggregate references a link that cannot be resolved by any registry in the search domain, a field called `registryHint` will be
used to add the registry will the required content to a discovered zone.

### Reference Types/Links

#### OCI Reference types

According to the OCI Referrers spec, referrers using a tag schema to declare relationships in the
`subject` field. When querying the referrer's API, an index manifest response is generated. This type of workflow could be used when creating supporting artifacts for artifact that would only ever reference
one artifact as the parent.

An important note here is that a referring manifest can only ever have one subject. Meaning this relationship takes the form of a tree structure.

##### Use Case

```mermaid
graph LR;
A[Signature]-->B[Artifact];
C[Attestation] --> B
D[Other] --> B
```

#### Artifact Link

Links are a UOR specific type and differ from references because they define relationships that can be cross repo. These links can also form a DAG structure.

https://github.com/uor-framework/uor-client-go/blob/main/docs/design/nodes.md#linked-collection

#### Methods

- Option 1: We could link artifact references directory in the artifact spec.
    - Pros
        - Allows the publisher to define artifact links without additional steps.
    - Cons
        - Rate of Change: Will an artifact change everytime a link does?
        - Non-publisher cannot create artifact links
- Option 2: Maintain an index manifest in each repo to track relationships. This would be created from running an attribute query. It would produce an index of attribute resolved links.
    - Pros
        - Anyone can link two artifacts and keep them update to date easily using the attribute API
    - Cons
        - Another manifest reference will have to be maintained. Scaling this could get complex from a lifecycle management perspective.

##### Use Cases

```mermaid
graph LR;
A[Terraform main.tf]-->B[AWS Provider];
A --> C[Terraform Module 1]
A --> D[Terraform Module 2]
E[Another Terraform main.tf] --> B
E --> D
```

```mermaid
graph LR;
A[CVE Artifact]--Links-->B[CPE Attribute]
B--Resolve-->C[Artifact Digest Reference]
```

## Deployment Records
Fully resolved immutable artifact indexes

## Supply Chain Security Areas

### Immutable/Mutable References

Our goal is to make artifacts tamper-proof without relying solely on
immutable references. Artifacts will, of course, always have a digest
but pulling by digest would not be a requirement to ensure the artifact contains the expected content.

**Why don't we want to use immutable references as the main security control?**

Using immutable references make upgrading more software difficult. Without signatures, using immutable references just shifts risk from one party to another because using immutable references do not provide proof of the publisher identity (i.e. non-repudiation).

Where is immutable reference usage important?

- Build reproducibility
- TOCTOU (Time of Check, Time of Use)

Solution: [Deployment Records](#Deployment Records)

### Signatures
Signatures are a better way to ensure the artifact has not been
tampered with and come from a trusted entity. Blobs must be individually signed so that they can be referenced as standalone descriptors. This functionality will come from the use of cosign which already supports OCI artifacts.

### Software Inventory

Software inventory fields are part of the core UOR schema and are required to be present in the manifest attributes. 
This information will be listed in artifact manifest attributes to assure they are queryable with the attribute searching API.
Software inventory information will be dynamically constructed for aggregates based on linked manifests and attribute results.
The software inventory will contain the required information for conversion into commonly adopted formats like SPDX and CycloneDX.

Signed Inventories - Each artifact will contain component information that will be stored in attributes and attestation predicate.
The artifact and attestation will be signed.

**Why are the attributes stored in the manifest and not as a separate artifact?**
    - Where attributes are stored should depend on their rate of change. Storing core attributes like component and build metadata in the manifest ensures that if they are changed the digest will change.
    - It will result in less calls to the referrer's API

#### Streamlined Access to Software Inventories

A requirement for security teams that manage many products is streamlined software inventory management.
This is possible with the proposed solution by allowing users to configure search domains (e.g. queryable registry endpoints) for content. Using the component schema and product schema, queries can be constructed to easily find application specific software inventories. This allows the storage of components to remain decentralized for distributed teams while still providing one method for content retrieval.

### Attestations
We will support in-toto attestation.
One core attestation during artifact publishing will include the artifact as the statement and the attributes as the predicate.
More information [here](https://github.com/in-toto/attestation/blob/main/spec/README.md)

Attestations from user input will be accepted during artifact publishing.

Tooling will be required to fail closed if attestations are not present.

### Vulnerability Management
- Attribute queries
- Auto-upgrades
- CVE Linking

### Transparency Logging
> TODO

# Diagrams

## System Context

```mermaid
graph LR;
A[UOR User]-->B[Client]
B --publishing and retrieval--> C[Artifact Service]
B --policy enforcement--> D[Host Policy Service]
C --storage and queries--> E[Registry]
E --> F[Organization Policy Service]
```


```mermaid
sequenceDiagram;
participant Client
participant Host Policy Service
participant Artifact Service
participant Registry
Client->>Host Policy Service: Check attributes
Host Policy Service-->>Client: Artifact Meets Criteria
Client->>Artifact Service: Publishing Operation
Artifact Service->>Registry: Publishing to Registry
Registry-->>Artifact Service: Success
Artifact Service-->>Client: Success
```

```mermaid
sequenceDiagram;
participant Client
participant Organization Policy Service
participant Artifact Service
participant Registry
Client->>Artifact Service: Publishing Operation
Artifact Service->>Registry: Publishing to Registry
Registry->>Organization Policy Service: Check attributes
Organization Policy Service-->>Registry: Meets criteria
Registry->>Registry: Store artifact
Registry-->Artifact Service: Success
Artifact Service-->>Client: Success
```

### Client

#### Searching

```mermaid
sequenceDiagram;
participant User
participant Client
participant Registry
User->>Client: Sending attribute query
Client->>Client: Reconstruct query for Registry API
Note left of Client: Use the search domain to determine which order to query the registries.
Client->>Registry: Send modified query
Registry-->>Client: Constructed and fully resolved aggregate
Client-->>User: Merged query results from search domain
```

### Registry

#### Components

##### Container Registry Service

#### Workflows

##### Publishing
```mermaid
sequenceDiagram;
participant Client
participant API Endpoint
participant Ingestion Service
participant Manifest Scraper
participant Database
Client->>API Endpoint: User Artifact
API Endpoint->>Ingestion Service: User Collectin
Ingestion Service-->>API Endpoint: Writen to disk
API Endpoint-->>Client: Written to digest X
Note right of Manifest Scraper: Triggered by hook or schedule?
Manifest Scraper->>Manifest Scraper: Scrape attributes from manifest
Manifest Scraper->>Database: Write attribute data to database
Note right of Manifest Scraper: Optional Component
```

##### Consumption
```mermaid
sequenceDiagram;
participant Client
participant Attributes API
participant Database
Client->>Attributes API: Attribute Query
Attributes API->>Database: Database Query
Database-->>Attributes API: Result Set
Attributes API-->>Client: Aggregate
```

### Artifact Service
> TODO

### Policy Service
> TODO

# Glossary

- Artifact - A file represented as an OCI artifact schema: the properties and data types that can be specified within an artifact.
- Aggregates - An aggregate that is constructed from attribute queries from existing artifacts to form a larger application.
- Dataset Configuration - A configuration file used during artifact publishing to configure file attributes, associate schema, and link existing artifacts.
- Attribute Query - A configuration file used with the UOR CLI that defines key and values to filter artifact contents during pulling operations.
- Software Inventory - An inventory list of all the software artifacts that comprise an artifact or aggregate.
- Multi-Schema Artifact - An artifact with an attribute set that satisfies multiple attribute schemas.

# Reference

Birkholz, H., Delignat-Lavaud, A., & Fournet, C. (2022). An Architecture for Trustworthy and Transparent Digital Supply Chains draft-birkholz-scitt-architecture-00. *Internet Engineering Task Force*. <https://www.ietf.org/archive/id/draft-birkholz-scitt-architecture-00.txt>