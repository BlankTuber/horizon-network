# 🌐 **Horizon - P2P Network Gateway**

---

## 📑 **Table of Contents**

1. Project Overview
2. System Architecture
3. Connection Flow
4. Security Model

- [Assets](./assets/assets.md)
- [Database Schemas](./database/database_schemas.md)
- [Roadmap](./roadmap/roadmap.md)
- [Specifications](./specifications/specifications.md)

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
  - Server update delivery# 🗓️ **Horizon P2P Network Gateway - MVP Development Roadmap**

## 📅 **Phase 1: Broker Foundation (Weeks 1-4)**

### Week 1: Database Setup

- **Setup Go Environment for Broker**
  - Configure Go development environment
  - Set up libSQL/SQLite database connection
  - Create initial project structure

- **Implement Core Database Tables**
  - `registration_keys`: For server registration authorization
  - `servers`: For tracking registered servers
  - `personal_keys`: For client authentication
  - `admins`: For administrative access

### Week 2: Broker Authentication

- **Admin System**
  - Implement admin login functionality
  - Create basic admin dashboard structure
  - Develop session management
  
- **Registration Key System**
  - Build registration key generation
  - Implement key validation logic
  - Create admin interface for key management

### Week 3: Broker Server Management

- **Server Registration Endpoints**
  - Implement server registration API endpoint
  - Create public key storage
  - Develop IP validation and tracking
  - Build server record management

- **Server Verification Logic**
  - Implement basic challenge-response verification
  - Create server status tracking
  - Build verification logging

### Week 4: Broker Client Authentication

- **Personal Key Management**
  - Implement personal key generation
  - Create key validation endpoint
  - Build user association logic

- **Connection Tracking**
  - Implement `connections` table functionality
  - Create connection recording logic
  - Build basic connection analytics

## 📅 **Phase 2: Server Component (Weeks 5-10)**

### Week 5: Server Setup

- **Setup Rust Environment for Server**
  - Configure Rust development environment
  - Set up project structure with tokio
  - Implement configuration loading

- **Server Identity**
  - Create keypair generation
  - Implement hardware fingerprinting
  - Build configuration persistence
  - Develop broker connection module

### Week 6: Server Registration

- **Registration Process**
  - Implement registration key input
  - Build broker communication for registration
  - Create public key submission
  - Develop registration state persistence

- **Heartbeat System**
  - Implement periodic heartbeat generation
  - Create signature generation for heartbeats
  - Build reconnection logic
  - Develop status reporting

### Week 7: Network Interface Setup

- **QUIC Implementation**
  - Set up Quinn server configuration
  - Implement TLS certificate management
  - Create connection acceptance logic
  - Build basic stream management

- **Virtual Network Interface**
  - Implement TUN/TAP interface creation
  - Create IP address management
  - Build routing table modification
  - Develop packet capture system

### Week 8-9: VPN Functionality

- **Traffic Tunneling**
  - Implement packet encapsulation
  - Create route management
  - Build traffic forwarding
  - Develop IP masking

- **Connection Management**
  - Implement client validation
  - Create connection metrics collection
  - Build graceful termination
  - Develop concurrent connection handling

### Week 10: LAN Emulation

- **Local Network Simulation**
  - Implement broadcast packet handling
  - Create basic network discovery
  - Build multicast forwarding
  - Develop local service advertisement

## 📅 **Phase 3: Client Component (Weeks 11-16)**

### Week 11: Client Setup

- **Setup Rust Environment for Client**
  - Configure Rust/Tauri development environment
  - Set up project structure
  - Implement configuration management

- **Personal Key Management**
  - Integrate keyring for secure storage
  - Create broker authentication module
  - Build key validation logic
  - Develop device identity system

### Week 12: Server Discovery

- **Broker Communication**
  - Implement broker API client
  - Create server listing retrieval
  - Build server filtering logic
  - Develop error handling for broker communication

- **Server Selection**
  - Create server display logic
  - Implement server sorting by performance/location
  - Build server selection UI
  - Develop server connection history tracking

### Week 13-14: Connection Implementation

- **QUIC Client**
  - Implement Quinn client configuration
  - Create connection establishment
  - Build certificate validation
  - Develop reconnection logic

- **Network Integration**
  - Implement virtual adapter configuration
  - Create route table modification
  - Build DNS configuration
  - Develop connection state management

### Week 15-16: Client UI

