# 🌐 **Horizon - P2P Network Gateway**

---

## 📑 **Table of Contents**

1. [Project Overview](#-project-overview)
2. [System Architecture](#️-system-architecture)
3. [Connection Flow](#-connection-flow)
4. [Security Model](#-security-model)
5. [Development Roadmap](#-development-roadmap)

- [Database Schema](./database/database_schemas.md)
- [Component Specifications](./specifications/specifications.md)
- [Detailed Roadmap](./roadmap/roadmap.md)

---

## 🚀 **Project Overview**

Horizon is a secure peer-to-peer network gateway with three primary components:

- A central broker that manages server registration and client authentication
- Servers that provide network services including VPN functionality and LAN emulation
- Client applications that establish direct connections to servers

The system enables:

- P2P client-server connections with WebRTC (initial) and optional QUIC (advanced)
- Network functionality (VPN, LAN emulation for various applications)
- Secure authentication through personal keys
- Named server identification and management
- Operation with intermittent broker availability
- Server verification and update mechanisms

---

## 🏗️ **System Architecture**

### Core Components

#### 🖥️ **Server Component**

- **Language**: Rust
  - `tokio` (async runtime)
  - `webrtc-rs` (WebRTC protocol implementation)
  - Optional `quinn` (QUIC protocol implementation) in later phases
- **Key Functionality**:
  - Direct P2P connection with client
  - Network traffic tunneling
  - Virtual LAN emulation
  - Geolocation services
  - Server identity verification with friendly naming
  - Automatic process monitoring and self-healing
  - Secure update process

#### 🔄 **Broker Component**

- **Language**: Go
  - **Gin** (HTTP/REST API framework)
  - **PostgreSQL** (Robust relational database)
- **Key Functionality**:
  - Server validation and registration with naming support
  - Client authentication via personal keys
  - Connection tracking and analytics
  - Server discovery for clients
  - Geolocation tracking and display
  - Administrative interface with server management
  - Security monitoring and alerts
  - Server update delivery

#### 📱 **Client Component**

- **Language**: Rust
  - `webrtc-rs` (WebRTC client implementation)
  - Optional `quinn` (QUIC client implementation) in later phases
  - `tauri` (cross-platform UI framework)
  - `keyring` (secure key storage)
- **Key Functionality**:
  - Personal key management
  - Server discovery via broker with name and location info
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
   - Server operator inputs key and server name during setup
   - Server generates encryption keypair
   - Server collects location and hardware identity information

2. **Registration**:
   - Server connects to broker registration endpoint
   - Server provides registration key, name, public key, and location data
   - Broker validates registration key
   - Broker extracts public IP from connection
   - Broker determines geolocation information
   - Broker generates verification secret
   - Broker registers server with name and returns verification secret

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
   - Broker returns servers with names, capabilities, and locations
   - Client displays server names and locations visually
   - Client selects appropriate server

3. **Connection Establishment**:
   - Broker records connection intent
   - Client initiates direct P2P connection to server using WebRTC
   - Server accepts connection if authorized
   - WebRTC establishes secure data channels
   - Broker updates connection status
   - Client UI shows real-time connection status

4. **Active Connection**:
   - Server provides network services
   - Client uses VPN/LAN functionality
   - Client displays performance metrics
   - Both maintain security with WebRTC encryption

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
  - Server name validation for uniqueness
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
  - WebRTC with DTLS encryption (initial phases)
  - Optional QUIC protocol with TLS 1.3 (later phases)
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
  - PostgreSQL security features

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

## 🗓️ **Development Roadmap**

### Phase 1: Foundation & MVP (Weeks 1-8)
- Project setup and environment configuration
- Core broker with PostgreSQL implementation
- Basic server with WebRTC connectivity
- Simple client with minimal UI
- End-to-end connection testing

### Phase 2: Core Functionality (Weeks 9-16)
- Enhanced broker with admin interface
- Improved server with better P2P connectivity
- Enhanced client with better UI
- Security improvements and testing

### Phase 3: Advanced Features (Weeks 17-24)
- LAN emulation implementation
- Advanced broker features with geolocation
- Optional QUIC protocol transition
- Production preparation

### Phase 4: Scaling & Refinement (Weeks 25-32)
- System resilience enhancements
- Security hardening
- User experience improvements
- Final testing and deployment preparation

For detailed roadmaps of each component, see [Detailed Roadmap](./roadmap/roadmap.md).
