# Redfish Spec Summary (converted from Redfish_Spec_Summary.docx)

Summary / Interface checklist (Redfish)

1) Root
- Path
  - GET /redfish/v1/
- Purpose
  - Entry point: lists available top-level services and collections (Systems, Chassis, Managers, AccountService, SessionService, etc.).
- Main properties
  - `@odata.type` — The JSON schema/type identifier for the resource (e.g., `#ServiceRoot.v1_15_0.ServiceRoot`).
  - `Name` — Human-readable name of the service (e.g., "Root Service").
  - `RedfishVersion` — The Redfish protocol version supported by this service.
  - `UUID` — Unique identifier for the system managed by this service (persistent across reboots where possible).
  - `Links` — Object containing references to related resources; keys typically include `Sessions`, `Registries`, `Managers`, etc. Each link is an object with an `@odata.id` pointing to the target resource.

2) Systems
- Paths
  - GET /redfish/v1/Systems
  - GET /redfish/v1/Systems/{SystemId}
- Primary purpose
  - Represents computer systems (hosts) managed by the service.
- Important properties (typical)
  - `Id` — Unique identifier for the resource within the collection (string). Used in URIs.
  - `Name` — Human readable name for the system.
  - `Description` — Short description of the resource.
  - `Manufacturer` — Vendor name for the system hardware.
  - `Model` — Product model identifier.
  - `SKU` — Stock-keeping unit (vendor supplied identifier).
  - `SerialNumber` — Hardware serial number.
  - `PartNumber` — Vendor part number.
  - `Bios` — Object or link to the BIOS resource; contains BIOS version and settings link. (e.g., `@odata.id` to /redfish/v1/Systems/{id}/Bios)
  - `Boot` — Object describing boot configuration and override properties (`BootSourceOverrideTarget`, `BootSourceOverrideEnabled`, `UefiTarget`). Use to control boot behavior.
  - `Status` — Object with `{ State, Health, HealthRollup }` summarizing operational state and health. State examples: `Enabled`, `Disabled`. Health: `OK`, `Warning`, `Critical`.
  - `ProcessorSummary` — Aggregate processor info (count, status).
  - `MemorySummary` — Aggregate memory info (total, health).
  - `Storage` — Link to Storage collection for this system.
  - `EthernetInterfaces` — Link to Ethernet interfaces collection for this system.
  - `Links` — Relationship links such as `Chassis`, `ManagedBy` (who manages the system).
- Example GET (abbreviated)
  - Request:
    - GET /redfish/v1/Systems/1
    - Accept: application/json
  - Response (200):

  ```json
  {
    "@odata.id": "/redfish/v1/Systems/1",
    "Id": "1",
    "Name": "Host System",
    "Manufacturer": "ACME",
    "Model": "X1000",
    "SerialNumber": "SN123456",
    "Status": { "State": "Enabled", "Health": "OK" },
    "Bios": { "@odata.id": "/redfish/v1/Systems/1/Bios" },
    "EthernetInterfaces": { "@odata.id": "/redfish/v1/Systems/1/EthernetInterfaces" }
  }
  ```

3) Chassis
- Paths
  - GET /redfish/v1/Chassis
  - GET /redfish/v1/Chassis/{ChassisId}
- Purpose
  - Physical containers (racks, enclosures, boards).
- Key properties
  - `Id` — Resource id used in the collection.
  - `Name` — Human-readable name.
  - `Location` — Physical or logical location information object (may include Info and URI).
  - `Thermal` — Link to thermal information (sensors, fans, temperatures).
  - `Power` — Link to power metrics and control.
  - `Status` — Operational state and health.
  - `Links/Contains` — Array of contained chassis or resources (with `@odata.id`).

4) Managers
- Paths
  - GET /redfish/v1/Managers
  - GET /redfish/v1/Managers/{ManagerId}
- Purpose
  - Management controllers (BMC/IMM/iLO).
- Key properties
  - `FirmwareVersion` — Version string for manager firmware (BMC firmware version).
  - `DateTime` — Current date/time on manager, often used for logs and scheduling.
  - `ManagerType` — Type of manager (e.g., BMC).
  - `GraphicalConsole` — Object describing graphical console capabilities (VNC, etc.).
  - `SerialConsole` — Object describing serial console settings and status.
  - `CommandShell` — Object describing command shell capability and status.
  - `Links/ManagerForServers` — Links to systems that this manager controls.
- Actions
  - `Manager.Reset`: POST /redfish/v1/Managers/{id}/Actions/Manager.Reset
    - Body: `{ "ResetType": "GracefulRestart" }` (ResetType values per schema)
    - Response: 204 or 202

5) AccountService & Accounts
- Paths
  - GET /redfish/v1/AccountService
  - GET /redfish/v1/AccountService/Accounts/{AccountId}
  - POST /redfish/v1/AccountService/Accounts (create account) — may be restricted