- **Core Interface**
  - Implement main application window
  - Create connection status display
  - Build settings interface
  - Develop server browser
  
- **Connection Management UI**
  - Create connect/disconnect controls
  - Implement status indicators
  - Build basic performance display
  - Develop connection profiles

## 📅 **Phase 4: Security Enhancement (Weeks 17-20)**

### Week 17: Server Security

- **Challenge-Response System**
  - Enhance verification challenge processing
  - Implement cryptographic response generation
  - Create attestation evidence collection
  - Build verification metrics

- **Traffic Validation**
  - Implement basic traffic signature validation
  - Create replay attack prevention
  - Build traffic integrity checks

### Week 18: Broker Security

- **Security Event System**
  - Implement `security_events` table
  - Create security event recording
  - Build basic alerting system
  - Develop event severity classification

- **Connection Security**
  - Enhance connection validation
  - Implement suspicious activity detection
  - Create connection pattern analysis

### Week 19: Client Security

- **Connection Verification**
  - Implement server verification
  - Create connection encryption validation
  - Build visual security indicators
  - Develop security alert display

- **Secure Storage**
  - Enhance credential security
  - Implement sensitive data protection
  - Create secure configuration storage

### Week 20: End-to-End Encryption

- **Secure Communication**
  - Implement message encryption
  - Create secure key exchange
  - Build encrypted channel management
  - Develop crypto algorithm negotiation

## 📅 **Phase 5: Process Resilience (Weeks 21-24)**

### Week 21-22: Process Monitoring

- **Server Process Monitor**
  - Implement watchdog process
  - Create automatic restart capability
  - Build health checking
  - Develop crash recovery

- **Broker Monitoring Endpoints**
  - Create server status endpoints
  - Implement health reporting API
  - Build basic monitoring dashboard
  - Develop server status visualization

### Week 23-24: Broker Unavailability Handling

- **Server Resilience**
  - Implement standby mode
  - Create cached operation capabilities
  - Build connection maintenance during outages
  - Develop reconnection strategies

- **Client Offline Capabilities**
  - Implement connection maintenance during outages
  - Create offline status indicators
  - Build reconnection logic
  - Develop user guidance system

## 📅 **Phase 6: Admin Interface and Refinement (Weeks 25-30)**

### Week 25-26: Admin Dashboard - Core Features

- **Server Management**
  - Implement server listing with status
  - Create detailed server view
  - Build server action controls
  - Develop performance visualization

- **User Management**
  - Implement user listing
  - Create personal key management
  - Build device management
  - Develop connection history display

### Week 27-28: Client Refinement

- **User Experience Improvements**
  - Refine connection workflow
  - Implement error handling improvements
  - Create helpful notifications
  - Build user onboarding experience

- **Performance Display**
  - Implement basic performance metrics
  - Create connection quality indicators
  - Build bandwidth usage display
  - Develop latency visualization

### Week 29-30: Final Integration and Testing

- **System Integration**
  - Ensure all components work together
  - Verify authentication flow
  - Validate connection establishment
  - Test broker unavailability scenarios

- **Refinement Based on Testing**
  - Address critical issues
  - Implement performance improvements
  - Create necessary documentation
  - Prepare for initial deployment

## 🔍 **Component Dependency Map**

1. **Core Dependencies**
   - Database schema → Broker authentication → Server registration
   - Server identity → Server registration → Heartbeat system
   - Client authentication → Server discovery → Connection establishment

2. **Network Dependencies**
   - QUIC implementation → Connection system → VPN functionality
   - Virtual network interfaces → Traffic routing → LAN emulation
   - Client network integration → UI status display

3. **Security Dependencies**
   - Server verification → Challenge-response system
   - Client key management → Connection security
   - Traffic validation → End-to-end encryption

## 🔄 **Critical Path Components**

1. **Broker Foundation**
   - Database implementation
   - Authentication system
   - Server registration
   - Connection tracking

2. **Server Core**
   - Identity and registration
   - QUIC implementation
   - Network interface
   - Traffic tunneling

3. **Client Essentials**
   - Authentication
   - Server discovery
   - Connection establishment
   - Network integration
   - Basic UI

4. **Security Baseline**
   - Server verification
   - Traffic validation
   - Secure storage
   - Encrypted communication

5. **Operational Resilience**
   - Process monitoring
   - Broker unavailability handling
   - Connection maintenance

### 📱 **Client Component**

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

### Auth System

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
