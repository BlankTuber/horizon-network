# 🌐 **Horizon - P2P Network Gateway**

---

## 📑 **Table of Contents**
1. Project Overview
2. System Architecture
3. Component Specifications
4. Connection Flow
5. Database Schema
6. Security Model
7. Implementation Roadmap

---

## 🚀 **Project Overview**

Horizon is a secure peer-to-peer network gateway with three primary components:
- A central broker that manages server registration and client authentication
- Servers that provide network services including VPN functionality and LAN emulation
- Client applications that establish direct connections to servers

The system enables:
- P2P client-server connections
- Network functionality (VPN, LAN emulation for various applications)
- Secure authentication through personal keys
- Operation with intermittent broker availability
- Server verification and update mechanisms

---

## 🏗️ **System Architecture**

### Core Components

#### 🖥️ **Server Component**
- **Language**: Rust
  - `tokio` (async runtime)
  - `quinn` (QUIC protocol implementation)
- **Key Functionality**:
  - Direct P2P connection with client
  - Network traffic tunneling
  - Virtual LAN emulation
  - Geolocation services
  - Server identity verification
  - Automatic process monitoring and self-healing
  - Secure update process

#### 🔄 **Broker Component**
- **Language**: Go
  - **Gin** (HTTP/REST API framework)
  - **libSQL** (SQLite-compatible database with better concurrency)
- **Key Functionality**:
  - Server validation and registration
  - Client authentication via personal keys
  - Connection tracking
  - Server discovery for clients
  - Geolocation tracking and display
  - Basic administrative interface
  - Security monitoring
  - Server update delivery

#### 📱 **Client Component**
- **Language**: Rust
  - `quinn` (QUIC client implementation)
  - `tauri` (cross-platform UI framework)
  - `keyring` (secure key storage)
- **Key Functionality**:
  - Personal key management
  - Server discovery via broker with location info
  - Direct P2P connection to servers
  - Network service utilization
  - Modern, intuitive UI with real-time status
  - One-click installation process
  - Secure communication with keypair encryption

---

## 📊 **Component Specifications**

### 🖥️ **Server Specifications**

#### Network Services
- **VPN Capabilities**:
  - Encrypted tunnel creation
  - Traffic routing
  - IP masking for privacy/geo-bypassing
  - Network protocol handling

- **LAN Emulation**:
  - Virtual network adapter creation
  - Local network presence simulation
  - Network discovery broadcasts
  - UDP broadcast/multicast forwarding

#### Broker Communication
- Report version, location, and capabilities during registration
- Accept verification challenges
- Submit periodic heartbeats
- Handle registration with minimal configuration
- Receive and verify updates from broker

#### Process Monitor
- Independent watchdog process
- Automatic server process restart on failure
- Resource usage monitoring
- Crash reporting
- Startup failure detection

#### Client Connectivity
- P2P connection establishment
- Network service provisioning
- Connection state management
- Graceful disconnection handling

#### Server Identity
- Identity verification
- Hardware fingerprinting
- Location-based verification
- Network identity verification

#### Update Mechanism
- Receive updates from broker
- Verify update authenticity with keypair verification
- Apply updates
- Report update status

### 🔄 **Broker Specifications**

#### Server Management
- Validate servers through verification challenges
- Track server availability and location
- Record server capabilities
- Deliver and verify server updates
- Notice major server irregularities

#### Authentication System
- Validate server registration
- Authenticate clients via personal keys
- Associate keys with user identities
- Handle key rotation when needed
- Process manual key updates
- Revoke compromised keys

#### Connection Orchestration
- Record connection start/end times
- Track current connections
- Handle server unavailability
- Verify connection security

#### Administrative Interface
- Key management dashboard with user info
- Server monitoring with location data
- Connection tracking
- Security alert display

### 📱 **Client Specifications**

#### Installation
- One-click installer
- Automatic dependency resolution
- Secure initial setup
- Default security settings configuration

