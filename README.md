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

---

## 🚀 **Project Overview**

Horizon is a secure peer-to-peer network gateway with three primary components:
- A central broker that manages server registration and client authentication
- Servers that provide network services including VPN functionality and LAN emulation
- Client applications that establish direct connections to servers

The system enables:
- P2P client-server connections
- Network functionality (VPN, LAN emulation for various applications)
- Secure authentication through personal keys with regular rotation
- Operation with intermittent broker availability
- Server integrity verification and update mechanisms

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
  - Auto-reporting of capabilities to broker
  - Multiple identity verification mechanisms
  - Secure update process

#### 🔄 **Broker Component**
- **Language**: Go
  - **Gin** (HTTP/REST API framework)
  - **PostgreSQL** (relational database)
  - **Redis** (caching and real-time updates)
- **Key Functionality**:
  - Server validation and registration
  - Client authentication via personal keys
  - Connection tracking
  - Server discovery for clients
  - Public IP determination from incoming connections
  - Server verification through challenge-response
  - Administrative interface for key management
  - Security monitoring and alerting
  - Server update delivery and verification
  - Key rotation management

#### 📱 **Client Component**
- **Language**: Rust
  - `quinn` (QUIC client implementation)
  - `tauri` (cross-platform UI framework)
  - `keyring` (secure key storage)
- **Key Functionality**:
  - Personal key management
  - Server discovery via broker
  - Direct P2P connection to servers
  - Multiple device support for same user (allow user to export their key)
  - Network service utilization
  - Key rotation handling
  - Secure communication enforcement

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
  - Network discovery broadcasts
  - Network address translation
  - UDP broadcast/multicast forwarding

#### Broker Communication
- Report version and capabilities during registration
- Accept challenge-response verification
- Submit periodic heartbeats with status
- Handle registration without manual configuration
- Maintain NAT traversal channels for broker commands
- Accept and process commands from broker when idle
- Receive and verify updates from broker
- Verify that all client connection attempts passes through broker

#### Client Connectivity
- P2P connection establishment
- Network service provisioning
- Connection state management
- Graceful disconnection handling
- Security boundary enforcement

#### Server Identity
- Multiple identity verification factors
- Hardware fingerprinting
- Cryptographic attestation
- Network identity verification
- Manual verification for significant changes (hardware changes, location changes, etc.)
- Protection against spoofing/hijacking attempts

#### Update Mechanism
- Receive updates from broker
- Verify update authenticity
- Apply updates securely
- Report update status
- Rollback capability

### 🔄 **Broker Specifications**

#### Server Management
- Extract public IP from incoming connections to use for futher information gathering
- Validate servers through verification challenges
- Track server availability status
- Record server capabilities automatically
- Store NAT traversal information
- Maintain command channel
- Flag unresponsive servers for investigation
- Deliver and verify server updates
- Monitor for unauthorized server changes
- Track multiple server identity factors
- Notice server irregularities and flag them

#### Authentication System
- Validate server registration credentials
- Authenticate clients via personal keys
- Support multi-device usage per key
- Prevent unauthorized server registration
- Handle regular key rotation
- Process manual key updates
- Revoke compromised keys
- Enforce key security policies
- Notice user irregularities and flag them

#### Connection Orchestration
- Record connection start/end times
- Track current connections
- Handle server unavailability
- Manage connection termination for new requests
- Verify all client traffic passes through broker
- Handle connections so that a user can only have one connection at a time

#### Administrative Interface
- Key management dashboard
- Server monitoring system
- Connection tracking
- Security alert management
- User management
- System performance monitoring
- Audit logging
- Update management

### 📱 **Client Specifications**

#### Authentication
- Store personal key in system secure storage
- Authenticate with broker
- Support usage across multiple devices
- Handle key revocation
- Process regular key updates
- Allow manual key updates
- Secure key transition
- Secure key transmission

#### Server Selection
- Discover available servers via broker
- Verify broker-mediated connections

#### Network Integration
- Utilize server VPN capabilities
- Access virtual LAN functionality
- Maintain connection across network changes
- Enforce security boundaries

