# 🌐 **Horizon - P2P Network Gateway: Project Plan**

---

## 📑 **Table of Contents**
1. Project Overview
2. System Architecture
3. Component Specifications
4. Connection Flow
5. Database Schema
6. Security Model
7. Implementation Roadmap
8. Deployment Guidelines

---

## 🚀 **Project Overview**

Horizon is a secure peer-to-peer network gateway with three primary components:
- A central broker that manages server registration and client authentication
- Servers that provide network services including VPN functionality and LAN emulation
- Client applications that establish direct connections to servers

The system enables:
- P2P client-server connections
- Network functionality (VPN, LAN emulation for gaming, geo-restriction bypass)
- Secure authentication through personal keys
- Operation with intermittent broker availability

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
  - Virtual LAN emulation for gaming/applications
  - Geolocation services
  - Operation during broker unavailability
  - Auto-reporting of capabilities to broker

#### 🔄 **Broker Component**
- **Language**: Go
  - **Gin** (HTTP/REST API framework)
  - **SQLite** (local database)
  - **BoltDB** (embedded key-value store)
- **Key Functionality**:
  - Server validation and registration
  - Client authentication via personal keys
  - Connection tracking and time limits
  - Server discovery for clients
  - Public IP determination from incoming connections
  - Server verification through challenge-response

#### 📱 **Client Component**
- **Language**: Rust
  - `quinn` (QUIC client implementation)
  - `tauri` (cross-platform UI framework)
  - `keyring` (secure key storage)
- **Key Functionality**:
  - Personal key management
  - Server discovery via broker
  - Direct P2P connection to servers
  - Multiple device support for same user
  - Network service utilization

---

## 📊 **Component Specifications**

### 🖥️ **Server Specifications**

#### Network Services
- **VPN Capabilities**:
  - Encrypted tunnel creation
  - Traffic routing between networks
  - IP masking for privacy/geo-bypassing
  - Network protocol handling

- **LAN Emulation**:
  - Virtual network adapter creation
  - Local network presence simulation
  - Game/application discovery broadcasts
  - Network address translation for gaming
  - UDP broadcast/multicast forwarding

#### Broker Communication
- Report version and capabilities during registration
- Accept challenge-response verification
- Submit periodic heartbeats with status
- Handle registration without manual configuration
- Fall back to last-known-good state if broker unavailable
- Maintain NAT traversal channels for broker commands
- Accept and process commands from broker when idle

#### Client Connectivity
- P2P connection establishment
- Network service provisioning
- Connection state management
- Time limit enforcement
- Graceful disconnection handling

### 🔄 **Broker Specifications**

#### Server Management
- Extract public IP from incoming connections
- Validate servers through verification challenges
- Track server availability status
- Record server capabilities automatically
- Store NAT traversal information for idle servers
- Maintain command channel to idle servers
- Remove servers after inactivity threshold

#### Authentication System
- Validate server registration credentials
- Authenticate clients via personal keys
- Support multi-device usage per key
- Prevent unauthorized server registration

#### Connection Orchestration
- Record connection start/end times
- Enforce 8-hour connection limits
- Track current connections
- Handle server unavailability
- Manage connection termination for new requests

### 📱 **Client Specifications**

#### Authentication
- Store personal key in system secure storage
- Authenticate with broker
- Support usage across multiple devices
- Handle key revocation

#### Server Selection
- Discover available servers via broker
- Cache server information for offline operation
- Select optimal server based on needs
- Handle broker unavailability

#### Network Integration
- Utilize server VPN capabilities
- Access virtual LAN functionality
- Configure local network settings automatically
- Maintain connection across network changes

#### Multi-Device Support
- Use same personal key across devices
- Handle concurrent connection attempts
- Synchronize settings when possible
- Manage device-specific configurations

---

## 🔄 **Connection Flow**

### Server Registration Process
1. **Initial Setup**:
   - Administrator generates one-time registration key in broker
   - Administrator provides key to server operator
   - Server operator inputs key during initial setup
   - Server generates encryption keypair

2. **First-Time Registration**:
   - Server connects to broker registration endpoint
   - Server provides one-time registration key and public key
   - Broker validates registration key
   - Broker extracts public IP from connection
   - Broker binds registration key to server's public key
   - Broker generates verification secret for future authentication
   - Broker registers server and returns verification secret
   - Server stores verification secret securely

