# Redfish Specification — Organized (Generated from panyue8/bmcweb)

This document organizes the Redfish endpoints and schemas implemented in the repository panyue8/bmcweb following the style of the sample you provided (UpdateService example). It is generated from files present in the repository and from docs/Redfish.md. This is a Markdown version; if you confirm I will convert it to a Word document (.docx) and commit it to a branch.

---

## 3.30 UpdateService
This resource shall be used to update the BMC/BIOS firmware with a remote http/ftp server. BMC will download the firmware image from the remote server, verify it, and upgrade the firmware. BMC redfish interface will return a task id to track the progress of the firmware update.

### 3.30.1 {SR}/UpdateService

| Property | Type | Value |
|---|---:|---|
| Actions | Object | UpdateService allows the user to perform UpdateService.SimpleUpdate Action. It can also contain an Oem Object under Oem attribute under this Actions
|  | | - `#UpdateService.SimpleUpdate` (Object): None
|  | | - `Oem` (Object): None
|  | | &nbsp;&nbsp; - `#UpdateService.BMCFwUpdate` (Object): None
|  | | &nbsp;&nbsp; - `#UpdateService.UploadFirmwareImage` (Object): None
|  | | &nbsp;&nbsp; - `#UpdateService.UploadCABundle` (Object): None
| Description | String | "Redfish Update Service" |
| Id | String | "UpdateService" |
| MaxImageSizeBytes | Integer | The maximum size in bytes of the software update image that this Service supports. |
| Name | String | "Update Service" |
| Oem | Object | AMI OEM TBD
|  |  | - `AMIUpdateService` (Object): None |
| ServiceEnabled | Boolean | This indicates whether this service is enabled. |
| Status | Object | Refer Terms and Definitions under Section 2.6 |
| FirmwareInventory | Object | This property shall contain a link to a Resource of type SoftwareInventoryCollection. |
| MultipartHttpPushUri | String | The URI used to perform a Redfish Specification-defined Multipart HTTP or HTTPS push update to the Update Service. URL string: `{SR}/UpdateService/upload` |

### 3.30.2 {SR}/UpdateService/FirmwareInventory

| Property | Type | Value |
|---|---:|---|
| Description | String | "Collection of Firmware Inventory resources available to the UpdateService" |
| Name | String | "Firmware Inventory Collection" |
| Members | Array | Contains the members of the firmware collection. |
| Members@odata.count | Number | Collection members count. |

### 3.30.3 {SR}/UpdateService/FirmwareInventory/{Firmware_Type}

| Property | Type | Value |
|---|---:|---|
| Id | String | Resource Identifier |
| Name | String | Name of the Resource |
| Updateable | Boolean | An indication of whether the Update Service can update this firmware. |
| Version | String | The version of this software. (Only for BMC) |

### 3.30.4 REST operations

- PATCH

### 3.30.5 Actions operations

Action URI: `{SR}/UpdateService/upload`

Note: MultipartHttpPushUri URI

- UpdateFile (File) — M — The image file for firmware update.
- UpdateParameters (File) — M — The JSON file to indicate the firmware target. The format should refer to `{SR}/UpdateService/UpdateService/FirmwareInventory`.

Example UpdateParameters JSON:

```json
{
    "Targets": [
        "/redfish/v1/UpdateService/FirmwareInventory/BMC"
    ]
}
```

- OemParameters (File) — M — The JSON file to indicate the image file type. The format should look like:

```json
{
    "ImageType": "BMC"
}
```

Enum Description

- BMC — Indicate the file is a BMC image.
- BIOS — Indicate the file is a BIOS image.


#### Examples

Ex1: Update BMC ROM1

1. Set the BMC image file `rom.ima` as `UpdateFile`.
2. Create a file `parameters.json` with following content and set as `UpdateParameters`:

```json
{
    "Targets": [
        "/redfish/v1/UpdateService/FirmwareInventory/BMCImage1"
    ]
}
```

3. Create a file `oem.json` with following content and set as `OemParameters`:

```json
{
    "ImageType": "BMC"
}
```

4. Start to upload image. If using POSTMAN, do a multipart/form-data POST to `{SR}/UpdateService/upload` with files `UpdateFile`, `UpdateParameters`, `OemParameters`.
5. The response will show a Task created; user can access the Task resource to get the update status.

Ex2: Update BIOS both ROMs

1. Set the BIOS image file `xge102o_0.21.bin` as `UpdateFile`.
2. Create a file `parameters.json` with following content and set as `UpdateParameters`:

```json
{
    "Targets": [
        "/redfish/v1/UpdateService/FirmwareInventory/BIOSImage1",
        "/redfish/v1/UpdateService/FirmwareInventory/BIOSImage2"
    ]
}
```

3. Create a file `oem.json` with following content and set as `OemParameters`:

```json
{
    "ImageType": "BIOS"
}
```

4. Follow the same upload steps as Ex1.

---

## Other resources (summary entries)

Below are other Redfish resources and endpoints present in the repository. For each, I can expand into the same detailed format as above on request.

- Service Root (`/redfish/v1`) — implemented in `redfish-core/lib/service_root.hpp`.
  - Properties: Id, Name, RedfishVersion, Links to AccountService, Managers, SessionService, Systems, UpdateService, etc.
- JsonSchemas (`/redfish/v1/JsonSchemas`) — index and file endpoints implemented in `redfish-core/lib/redfish_v1.hpp`. Reads schema files from `/usr/share/www/redfish/v1/JsonSchemas`.
- SessionService (`/redfish/v1/SessionService`) — implemented in `redfish-core/lib/redfish_sessions.hpp` (SessionTimeout, Sessions collection, per-session routes).
- Metadata (`/redfish/v1/$metadata`) — generated from local XML schema files in `/usr/share/www/redfish/v1/schema` via `redfish-core/lib/metadata.hpp`.
- Aggregation behaviours documented in `docs/AGGREGATION.md` and schema parsing scripts in `scripts/`.

---

If this structure looks good, reply with:

- "Convert to Word and commit" — I will convert this Markdown to `docs/Redfish-Spec.docx`, create a branch `redfish-spec-doc`, and commit the .docx to that branch.
- "Expand [resource]" — e.g., "Expand UpdateService and SessionService" and I will add detailed sections for those resources in the same format.

Notes:
- Search results used to build this file may be incomplete; view the code search for more: https://github.com/panyue8/bmcweb/search?q=%2Fredfish%2Fv1&type=code
- The document is generated from repository files only (as you requested).