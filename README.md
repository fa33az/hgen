# HGEN - HTTP Custom Configuration Hub

HGEN is a modern, high-performance web utility built with Vue 3, TypeScript, and Vite, designed to automate and simplify the generation of configuration profiles for the **HTTP Custom** VPN application.

## Key Features

1. **🚀 Automatic GeoIP & ISP Detection**: Identifies your local network ISP automatically to suggest active bug domains.
2. **🧬 Multi-Protocol Tunneling**: Generates Direct Payload (HTTP), SSL/TLS (SNI), SSH WebSocket (CDN), SSL + WebSocket (SNI + WS), OpenVPN WS, and V2Ray (VMess/VLess WS CDN) formats.
3. **🛠️ Visual Payload Builder**: An interactive checkbox dashboard to customize injection payload templates (Keep-Alive, Online Host, User-Agent, Referer, Front Query).
4. **🔗 V2Ray Link Converter**: Automatically generates standard shareable `vmess://` and `vless://` URLs.
5. **⚡ Server Latency / Ping Tester**: Live millisecond round-trip latency testing (RTT) for pre-configured SSH servers.
6. **📱 QR Code Scanner & Sharing**: Share config URLs or scan config clipboards instantly on phones.
7. **🔒 Lock & Expiry Controls**: Password lock your configurations and set expiration dates.
8. **🌐 Utilities Hub**: Built-in Base64 translator and DNS Host-to-IP resolver.

## Quick Start

### Prerequisites
- Node.js (v18 or higher)
- npm

### Installation

1. Clone the repository:
   ```bash
   git clone <repository_url>
   cd hgen
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Run the development server:
   ```bash
   npm run dev
   ```

4. Build the production application:
   ```bash
   npm run build
   ```

## Development Stack
- **Framework**: Vue 3 (Composition API)
- **Language**: TypeScript
- **Bundler**: Vite
- **Styling**: Vanilla CSS (Cyberpunk theme)
- **Icons**: Lucide Icons
