# No Longer Evil Documentation

Comprehensive documentation for the No Longer Evil right-to-repair firmware for Nest Thermostats.

## Overview

This repository contains the Mintlify documentation for the NoLongerEvil project, which provides:

- **Hosted Service Setup Guide** - Easy plug-and-play setup for beginners
- **Self-Hosted Setup Guide** - Complete control for advanced users
- **API Reference** - Comprehensive documentation for both Thermostat Communication API (port 443) and Control API (port 8081)
- **Usage Guides** - Web dashboard and thermostat control tutorials
- **Technical Reference** - Architecture, security, and implementation details
- **Troubleshooting** - Solutions to common issues

## Local Development

### Prerequisites

- Node.js 18+ installed
- npm or yarn package manager

### Setup

1. Install Mintlify CLI:

```bash
npm i -g mintlify
```

2. Navigate to the documentation directory:

```bash
cd NoLongerEvil-docs
```

3. Start the development server:

```bash
mintlify dev
```

4. Open your browser to `http://localhost:3000`

The documentation will auto-reload as you edit `.mdx` files.

## API Documentation Guidelines

The API reference (`api-reference/`) documents both server APIs (NO frontend API routes):

### Thermostat Communication API (Port 443)

- Used by Nest thermostats only
- HTTPS with Basic Auth
- Endpoints: `/entry`, `/passphrase`, `/transport/*`, `/nest/weather/*`

### Control API (Port 8081)

- Used by dashboards and automation tools
- HTTP with optional auth
- Endpoints: `/command`, `/status`, `/api/devices`

### API Page Format

Each endpoint should include:

1. **Overview** - What the endpoint does
2. **Endpoint** - HTTP method and path
3. **Authentication** - Auth requirements
4. **Request** - Headers, body, parameters
5. **Response** - Success and error responses
6. **Examples** - cURL, JavaScript, Python
7. **Use Cases** - Real-world scenarios

## Maintenance

### Regular Updates

- **Firmware changes** - Update API docs when endpoints change
- **New features** - Document as they're added
- **Troubleshooting** - Add common issues from Discord/GitHub
- **Images** - Keep device identification images current

## License

Documentation is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

Code examples in documentation are licensed under MIT

---

Built with [Mintlify](https://mintlify.com) 🌿
