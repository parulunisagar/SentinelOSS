# SentinelOSS

SentinelOSS is an open-source Linux monitoring and security analytics platform built in C++17.

## Features

* Real-time CPU monitoring
* Memory monitoring
* Disk usage monitoring
* Network statistics
* Uptime tracking
* REST API
* Security-focused architecture
* Oracle Linux support

 Live Demo

http://80.225.199.123:8080

## Screenshots

Add screenshots here.

## Installation

```bash
sudo dnf install gcc gcc-c++ cmake make sqlite-devel openssl-devel git -y

git clone https://github.com/YOUR_USERNAME/SentinelOSS.git
cd SentinelOSS

mkdir build
cd build

cmake ..
make -j$(nproc)

./sentinel
```

## Roadmap

* Alert system
* Dashboard improvements
* Historical metrics
* Security event tracking

## License

MIT License
