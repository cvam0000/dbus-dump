# dbus-dump
A Bash tool for Linux that introspects the entire D-Bus tree and outputs it in YAML format, making it easy to search, debug, and document system services. Users can quickly look up any property, interface, or object path, and easily identify the relationships between them (e.g., which object path contains a particular interface and property).

## Features
- Discover all D-Bus services.
- List all object paths for each service.
- Enumerate all properties, methods, and interfaces for every object path.
- Output the full D-Bus tree in a structured, searchable YAML format.
- Map telemetry and Redfish data to D-Bus objects for integration.
- Debug D-Bus issues with a complete introspection dump.
- Document system interfaces for development, QA, or integration.

---

## Requirements
- Bash (version 4.0 or higher recommended)
- At least one of the following tools:
  - `busctl` (from systemd) — Preferred
  - `gdbus` (from GLib) — Good for introspection
  - `dbus-send` (from dbus) — Fallback option
- Standard Unix utilities: `awk`, `sed`, `grep`, `sort`, `uniq`
- (Optional) `yq` for advanced YAML querying
- May require `sudo` for some system services

---

## How to Use

### 1. Make the script executable
```bash
chmod +x dbus_dump.sh
```

### 2. Basic usage examples
- **Dump all services (system bus) to default file:**
  ```bash
  ./dbus_dump.sh
  ```
- **Dump a specific service:**
  ```bash
  ./dbus_dump.sh org.freedesktop.NetworkManager
  ```
- **Dump to a custom output file:**
  ```bash
  ./dbus_dump.sh -o my_dump.yaml
  ```
- **Use the session bus instead of the system bus:**
  ```bash
  ./dbus_dump.sh --session
  ```
- **Dump a specific service to a custom file:**
  ```bash
  ./dbus_dump.sh org.freedesktop.NetworkManager custom_output.yaml
  ```
- **Show help:**
  ```bash
  ./dbus_dump.sh --help
  ```

### 3. Command-Line Options
| Option                | Description                                  |
|-----------------------|----------------------------------------------|
| `-h`, `--help`        | Show help message                            |
| `-s`, `--system`      | Use system bus (default)                     |
| `-u`, `--session`     | Use session bus                              |
| `-o`, `--output FILE` | Specify output file (default: dbus_dump.yaml)|

---

## Output Format
The script generates YAML with the following structure:
```yaml
# D-Bus Tree Dump
# Generated on: [timestamp]
# Bus type: system
# Format: service -> tree_structure + object_paths -> introspection_data

dbus_dump:
  "service.name":
    tree_structure: |
      `- /xyz
        `- /xyz/openbmc_project
          |- /xyz/openbmc_project/FruDevice
          | |- /xyz/openbmc_project/FruDevice/10
          | `- /xyz/openbmc_project/FruDevice/12
    object_paths:
      "/xyz/openbmc_project":
        introspection: |
          NAME                                TYPE      SIGNATURE RESULT/VALUE FLAGS
          org.freedesktop.DBus.Introspectable interface -         -            -
          .Introspect                         method    -         s            -
      "/xyz/openbmc_project/FruDevice/10":
        introspection: |
          NAME                                TYPE      SIGNATURE RESULT/VALUE                           FLAGS
          org.freedesktop.DBus.Introspectable interface -         -                                      -
          .Introspect                         method    -         s                                      -
          xyz.openbmc_project.FruDevice       interface -         -                                      -
          .DEVICE_TYPE                        property  y         4                                      emits-change
```
- **tree_structure:** Visualizes the hierarchy of object paths for the service.
- **object_paths:** Maps each object path to its introspection data.
- **introspection:** Lists interfaces, methods, and properties for the object path.

---

## Use Cases and Benefits
- Discover available properties across all D-Bus services
- Find object paths for specific interfaces or properties
- Map telemetry data by correlating with Redfish identifiers
- Debug D-Bus issues by having complete service visibility
- Document system interfaces for development and integration

---

## Example: Searching the YAML Output
To find all object paths containing a specific property (e.g., `DEVICE_TYPE`):
```bash
grep -B 2 DEVICE_TYPE dbus_dump.yaml
```
Or, using `yq`:
```bash
yq '.. | select(has("introspection")) | select(.introspection | test("DEVICE_TYPE"))' dbus_dump.yaml
```

---

## Troubleshooting
- If you see malformed YAML output, ensure you're using the latest version of the script where log messages are properly directed to stderr.
- The script automatically tries different methods to enumerate object paths and falls back to common patterns if automated discovery fails.
- If a service cannot be introspected, the script logs a warning and continues with the rest.
- On some systems or containers, D-Bus access may be restricted; try running with `sudo` if needed.

---

## Example: Dumping a Service
```bash
./dbus_dump.sh xyz.openbmc_project.EntityManager
```

## Example: Dumping All Services to a Custom File
```bash
./dbus_dump.sh -o all_services.yaml
```

---

## Summary
This script is a one-stop solution for exploring, documenting, and debugging the D-Bus interface on Linux systems. It is especially useful for developers, integrators, and anyone working with system services, telemetry, or Redfish integration.