3. **Ongoing Communication & Verification**:
   - Server sends heartbeat every 30 seconds
   - Server includes HMAC signature using verification secret
   - Server reports capabilities and version
   - Broker validates HMAC signature
   - Broker periodically issues new challenges during heartbeats
   - Server responds to challenges using verification secret
   - Broker tracks last-seen and last-verified timestamps
   - Broker updates server status

### Client Connection Process
1. **Authentication**:
   - Client provides personal key to broker
   - Broker validates key
   - Broker recognizes device if previously used
   - Broker registers new device if first connection

2. **Server Discovery**:
   - Client requests available server list
   - Broker returns servers with capabilities
   - Client selects appropriate server
   - Client requests connection permission

3. **Connection Establishment**:
   - Broker records connection intent
   - Broker notifies server of pending connection
   - Client initiates direct P2P connection to server
   - Server accepts connection if authorized
   - Broker updates connection status

4. **Active Connection**:
   - Server provides network services
   - Client uses VPN/LAN functionality
   - Both track 8-hour time limit
   - Broker periodically verifies connection status

5. **Connection Termination**:
   - Time limit reached or user initiated
   - Graceful disconnection process
   - Server returns to available status
   - Broker records connection end

### Broker Unavailability Handling
1. **Server Behavior**:
   - If serving client: continue operation normally
   - If idle: enter standby mode with periodic broker reconnection attempts
   - Handle broker unavailability gracefully without crashing
   - Store last known good state for recovery
   - Resume normal operation when broker returns online

2. **Client Behavior**:
   - Maintain current connection if active
   - Attempt connection to cached servers if needed
   - Continue broker reconnection attempts
   - Store connection history locally

### Multi-Device Management
1. **Same-User Authentication**:
   - All devices use identical personal key
   - Broker identifies devices by hardware fingerprint
   - Connection history tracked per device

2. **Concurrent Connection Handling**:
   - Broker detects connection from new device
   - If existing connection present:
     - Prompt user for action
     - Terminate previous connection if confirmed
     - Establish new connection
     - Log device transition

---

## 📋 **Database Schema**

### `server_registration_keys` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `key_hash` | BLOB | Hashed registration key |
| `description` | TEXT | Purpose/owner description |
| `created_at` | INTEGER | Creation timestamp |
| `expires_at` | INTEGER | Expiration timestamp |
| `used` | INTEGER | 0=Unused, 1=Used |
| `server_id` | TEXT | Associated server after use (NULL if unused) |
| `used_at` | INTEGER | Timestamp of usage |

### `users` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `name` | TEXT | User identifier (optional) |
| `created_at` | INTEGER | Account creation timestamp |
| `last_active` | INTEGER | Last activity timestamp |
| `status` | INTEGER | 0=Active, 1=Suspended, 2=Deleted |

### `user_devices` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `user_id` | TEXT | Foreign key to users |
| `hardware_id` | TEXT | Device hardware fingerprint |
| `device_name` | TEXT | Auto-detected device name |
| `first_seen` | INTEGER | First connection timestamp |
| `last_seen` | INTEGER | Last connection timestamp |
| `client_version` | TEXT | Client software version |

### `access_keys` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `user_id` | TEXT | Foreign key to users |
| `key_hash` | BLOB | Hashed key value |
| `created_at` | INTEGER | Key creation timestamp |
| `revoked` | INTEGER | 0=Active, 1=Revoked |
| `description` | TEXT | Optional key description |

### `servers` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `public_ip` | TEXT | Public IP address (auto-detected) |
| `public_key` | BLOB | Server's public key |
| `registration_key_id` | TEXT | Foreign key to server_registration_keys |
| `verification_secret` | BLOB | Secret for ongoing verification |
| `first_seen` | INTEGER | First registration timestamp |
| `last_heartbeat` | INTEGER | Last heartbeat timestamp |
| `last_verified` | INTEGER | Last successful verification timestamp |
| `status` | INTEGER | 0=Available, 1=Connected, 2=Offline |
| `version` | TEXT | Server software version |
| `capabilities` | TEXT | JSON of server capabilities |
| `region` | TEXT | Geographic region (derived from IP) |
| `nat_info` | TEXT | NAT traversal information |
| `command_port` | INTEGER | Port for broker commands |
| `last_command` | INTEGER | Last successful command timestamp |

