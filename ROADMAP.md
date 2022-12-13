UOR Project Roadmap
====================

### How should I use this document?

This document provides description of items that the project decided to prioritize. Each item is ranked and will
be completed in the ranked order. This should serve as a reference point for contributors to understand where 
the project is going, and help determine if a contribution could be conflicting with some longer term plans.

### How can I get involved?

Our [projects](https://github.com/orgs/uor-framework/projects) show upcoming work for the
various projects under the organization and is a good way to see what issue are being tackled. 
We encourage all types of contributions, but please be sure to comment on issues or 
communicate with the maintainers to avoid duplicated efforts.

#### How can I add something to the roadmap?

UOR is an emerging technology so the road-mapping process is not well-defined yet.

To propose a new feature or discuss the design we encourage participation in GitHub Discussions 
on the main `uor-framework` repository in the `uor-framework` organization.

# 1. Package Management Plugins

The UOR team is currently working toward universal content management
and focusing on integrating existing packaging format is a way to encourage collection
publishing and help add functionality to existing package management workflows.

## 1.1 Pip Plugin

Integration with `pip`:
1. Complete initial design proposal
2. Refine gRPC API as needed. Related [issue](https://github.com/uor-framework/uor-client-go/pull/90) in `uor-client-go`.
3. Define required package management schemas
4. Implementation

# 2. Package Format Ingestion

The UOR team is working on collecting metadata for packages hosted in other locations.
This task focuses on creating a solution to discover new packages and ingest the metadata.

## 2.1 RPM Ingestion

1. Complete initial design proposal and planning
2. Design relevant schemas
3. Create integration services for automated publishing
4. Accept provenance data during publishing
5. Ingest dependency information

