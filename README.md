# 🌐 **Horizon - P2P Network Gateway**

---

## 📑 **Table of Contents**

1. Project Overview
2. System Architecture
3. Connection Flow
4. Security Model

- [Assets](./assets/readme.md)
- [Database Schemas](./database/readme.md)
- [Roadmap](./roadmap/readme.md)
- [Specifications](./spesification/readme.md)

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