- Key properties
  - `Accounts` — Link to accounts collection.
  - `Roles` — Link to roles collection.
  - `LDAP` / `Radius` configuration — Objects describing external authentication sources.
  - Account object fields:
    - `UserName` — Login name for the account.
    - `Password` — Write-only field used when creating an account (never returned).
    - `RoleId` — Role assigned to the account (Admin, Operator, etc.).
    - `Enabled` — Boolean indicating if the account is enabled.
    - `Locked` — Boolean indicating if account is locked due to lockout policy.
- Examples
  - Create account:
    - POST /redfish/v1/AccountService/Accounts
    - Body: `{ "UserName":"bob", "Password":"pass", "RoleId":"Admin" }`
    - Response: 201 Created with Location header
  - Update (PATCH) an account:
    - PATCH /redfish/v1/AccountService/Accounts/2
    - Body: `{ "Enabled": false }`
    - Response: 200 OK

6) SessionService (authentication)
- Paths
  - GET /redfish/v1/SessionService
  - POST /redfish/v1/SessionService/Sessions
  - DELETE /redfish/v1/SessionService/Sessions/{SessionId}
- Purpose
  - Create/terminate sessions, return X-Auth-Token
- Key properties
  - `SessionTimeout` — Integer seconds before an idle session expires.
  - `ServiceEnabled` — Boolean indicating whether session creation is enabled.
  - `Sessions` — Link to sessions collection.
- Example: create a session (login)
  - Request:
    - POST /redfish/v1/SessionService/Sessions
    - Body: `{ "UserName": "admin", "Password": "pass" }`
  - Response (201):
    - Headers: Location: /redfish/v1/SessionService/Sessions/123
               X-Auth-Token: <token>
    - Body: `{ "@odata.id": "/redfish/v1/SessionService/Sessions/123", "Id": "123", "UserName": "admin" }`

7) Roles
- Paths
  - GET /redfish/v1/AccountService/Roles
  - GET /redfish/v1/AccountService/Roles/{RoleId}
- Purpose
  - Define privilege sets (ReadOnly, Operator, Administrator)
- Key fields
  - `Id` — Role id.
  - `IsPredefined` — Boolean indicating if the role is builtin.
  - `AssignedPrivileges` — Array of privilege strings (e.g., `"Login"`, `"ConfigureManager"`).
  - `OemPrivileges` — Vendor-specific additional privileges.

8) EthernetInterfaces / Network
- Paths
  - GET /redfish/v1/Managers/{mgr}/NetworkInterfaces
  - GET /redfish/v1/Systems/{sys}/EthernetInterfaces
- Key properties
  - `Id` — Identifier for the interface resource (e.g., `"eth0"`).
  - `Name` — Human-readable name.
  - `MACAddress` — MAC address string for the interface.
  - `IPv4Addresses` — Array of IPv4 address objects with fields: `Address`, `SubnetMask`, `Gateway`, `AddressOrigin` (Static/DHCP).
  - `IPv6Addresses` — Array of IPv6 address objects.
  - `SpeedMbps` — Link speed in megabits per second.
  - `MTUSize` — Maximum transmission unit in bytes.
  - `Status` — Operational state/health.
- Example GET interface entry:
  ```json
  {
    "Id": "eth0",
    "Name": "Ethernet Interface",
    "MACAddress": "AA:BB:CC:DD:EE:FF",
    "IPv4Addresses": [ { "Address": "192.0.2.10", "SubnetMask": "255.255.255.0", "AddressOrigin": "Static" } ],
    "Status": { "State": "Enabled", "Health": "OK" }
  }
  ```

9) Storage, Volumes, Drives
- Paths
  - GET /redfish/v1/Systems/{sys}/Storage
  - GET /redfish/v1/Storage/{StorageId}
  - GET /redfish/v1/Systems/{sys}/Storage/{storageId}/Volumes
  - GET /redfish/v1/Storage/{storageId}/Drives/{driveId}
- Key properties
  - `StorageControllers` — Array describing controllers attached to the storage.
  - `CapacityBytes` — Total capacity in bytes for a storage or volume.
  - `Volumes` — Link to volumes collection; each Volume has CapacityBytes, Name, and Links.Drives.
  - `Drives` — Collection of drive resources; each Drive has Model, Manufacturer, CapacityBytes, Status.
- Example: create a volume (if supported)
  - POST /redfish/v1/Systems/1/Storage/1/Volumes
  - Body: `{ "Name": "vol1", "CapacityBytes": 100000000000 }`
  - Response: 201 Created (may be asynchronous via Task)

10) UpdateService (firmware update)
- Paths
  - GET /redfish/v1/UpdateService
  - POST /redfish/v1/UpdateService/Actions/UpdateService.SimpleUpdate