#### User Interface
- Clean, modern design
- Real-time connection status
- Server location map/visualization
- Bandwidth and performance metrics
- Dark/light theme
- Accessibility features

#### Authentication
- Store personal key in system secure storage
- Authenticate with broker
- Handle key updates when needed
- Associate with user identity

#### Server Selection
- Discover available servers via broker
- Visual server location map
- Performance metrics for selection
- Connect to selected server

#### Network Integration
- Utilize server VPN capabilities
- Access virtual LAN functionality
- Maintain connection across network changes

#### Security Enforcement
- Prevent reverse connections from servers
- Keypair encryption for all communications
- Enforce communication policies
- Report security violations

---

## 🔄 **Connection Flow**

### Server Registration Process
1. **Initial Setup**:
   - Administrator generates registration key in broker
   - Server operator inputs key during setup
   - Server generates encryption keypair
   - Server collects location and hardware identity information

2. **Registration**:
   - Server connects to broker registration endpoint
   - Server provides registration key, public key, and location data
   - Broker validates registration key
   - Broker extracts public IP from connection
   - Broker determines geolocation information
   - Broker generates verification secret
   - Broker registers server and returns verification secret

3. **Ongoing Communication**:
   - Server sends heartbeat every 30 seconds
   - Server includes signature using keypair encryption
   - Server reports capabilities, location, and version
   - Broker validates signature
   - Process monitor ensures server stays active

### Client Connection Process
1. **Authentication**:
   - Client provides personal key to broker
   - Broker validates key and identifies user
   - Broker registers device if needed

2. **Server Discovery**:
   - Client requests available server list
   - Broker returns servers with capabilities and locations
   - Client displays server locations visually
   - Client selects appropriate server

3. **Connection Establishment**:
   - Broker records connection intent
   - Client initiates direct P2P connection to server
   - Server accepts connection if authorized
   - Broker updates connection status
   - Client UI shows real-time connection status

4. **Active Connection**:
   - Server provides network services
   - Client uses VPN/LAN functionality
   - Client displays performance metrics
   - Both maintain security with keypair encryption

5. **Connection Termination**:
   - Graceful disconnection process
   - Server returns to available status
   - Broker records connection end
   - Client UI updates to show disconnected state

### Broker Unavailability Handling
1. **Server Behavior**:
   - If serving client: continue operation normally
   - If idle: enter standby mode with reconnection attempts
   - Process monitor ensures stability during broker outage

2. **Client Behavior**:
   - Maintain current connection if active
   - Display appropriate status in UI
   - Cannot establish new connections during broker unavailability

### Key Management
1. **Regular Updates**:
   - System generates new keys when needed
   - Client receives key update notification
   - Automatic transition to new key

2. **Manual Updates**:
   - Administrator initiates key revocation
   - Previous key invalidated
   - User receives notification with instructions
   - User must manually input new key

---

## 📋 **Database Schema**

### Core Tables

#### `servers` Table
```sql
CREATE TABLE servers (
    server_id TEXT PRIMARY KEY,
    hostname TEXT NOT NULL,
    public_ip TEXT NOT NULL,
    public_key TEXT NOT NULL,
    verification_secret TEXT NOT NULL,
    version TEXT NOT NULL,
    location TEXT, -- Country/region
    latitude REAL,
    longitude REAL,
    registration_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_heartbeat TIMESTAMP,
    status TEXT NOT NULL DEFAULT 'pending',
    available BOOLEAN DEFAULT FALSE
);
```

#### `server_capabilities` Table
```sql
CREATE TABLE server_capabilities (
    capability_id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id TEXT REFERENCES servers(server_id) ON DELETE CASCADE,
    capability_type TEXT NOT NULL,
    capability_details TEXT NOT NULL,
    enabled BOOLEAN DEFAULT TRUE
);
```