#### Multi-Device Support
- Use same personal key across devices
- Synchronize settings when possible
- Manage device-specific configurations

#### Security Enforcement
- Prevent reverse connections from servers
- Detect malicious payloads
- Enforce communication policies
- Report security violations
- Verify traffic routing
- Secure traffic transmission

---

## 🔄 **Connection Flow**

### Server Registration Process
1. **Initial Setup**:
   - Administrator generates one-time registration key in broker
   - Administrator provides key to server operator
   - Server operator inputs key during initial setup
   - Server generates encryption keypair
   - Server collects hardware/network identity information

2. **First-Time Registration**:
   - Server connects to broker registration endpoint
   - Server provides one-time registration key, public key, and identity factors
   - Broker validates registration key
   - Broker extracts public IP from connection
     - Broker uses public IP to generate other information about server
   - Broker binds registration key to server's multiple identity factors
   - Broker generates verification secret for future authentication
   - Broker registers server and returns verification secret
   - Server stores verification secret securely

3. **Ongoing Communication & Verification**:
   - Server sends heartbeat every 30 seconds
   - Server includes HMAC signature using verification secret
   - Server reports capabilities, version, and identity information
   - Broker validates HMAC signature and identity factors
   - Broker periodically issues new challenges during heartbeats
   - Server responds to challenges using verification secret
   - Broker tracks last-seen and last-verified timestamps
   - Broker updates server status
   - Broker periodically updates verification secret, and server recieves and updates this

### Client Connection Process
1. **Authentication**:
   - Client provides personal key to broker
   - Broker validates key and checks rotation status
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
   - Server verifies all traffic routes through broker

4. **Active Connection**:
   - Server provides network services
   - Client uses VPN/LAN functionality
   - Both maintain security boundaries
   - Broker periodically verifies connection status

5. **Connection Termination**:
   - User initiated disconnection | Broker termination request | Connection timeout
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
   - Cannot establish new connections during broker unavailability
   - Continue broker reconnection attempts

### Server Change Detection
1. **Change Monitoring**:
   - Broker tracks multiple server identity factors
   - Server reports identity information with each heartbeat
   - Significant changes trigger verification requirement

2. **Verification Process**:
   - Broker flags server as requiring verification
   - Administrator reviews change details
   - Manual verification process initiated
   - Server remains unavailable until verified
   - New verification credentials issued after approval

### Key Rotation System
1. **Regular Updates**:
   - System generates new keys on schedule
   - Client receives key update notification
   - Automatic transition without manual input
   - Old key remains valid during transition period
   - Client confirms receipt of new key

2. **Manual Updates**:
   - Administrator initiates key revocation
   - All devices using key receive notification
   - Previous key immediately invalidated
   - User must manually input new key
   - Broker records key transition

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

### Server Update Process
1. **Update Preparation**:
   - Administrator uploads update to broker
   - Broker verifies update package
   - Update assigned version number and verification hash

2. **Update Delivery**:
   - Broker notifies eligible servers
   - Servers request update package
   - Broker delivers update securely
   - Server verifies package authenticity
   - Server applies update during maintenance window
   - Server reports update status to broker

---

## 📋 **Database Schemas**

