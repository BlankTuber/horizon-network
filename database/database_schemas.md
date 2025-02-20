# 📋 **Database Schema**

## Core Tables

### `registration_keys` Table

```sql
CREATE TABLE registration_keys (
    registration_key TEXT PRIMARY KEY,
    created_by TEXT REFERENCES admins(admin_id),
    creation_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expiration_date TIMESTAMP,
    used BOOLEAN DEFAULT FALSE,
    used_by_server_id TEXT,
    used_date TIMESTAMP,
    notes TEXT,
    status TEXT NOT NULL DEFAULT 'active'
);
```

### `servers` Table

```sql
CREATE TABLE servers (
    server_id TEXT PRIMARY KEY,
    hostname TEXT NOT NULL,
    public_ip TEXT NOT NULL,
    public_key TEXT NOT NULL,
    version TEXT NOT NULL,
    location TEXT,
    registration_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_heartbeat TIMESTAMP,
    status TEXT NOT NULL DEFAULT 'pending',
    available BOOLEAN DEFAULT FALSE,
    registered_with_key TEXT REFERENCES registration_keys(registration_key),
    verification_hash TEXT NOT NULL
);
```

### `server_capabilities` Table

```sql
CREATE TABLE server_capabilities (
    capability_id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id TEXT REFERENCES servers(server_id) ON DELETE CASCADE,
    capability_type TEXT NOT NULL,
    capability_details TEXT NOT NULL,
    enabled BOOLEAN DEFAULT TRUE
);
```

### `personal_keys` Table

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
    created_by TEXT REFERENCES admins(admin_id),
    notes TEXT
);
```

### `devices` Table

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

### `connections` Table

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
    client_ip TEXT NOT NULL,
    disconnection_reason TEXT
);
```

### `heartbeats` Table

```sql
CREATE TABLE heartbeats (
    heartbeat_id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id TEXT REFERENCES servers(server_id) ON DELETE CASCADE,
    received_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    signature_valid BOOLEAN NOT NULL,
    active_connections INTEGER NOT NULL DEFAULT 0,
    current_location TEXT,
    system_metrics TEXT
);
```

### `server_verifications` Table

```sql
CREATE TABLE server_verifications (
    verification_id INTEGER PRIMARY KEY AUTOINCREMENT,
    server_id TEXT REFERENCES servers(server_id) ON DELETE CASCADE,
    verification_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    challenge_sent TEXT NOT NULL,
    response_received TEXT,
    verification_result BOOLEAN,
    verification_method TEXT NOT NULL,
    failure_reason TEXT
);
```

### `security_events` Table

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
    handled_by TEXT REFERENCES admins(admin_id),
    handling_notes TEXT,
    user_name TEXT
);
```

### `admins` Table

```sql
CREATE TABLE admins (
    admin_id TEXT PRIMARY KEY,
    username TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    role TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'active',
    last_login TIMESTAMP,
    failed_login_attempts INTEGER DEFAULT 0,
    require_2fa BOOLEAN DEFAULT TRUE
);
```

### `admin_actions` Table

```sql
CREATE TABLE admin_actions (
    action_id INTEGER PRIMARY KEY AUTOINCREMENT,
    admin_id TEXT REFERENCES admins(admin_id),
    action_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    action_type TEXT NOT NULL,
    target_type TEXT NOT NULL,
    target_id TEXT NOT NULL,
    action_details TEXT,
    ip_address TEXT
);
```

### `server_updates` Table

```sql
CREATE TABLE server_updates (
    update_id TEXT PRIMARY KEY,
    version TEXT NOT NULL,
    release_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    package_hash TEXT NOT NULL,
    signature TEXT NOT NULL,
    update_description TEXT,
    created_by TEXT REFERENCES admins(admin_id),
    mandatory BOOLEAN DEFAULT FALSE,
    min_version_required TEXT
);
```

### `server_update_deployments` Table

```sql
CREATE TABLE server_update_deployments (
    deployment_id INTEGER PRIMARY KEY AUTOINCREMENT,
    update_id TEXT REFERENCES server_updates(update_id),
    server_id TEXT REFERENCES servers(server_id),
    deployment_time TIMESTAMP,
    status TEXT NOT NULL DEFAULT 'pending',
    installation_time TIMESTAMP,
    failure_reason TEXT,
    previous_version TEXT
);
```

## Important Indexes

```sql
-- Registration key indexes
CREATE INDEX idx_registration_keys_status ON registration_keys(status);
CREATE INDEX idx_registration_keys_used ON registration_keys(used);
CREATE INDEX idx_registration_keys_expiration ON registration_keys(expiration_date);

-- Server performance indexes
CREATE INDEX idx_servers_status ON servers(status);
CREATE INDEX idx_servers_last_heartbeat ON servers(last_heartbeat);
CREATE INDEX idx_servers_location ON servers(location);
CREATE INDEX idx_servers_registered_with_key ON servers(registered_with_key);

-- Authentication indexes
CREATE INDEX idx_personal_keys_status ON personal_keys(status);
CREATE INDEX idx_personal_keys_user ON personal_keys(user_name);
CREATE INDEX idx_devices_key_id ON devices(key_id);
CREATE INDEX idx_devices_last_seen ON devices(last_seen);
CREATE INDEX idx_devices_status ON devices(status);

-- Connection tracking indexes
CREATE INDEX idx_connections_status ON connections(status);
CREATE INDEX idx_connections_server_id ON connections(server_id);
CREATE INDEX idx_connections_device_id ON connections(device_id);
CREATE INDEX idx_connections_user ON connections(user_name);
CREATE INDEX idx_connections_start ON connections(connection_start);
CREATE INDEX idx_connections_end ON connections(connection_end);

-- Security monitoring indexes
CREATE INDEX idx_security_events_severity ON security_events(severity);
CREATE INDEX idx_security_events_handled ON security_events(handled);
CREATE INDEX idx_security_events_timestamp ON security_events(timestamp);
CREATE INDEX idx_heartbeats_server_id ON heartbeats(server_id);
CREATE INDEX idx_heartbeats_received_time ON heartbeats(received_time);

-- Admin and update indexes
CREATE INDEX idx_admin_actions_admin_id ON admin_actions(admin_id);
CREATE INDEX idx_admin_actions_action_type ON admin_actions(action_type);
CREATE INDEX idx_server_updates_version ON server_updates(version);
CREATE INDEX idx_server_update_deployments_status ON server_update_deployments(status);
CREATE INDEX idx_server_verifications_server_id ON server_verifications(server_id);
CREATE INDEX idx_server_verifications_result ON server_verifications(verification_result);
```