### `server_challenges` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `server_id` | TEXT | Foreign key to servers |
| `challenge` | BLOB | Challenge data |
| `expected_response` | BLOB | Expected verification response |
| `created_at` | INTEGER | Challenge creation timestamp |
| `completed` | INTEGER | 0=Pending, 1=Completed, 2=Failed |
| `completed_at` | INTEGER | Completion timestamp |

### `connections` Table
| Column | Type | Description |
|--------|------|-------------|
| `id` | TEXT | Primary key (UUID) |
| `user_id` | TEXT | Foreign key to users |
| `device_id` | TEXT | Foreign key to user_devices |
| `server_id` | TEXT | Foreign key to servers |
| `started_at` | INTEGER | Connection start timestamp |
| `ended_at` | INTEGER | Connection end timestamp (NULL if active) |
| `max_duration_seconds` | INTEGER | Time limit (default 28800 = 8 hours) |
| `termination_reason` | INTEGER | 0=InProgress, 1=Timeout, 2=UserRequest, 3=NewConnection, 4=ServerDisconnect, 5=Error |

### Indices
- `validation_credentials`: Indices on `credential_hash`, `revoked` 
- `users`: Index on `status`
- `user_devices`: Indices on `user_id`, `hardware_id`
- `access_keys`: Indices on `user_id`, `key_hash`, `revoked`
- `servers`: Indices on `public_ip`, `status`, `last_heartbeat`
- `server_challenges`: Indices on `server_id`, `completed`
- `connections`: Indices on `user_id`, `server_id`, `started_at`, `ended_at`

---

## 🔒 **Security Model**

### Server Validation
- **Initial Registration**:
  - One-time registration key required for first connection
  - Key distributed securely to authorized server operators
  - Keys become bound to server's public key after first use
  - Registration keys expire after successful onboarding

- **Ongoing Verification**:
  - Challenge-response verification on each reconnection
  - Public key continuity verification
  - Connection origin validation
  - Periodic re-verification during heartbeats

- **Monitoring**:
  - Behavioral analysis
  - Version verification
  - Capability validation
  - Connection pattern monitoring

### Authentication System
- **Personal Keys**:
  - 256-bit random keys
  - Stored as salted hashes
  - Device-independent usage
  - Manual distribution to users

- **Device Management**:
  - Hardware fingerprinting
  - Device registration tracking
  - Suspicious activity detection
  - Multi-device coordination

### Secure Communication
- **Transport Security**:
  - QUIC protocol with TLS 1.3
  - Certificate verification
  - Perfect forward secrecy
  - Connection encryption

- **Data Protection**:
  - No traffic content logging
  - Minimal metadata storage
  - Regular data purging
  - Encrypted database storage

### Operational Security
- **Broker Hardening**:
  - Minimal attack surface
  - Connection rate limiting
  - Input validation
  - Privilege separation

- **Server Protection**:
  - Isolated network operation
  - Minimal persistent storage
  - Limited broker dependency
  - Secure failure modes

- **Client Security**:
  - Secure key storage
  - Connection verification
  - Application sandboxing
  - Update verification

---

## 📅 **Implementation Roadmap**

### Phase 1: Broker Development (Weeks 1-6)
1. **Week 1: Core Foundation**
   - Set up Go project structure
   - Implement SQLite database schema
   - Create basic HTTP server with Gin
   - Develop configuration system

2. **Week 2: Authentication System**
   - Implement server registration key system
   - Create user and access key management
   - Build challenge-response verification
   - Develop verification secret management
   - Implement HMAC verification for heartbeats
   - Develop multi-device support logic

3. **Week 3: Server Management**
   - Create server registration endpoints
   - Implement public IP extraction
   - Build server capability tracking
   - Develop server status monitoring
   - Implement NAT traversal information storage
   - Create server command channel system

4. **Week 4: Connection Management**
   - Implement connection logging
   - Create time limit enforcement
   - Build connection termination system
   - Develop concurrent connection handling

5. **Week 5: Client Interfaces**
   - Create client authentication endpoints
   - Implement server discovery API
   - Build connection request handling
   - Develop device tracking system

6. **Week 6: Broker Hardening**
   - Implement comprehensive testing
   - Create security enhancements
   - Build failure recovery mechanisms
   - Develop monitoring and logging

### Phase 2: Server Development (Weeks 7-14)
1. **Week 7: Server Foundation**
   - Set up Rust project structure
   - Implement configuration system
   - Create broker communication module
   - Build registration process

