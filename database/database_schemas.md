# 📋 **PostgreSQL Database Schema**

## Core Tables

### `registration_keys` Table

```sql
CREATE TABLE registration_keys (
    registration_key UUID PRIMARY KEY,
    created_by UUID REFERENCES admins(admin_id),
    creation_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expiration_date TIMESTAMP WITH TIME ZONE,
    used BOOLEAN DEFAULT FALSE,
    used_by_server_id UUID,
    used_date TIMESTAMP WITH TIME ZONE,
    notes TEXT,
    status VARCHAR(20) NOT NULL DEFAULT 'active'
);
```

### `servers` Table

```sql
CREATE TABLE servers (
    server_id UUID PRIMARY KEY,
    name VARCHAR(100) NOT NULL, -- User-friendly server name
    hostname VARCHAR(255) NOT NULL,
    public_ip INET NOT NULL, -- PostgreSQL INET type
    public_key TEXT NOT NULL,
    version VARCHAR(50) NOT NULL,
    location JSONB, -- Enhanced location storage with JSON
    registration_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_heartbeat TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    available BOOLEAN DEFAULT FALSE,
    registered_with_key UUID REFERENCES registration_keys(registration_key),
    verification_hash TEXT NOT NULL
);
```

### `server_capabilities` Table

```sql
CREATE TABLE server_capabilities (
    capability_id SERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    capability_type VARCHAR(50) NOT NULL,
    capability_details JSONB NOT NULL, -- JSON for flexible capability details
    enabled BOOLEAN DEFAULT TRUE
);
```

### `personal_keys` Table

```sql
CREATE TABLE personal_keys (
    key_id UUID PRIMARY KEY,
    user_name VARCHAR(100) NOT NULL,
    user_email VARCHAR(255),
    key_hash TEXT NOT NULL,
    key_salt TEXT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    creation_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expiration_date TIMESTAMP WITH TIME ZONE,
    last_rotation_date TIMESTAMP WITH TIME ZONE,
    created_by UUID REFERENCES admins(admin_id),
    notes TEXT
);
```

### `devices` Table

```sql
CREATE TABLE devices (
    device_id UUID PRIMARY KEY,
    key_id UUID REFERENCES personal_keys(key_id),
    hardware_fingerprint TEXT NOT NULL,
    device_name VARCHAR(100),
    platform VARCHAR(50) NOT NULL,
    first_seen TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_seen TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT 'active'
);
```

### `connections` Table

```sql
CREATE TABLE connections (
    connection_id UUID PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id),
    device_id UUID REFERENCES devices(device_id),
    key_id UUID REFERENCES personal_keys(key_id),
    user_name VARCHAR(100),
    connection_start TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    connection_end TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    connection_type VARCHAR(50) NOT NULL,
    client_ip INET NOT NULL,
    disconnection_reason TEXT,
    connection_metrics JSONB -- Store connection quality metrics as JSON
);
```

### `heartbeats` Table

```sql
CREATE TABLE heartbeats (
    heartbeat_id BIGSERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    received_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    signature_valid BOOLEAN NOT NULL,
    active_connections INTEGER NOT NULL DEFAULT 0,
    current_location JSONB,
    system_metrics JSONB -- Store system metrics as structured JSON
);
```

### `server_verifications` Table

```sql
CREATE TABLE server_verifications (
    verification_id BIGSERIAL PRIMARY KEY,
    server_id UUID REFERENCES servers(server_id) ON DELETE CASCADE,
    verification_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    challenge_sent TEXT NOT NULL,
    response_received TEXT,
    verification_result BOOLEAN,
    verification_method VARCHAR(50) NOT NULL,
    failure_reason TEXT
);
```

### `security_events` Table

```sql
CREATE TABLE security_events (
    event_id BIGSERIAL PRIMARY KEY,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    event_type VARCHAR(50) NOT NULL,
    severity VARCHAR(20) NOT NULL,
    component VARCHAR(50) NOT NULL,
    source_id UUID,
    description TEXT NOT NULL,
    handled BOOLEAN DEFAULT FALSE,
    handled_by UUID REFERENCES admins(admin_id),
    handling_notes TEXT,
    user_name VARCHAR(100),
    event_details JSONB -- Store additional event details as JSON
);
```

### `admins` Table

```sql
CREATE TABLE admins (
    admin_id UUID PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    role VARCHAR(50) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    last_login TIMESTAMP WITH TIME ZONE,
    failed_login_attempts INTEGER DEFAULT 0,
    require_2fa BOOLEAN DEFAULT TRUE,
    totp_secret TEXT -- TOTP secret for 2FA
);
```