- Key properties
  - `HttpPushUri` / `MultipartHttpPushUri` — URI(s) used to push firmware images via multipart HTTP uploads to the UpdateService.
  - `ServiceEnabled` — Boolean indicating whether the update service is enabled.
  - `FirmwareInventory` — Link to a SoftwareInventory / FirmwareInventory collection that lists updatable firmware entries and their versions.
  - `MaxImageSizeBytes` — Integer limiting upload size supported by this service.
- Example SimpleUpdate
  - POST /redfish/v1/UpdateService/Actions/UpdateService.SimpleUpdate
  - Body: `{ "ImageURI": "http://example.com/firmware.bin" }`
  - Response: 202 Accepted (operation may be an asynchronous Task)

11) EventService
- Paths
  - GET /redfish/v1/EventService
  - POST /redfish/v1/EventService/Subscriptions
- Purpose
  - Subscribe to Redfish events; includes delivery and protocol configuration.
- Key properties
  - `ServiceEnabled` — Whether event delivery is enabled.
  - `Subscriptions` — Link to subscriptions collection.
  - `EventTypes` — Supported event types (e.g., StatusChange, ResourceUpdated).
  - Typical fields for subscription
  - `Destination` — URL to send event notifications.
  - `EventTypes` — Array selecting which events trigger notifications.
  - `Context` — Client-provided context string included in notifications.
  - `Protocol` — e.g., Redfish, SMTP, etc.

12) Tasks / Asynchronous operations
- Path
  - GET /redfish/v1/TaskService
  - GET /redfish/v1/TaskService/Tasks/{TaskId}
- Structure
  - Task object has:
    - `TaskState` — Running, Completed, Exception, Exception, Cancelled, etc.
    - `StartTime`, `EndTime` — timestamps for task lifecycle.
    - `Messages` — Array of message objects (MessageId, Message) providing diagnostic info.
- Key properties
  - `StartTime` — When task started.
  - `EndTime` — When task ended (may be null if running).
  - `TaskState` — Current state of task execution.

13) Schema & OData semantics
- JSON schema-based; each resource has:
  - `@odata.type` — schema type identifier referencing Json-schema/metadata.
  - `@odata.id` — resource canonical URI.
  - `@odata.context` — context URI pointing to $metadata.
  - `Id`, `Name` — standard identity fields.
- Actions and Action targets are exposed under `Actions`:
  - Example: `"Actions": { "#UpdateService.SimpleUpdate": { "target": "/redfish/v1/UpdateService/Actions/UpdateService.SimpleUpdate" } }`
- Links represent relationships and are usually objects with `@odata.id` or arrays of link objects.

14) Common property patterns
- Status — Object with fields:
  - `State` — Operational state (Enabled, Disabled, StandbyOffline, etc.).
  - `Health` — Health indicator (OK, Warning, Critical).
  - `HealthRollup` — Aggregate health for child resources.
- Links — Relationship container; usually contains arrays/objects pointing to related resources via `@odata.id`.
- Oem — Vendor-specific extension subtree; clients must ignore unknown Oem properties if unsupported.
- Actions — Action definitions live under `Actions` with action name and target.

15) HTTP behavior and headers
- Authentication
  - Use `X-Auth-Token` header for authenticated sessions; Basic auth may be supported for some endpoints.
- POST create responses
  - 201 Created with Location header to the new resource and possibly a body containing `@odata.id`.
- Asynchronous operations
  - 202 Accepted for operations started; may return a Task resource location to poll for status.
- Error responses
  - Standard OData error format (Redfish):
    {
      "error": {
        "code": "Base.1.0.GeneralError",
        "message": "A general error has occurred. See ExtendedInfo for more information.",
        "details": [...]
      }
    }

16) Example: Reset action (manager)
- POST /redfish/v1/Managers/1/Actions/Manager.Reset
- Body:
  `{ "ResetType": "GracefulRestart" }`
- Response:
  - 200/204 or 202 with Task

17) Example: Change boot order or set BootSourceOverride
- PATCH /redfish/v1/Systems/1
- Body:
  `{ "Boot": { "BootSourceOverrideEnabled": "Once", "BootSourceOverrideTarget": "Pxe" } }`
- Response: 200 OK or 202 if async

18) Example: Query collection
- GET /redfish/v1/Systems
- Response:
  ```json
  {
    "@odata.id": "/redfish/v1/Systems",
    "Members@odata.count": 1,
    "Members": [
      { "@odata.id": "/redfish/v1/Systems/1" }
    ]
  }
  ```

19) OEM extensions
- `Oem` objects under an `Oem` property provide vendor-specific fields and actions; clients should tolerate unknown properties.

20) Useful references (official)
- Redfish specification root and JSON schema docs: https://www.dmtf.org/standards/redfish
- Redfish examples and model definitions: Redfish Schema (e.g., ComputerSystem.v1_10_0.json) on DMTF site or Github redfish-schema repos.