### Table of Contents
1. [Core Schemas](#core-schemas)
2. [Authentication & Identity Schemas](#authentication--identity-schemas)
3. [Connection & Monitoring Schemas](#connection--monitoring-schemas)
4. [Security & Auditing Schemas](#security--auditing-schemas)
5. [Administrative Schemas](#administrative-schemas)
6. [Schema Relationships](#schema-relationships)
7. [Indexes & Performance Considerations](#indexes--performance-considerations)

### Core Schemas

#### `servers` Table
```sql
CREATE TABLE servers (
    server_id UUID PRIMARY KEY,
    hostname VARCHAR(255) NOT NULL,
    public_ip INET NOT NULL,
    public_key TEXT NOT NULL,
    verification_secret TEXT NOT NULL,
    version VARCHAR(50) NOT NULL,
    registration_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    last_heartbeat TIMESTAMP WITH TIME ZONE,
    last_verified TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    available BOOLEAN DEFAULT FALSE,
    requires_verification BOOLEAN DEFAULT FALSE,
    verification_reason TEXT,
    maintenance_mode BOOLEAN DEFAULT FALSE,
    decommissioned BOOLEAN DEFAULT FALSE,
    decommission_reason TEXT,
    decommission_date TIMESTAMP WITH TIME ZONE,
    created_by UUID REFERENCES admins(admin_id),
    CONSTRAINT status_check CHECK (status IN ('pending', 'active', 'unavailable', 'maintenance', 'decommissioned', 'compromised'))
);
```

#### `server_capabilities` Table
```sql
CREATE TABLE server_capabilities (
    capability_id SERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    capability_type VARCHAR(50) NOT NULL,
    capability_details JSONB NOT NULL,
    enabled BOOLEAN DEFAULT TRUE,
    added_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT capability_types CHECK (capability_type IN ('vpn', 'lan_emulation', 'geo_location', 'traffic_routing', 'nat_traversal'))
);
```

#### `server_identity_factors` Table
```sql
CREATE TABLE server_identity_factors (
    factor_id SERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    factor_type VARCHAR(50) NOT NULL,
    factor_value TEXT NOT NULL,
    first_seen TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_seen TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    confidence NUMERIC(5,2) DEFAULT 100.0,
    is_current BOOLEAN DEFAULT TRUE,
    CONSTRAINT factor_types CHECK (factor_type IN ('hardware_fingerprint', 'network_signature', 'geolocation', 'cryptographic_attestation', 'runtime_environment'))
);
```

#### `server_updates` Table
```sql
CREATE TABLE server_updates (
    update_id SERIAL PRIMARY KEY,
    version VARCHAR(50) NOT NULL,
    update_package_path TEXT NOT NULL,
    package_hash TEXT NOT NULL,
    signature TEXT NOT NULL,
    release_notes TEXT,
    required BOOLEAN DEFAULT FALSE,
    release_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expiration_date TIMESTAMP WITH TIME ZONE,
    created_by UUID REFERENCES admins(admin_id)
);
```

#### `server_update_status` Table
```sql
CREATE TABLE server_update_status (
    status_id SERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    update_id INTEGER REFERENCES server_updates(update_id) ON DELETE CASCADE,
    status VARCHAR(20) DEFAULT 'pending',
    notification_sent BOOLEAN DEFAULT FALSE,
    notification_date TIMESTAMP WITH TIME ZONE,
    download_started TIMESTAMP WITH TIME ZONE,
    download_completed TIMESTAMP WITH TIME ZONE,
    verification_status VARCHAR(20),
    installation_started TIMESTAMP WITH TIME ZONE,
    installation_completed TIMESTAMP WITH TIME ZONE,
    current_version_before VARCHAR(50),
    rollback_status VARCHAR(20),
    failure_reason TEXT,
    CONSTRAINT update_status_check CHECK (status IN ('pending', 'notified', 'downloading', 'verifying', 'installing', 'completed', 'failed', 'rolled_back'))
);
```

### Authentication & Identity Schemas

#### `personal_keys` Table
```sql
CREATE TABLE personal_keys (
    key_id UUID PRIMARY KEY,
    key_hash TEXT NOT NULL,
    key_salt TEXT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    creation_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    activation_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    expiration_date TIMESTAMP WITH TIME ZONE,
    last_rotation_date TIMESTAMP WITH TIME ZONE,
    next_rotation_date TIMESTAMP WITH TIME ZONE,
    rotation_notifications_sent BOOLEAN DEFAULT FALSE,
    manual_rotation_required BOOLEAN DEFAULT FALSE,
    deactivation_date TIMESTAMP WITH TIME ZONE,
    deactivation_reason TEXT,
    created_by UUID REFERENCES admins(admin_id),
    notes TEXT,
    CONSTRAINT key_status_check CHECK (status IN ('pending', 'active', 'rotating', 'deactivated', 'compromised', 'expired'))
);
```

#### `key_rotation_history` Table
```sql
CREATE TABLE key_rotation_history (
    rotation_id SERIAL PRIMARY KEY,
    previous_key_id UUID REFERENCES personal_keys(key_id),
    new_key_id UUID REFERENCES personal_keys(key_id),
    rotation_type VARCHAR(20) NOT NULL,
    rotation_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    initiated_by UUID REFERENCES admins(admin_id),
    reason TEXT,
    CONSTRAINT rotation_type_check CHECK (rotation_type IN ('scheduled', 'manual', 'emergency', 'compromised'))
);
```

#### `devices` Table
```sql
CREATE TABLE devices (
    device_id UUID PRIMARY KEY,
    key_id UUID REFERENCES personal_keys(key_id),
    hardware_fingerprint TEXT NOT NULL,
    device_name VARCHAR(255),
    platform VARCHAR(50) NOT NULL,
    os_version VARCHAR(50),
    app_version VARCHAR(50) NOT NULL,
    first_seen TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    last_seen TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    suspicious BOOLEAN DEFAULT FALSE,
    suspicious_reason TEXT,
    blocked BOOLEAN DEFAULT FALSE,
    blocked_date TIMESTAMP WITH TIME ZONE,
    blocked_reason TEXT,
    CONSTRAINT device_status_check CHECK (status IN ('active', 'inactive', 'suspicious', 'blocked'))
);
```

#### `registration_keys` Table
```sql
CREATE TABLE registration_keys (
    registration_key_id SERIAL PRIMARY KEY,
    registration_key TEXT NOT NULL UNIQUE,
    status VARCHAR(20) NOT NULL DEFAULT 'unused',
    creation_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    expiration_date TIMESTAMP WITH TIME ZONE NOT NULL,
    used_date TIMESTAMP WITH TIME ZONE,
    used_by_server_id UUID REFERENCES servers(server_id),
    created_by UUID REFERENCES admins(admin_id),
    notes TEXT,
    CONSTRAINT reg_key_status_check CHECK (status IN ('unused', 'used', 'expired', 'revoked'))
);
```

#### `admins` Table
```sql
CREATE TABLE admins (
    admin_id UUID PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(255),
    role VARCHAR(50) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    last_login TIMESTAMP WITH TIME ZONE,
    failed_login_attempts INTEGER DEFAULT 0,
    lockout_until TIMESTAMP WITH TIME ZONE,
    created_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    created_by UUID REFERENCES admins(admin_id),
    last_password_change TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    require_password_change BOOLEAN DEFAULT FALSE,
    mfa_enabled BOOLEAN DEFAULT TRUE,
    mfa_secret TEXT,
    CONSTRAINT admin_status_check CHECK (status IN ('active', 'inactive', 'locked', 'suspended'))
);
```

### Connection & Monitoring Schemas

#### `connections` Table
```sql
CREATE TABLE connections (
    connection_id UUID PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id),
    device_id UUID REFERENCES devices(device_id),
    key_id UUID REFERENCES personal_keys(key_id),
    connection_start TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    connection_end TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    connection_type VARCHAR(50) NOT NULL,
    client_ip INET NOT NULL,
    allocated_resources JSONB,
    termination_reason TEXT,
    initiated_by VARCHAR(20),
    connection_details JSONB,
    CONSTRAINT conn_status_check CHECK (status IN ('active', 'terminated', 'failed', 'timeout', 'interrupted')),
    CONSTRAINT conn_type_check CHECK (connection_type IN ('vpn', 'lan_emulation', 'both')),
    CONSTRAINT init_check CHECK (initiated_by IN ('client', 'server', 'broker', 'system', 'admin'))
);
```

#### `connection_metrics` Table
```sql
CREATE TABLE connection_metrics (
    metric_id SERIAL PRIMARY KEY,
    connection_id UUID REFERENCES connections(connection_id) ON DELETE CASCADE,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    bytes_sent BIGINT NOT NULL DEFAULT 0,
    bytes_received BIGINT NOT NULL DEFAULT 0,
    latency_ms INTEGER,
    packet_loss_percent NUMERIC(5,2),
    active_services JSONB,
    resource_usage JSONB
);
```

#### `heartbeats` Table
```sql
CREATE TABLE heartbeats (
    heartbeat_id SERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    received_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    server_time TIMESTAMP WITH TIME ZONE,
    signature_valid BOOLEAN NOT NULL,
    server_status JSONB NOT NULL,
    active_connections INTEGER NOT NULL DEFAULT 0,
    system_metrics JSONB,
    verification_response TEXT,
    verification_status BOOLEAN,
    identity_verified BOOLEAN DEFAULT TRUE
);
```

#### `service_status` Table
```sql
CREATE TABLE service_status (
    status_id SERIAL PRIMARY KEY,
    service_name VARCHAR(100) NOT NULL,
    component VARCHAR(20) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'operational',
    last_check TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    last_operational TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    incident_details TEXT,
    affected_components JSONB,
    next_check TIMESTAMP WITH TIME ZONE,
    CONSTRAINT component_check CHECK (component IN ('broker', 'server', 'client', 'system')),
    CONSTRAINT status_check CHECK (status IN ('operational', 'degraded', 'outage', 'maintenance'))
);
```

### Security & Auditing Schemas

#### `security_events` Table
```sql
CREATE TABLE security_events (
    event_id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    event_type VARCHAR(100) NOT NULL,
    severity VARCHAR(20) NOT NULL,
    component VARCHAR(20) NOT NULL,
    source_id UUID,
    source_ip INET,
    description TEXT NOT NULL,
    raw_data JSONB,
    handled BOOLEAN DEFAULT FALSE,
    handled_by UUID REFERENCES admins(admin_id),
    handled_at TIMESTAMP WITH TIME ZONE,
    resolution_notes TEXT,
    CONSTRAINT severity_check CHECK (severity IN ('info', 'low', 'medium', 'high', 'critical')),
    CONSTRAINT component_check CHECK (component IN ('broker', 'server', 'client', 'system'))
);
```

#### `verification_challenges` Table
```sql
CREATE TABLE verification_challenges (
    challenge_id SERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    challenge_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    challenge_type VARCHAR(50) NOT NULL,
    challenge_data TEXT NOT NULL,
    expected_response TEXT,
    actual_response TEXT,
    response_time TIMESTAMP WITH TIME ZONE,
    verification_success BOOLEAN,
    attempt_count INTEGER DEFAULT 1,
    CONSTRAINT challenge_type_check CHECK (challenge_type IN ('hmac', 'public_key', 'identity_verification', 'capability_check', 'behavior_verification'))
);
```

#### `audit_logs` Table
```sql
CREATE TABLE audit_logs (
    log_id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    admin_id UUID REFERENCES admins(admin_id),
    action VARCHAR(100) NOT NULL,
    component VARCHAR(50) NOT NULL,
    target_id UUID,
    target_type VARCHAR(50) NOT NULL,
    previous_state JSONB,
    new_state JSONB,
    ip_address INET,
    user_agent TEXT,
    success BOOLEAN NOT NULL,
    notes TEXT
);
```

#### `security_policy` Table
```sql
CREATE TABLE security_policy (
    policy_id SERIAL PRIMARY KEY,
    policy_name VARCHAR(100) NOT NULL UNIQUE,
    policy_type VARCHAR(50) NOT NULL,
    settings JSONB NOT NULL,
    description TEXT,
    created_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    last_modified TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    modified_by UUID REFERENCES admins(admin_id),
    active BOOLEAN DEFAULT TRUE,
    applies_to JSONB NOT NULL,
    CONSTRAINT policy_type_check CHECK (policy_type IN ('authentication', 'connection', 'key_rotation', 'verification', 'monitoring', 'updates'))
);
```

#### `blocked_ips` Table
```sql
CREATE TABLE blocked_ips (
    block_id SERIAL PRIMARY KEY,
    ip_address INET NOT NULL,
    ip_range CIDR,
    reason TEXT NOT NULL,
    severity VARCHAR(20) NOT NULL,
    block_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    expiration_date TIMESTAMP WITH TIME ZONE,
    blocked_by UUID REFERENCES admins(admin_id),
    block_source VARCHAR(50) NOT NULL,
    notes TEXT,
    CONSTRAINT severity_check CHECK (severity IN ('temporary', 'suspicious', 'malicious', 'persistent')),
    CONSTRAINT source_check CHECK (block_source IN ('manual', 'automatic', 'system', 'threshold'))
);
```

#### `anomaly_detection` Table
```sql
CREATE TABLE anomaly_detection (
    anomaly_id SERIAL PRIMARY KEY,
    detection_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    anomaly_type VARCHAR(100) NOT NULL,
    component VARCHAR(20) NOT NULL,
    source_id UUID,
    confidence_score NUMERIC(5,2) NOT NULL,
    baseline_data JSONB,
    anomalous_data JSONB,
    addressed BOOLEAN DEFAULT FALSE,
    addressed_by UUID REFERENCES admins(admin_id),
    addressed_time TIMESTAMP WITH TIME ZONE,
    resolution_action TEXT,
    false_positive BOOLEAN DEFAULT FALSE,
    CONSTRAINT component_check CHECK (component IN ('broker', 'server', 'client', 'connection', 'authentication'))
);
```

### Administrative Schemas

#### `system_settings` Table
```sql
CREATE TABLE system_settings (
    setting_id SERIAL PRIMARY KEY,
    setting_name VARCHAR(100) NOT NULL UNIQUE,
    setting_value JSONB NOT NULL,
    description TEXT,
    editable BOOLEAN DEFAULT TRUE,
    requires_restart BOOLEAN DEFAULT FALSE,
    modified_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    modified_by UUID REFERENCES admins(admin_id),
    category VARCHAR(50) NOT NULL
);
```

#### `admin_sessions` Table
```sql
CREATE TABLE admin_sessions (
    session_id UUID PRIMARY KEY,
    admin_id UUID REFERENCES admins(admin_id) ON DELETE CASCADE,
    login_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    expiry_time TIMESTAMP WITH TIME ZONE NOT NULL,
    last_activity TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    ip_address INET NOT NULL,
    user_agent TEXT NOT NULL,
    session_token TEXT NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    logout_time TIMESTAMP WITH TIME ZONE,
    logout_reason VARCHAR(50),
    CONSTRAINT logout_reason_check CHECK (logout_reason IN ('user_initiated', 'expired', 'admin_terminated', 'security_policy', 'session_hijacking_suspected'))
);
```

#### `notifications` Table
```sql
CREATE TABLE notifications (
    notification_id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    notification_type VARCHAR(50) NOT NULL,
    urgency VARCHAR(20) NOT NULL DEFAULT 'normal',
    target_admin_id UUID REFERENCES admins(admin_id),
    target_all_admins BOOLEAN DEFAULT FALSE,
    subject TEXT NOT NULL,
    content TEXT NOT NULL,
    related_entity_type VARCHAR(50),
    related_entity_id UUID,
    read BOOLEAN DEFAULT FALSE,
    read_time TIMESTAMP WITH TIME ZONE,
    expires TIMESTAMP WITH TIME ZONE,
    action_required BOOLEAN DEFAULT FALSE,
    action_taken BOOLEAN DEFAULT FALSE,
    action_details TEXT,
    CONSTRAINT urgency_check CHECK (urgency IN ('low', 'normal', 'high', 'critical')),
    CONSTRAINT notification_type_check CHECK (notification_type IN ('security', 'system', 'update', 'maintenance', 'policy', 'user', 'server'))
);
```

#### `scheduled_tasks` Table
```sql
CREATE TABLE scheduled_tasks (
    task_id SERIAL PRIMARY KEY,
    task_name VARCHAR(100) NOT NULL,
    task_type VARCHAR(50) NOT NULL,
    schedule JSONB NOT NULL,
    last_run TIMESTAMP WITH TIME ZONE,
    next_run TIMESTAMP WITH TIME ZONE,
    enabled BOOLEAN DEFAULT TRUE,
    task_parameters JSONB,
    description TEXT,
    created_by UUID REFERENCES admins(admin_id),
    created_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    modified_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    last_status VARCHAR(20),
    last_output TEXT,
    retry_count INTEGER DEFAULT 0,
    max_retries INTEGER DEFAULT 3,
    CONSTRAINT task_type_check CHECK (task_type IN ('key_rotation', 'maintenance', 'backup', 'cleanup', 'security_scan', 'report_generation', 'server_update'))
);
```

#### `reports` Table
```sql
CREATE TABLE reports (
    report_id SERIAL PRIMARY KEY,
    report_name VARCHAR(100) NOT NULL,
    report_type VARCHAR(50) NOT NULL,
    generation_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    parameters JSONB,
    generated_by UUID REFERENCES admins(admin_id),
    file_path TEXT,
    file_size INTEGER,
    file_hash TEXT,
    expiry_date TIMESTAMP WITH TIME ZONE,
    public_access BOOLEAN DEFAULT FALSE,
    access_token TEXT,
    description TEXT,
    CONSTRAINT report_type_check CHECK (report_type IN ('security', 'connection', 'performance', 'audit', 'system_health', 'user_activity', 'server_status'))
);
```

### Schema Relationships

The key relationships in this database design include:

1. **Server Identity and Verification**
   - `servers` ↔ `server_identity_factors` (1:N)
   - `servers` ↔ `verification_challenges` (1:N)
   - `servers` ↔ `heartbeats` (1:N)
   - `servers` ↔ `server_capabilities` (1:N)
   - `servers` ↔ `server_update_status` (1:N)
   - `registration_keys` ↔ `servers` (1:1)

2. **Authentication and Access Control**
   - `personal_keys` ↔ `devices` (1:N)
   - `personal_keys` ↔ `key_rotation_history` (1:N)
   - `personal_keys` ↔ `connections` (1:N)
   - `admins` ↔ `admin_sessions` (1:N)
   - `admins` ↔ `audit_logs` (1:N)

3. **Connection Management**
   - `connections` ↔ `servers` (N:1)
   - `connections` ↔ `devices` (N:1)
   - `connections` ↔ `connection_metrics` (1:N)

4. **Security and Monitoring**
   - `security_events` → `servers`, `devices`, `connections` (M:N)
   - `anomaly_detection` → `servers`, `devices`, `connections` (M:N)
   - `audit_logs` → multiple entities (polymorphic)
   - `blocked_ips` with broker monitoring

5. **Administrative Operations**
   - `admins` ↔ `notifications` (1:N)
   - `admins` ↔ `scheduled_tasks` (1:N)
   - `admins` ↔ `reports` (1:N)
   - `admins` ↔ `server_updates` (1:N)

### Indexes & Performance Considerations

### Critical Indexes

```sql
-- Server performance indexes
CREATE INDEX idx_servers_status ON servers(status);
CREATE INDEX idx_servers_last_heartbeat ON servers(last_heartbeat);
CREATE INDEX idx_servers_public_ip ON servers(public_ip);

-- Authentication indexes
CREATE INDEX idx_personal_keys_status ON personal_keys(status);
CREATE INDEX idx_personal_keys_expiration ON personal_keys(expiration_date);
CREATE INDEX idx_devices_key_id ON devices(key_id);
CREATE INDEX idx_devices_hardware_fingerprint ON devices(hardware_fingerprint);
CREATE INDEX idx_devices_last_seen ON devices(last_seen);

-- Connection tracking indexes
CREATE INDEX idx_connections_status ON connections(status);
CREATE INDEX idx_connections_server_id ON connections(server_id);
CREATE INDEX idx_connections_device_id ON connections(device_id);
CREATE INDEX idx_connections_key_id ON connections(key_id);
CREATE INDEX idx_connections_times ON connections(connection_start, connection_end);

-- Security monitoring indexes
CREATE INDEX idx_security_events_severity_time ON security_events(severity, timestamp);
CREATE INDEX idx_security_events_handled ON security_events(handled);
CREATE INDEX idx_heartbeats_server_time ON heartbeats(server_id, received_time);
CREATE INDEX idx_audit_logs_action_time ON audit_logs(action, timestamp);
CREATE INDEX idx_blocked_ips_address ON blocked_ips(ip_address);
CREATE INDEX idx_anomaly_detection_unaddressed ON anomaly_detection(addressed, detection_time);

-- Administrative indexes
CREATE INDEX idx_admin_sessions_active ON admin_sessions(admin_id, active);
CREATE INDEX idx_notifications_unread ON notifications(target_admin_id, read);
CREATE INDEX idx_scheduled_tasks_next_run ON scheduled_tasks(next_run, enabled);
```

### Partitioning Strategy

For high-volume tables, consider implementing the following partitioning strategies:

```sql
-- Partitioning heartbeats by month
CREATE TABLE heartbeats_partition (
    LIKE heartbeats INCLUDING ALL
) PARTITION BY RANGE (received_time);

-- Create monthly partitions
CREATE TABLE heartbeats_y2024m01 PARTITION OF heartbeats_partition
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
-- Continue with additional partitions

-- Partitioning connection_metrics by server and month
CREATE TABLE connection_metrics_partition (
    LIKE connection_metrics INCLUDING ALL
) PARTITION BY LIST (server_id) PARTITION BY RANGE (timestamp);

-- Partitioning audit_logs by quarter
CREATE TABLE audit_logs_partition (
    LIKE audit_logs INCLUDING ALL
) PARTITION BY RANGE (timestamp);
```

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
  - Multiple identity factors verification
  - Hardware fingerprint validation
  - Network characteristic monitoring
  - Periodic verification secret updates

- **Change Management**:
  - Significant changes trigger verification requirement
  - Server flagged as requiring verification
  - Manual administrator review required
  - Multiple verification factors checked
  - Change history maintained

- **Monitoring**:
  - Behavioral analysis
  - Version verification
  - Capability validation
  - Connection pattern monitoring
  - Traffic pattern analysis
  - Identity consistency checks

### Authentication System
- **Personal Keys**:
  - 256-bit random keys
  - Stored as salted hashes
  - Device-independent usage
  - Manual distribution to users
  - Regular rotation schedule
  - Manual revocation capability

- **Key Rotation**:
  - Automatic scheduled rotations
  - Transparent transition for regular updates
  - Manual revocation capability
  - Immediate invalidation for security concerns
  - Rotation history tracking

- **Device Management**:
  - Hardware fingerprinting
  - Device registration tracking
  - Suspicious activity detection
  - Multi-device coordination
  - Device-specific security policies

### Secure Communication
- **Transport Security**:
  - QUIC protocol with TLS 1.3
  - Certificate verification
  - Perfect forward secrecy
  - Connection encryption
  - Traffic verification
  - Keypair encryption and decryption

- **Security Boundaries**:
  - Server cannot initiate reverse connections
  - Payload inspection for malicious content
  - Traffic routing verification
  - Communication policy enforcement
  - Violation reporting and handling

- **Data Protection**:
  - No traffic content logging
  - Minimal metadata storage
  - Encrypted database storage
  - Secure key management

### Operational Security
- **Broker Hardening**:
  - Minimal attack surface
  - Connection rate limiting
  - Input validation
  - Privilege separation
  - Administrative access controls
  - Audit logging
  - Intrusion detection

- **Server Protection**:
  - Isolated network operation
  - Minimal persistent storage
  - Limited broker dependency
  - Secure failure modes
  - Update verification
  - Rollback capability
  - Secondary process monitor in case of main process failure

- **Client Security**:
  - Secure key storage
  - Connection verification
  - Application sandboxing
  - Update verification
  - Traffic validation
  - Security boundary enforcement

- **Update Security**:
  - Cryptographically signed updates
  - Package hash verification
  - Secure delivery channel
  - Integrity checking during installation
  - Version compatibility validation
  - Rollback capability for failed updates

---

## 📅 **Implementation Roadmap**