### `admin_actions` Table

```sql
CREATE TABLE admin_actions (
    action_id BIGSERIAL PRIMARY KEY,
    admin_id UUID REFERENCES admins(admin_id),
    action_time TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    action_type VARCHAR(50) NOT NULL,
    target_type VARCHAR(50) NOT NULL,
    target_id UUID NOT NULL,
    action_details JSONB, -- Store action details as JSON
    ip_address INET -- PostgreSQL INET type
);
```

### `server_updates` Table

```sql
CREATE TABLE server_updates (
    update_id UUID PRIMARY KEY,
    version VARCHAR(50) NOT NULL,
    release_date TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    package_hash TEXT NOT NULL,
    signature TEXT NOT NULL,
    update_description TEXT,
    created_by UUID REFERENCES admins(admin_id),
    mandatory BOOLEAN DEFAULT FALSE,
    min_version_required VARCHAR(50),
    update_files JSONB -- Store update file information as JSON
);
```

### `server_update_deployments` Table

```sql
CREATE TABLE server_update_deployments (
    deployment_id BIGSERIAL PRIMARY KEY,
    update_id UUID REFERENCES server_updates(update_id),
    server_id UUID REFERENCES servers(server_id),
    deployment_time TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    installation_time TIMESTAMP WITH TIME ZONE,
    failure_reason TEXT,
    previous_version VARCHAR(50),
    deployment_logs JSONB -- Store deployment logs as JSON
);
```

## Important Indexes

```sql
-- Registration key indexes
CREATE INDEX idx_registration_keys_status ON registration_keys(status);
CREATE INDEX idx_registration_keys_used ON registration_keys(used);
CREATE INDEX idx_registration_keys_expiration ON registration_keys(expiration_date);

-- Server performance indexes
CREATE INDEX idx_servers_name ON servers(name);
CREATE INDEX idx_servers_status ON servers(status);
CREATE INDEX idx_servers_last_heartbeat ON servers(last_heartbeat);
CREATE INDEX idx_servers_location ON servers USING GIN (location); -- JSON indexing
CREATE INDEX idx_servers_registered_with_key ON servers(registered_with_key);

-- Authentication indexes
CREATE INDEX idx_personal_keys_status ON personal_keys(status);
CREATE INDEX idx_personal_keys_user ON personal_keys(user_name);
CREATE INDEX idx_devices_key_id ON devices(key_id);
CREATE INDEX idx_devices_last_seen ON devices(last_seen);
CREATE INDEX idx_devices_status ON devices(status);
CREATE INDEX idx_devices_platform ON devices(platform);

-- Connection tracking indexes
CREATE INDEX idx_connections_status ON connections(status);
CREATE INDEX idx_connections_server_id ON connections(server_id);
CREATE INDEX idx_connections_device_id ON connections(device_id);
CREATE INDEX idx_connections_user ON connections(user_name);
CREATE INDEX idx_connections_start ON connections(connection_start);
CREATE INDEX idx_connections_end ON connections(connection_end);
CREATE INDEX idx_connections_status_server_id ON connections(status, server_id);
CREATE INDEX idx_connections_timerange ON connections(connection_start, connection_end);

-- Security monitoring indexes
CREATE INDEX idx_security_events_severity ON security_events(severity);
CREATE INDEX idx_security_events_handled ON security_events(handled);
CREATE INDEX idx_security_events_timestamp ON security_events(timestamp);
CREATE INDEX idx_heartbeats_server_id ON heartbeats(server_id);
CREATE INDEX idx_heartbeats_received_time ON heartbeats(received_time);
CREATE INDEX idx_heartbeats_server_time ON heartbeats(server_id, received_time);

-- Admin and update indexes
CREATE INDEX idx_admin_actions_admin_id ON admin_actions(admin_id);
CREATE INDEX idx_admin_actions_action_type ON admin_actions(action_type);
CREATE INDEX idx_server_updates_version ON server_updates(version);
CREATE INDEX idx_server_update_deployments_status ON server_update_deployments(status);
CREATE INDEX idx_server_verifications_server_id ON server_verifications(server_id);
CREATE INDEX idx_server_verifications_result ON server_verifications(verification_result);
CREATE INDEX idx_server_verifications_server_time ON server_verifications(server_id, verification_time);
```