2. **Week 8: Broker Integration**
   - Implement registration key handling
   - Store verification secret securely
   - Create HMAC signing for heartbeats
   - Build challenge-response mechanism
   - Develop public key management
   - Implement capability reporting
   - Build NAT traversal channel

3. **Week 9: Connection Handling**
   - Implement QUIC listener
   - Create client connection acceptance
   - Build connection state management
   - Develop time limit enforcement

4. **Week 10: VPN Core Functionality**
   - Implement virtual network adapter
   - Create traffic routing
   - Build IP masking
   - Develop protocol handling

5. **Week 11: LAN Emulation**
   - Implement virtual LAN capabilities
   - Create broadcast/multicast forwarding
   - Build game discovery support
   - Develop NAT for gaming applications

6. **Week 12: Advanced Networking**
   - Implement geolocation services
   - Create traffic optimization
   - Build protocol-specific handling
   - Develop performance monitoring

7. **Week 13: Failure Handling**
   - Implement broker unavailability handling
   - Create connection persistence
   - Build recovery mechanisms
   - Develop graceful degradation
   - Implement standby mode during broker outages
   - Create broker command processing system

8. **Week 14: Server Testing**
   - Implement comprehensive testing
   - Create performance optimization
   - Build security hardening
   - Develop deployment packaging

### Phase 3: Client Development (Weeks 15-22)
1. **Week 15: Client Foundation**
   - Set up Rust/Tauri project
   - Implement configuration system
   - Create personal key management
   - Build secure storage integration

2. **Week 16: Broker Communication**
   - Implement authentication system
   - Create server discovery
   - Build connection requesting
   - Develop device identification

3. **Week 17: Connection Management**
   - Implement QUIC client
   - Create direct server connection
   - Build connection monitoring
   - Develop reconnection handling

4. **Week 18: Network Integration**
   - Implement VPN utilization
   - Create LAN functionality access
   - Build system network configuration
   - Develop connection monitoring

5. **Week 19: User Interface - Core**
   - Create main application window
   - Implement server selection interface
   - Build connection management UI
   - Develop status monitoring display

6. **Week 20: User Interface - Advanced**
   - Implement settings management
   - Create connection statistics
   - Build notification system
   - Develop help/documentation

7. **Week 21: Multi-Device Support**
   - Implement device-specific storage
   - Create settings synchronization
   - Build concurrent connection handling
   - Develop device transition process

8. **Week 22: Client Testing**
   - Implement cross-platform testing
   - Create performance optimization
   - Build security hardening
   - Develop installation packages

### Phase 4: Integration & Finalization (Weeks 23-26)
1. **Week 23: System Integration**
   - Test complete system flow
   - Verify component interactions
   - Validate time limit enforcement
   - Test multi-device scenarios
   - Verify broker command channel to idle servers

2. **Week 24: Failure Testing**
   - Test broker unavailability
   - Verify component recovery
   - Validate data consistency
   - Test security mechanisms

3. **Week 25: Performance Optimization**
   - Measure system performance
   - Optimize resource usage
   - Improve connection speeds
   - Reduce startup times

4. **Week 26: Final Packaging**
   - Create deployment documentation
   - Build installation scripts
   - Finalize configuration options
   - Prepare administrator guidelines

---

## 📦 **Deployment Guidelines**

### Broker Deployment
- **Hardware Recommendations**:
  - 2GB RAM minimum
  - 20GB storage
  - Static IP recommended
  - Moderate CPU requirements

- **Installation**:
  - Self-contained executable
  - SQLite database file
  - Configuration through YAML file
  - TLS certificate configuration

- **Administration**:
  - Key management CLI tools
  - Connection monitoring dashboard
  - Database backup utility
  - Log management configuration

### Server Deployment
- **Hardware Recommendations**:
  - 1GB RAM minimum
  - Stable internet connection
  - Minimal storage requirements
  - Single core sufficient

- **Installation**:
  - Single executable
  - Configuration file (minimal settings required)
  - Validation credential file
  - Service installation scripts

- **Administration**:
  - Status monitoring page
  - Performance statistics
  - Connection logging (configurable)
  - Update mechanism

### Client Deployment
- **Supported Platforms**:
  - Windows 10/11
  - macOS 11+
  - Ubuntu/Debian Linux

- **Installation**:
  - Platform-specific installers
  - Minimal dependencies
  - Automatic updates
  - Configuration backup/restore

- **User Setup**:
  - Personal key input on first run
  - Automatic network configuration
  - Optional advanced settings
  - Connection preferences