#### `personal_keys` Table
```sql
CREATE TABLE personal_keys (
    key_id TEXT PRIMARY KEY,
    user_name TEXT NOT NULL,
    user_email TEXT,
    key_hash TEXT NOT NULL,
    key_salt TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'active',
    creation_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expiration_date TIMESTAMP,
    last_rotation_date TIMESTAMP,
    notes TEXT
);
```

#### `devices` Table
```sql
CREATE TABLE devices (
    device_id TEXT PRIMARY KEY,
    key_id TEXT REFERENCES personal_keys(key_id),
    hardware_fingerprint TEXT NOT NULL,
    device_name TEXT,
    platform TEXT NOT NULL,
    first_seen TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_seen TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status TEXT NOT NULL DEFAULT 'active'
);
```

#### `connections` Table
```sql
CREATE TABLE connections (
    connection_id TEXT PRIMARY KEY,
    server_id TEXT REFERENCES servers(server_id),
    device_id TEXT REFERENCES devices(device_id),
    key_id TEXT REFERENCES personal_keys(key_id),
    user_name TEXT,
    connection_start TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    connection_end TIMESTAMP,
    status TEXT NOT NULL DEFAULT 'active',
    connection_type TEXT NOT NULL,
    client_ip TEXT NOT NULL
);
```

#### `heartbeats` Table
```sql
CREATE TABLE heartbeats (
    heartbeat_id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id TEXT REFERENCES servers(server_id) ON DELETE CASCADE,
    received_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    signature_valid BOOLEAN NOT NULL,
    active_connections INTEGER NOT NULL DEFAULT 0,
    current_location TEXT,
    system_metrics TEXT -- JSON with CPU, memory, network usage
);
```

#### `security_events` Table
```sql
CREATE TABLE security_events (
    event_id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    event_type TEXT NOT NULL,
    severity TEXT NOT NULL,
    component TEXT NOT NULL,
    source_id TEXT,
    description TEXT NOT NULL,
    handled BOOLEAN DEFAULT FALSE,
    user_name TEXT -- If applicable
);
```

#### `admins` Table
```sql
CREATE TABLE admins (
    admin_id TEXT PRIMARY KEY,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    role TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'active',
    last_login TIMESTAMP
);
```

### Important Indexes

```sql
-- Server performance indexes
CREATE INDEX idx_servers_status ON servers(status);
CREATE INDEX idx_servers_last_heartbeat ON servers(last_heartbeat);
CREATE INDEX idx_servers_location ON servers(location);

-- Authentication indexes
CREATE INDEX idx_personal_keys_status ON personal_keys(status);
CREATE INDEX idx_personal_keys_user ON personal_keys(user_name);
CREATE INDEX idx_devices_key_id ON devices(key_id);
CREATE INDEX idx_devices_last_seen ON devices(last_seen);

-- Connection tracking indexes
CREATE INDEX idx_connections_status ON connections(status);
CREATE INDEX idx_connections_server_id ON connections(server_id);
CREATE INDEX idx_connections_device_id ON connections(device_id);
CREATE INDEX idx_connections_user ON connections(user_name);

-- Security monitoring indexes
CREATE INDEX idx_security_events_severity ON security_events(severity);
CREATE INDEX idx_heartbeats_server_id ON heartbeats(server_id);
```

---

## 🔒 **Security Model**

### Server Validation
- **Initial Registration**:
  - Registration key required for first connection
  - Key becomes bound to server's public key
  - Keys expire after successful registration

- **Ongoing Verification**:
  - Challenge-response verification
  - Public key verification  
  - Connection origin validation
  - Location verification
  - Process monitoring for tampering detection

- **Monitoring**:
  - Version verification
  - Capability validation
  - Location consistency checking
  - Connection pattern monitoring

### Authentication System
- **Personal Keys**:
  - Strong random keys
  - Stored as salted hashes
  - Associated with real user identities
  - Manual distribution to users
  - Manual revocation capability

