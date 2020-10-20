# Design Data Storage

## Choose a data storage approach

- Classify data:
  - Structured
    - AKA relational data, adheres to strict schema
  - Semi-Structured
    - Non-relational data
    - Expression and structure defined by a _serialization language_ (e.g. XML, JSON, YAML)
  - Unstructured
    - Media files, office files, text files, log files
- Operational needs:
  - Understand how data is accessed
  - Understand how data is changed
  - What are the latency requirements?
- Transactions:
  - Are transactions needed?
  - ACID
  - OLTP vs OLAP

## Choose appropriate storage account

### Settings

- Location
- Performance
  - Standard:
    - Uses magnetic disk drives
  - Premium:
    - Use SSDs
    - Enable block blobs and append blobs
- Replication:
  - LRS
    - 3 copies across fault domains
  - ZRS
    - 3 copies across storage clusters
  - GRS
    - 3 copies in 2 regions
  - RA-GRS
    - Same as GRS, except can read from either region
  - GZRS, RA-GZRS
- Access tier:
  - Hot, Cold
- Kind:
  - StorageV2 (general purpose v2): recommended one, all types supported
  - Storage (general purpose v1): legacy, all types supported
  - Blob storage: legacy, only block and append blobs
- How to choose how many needed:
  - Data diversity:
    - Want to keep in specific country?
    - Is the data public or private?
  - Cost sensitivity
    - E.g. different performance and replication
  - Management overhead

### Blob storage

- Block blobs
  - Up to ~5 TB
  - Stored as 100 MB blocks
  - Read from beginning to an end
- Page blobs
  - Up to 8 TB
  - Stored in 512 byte pages
  - Provide random read/write
- Append blobs
  - Up to 198 GB
  - Like block blobs, but optimized for append operations
  - E.g. for logging application

### Files

- Network file shares,
- Accessed using SMB, REST interface or client libs

### Queues

- Messages up to 64 KB
- Can contain millions of messages

### Secure

- Encryption at rest
- Encryption in transit (HTTPS)
- Supports CORS
- AAD or RBAC for access control
- Audit access using Storage Analytics service

## Manage data

- Azure Storage Explorer

## Disk storage for VMs

- Disk roles
  - OS disk
  - Data disk
  - Temporary disk
- Ephemeral OS disks
  - Saves data on local VM storage
  - Fast, but VM failure might destroy all data
- Managed disks
  - Virtual hard disk, for which Azure manages all required physical infra
  - Stored as page blobs in Storage account
  - Support encryption with Azure Storage Service Encryption (SSE) or Azure Disk Encryption (ADE)
  - Automatically replicated
- Unmanaged disks
  - Stored as page blob in Storage account
  - You create and manage manually
    - You track IOPS limits
    - You manage security and RBAC access
    - No scalability or management features
- Disk performance
  - Ultra SSD:
    - Highest disk performance
    - IOPS depends on the disk size
    - 4 GB to 64 TB
    - Limitations
      - Only in certain regions
      - VMs need to be in avail zones
      - Only for ES/DS v3 VMs
      - Only as data disks
      - No disk snapshots, VM images, scale sets, ADE, Azure Backup or Site recovery
  - Premium SSD:
    - No ultra SSD limitations
    - Performance figures are guaranteed
    - Can't adjust performance without disk detach
  - Standard SSD:
    - Perf figures aren't guaranteed, but achieved 99% of time
  - Standard HDD:
    - Stored on magnetic disk

## Monitor, diagnose and troubleshoot Azure Storage

Metrics:

- Transaction metrics
- Capacity metrics
- 1 hour or 10 minute

Logging:
