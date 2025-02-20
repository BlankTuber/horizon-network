
# 📋 **Database Schema**

## Core Tables

### `servers` Table

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
    client_ip TEXT NOT NULL
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
    system_metrics TEXT -- JSON with CPU, memory, network usage
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
    user_name TEXT -- If applicable
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