- **Key Rotation**:
  - Manual revocation capability
  - Rotation history tracking
  - User notification system

- **Device Management**:
  - Hardware fingerprinting
  - Device registration tracking
  - Suspicious activity detection

### Secure Communication
- **Transport Security**:
  - QUIC protocol with TLS 1.3
  - Certificate verification
  - Connection encryption
  - Keypair encryption/decryption for message security

- **Security Boundaries**:
  - Server cannot initiate reverse connections
  - Traffic verification
  - Communication policy enforcement

- **Data Protection**:
  - Minimal metadata storage
  - Secure key management
  - User identity protection

### Operational Security
- **Broker Protection**:
  - Input validation
  - Administrative access controls
  - Audit logging
  - libSQL concurrency benefits for resilience

- **Server Protection**:
  - Process monitor for automatic recovery
  - Minimal persistent storage
  - Limited broker dependency
  - Update verification
  - Crash reporting and analysis

- **Client Security**:
  - Secure key storage
  - Connection verification
  - Traffic validation
  - Visual security status indicators

- **Update Security**:
  - Signed updates with keypair verification
  - Package hash verification
  - Integrity checking during installation

---

## 📅 **Implementation Roadmap**

## Phase 1: Foundation and Core Architecture

### 🛠️ 1.1 Project Setup
- Set up dev environments (Rust, Go)
- Initialize Git repos with proper branching structure
- Install necessary libraries (tokio, quinn, gin)
- Configure basic CI pipeline with GitHub Actions
- Create libSQL schema definitions and test connections

### 🧱 1.2 Core Infrastructure Development
- Implement network communication libraries
  - QUIC protocol integration in Rust
  - REST endpoints in Go
- Set up libSQL database with connection pooling
- Create broker-server communication protocol
- Implement basic logging and error handling
- Test network connectivity between components

### 🔄 1.3 Broker Core Development
- Implement broker service with HTTP endpoints
  - `/register` - Server registration
  - `/auth` - Client authentication
  - `/servers` - Server discovery
  - `/status` - System status
- Set up database connection with prepared statements
- Create server registration handler with location detection
- Implement client authentication with user identity
- Build connection tracking system
- Test concurrent request handling with libSQL

## Phase 2: Security Framework

### 🔐 2.1 Authentication System
- Develop personal key generation with user association
  - Implement key derivation function (Argon2id)
  - Create salted hash storage
  - Build user identity management
- Implement secure key storage
  - Database encryption for stored keys
  - Memory protection for keys in use
- Create key distribution mechanism with user verification
- Build key validation logic with timing-attack resistance
- Test authentication with multiple concurrent users

### 🛡️ 2.2 Server Verification System
- Develop registration key generation
  - Implement cryptographically secure PRNG
  - Create key expiration logic
- Build registration validation pipeline
  - Key verification
  - Server identity collection
  - Location verification
- Implement challenge-response verification
  - Create challenge generation
  - Build response validation
  - Test replay attack resistance
- Create process monitor for server self-healing
  - Independent watchdog process
  - Health check mechanism
  - Automatic restart capability
  - Failure notification system

### 🔒 2.3 Secure Communication
- Implement TLS 1.3 with QUIC
  - Certificate generation and validation
  - Session resumption handling
- Develop keypair encryption system
  - Key exchange protocol
  - Message encryption/decryption
  - Forward secrecy implementation
- Create traffic verification mechanisms
  - Packet validation
  - Tamper detection
  - Replay protection
- Test against common attack vectors
  - MitM attacks
  - Replay attacks
  - Protocol downgrade attacks

## Phase 3: Network Services

### 🌐 3.1 VPN Capabilities
- Implement encrypted tunnel creation
  - TUN/TAP interface management
  - Packet encapsulation
  - Route table manipulation
- Develop traffic routing with location awareness
  - IP forwarding
  - NAT traversal
  - Traffic prioritization
- Create IP masking functionality
  - Address translation
  - Header rewriting
  - DNS leak prevention
