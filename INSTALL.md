# INSTALL

This document explains how to build, install, run and troubleshoot Sentinel OSS (C++17, CMake, SQLite).

Supported environments
- Oracle Linux / RHEL / CentOS 8/9, Fedora, Ubuntu (Linux): primary target for production deployment.
- WSL2 (Windows) or native Linux for local development.
- Docker: quick evaluation container.

Prerequisites
- C++17 toolchain: `g++` (>=9) or `clang++`
- CMake >= 3.10
- make / ninja
- SQLite3 development headers
- pkg-config (recommended)
- systemd (for service installation)
- firewall-cmd (firewalld) or iptables for firewall config

Install packages (Oracle Linux 9 example)
```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y cmake gcc-c++ sqlite-devel pkgconfig
```
Ubuntu/Debian example
```bash
sudo apt update
sudo apt install -y build-essential cmake libsqlite3-dev pkg-config
```

Build from source
1. Clone the repo (replace URL with your fork):
```bash
git clone https://github.com/<your-org>/SentinelOSS.git "sentineloss" && cd "sentineloss"
```
2. Create a build directory, run CMake and build:
```bash
mkdir -p build && cd build
cmake ..
cmake --build . -- -j$(nproc)
```
3. After a successful build the server binary will be located in `build/src/sentinel_server`.

Install the binary (system-wide)
```bash
sudo cp build/src/sentinel_server /usr/local/bin/sentinel_server
sudo chmod 755 /usr/local/bin/sentinel_server
```

Systemd service (recommended for production)
Create `/etc/systemd/system/sentinel.service` with the following contents (edit paths/user as needed):

```
[Unit]
Description=Sentinel OSS HTTP server
After=network.target

[Service]
Type=simple
User=opc
Group=opc
WorkingDirectory=/home/opc/sentinel   # adjust to your workspace without spaces
ExecStart=/usr/local/bin/sentinel_server
Restart=always
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

Notes:
- Avoid spaces in `WorkingDirectory` (create a symlink without spaces if needed, e.g. `/home/opc/sentinel -> /home/opc/SentinelOSS/project v2`).
- Ensure `ExecStart` points to the installed binary and the file is executable.

Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable sentinel.service
sudo systemctl start sentinel.service
sudo systemctl status sentinel.service
```

Firewall / Networking
- To allow port 8080 (default) through firewalld:
```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```
- Verify listener:
```bash
sudo ss -tulnp | grep 8080
```
- If you use a cloud provider, also open port 8080 in the cloud security group / network ACLs.

Run locally (non-systemd)
You can run the binary in foreground for development and debugging:
```bash
/home/opc/SentinelOSS/project\ v2/build/src/sentinel_server
# or
/usr/local/bin/sentinel_server
```
Stop any running systemd service first to free the port:
```bash
sudo systemctl stop sentinel.service
```

Verify endpoints
- Metrics: `http://<host>:8080/api/metrics`
- Health: `http://<host>:8080/api/health`
- Alerts: `http://<host>:8080/api/alerts`
- Dashboard UI: `http://<host>:8080/`

Troubleshooting
- Failed to bind port 8080: another process is listening or previous instance is still running. Find and kill it:
```bash
sudo ss -tulnp | grep ':8080'
sudo kill <PID> || sudo kill -9 <PID>
```
- Systemd CHDIR/No such file or directory: fix `WorkingDirectory` in the service file to point to an existing directory (no spaces), or create a symlink without spaces.
- Service keeps restarting: inspect logs:
```bash
sudo journalctl -u sentinel.service -n 200 --no-pager
sudo journalctl -u sentinel.service -f
```
- Binary not found after `cp`: ensure you built under `build/src/sentinel_server` and the `cp` path matches. Use `find build -type f -name sentinel_server -ls` to locate the executable.

Docker (optional quick run)
- Example Dockerfile (minimal):
```
FROM fedora:latest
RUN dnf -y install cmake gcc-c++ sqlite-devel && dnf clean all
COPY . /src
WORKDIR /src/build
RUN cmake .. && cmake --build . -- -j
EXPOSE 8080
CMD ["/src/build/src/sentinel_server"]
```
- Build and run container:
```bash
docker build -t sentineloss:latest .
docker run --rm -p 8080:8080 sentineloss:latest
```

Development tips
- Disable browser caching when testing UI changes (Ctrl+Shift+R or open in an Incognito window).
- The dashboard HTML may be embedded in the binary; rebuild+install the binary to update the UI served by systemd.
- Use `curl` to inspect server responses from the host to avoid browser caching confusion:
```bash
curl -sS http://127.0.0.1:8080/ | sed -n '1,120p'
```

Contributing and testing
- Run unit tests (if present) after changes; follow the project's `CI` or `tests` instructions in `README.md`.

If something fails, paste the exact `journalctl` output and the output of `ss -tulnp | grep 8080` and I will help diagnose.
