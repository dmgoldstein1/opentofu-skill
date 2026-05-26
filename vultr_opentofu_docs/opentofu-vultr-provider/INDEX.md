# OpenTofu Vultr Provider Documentation Index

This directory contains comprehensive documentation for the Vultr provider for OpenTofu v2.31.2 (latest version as of May 12, 2026).

## Overview

- **Provider**: vultr/vultr
- **Owner**: [Vultr](https://github.com/vultr)
- **Repository**: [vultr/terraform-provider-vultr](https://github.com/vultr/terraform-provider-vultr)
- **License**: MPL-2.0
- **Latest Version**: v2.31.2 (Published: 5/12/2026)
- **Source Registry**: https://search.opentofu.org/provider/vultr/vultr/latest

For general provider information and configuration, see [00_provider_overview.md](./00_provider_overview.md).

---

## Resources (51)

Resources are used to create, manage, and destroy infrastructure with Vultr.

### Compute Resources
- [bare_metal_server](./resources/01_bare_metal_server.md) - Bare Metal Server resource
- [instance](./resources/17_instance.md) - Cloud Compute Instance resource
- [instance_ipv4](./resources/18_instance_ipv4.md) - Instance IPv4 resource

### Storage Resources
- [block_storage](./resources/02_block_storage.md) - Block Storage volume
- [object_storage](./resources/26_object_storage.md) - Object Storage subscription
- [object_storage_bucket](./resources/27_object_storage_bucket.md) - Object Storage bucket
- [virtual_file_system_storage](./resources/49_virtual_file_system_storage.md) - Virtual File System Storage

### Networking Resources
- [vpc](./resources/50_vpc.md) - Virtual Private Cloud (VPC)
- [vpc2](./resources/51_vpc2.md) - Virtual Private Cloud v2
- [firewall_group](./resources/14_firewall_group.md) - Firewall group
- [firewall_rule](./resources/15_firewall_rule.md) - Firewall rule
- [nat_gateway](./resources/23_nat_gateway.md) - NAT Gateway
- [nat_gateway_firewall_rule](./resources/24_nat_gateway_firewall_rule.md) - NAT Gateway Firewall Rule
- [nat_gateway_port_forwarding_rule](./resources/25_nat_gateway_port_forwarding_rule.md) - NAT Gateway Port Forwarding Rule
- [load_balancer](./resources/22_load_balancer.md) - Load Balancer
- [reserved_ip](./resources/41_reserved_ip.md) - Reserved IP
- [reverse_ipv4](./resources/42_reverse_ipv4.md) - Reverse IPv4
- [reverse_ipv6](./resources/43_reverse_ipv6.md) - Reverse IPv6

### DNS Resources
- [dns_domain](./resources/12_dns_domain.md) - DNS Domain
- [dns_record](./resources/13_dns_record.md) - DNS Record

### Database Resources
- [database](./resources/04_database.md) - Managed Database
- [database_connection_pool](./resources/05_database_connection_pool.md) - Database Connection Pool
- [database_connector](./resources/06_database_connector.md) - Database Connector
- [database_db](./resources/07_database_db.md) - Database (schema)
- [database_quota](./resources/08_database_quota.md) - Database Quota
- [database_replica](./resources/09_database_replica.md) - Database Replica
- [database_topic](./resources/10_database_topic.md) - Database Topic
- [database_user](./resources/11_database_user.md) - Database User

### Kubernetes Resources
- [kubernetes](./resources/20_kubernetes.md) - Kubernetes Cluster
- [kubernetes_node_pools](./resources/21_kubernetes_node_pools.md) - Kubernetes Node Pools

### Container Resources
- [container_registry](./resources/03_container_registry.md) - Container Registry
- [inference](./resources/16_inference.md) - Inference (AI/ML) resource

### Snapshot Resources
- [snapshot](./resources/44_snapshot.md) - Snapshot
- [snapshot_from_url](./resources/45_snapshot_from_url.md) - Snapshot from URL
- [iso](./resources/19_iso.md) - ISO image

### Identity & Access Resources
- [organization](./resources/31_organization.md) - Organization
- [organization_group](./resources/32_organization_group.md) - Organization Group
- [organization_policy](./resources/33_organization_policy.md) - Organization Policy
- [organization_policy_group_attachment](./resources/34_organization_policy_group_attachment.md) - Organization Policy Group Attachment
- [organization_policy_user_attachment](./resources/35_organization_policy_user_attachment.md) - Organization Policy User Attachment
- [organization_role](./resources/36_organization_role.md) - Organization Role
- [organization_role_group_attachment](./resources/37_organization_role_group_attachment.md) - Organization Role Group Attachment
- [organization_role_policy_attachment](./resources/38_organization_role_policy_attachment.md) - Organization Role Policy Attachment
- [organization_role_session](./resources/39_organization_role_session.md) - Organization Role Session
- [organization_role_trust](./resources/40_organization_role_trust.md) - Organization Role Trust
- [oidc_issuer](./resources/28_oidc_issuer.md) - OIDC Issuer
- [oidc_provider](./resources/29_oidc_provider.md) - OIDC Provider
- [oidc_token](./resources/30_oidc_token.md) - OIDC Token

### User & Key Management
- [ssh_key](./resources/46_ssh_key.md) - SSH Key
- [startup_script](./resources/47_startup_script.md) - Startup Script
- [user](./resources/48_user.md) - User account

---

## Data Sources (42)

Data sources are used to fetch information about existing Vultr infrastructure.

### Account & Organization Data Sources
- [account](./data-sources/01_account.md) - Account information
- [organization](./data-sources/26_organization.md) - Organization data
- [organization_group](./data-sources/27_organization_group.md) - Organization Group data
- [organization_policy](./data-sources/28_organization_policy.md) - Organization Policy data
- [organization_role](./data-sources/29_organization_role.md) - Organization Role data

### Compute Data Sources
- [bare_metal_plan](./data-sources/04_bare_metal_plan.md) - Bare Metal plans
- [bare_metal_server](./data-sources/05_bare_metal_server.md) - Bare Metal servers
- [instance](./data-sources/12_instance.md) - Cloud Compute instance
- [instance_ipv4](./data-sources/13_instance_ipv4.md) - Instance IPv4 addresses
- [instances](./data-sources/14_instances.md) - List of instances
- [plan](./data-sources/31_plan.md) - Pricing plans
- [region](./data-sources/32_region.md) - Available regions

### Storage Data Sources
- [block_storage](./data-sources/06_block_storage.md) - Block Storage volumes
- [object_storage](./data-sources/20_object_storage.md) - Object Storage subscriptions
- [object_storage_cluster](./data-sources/21_object_storage_cluster.md) - Object Storage clusters
- [object_storage_tier](./data-sources/22_object_storage_tier.md) - Object Storage tiers
- [virtual_file_system_storage](./data-sources/40_virtual_file_system_storage.md) - Virtual File System Storage

### Networking Data Sources
- [vpc](./data-sources/41_vpc.md) - Virtual Private Cloud
- [vpc2](./data-sources/42_vpc2.md) - Virtual Private Cloud v2
- [firewall_group](./data-sources/10_firewall_group.md) - Firewall groups
- [load_balancer](./data-sources/18_load_balancer.md) - Load Balancers
- [reserved_ip](./data-sources/33_reserved_ip.md) - Reserved IPs
- [reverse_ipv4](./data-sources/34_reverse_ipv4.md) - Reverse IPv4 entries
- [reverse_ipv6](./data-sources/35_reverse_ipv6.md) - Reverse IPv6 entries

### DNS Data Sources
- [dns_domain](./data-sources/09_dns_domain.md) - DNS domains

### Database Data Sources
- [database](./data-sources/08_database.md) - Managed Databases

### Kubernetes Data Sources
- [kubernetes](./data-sources/17_kubernetes.md) - Kubernetes clusters

### Container Data Sources
- [container_registry](./data-sources/07_container_registry.md) - Container registries
- [inference](./data-sources/11_inference.md) - Inference resources

### Image & Application Data Sources
- [application](./data-sources/02_application.md) - Applications
- [iso_private](./data-sources/15_iso_private.md) - Private ISO images
- [iso_public](./data-sources/16_iso_public.md) - Public ISO images
- [os](./data-sources/30_os.md) - Operating systems

### Snapshot Data Sources
- [snapshot](./data-sources/36_snapshot.md) - Snapshots
- [backup](./data-sources/03_backup.md) - Backups

### OIDC Data Sources
- [oidc_discovery](./data-sources/23_oidc_discovery.md) - OIDC Discovery
- [oidc_provider](./data-sources/24_oidc_provider.md) - OIDC Provider
- [oidc_issuer](./data-sources/25_oidc_issuer.md) - OIDC Issuer

### User Data Sources
- [ssh_key](./data-sources/37_ssh_key.md) - SSH keys
- [startup_script](./data-sources/38_startup_script.md) - Startup scripts
- [user](./data-sources/39_user.md) - User accounts

### Monitoring Data Sources
- [logs](./data-sources/19_logs.md) - Log data

---

## Quick Links

- **Official Provider Repository**: https://github.com/vultr/terraform-provider-vultr
- **OpenTofu Registry**: https://search.opentofu.org/provider/vultr/vultr/latest
- **Vultr API Documentation**: https://www.vultr.com/docs/api/
- **OpenTofu Documentation**: https://opentofu.org/docs/

---

## Notes

- This documentation was generated from the OpenTofu Registry on May 25, 2026
- The Vultr provider for OpenTofu/Terraform is compatible with both tools
- All resources support OpenTofu configuration syntax
- For the most up-to-date information, always refer to the official Vultr provider repository

---

**Total Documentation Files**: 94 (1 provider overview + 51 resources + 42 data sources)
