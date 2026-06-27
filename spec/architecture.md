# Architecture Overview

This document provides a high-level overview of the Device Locator application architecture, built using Tauri (Rust backend) with a React frontend and SQLite database.

## Technology Stack

- **Frontend**: React with TypeScript
- **Backend**: Tauri (Rust)
- **Database**: SQLite
- **Platform Support**: Windows and macOS

### Why Tauri?

Tauri is ideal for this network device locator because:
- **Native Network Access**: Rust backend provides low-level access to network interfaces and protocols
- **Cross-Platform**: Single codebase supports both Windows and macOS with native APIs
- **Small Binaries**: Significantly smaller distribution size compared to Electron
- **Performance**: Rust's performance benefits for network scanning operations
- **Security**: Strong security model with explicit permission system
- **Built-in SQLite Support**: Native database integration via Tauri plugins

---

## Frontend Architecture (React)

### Overview
The React frontend provides the user interface for device discovery, filtering, and management. It communicates with the Tauri backend via asynchronous commands and event listeners.

### Key Components

**DeviceList Component**
- Displays discovered devices in a sortable, filterable table
- Shows IP address (clickable), MAC address, device name, and device type
- Highlights newly discovered devices at the top of the list
- Updates dynamically as devices come online

**FilterControls Component**
- MAC address OUI filter input with format validation (supports 00:1A:2B, 00-1A-2B, 001A2B)
- Toggle between "show all devices" and "show only matching devices"
- Filtering applied at display level, not discovery level

**ScanControls Component**
- Rescan button to trigger new network scan
- Network adapter selection (if multiple interfaces available)
- Scan status indicator

**ExportPanel Component**
- Export discovered devices to CSV or TXT format
- File system save dialog integration

### State Management

- **Device State**: List of discovered devices, updated in real-time
- **Filter State**: Current OUI filter and display mode settings
- **Scan State**: Active scan status, selected network adapters
- **Settings State**: User preferences persisted to database

### Frontend-Backend Communication

- **Tauri Commands**: Frontend invokes Rust functions (scan_network, export_devices, update_settings)
- **Event Listeners**: Backend emits events when new devices discovered (device_found, scan_complete)
- **Real-time Updates**: WebSocket-style event system for live device list updates

---

## Backend Architecture (Tauri/Rust)

### Overview
The Rust backend handles all network operations, device discovery, and data persistence. It exposes commands to the frontend and emits events for real-time updates.

### Core Services

**Network Scanner Service**
- **ARP Scanning**: Enumerate active devices on local network segments
- **mDNS Listener**: Detect Algo devices via multicast DNS broadcasts when they boot
- **Multi-Interface Handler**: Scan across all available network adapters (Wi-Fi, Ethernet, VPN)
- **Background Scanning**: Continuous discovery process running in separate thread

**Device Discovery Engine**
- Parse network responses and extract device information
- Resolve device names via DNS when available
- Identify device types using MAC OUI database and other heuristics
- Deduplicate devices across multiple interfaces

**OUI Filter Module**
- Validate and normalize MAC address OUI input formats
- Match discovered devices against configured OUI (default: 00:22:EE for Algo devices)
- Provide all discovered devices to frontend (filtering at display layer)

**Export Service**
- Format device data as CSV or plain text
- Handle file I/O operations with OS-specific file dialogs
- Generate timestamped export filenames

### Tauri Commands (Frontend-Callable)

```rust
// Pseudo-code examples
scan_network() -> Result<Vec<Device>>
get_network_adapters() -> Vec<NetworkAdapter>
update_filter_settings(oui: String, enabled: bool) -> Result<()>
export_devices(format: ExportFormat, path: String) -> Result<()>
```

### Event Emission

- `device-found`: Emitted when new device discovered (especially mDNS broadcasts)
- `scan-progress`: Progress updates during network scan
- `scan-complete`: Scan finished with summary statistics

### Concurrency & Threading

- Main thread handles UI and Tauri commands
- Background thread runs continuous mDNS listener
- Thread pool for parallel scanning of multiple network segments/interfaces

### Platform-Specific Considerations

**Windows**
- Windows Sockets API for network operations
- IP Helper API for ARP table access
- WMI for network adapter enumeration

**macOS**
- BSD sockets for network operations
- System Configuration framework for network adapter info
- Bonjour SDK for mDNS discovery