- Implement network protocol handling
  - TCP optimization
  - UDP forwarding
  - ICMP response

### 🖧 3.2 LAN Emulation
- Implement virtual network adapter
  - Interface creation
  - MAC address assignment
  - MTU configuration
- Develop local network presence simulation
  - mDNS response
  - SSDP handling
  - NetBIOS name service
- Create network discovery broadcasts
  - Broadcast forwarding
  - ARP handling
  - Service advertisement
- Implement multicast forwarding
  - Group management
  - Packet duplication
  - TTL handling
- Test with common LAN applications
  - File sharing
  - Network printing
  - Media streaming
  - Games with LAN support

## Phase 4: Client and Management Interface

### 📱 4.1 Client Application
- Develop Tauri-based UI
  - Modern, responsive design
  - Dark/light theme support
  - Accessibility features
  - Server location map visualization
- Implement authentication flow
  - Secure key input
  - User identity display
  - Multi-device support
- Create server selection interface
  - Location-based sorting
  - Performance metrics
  - Favorite servers
- Develop connection management
  - Status indicators
  - Real-time metrics
  - Troubleshooting tools
- Build one-click installer
  - Dependency bundling
  - Auto-configuration
  - Update mechanism
  - Secure initial setup

### 🎛️ 4.2 Administrative Interface
- Develop key management with user identity
  - Key generation
  - User association
  - Revocation tools
  - Rotation scheduling
- Implement server monitoring
  - Status dashboard
  - Location tracking
  - Performance metrics
  - Health checks
- Create connection tracking
  - Active connections
  - Historical data
  - User activity
  - Bandwidth usage
- Implement security alert system
  - Real-time notifications
  - Severity classification
  - Response actions
  - Audit logging

## Phase 5: Testing and Deployment

### 🧪 5.1 Testing
- Implement unit tests
  - Component-level testing
  - Mocked dependencies
  - Edge case handling
- Develop integration testing
  - Inter-component communication
  - Database interactions
  - Authentication flows
- Create end-to-end test scenarios
  - Full connection lifecycle
  - Broker unavailability
  - Server failover
- Implement security testing
  - Penetration testing
  - Fuzzing
  - Dependency scanning
  - Compliance verification

### 🚀 5.2 Deployment Preparation
- Create installation packages
  - Windows installer (.msi)
  - macOS package (.pkg)
  - Linux packages (deb, rpm)
  - Automatic dependency resolution
- Implement basic backup procedures
  - Database backup script
  - Key storage backup
  - Configuration backup
- Develop monitoring setup
  - Health check endpoints
  - Metrics collection
  - Alert configuration
- Prepare user documentation
  - Installation guide
  - User manual
  - Troubleshooting
  - FAQ

## Phase 6: User Experience and Refinement

### 🔄 6.1 Feedback Loop
- Set up feedback channels
  - In-app feedback
  - Issue tracker
  - User community
- Implement automated crash reporting
  - Stack trace collection
  - Environment details
  - Frequency analysis
- Establish bug triage process
  - Severity classification
  - Reproduction steps
  - Fix prioritization

### 🎨 6.2 UX Improvements
- Refine client interface based on feedback
  - Workflow optimization
  - Visual enhancements
  - Performance improvements
- Add advanced visualizations
  - Connection quality graphs
  - Geolocation mapping
  - Usage statistics
- Implement guided setup wizard
  - First-run experience
  - Configuration assistance
  - Network optimization

### 📦 6.3 Packaging and Distribution
- Build zero-config installer
  - Silent installation option
  - Default security settings
  - Minimal user interaction required
- Implement auto-update mechanism
  - Update checking
  - Delta updates
  - Verification
  - Automatic installation
- Create distribution channels
  - Direct download
  - Package repositories
  - Update server
- Test deployment across platforms
  - Windows (10/11)
  - macOS (Intel/ARM)
  - Linux (major distros)
