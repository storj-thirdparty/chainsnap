This document outlines the requirements and specifications for contributing blockchain snapshots to [chainsnap.io](https://chainsnap.io). Snapshots are crucial for initializing blockchain nodes quickly by providing a point-in-time state of the blockchain. This specification ensures consistency and accessibility of snapshots contributed by various providers.

## Snapshot API Specification for Providers

Providers must maintain an API that details the snapshots they are hosting.

**Endpoint:** GET /api/snapshots

This endpoint returns an array of Snapshot objects. Each Snapshot object represents a static image of the state of a blockchain at a specific point in time. Below is the structure of a Snapshot object, along with a description of its fields.

### Snapshot Object Structure

```
export type Snapshot = {
  id: string; 
  chain: string; 
  client: string;
  version: string;
  mode: string;
  size_bytes: number; 
  path: string;
  created: string;
  updated: string; 
};
```

### Field Descriptions

* **id:** A unique identifier for each snapshot. This is used to reference and access specific snapshots within the system.
* **chain:** Specifies the blockchain network that the snapshot pertains to. This allows users to identify the relevant blockchain, such as Ethereum (eth), Binance Smart Chain (bsc), or Polygon (polygon).
* **client:** The software client used to interact with the blockchain and create the snapshot. Different clients may offer different features or optimizations, such as Erigon or Geth for Ethereum.
* **version:** The version of the client software used to generate the snapshot
* **mode:** Indicates the synchronization mode of the blockchain node at the time the snapshot was taken. For example, "archive" mode means the snapshot includes all historical data of the blockchain, suitable for intensive data analysis or forensics.
* **size_bytes:** The total size of the compressed snapshot file in bytes. 
* **path:** A Storj path where the snapshot can be accessed (e.g. sj://snapshots/network/client/version/snapshot.tar.zst ). See below for conventions
* **created:** The creation date and time of the snapshot, represented in an ISO 8601 format. 
* **updated:** The last updated date and time of the snapshot, in ISO 8601 format.

### Example API Request and Response for GET /api/snapshots

Request

```
GET /api/snapshots HTTP/1.1
Accept: application/json
```

Response

```
HTTP/1.1 200 OK
Content-Type: application/json
{
 "snapshots": [
   {
      "id": "6cfb2410-0278-4ef9-8d4a-9953a826c710",
      "chain": "bsc",
      "client": "geth",
      "created": "2024-02-07T12:00:00Z",
      "version": "v1.1.12",
      "mode": "archive",
      "public_size": 8558995193856,
      "path": "sj://snapshots-binance/mainnet/erigon/v1.1.12/archive-2024-02-07.tar.zst",
      "updated": "2024-02-07T12:00:00Z"
    },
    {
      "id": "39890f5a-0c52-4a71-a39c-d876c2370587",
      "chain": "eth",
      "client": "erigon",
      "created": "2023-11-12T12:00:00Z",
      "version": "2.53.4",
      "mode": "archive",
      "public_size": 3006972723200,
      "path": "sj://snapshots-ethereum/mainnet/erigon/2.53.4/archive-2023-11-12.tar.zst",
      "updated": "2023-11-12T12:00:00Z"
    }
  ]
}
```

### Listing and Delisting Snapshots

* To List: Include the snapshot's details in the endpoint's snapshot array.
* To Delist: Remove the snapshot from the endpoint's snapshot array.

## Synchronization with Chainsnap

Chainsnap periodically updates its listings based on the array from the endpoint. This ensures the store reflects the latest available snapshots or any removed ones.

### Snapshot Format and Naming Convention

Each snapshot is stored in a Storj bucket or buckets to facilitate organization and access.

When preparing and uploading snapshots to the Storj bucket, it's crucial to adhere to a specific format and naming convention to ensure consistency and ease of access.

* Compression and Format: Compress the snapshot using Zstandard (zstd) and package it in a tarball format. This compression method is chosen for its balance between high compression ratios and fast decompression speeds, making it ideal for large blockchain data sets. The command tar -xv --zstd -C ~/chain/data can be used for decompression, indicating the need for a .tar.zst file format.
* Path Naming Convention: Ensure snapshots are easily identifiable and organized. While no strict naming of the path is enforced, a structured naming convention might look like one that includes the blockchain platform, network, client software, client version, synchronization mode, and the date the snapshot was taken.
    
Example: For a snapshot of the Polygon mainnet, created using the Erigon client version 2.53.4, in archive synchronization mode, and captured on November 12, 2023, the file name and path in the Storj bucket could be:

`sj://snapshots-polygon/mainnet/erigon/2.53.4/archive-2023-11-12.tar.zst`

This path breaks down as follows:

* snapshots-polygon is the bucket dedicated to Polygon snapshots.
* mainnet specifies the network type.
* erigon/2.53.4 specifies the client and its version.
* archive-2023-11-12.tar.zst specifies the synchronization mode (archive), the date of the snapshot, and the compression format.

## Uploading Procedure

1. Prepare the Snapshot:

    * Ensure the snapshot conforms to the required structure.
    * Compress the snapshot using Zstandard (zstd) to reduce storage and egress costs. The file should be in a format compatible when piped to the command tar -xv --zstd -C ~/chain/data, allowing for direct extraction to the blockchain data directory.

1. Create a Storj Account:

    * Providers must create a Storj account and set up a bucket specifically for storing blockchain snapshots.

1. Upload the Snapshot:

    * Use the Storj Uplink CLI or compatible client libraries to upload the compressed snapshot file to the designated bucket.
    * Ensure the uploaded snapshot is not set to public access. Access to the snapshot should be controlled through access grants.

1. Verify Upload:

    * After uploading, verify the integrity and accessibility of the snapshot within the Storj bucket through the Storj Uplink CLI or the Storj web interface, ensuring the file is intact and correctly uploaded.

## Sharing Access Grants with Chainsnap

### Generating Access Grants

* Providers must generate an single access grant with all permissions (Read, Write, List, Delete) to the bucket (or buckets) where snapshots are stored.
* This access grant allows Chainsnap to create limited-time access grants for purchasers of the snapshots.

### Secure Sharing Procedure

1. Generate an Access Grant: Use the Storj console to create an access grant with permissions to read, write, delete, and list objects in the bucket.
2. Secure Communication Channel: Share the access grant with Storj’s Chainsnap representative over a secure communication channel, such as Keybase, to ensure the confidentiality and integrity of the access grant.
3. Chainsnap's Use of Access Grant: Chainsnap will use the provided access grant to generate limited-time access grants for snapshot purchasers. These limited access grants ensure secure and controlled distribution of the snapshots.