---

## Database Architecture (SQLite)

### Overview
SQLite provides lightweight, embedded data persistence for discovered devices, scan history, and user preferences.

### Schema Design

**devices table**
```sql
CREATE TABLE devices (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ip_address TEXT NOT NULL,
    mac_address TEXT NOT NULL UNIQUE,
    device_name TEXT,
    device_type TEXT,
    first_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    oui TEXT
);
```

**scan_history table**
```sql
CREATE TABLE scan_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    scan_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    adapter_name TEXT,
    devices_found INTEGER,
    scan_duration_ms INTEGER
);
```

**settings table**
```sql
CREATE TABLE settings (
    key TEXT PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Data Persistence Strategy

- **Device Records**: Update `last_seen` timestamp when device rediscovered
- **Historical Data**: Maintain scan history for troubleshooting and analytics
- **User Preferences**: Store OUI filters, display settings, window position
- **Session Persistence**: Restore previous session state on application launch

### Database Operations

**On Application Start**
- Initialize SQLite connection via Tauri plugin
- Load user settings and restore UI state
- Load previously discovered devices for display

**During Scanning**
- Insert new devices or update existing records
- Record scan events in history table
- Batch commits for performance during high-volume discovery

**On Export**
- Query devices with optional filters
- Format results according to export type (CSV/TXT)
- No modification to device records

### Indexes & Performance

```sql
CREATE INDEX idx_mac_address ON devices(mac_address);
CREATE INDEX idx_last_seen ON devices(last_seen);
CREATE INDEX idx_oui ON devices(oui);
```

---

## Architecture Flow

### Typical User Flow

1. **Application Launch**
   - Frontend loads, connects to Tauri backend
   - Backend initializes SQLite database
   - Load previous devices and settings from database
   - Display cached device list to user

2. **Network Scan**
   - User clicks "Rescan" button
   - Frontend invokes `scan_network()` command
   - Backend scans all network adapters using ARP
   - Backend emits `device-found` events as devices discovered
   - Frontend updates device list in real-time
   - New devices added to database with timestamp
   - Scan completion triggers `scan-complete` event

3. **mDNS Detection**
   - Background thread continuously listens for mDNS broadcasts
   - Algo device boots and sends mDNS announcement
   - Backend emits `device-found` event with new device
   - Frontend adds device to top of list (highlighted)
   - Device record inserted/updated in database

4. **Filtering**
   - User enters OUI filter (e.g., "00:22:EE")
   - Frontend validates and normalizes format
   - Filter stored in settings table
   - Frontend applies filter to device list display
   - Backend continues discovering all devices (no filtering at scan level)

5. **Export**
   - User selects export format (CSV/TXT)
   - Frontend invokes `export_devices()` command
   - Backend queries database for current device list
   - Format data and save to user-selected file path
   - Confirmation displayed to user

---

## Key Design Decisions

### Multi-Interface Handling
The application scans all available network adapters simultaneously to handle complex network environments (multiple subnets, VLANs, VPN connections). Devices discovered on multiple interfaces are deduplicated by MAC address.

### Display-Level Filtering
OUI filtering is applied at the frontend display layer rather than restricting backend discovery. This design allows users to toggle filter modes without rescanning, and ensures complete network visibility is maintained.

### Real-Time Updates via Events
The event-driven architecture enables the frontend to respond immediately to new device discoveries (especially mDNS broadcasts from booting Algo devices) without polling.

### Database-First Approach
All discovered devices are persisted immediately to SQLite, providing session continuity and enabling historical analysis of network changes.

### Background mDNS Listener
A dedicated background thread continuously monitors for mDNS broadcasts, ensuring Algo devices are detected immediately when they come online, independent of manual scan operations.

---

## Scalability Considerations

- **Large Networks**: Scanning optimized with parallel thread pool for multiple subnets
- **Many Devices**: SQLite handles thousands of device records efficiently with proper indexing
- **Continuous Discovery**: Background mDNS listener has minimal resource footprint
- **Memory Management**: Device list virtualization in frontend for smooth UI with 100+ devices

---

## Future Extensibility

The architecture supports potential enhancements:
- Additional discovery protocols (SNMP, UPnP)
- Device grouping and custom tagging
- Network topology visualization
- Remote scanning capabilities
- Custom device identification rules
