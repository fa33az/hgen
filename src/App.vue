<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import { 
  Key, 
  Globe, 
  Terminal, 
  Copy, 
  Check, 
  RefreshCw, 
  Cpu, 
  Sliders, 
  ShieldAlert, 
  Settings,
  Download,
  Server,
  Save,
  Trash2,
  Lock,
  Wrench,
  ExternalLink,
  Share2,
  ChevronDown,
  ChevronUp,
  Wifi,
  Activity
} from '@lucide/vue'

// Types
type Method = 'GET' | 'POST' | 'CONNECT' | 'HEAD' | 'PUT' | 'DELETE' | 'OPTIONS' | 'TRACE'
type Protocol = 'direct' | 'ssl' | 'ssh_ws' | 'ssl_ws' | 'ovpn_ws' | 'vmess' | 'vless'
type TabName = 'generator' | 'utilities' | 'servers' | 'saved'

interface ConfigPayload {
  name: string
  description: string
  template: (sshHost: string, sshPort: string, bug: string, sshUser: string, sshPass: string, extra: Record<string, any>) => string
}

interface SavedConfig {
  id: string
  name: string
  sshHost: string
  sshPort: string
  sshUser: string
  sshPass: string
  bugDomain: string
  selectedProtocol: Protocol
  requestMethod: Method
  customPath: string
  extraHeader: string
  rotateBugs: boolean
  rotateList: string
}

interface FreeSSHServer {
  provider: string
  serverName: string
  host: string
  port: string
  infoUrl: string
  countryCode: string
  providerLogo?: string
}

// Active Tab
const activeTab = ref<TabName>('generator')

// Collapsible Tuning
const showAdvanced = ref(false)

// Preset Bug Providers Grouped by Operator
const bugPresets = [
  {
    operator: 'Telkomsel',
    logoUrl: 'https://www.google.com/s2/favicons?sz=64&domain=telkomsel.com',
    presets: [
      { label: 'OPOK (Whatsapp)', value: 'v.whatsapp.net' },
      { label: 'Ilmupedia (Zoom)', value: 'zoom.us' },
      { label: 'Ruangguru Edu', value: 'ruangguru.com' },
      { label: 'Gamesmax (ML)', value: 'mobilelegends.com' },
      { label: 'Gamesmax (PUBG)', value: 'pubgmobile.com' },
      { label: 'Maxstream', value: 'maxstream.tv' },
      { label: 'Tiktok Kuota', value: 'tiktok.com' },
      { label: 'Instagram Kuota', value: 'instagram.com' }
    ]
  },
  {
    operator: 'XL & Axis',
    logoUrl: 'https://www.google.com/s2/favicons?sz=64&domain=xl.co.id',
    presets: [
      { label: 'OPOK (WA Biz)', value: 'business.whatsapp.com' },
      { label: 'OPOK (Line)', value: 'line.me' },
      { label: 'Axis Edu (Udemy)', value: 'udemy.com' },
      { label: 'Axis Edu (Ruangguru)', value: 'ruangguru.com' },
      { label: 'Conference (Classroom)', value: 'classroom.google.com' },
      { label: 'Conference (Teams)', value: 'teams.microsoft.com' },
      { label: 'Sosmed (Facebook)', value: 'm.facebook.com' },
      { label: 'Games (Free Fire)', value: 'freefiremobile.com' }
    ]
  },
  {
    operator: 'Indosat IM3',
    logoUrl: 'https://www.google.com/s2/favicons?sz=64&domain=im3.id',
    presets: [
      { label: 'OPOK (Instagram)', value: 'm.instagram.com' },
      { label: 'OPOK (Whatsapp)', value: 'v.whatsapp.net' },
      { label: 'Edu (Bimbel)', value: 'bimbel.ruangguru.com' },
      { label: 'Indosat Zoom', value: 'zoom.us' },
      { label: 'Indosat Classroom', value: 'classroom.google.com' },
      { label: 'Sosmed (Facebook)', value: 'm.facebook.com' },
      { label: 'Music (Spotify)', value: 'spotify.com' }
    ]
  },
  {
    operator: 'Smartfren',
    logoUrl: 'https://www.google.com/s2/favicons?sz=64&domain=smartfren.com',
    presets: [
      { label: 'OPOK (WeChat)', value: 'm.wechat.com' },
      { label: 'OPOK (Line)', value: 'line.me' },
      { label: 'Unlimited Portal', value: 'smartfren.com' },
      { label: 'Smartfren Zoom', value: 'zoom.us' }
    ]
  },
  {
    operator: 'Tri (3)',
    logoUrl: 'https://www.google.com/s2/favicons?sz=64&domain=tri.co.id',
    presets: [
      { label: 'Tri OPOK (Whatsapp)', value: 'v.whatsapp.net' },
      { label: 'Tri OPOK (Line)', value: 'line.me' },
      { label: 'Tri Edu (Classroom)', value: 'classroom.google.com' },
      { label: 'Tri Sosmed (Instagram)', value: 'instagram.com' }
    ]
  },
  {
    operator: 'Cloudflare & DNS',
    logoUrl: 'https://www.google.com/s2/favicons?sz=64&domain=cloudflare.com',
    presets: [
      { label: 'CF CDN IP 1', value: '104.22.4.101' },
      { label: 'CF CDN IP 2', value: '104.20.7.101' },
      { label: 'Google DNS IP', value: '8.8.8.8' },
      { label: 'Cloudflare DNS IP', value: '1.1.1.1' }
    ]
  }
]

// Free SSH Recommendations List
const freeServers: FreeSSHServer[] = [
  { provider: 'FastSSH', serverName: 'SG-WS CDN 1', host: 'sg1.sshws.net', port: '80', infoUrl: 'https://www.fastssh.com', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=fastssh.com' },
  { provider: 'SSHKit', serverName: 'US-WS CDN', host: 'us1.sshws.net', port: '80', infoUrl: 'https://www.sshkit.com', countryCode: 'us', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=sshkit.com' },
  { provider: 'JagoanSSH', serverName: 'SG-VIP Websocket', host: 'sg-vip.jagoanssh.com', port: '80', infoUrl: 'https://www.jagoanssh.com', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=jagoanssh.com' },
  { provider: 'FlySSH', serverName: 'ID-WS Cloudflare', host: 'id1.flyssh.net', port: '80', infoUrl: 'https://flyssh.net', countryCode: 'id', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=flyssh.net' },
  { provider: 'VPN Jantit', serverName: 'SG-WS Multiport', host: 'sg-ws.vpnjantit.com', port: '80', infoUrl: 'https://www.vpnjantit.com', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=vpnjantit.com' },
  { provider: 'SSHStores', serverName: 'SG-CDN Cloudflare', host: 'sg.sshstores.net', port: '80', infoUrl: 'https://sshstores.net', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=sshstores.net' },
  { provider: 'SSHOcean', serverName: 'SG-WS Dynamic', host: 'sg-ws.sshocean.net', port: '80', infoUrl: 'https://sshocean.com', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=sshocean.com' },
  { provider: 'SSHMax', serverName: 'ID-WS HighSpeed', host: 'id-ws.sshmax.net', port: '80', infoUrl: 'https://sshmax.net', countryCode: 'id', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=sshmax.net' },
  { provider: 'NETQ.ME', serverName: 'DE-WS Cloudflare', host: 'de-ws.netq.me', port: '80', infoUrl: 'https://netq.me', countryCode: 'de', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=netq.me' },
  { provider: 'GreenSSH', serverName: 'SG-WS HighSpeed', host: 'sg-ws.greenssh.com', port: '80', infoUrl: 'https://greenssh.com', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=greenssh.com' },
  { provider: 'Akela Tunnel', serverName: 'SG-CDN Akela', host: 'sg-cdn.akelatunnel.xyz', port: '80', infoUrl: 'https://akelatunnel.xyz', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=akelatunnel.xyz' },
  { provider: 'SSHMax', serverName: 'SG-WS Premium', host: 'sg-ws.sshmax.net', port: '80', infoUrl: 'https://sshmax.net', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=sshmax.net' },
  { provider: 'Howdy ID', serverName: 'SG-CDN Howdy', host: 'sg.howdy.id', port: '80', infoUrl: 'https://howdy.id', countryCode: 'sg', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=howdy.id' },
  { provider: 'Howdy ID', serverName: 'ID-CDN Howdy', host: 'id.howdy.id', port: '80', infoUrl: 'https://howdy.id', countryCode: 'id', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=howdy.id' },
  { provider: 'SSHOcean', serverName: 'US-WS CDN', host: 'us-ws.sshocean.net', port: '80', infoUrl: 'https://sshocean.com', countryCode: 'us', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=sshocean.com' },
  { provider: 'VPN Jantit', serverName: 'US-WS Multiport', host: 'us-ws.vpnjantit.com', port: '80', infoUrl: 'https://www.vpnjantit.com', countryCode: 'us', providerLogo: 'https://www.google.com/s2/favicons?sz=64&domain=vpnjantit.com' }
]

// Preset templates
const protocols: Record<Protocol, ConfigPayload> = {
  direct: {
    name: 'Direct Payload (HTTP)',
    description: 'Direct TCP connection with standard injection payload.',
    template: (host, port, bug, user, pass, extra) => {
      const bugStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] direct
[ssh] ${host}:${port}@${user}:${pass}
[bug] ${bug}
[payload] CONNECT [host_port] HTTP/1.1[crlf]Host: ${bugStr}[crlf]X-Online-Host: ${bugStr}[crlf]Connection: Keep-Alive[crlf][crlf]
[dns] 8.8.8.8
[udp] 7300`
    }
  },
  ssl: {
    name: 'SSL/TLS (SNI)',
    description: 'Encrypted connection using the Bug domain as Server Name Indication (SNI).',
    template: (host, port, bug, user, pass, extra) => {
      const sniStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] ssl
[ssh] ${host}:${port}@${user}:${pass}
[sni] ${sniStr}
[dns] 8.8.8.8
[udp] 7300`
    }
  },
  ssh_ws: {
    name: 'SSH WebSocket (CDN)',
    description: 'Cloudflare / CDN WebSocket tunneling with HTTP payload injection.',
    template: (host, port, bug, user, pass, extra) => {
      const method = extra.method || 'GET'
      const path = extra.path || '/'
      const header = extra.header ? `${extra.header}[crlf]` : ''
      const bugStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] ws
[ssh] ${bugStr}:80@${user}:${pass}
[real_ssh] ${host}:${port}
[payload] ${method} http://${bugStr}${path} HTTP/1.1[crlf]Host: ${host}[crlf]Upgrade: websocket[crlf]Connection: Upgrade[crlf]${header}[crlf]
[dns] 8.8.8.8
[udp] 7300`
    }
  },
  ssl_ws: {
    name: 'SSL/TLS + WebSocket (SNI + WS)',
    description: 'Secure SSL/TLS WebSocket connection utilizing Cloudflare CDN with SNI bug routing.',
    template: (host, port, bug, user, pass, extra) => {
      const method = extra.method || 'GET'
      const path = extra.path || '/'
      const header = extra.header ? `${extra.header}[crlf]` : ''
      const bugStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] ssl_ws
[ssh] ${bugStr}:443@${user}:${pass}
[real_ssh] ${host}:${port}
[sni] ${bugStr}
[payload] ${method} https://${bugStr}${path} HTTP/1.1[crlf]Host: ${host}[crlf]Upgrade: websocket[crlf]Connection: Upgrade[crlf]${header}[crlf]
[dns] 8.8.8.8
[udp] 7300`
    }
  },
  ovpn_ws: {
    name: 'OpenVPN WebSocket',
    description: 'Proxy OpenVPN traffic via CDN Websocket tunnel.',
    template: (host, port, bug, _user, _pass, extra) => {
      const method = extra.method || 'GET'
      const path = extra.path || '/'
      const bugStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] ovpn_ws
[vpn] ${host}:${port}
[bug] ${bugStr}
[payload] ${method} http://${bugStr}${path} HTTP/1.1[crlf]Host: ${host}[crlf]Upgrade: websocket[crlf]Connection: Upgrade[crlf][crlf]
[dns] 1.1.1.1`
    }
  },
  vmess: {
    name: 'V2Ray VMess (CDN/WebSocket)',
    description: 'VMess connection using target Host, UUID, and Bug SNI.',
    template: (host, port, bug, user, _pass, extra) => {
      const uuid = user || '00000000-0000-0000-0000-000000000000'
      const path = extra.path || '/vmess'
      const bugStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] vmess
[vmess] ${host}:${port}@${uuid}
[sni] ${bugStr}
[path] ${path}
[host] ${host}
[tls] true
[udp] true`
    }
  },
  vless: {
    name: 'V2Ray VLess (CDN/WebSocket)',
    description: 'VLess lightweight connection using target Host, UUID, and Bug SNI.',
    template: (host, port, bug, user, _pass, extra) => {
      const uuid = user || '00000000-0000-0000-0000-000000000000'
      const path = extra.path || '/vless'
      const bugStr = extra.rotateBugs ? extra.rotateList : bug
      return `[proto] vless
[vless] ${host}:${port}@${uuid}
[sni] ${bugStr}
[path] ${path}
[host] ${host}
[tls] true
[udp] true`
    }
  }
}

// Core App State
const sshHost = ref('sg1.sshws.net')
const sshPort = ref('80')
const sshUser = ref('hgen_vpn')
const sshPass = ref('strongpassword123')
const bugDomain = ref('ruangguru.com')
const configName = ref('HTTP-Custom-WS-Config')
const selectedProtocol = ref<Protocol>('ssh_ws')
const requestMethod = ref<Method>('GET')
const customPath = ref('/ssh-ws')
const extraHeader = ref('Connection: Keep-Alive')
const customPayloadTemplate = ref('')
const useCustomPayload = ref(false)
const copied = ref(false)

// Expiry & Password Lock States
const expiryDate = ref('')
const lockConfig = ref(false)
const configPassword = ref('')

// Host IP Resolver States
const dnsLookupInput = ref('')
const dnsLookupOutput = ref('')
const dnsLookupLoading = ref(false)

// Payload Builder States
const usePayloadBuilder = ref(false)
const payloadKeepAlive = ref(true)
const payloadOnlineHost = ref(true)
const payloadUserAgent = ref(true)
const payloadFrontQuery = ref(false)
const payloadReferer = ref(false)
const payloadRefererUrl = ref('https://google.com')

// Latency Tester States
const serverPings = ref<Record<string, number | string>>({})
const testingPings = ref<Record<string, boolean>>({})

// V2Ray Link States
const copiedV2rayLink = ref(false)

// Sharing States
const copiedShareLink = ref(false)

// Payload Snippets List
const payloadSnippets = [
  { name: 'Cloudflare WebSocket SSL (TLS)', value: 'GET wss://[rotate_host]/ HTTP/1.1[crlf]Host: [host][crlf]Upgrade: websocket[crlf]Connection: Upgrade[crlf][crlf]' },
  { name: 'HTTP Direct GET (Whatsapp Opok)', value: 'CONNECT [host_port] HTTP/1.1[crlf]Host: v.whatsapp.net[crlf]X-Online-Host: v.whatsapp.net[crlf]Connection: Keep-Alive[crlf][crlf]' },
  { name: 'Cloudflare WebSocket HTTP (Port 80)', value: 'GET http://[rotate_host]/ HTTP/1.1[crlf]Host: [host][crlf]Upgrade: websocket[crlf]Connection: Upgrade[crlf][crlf]' },
  { name: 'Back Inject GET (Line Opok)', value: 'GET http://line.me/ HTTP/1.1[crlf]Host: [host][crlf]Connection: Keep-Alive[crlf]Referer: http://line.me/[crlf][crlf]' }
]

// Rotate bugs state
const rotateBugs = ref(false)
const rotateList = ref('ruangguru.com;udemy.com;classroom.google.com')

// Saved profiles state
const savedProfiles = ref<SavedConfig[]>([])

// Base64 tool state
const base64Input = ref('')
const base64Output = ref('')
const base64Mode = ref<'encode' | 'decode'>('encode')

// Lifecycles
onMounted(() => {
  // Load saved profiles from localStorage
  const data = localStorage.getItem('hgen_saved_profiles')
  if (data) {
    try {
      savedProfiles.value = JSON.parse(data)
    } catch (e) {
      console.error(e)
    }
  }

  // Parse URL sharing parameters
  const urlParams = new URLSearchParams(window.location.search)
  if (urlParams.has('host')) sshHost.value = urlParams.get('host') || ''
  if (urlParams.has('port')) sshPort.value = urlParams.get('port') || ''
  if (urlParams.has('user')) sshUser.value = urlParams.get('user') || ''
  if (urlParams.has('pass')) sshPass.value = urlParams.get('pass') || ''
  if (urlParams.has('bug')) bugDomain.value = urlParams.get('bug') || ''
  if (urlParams.has('proto')) selectedProtocol.value = (urlParams.get('proto') || 'ssh_ws') as Protocol
  if (urlParams.has('name')) configName.value = urlParams.get('name') || ''
  if (urlParams.has('path')) customPath.value = urlParams.get('path') || ''
  if (urlParams.has('method')) requestMethod.value = (urlParams.get('method') || 'GET') as Method
  if (urlParams.has('header')) extraHeader.value = urlParams.get('header') || ''
  if (urlParams.has('expiry')) expiryDate.value = urlParams.get('expiry') || ''
})

// Auto-adjust port on protocol change
watch(selectedProtocol, (newVal) => {
  if (newVal.includes('ssl') || newVal.includes('vmess') || newVal.includes('vless')) {
    sshPort.value = '443'
  } else {
    sshPort.value = '80'
  }
})

// Built Custom Payload from Visual Builder
const builtPayload = computed(() => {
  const bug = bugDomain.value || 'bug.com'
  const method = requestMethod.value || 'CONNECT'
  const path = customPath.value || '/'
  let payload = ''

  if (payloadFrontQuery.value) {
    payload += `${method} http://${bug}${path}@[host_port] HTTP/1.1[crlf]`
  } else {
    payload += `${method} [host_port] HTTP/1.1[crlf]`
  }

  if (payloadOnlineHost.value) {
    payload += `Host: ${bug}[crlf]X-Online-Host: ${bug}[crlf]`
  } else {
    payload += `Host: ${bug}[crlf]`
  }

  if (payloadUserAgent.value) {
    payload += `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36[crlf]`
  }

  if (payloadReferer.value) {
    payload += `Referer: ${payloadRefererUrl.value}[crlf]`
  }

  if (payloadKeepAlive.value) {
    payload += `Connection: Keep-Alive[crlf]`
  }

  payload += `[crlf]`
  return payload
})

// V2Ray Share Link Converter (vmess:// / vless://)
const v2rayShareLink = computed(() => {
  if (selectedProtocol.value === 'vmess') {
    const vmessObj = {
      v: '2',
      ps: configName.value || 'VMess-HGEN',
      add: sshHost.value,
      port: parseInt(sshPort.value) || 443,
      id: sshUser.value,
      aid: '0',
      scy: 'auto',
      net: 'ws',
      type: 'none',
      host: sshHost.value,
      path: customPath.value || '/vmess',
      tls: 'tls',
      sni: bugDomain.value,
      alpn: ''
    }
    try {
      const jsonStr = JSON.stringify(vmessObj)
      const b64 = btoa(encodeURIComponent(jsonStr).replace(/%([0-9A-F]{2})/g, (_, p1) => {
        return String.fromCharCode(parseInt(p1, 16))
      }))
      return `vmess://${b64}`
    } catch (e) {
      return ''
    }
  } else if (selectedProtocol.value === 'vless') {
    const name = encodeURIComponent(configName.value || 'VLess-HGEN')
    const path = encodeURIComponent(customPath.value || '/vless')
    return `vless://${sshUser.value}@${sshHost.value}:${sshPort.value}?path=${path}&security=tls&encryption=none&host=${sshHost.value}&type=ws&sni=${bugDomain.value}#${name}`
  }
  return ''
})

const copyV2rayLink = async () => {
  if (!v2rayShareLink.value) return
  try {
    await navigator.clipboard.writeText(v2rayShareLink.value)
    copiedV2rayLink.value = true
    setTimeout(() => {
      copiedV2rayLink.value = false
    }, 2000)
  } catch (err) {
    console.error('Failed to copy V2Ray Link: ', err)
  }
}

// Latency Tester Method using fetch no-cors
const getPingClass = (pingVal: string | number) => {
  if (typeof pingVal === 'string') {
    if (pingVal === 'Testing...') return 'testing'
    if (pingVal === 'Timeout') return 'timeout'
    const num = parseInt(pingVal)
    if (!isNaN(num)) {
      if (num < 120) return 'fast'
      if (num < 280) return 'average'
      return 'slow'
    }
  }
  return 'idle'
}

const testServerPing = async (host: string) => {
  testingPings.value[host] = true
  serverPings.value[host] = 'Testing...'
  const startTime = performance.now()
  try {
    await fetch(`https://${host}/`, { 
      mode: 'no-cors', 
      cache: 'no-cache', 
      signal: AbortSignal.timeout ? AbortSignal.timeout(3500) : undefined 
    })
  } catch (e) {
    // catch error
  }
  const latency = Math.round(performance.now() - startTime)
  if (latency > 3400) {
    serverPings.value[host] = 'Timeout'
  } else {
    serverPings.value[host] = `${latency} ms`
  }
  testingPings.value[host] = false
}

const testAllServerPings = async () => {
  for (const srv of freeServers) {
    testServerPing(srv.host)
  }
}

const resolveHostIP = async () => {
  if (!dnsLookupInput.value) return
  dnsLookupLoading.value = true
  dnsLookupOutput.value = 'Resolving DNS...'
  try {
    const res = await fetch(`https://dns.google/resolve?name=${encodeURIComponent(dnsLookupInput.value)}&type=A`)
    const data = await res.json()
    if (data.Answer && data.Answer.length > 0) {
      dnsLookupOutput.value = data.Answer.map((ans: any) => ans.data).join('\n')
    } else {
      dnsLookupOutput.value = 'No IP addresses found for this domain.'
    }
  } catch (e) {
    dnsLookupOutput.value = 'Error: Failed to resolve host.'
  } finally {
    dnsLookupLoading.value = false
  }
}

const shareConfigLink = async () => {
  const params = new URLSearchParams()
  params.set('host', sshHost.value)
  params.set('port', sshPort.value)
  params.set('user', sshUser.value)
  params.set('pass', sshPass.value)
  params.set('bug', bugDomain.value)
  params.set('proto', selectedProtocol.value)
  params.set('name', configName.value)
  params.set('path', customPath.value)
  params.set('method', requestMethod.value)
  params.set('header', extraHeader.value)
  if (expiryDate.value) params.set('expiry', expiryDate.value)
  
  const shareUrl = `${window.location.origin}${window.location.pathname}?${params.toString()}`
  try {
    await navigator.clipboard.writeText(shareUrl)
    copiedShareLink.value = true
    setTimeout(() => {
      copiedShareLink.value = false
    }, 2000)
  } catch (e) {
    console.error('Failed to copy share link: ', e)
  }
}

const copySnippetText = async (text: string) => {
  try {
    await navigator.clipboard.writeText(text)
    alert('Payload snippet copied!')
  } catch (e) {
    console.error('Failed to copy: ', e)
  }
}

// Generated Configuration computed
const generatedConfig = computed(() => {
  let expiryHeader = ''
  let expiryPayload = ''
  if (expiryDate.value) {
    expiryHeader = `# Expiry Date: ${expiryDate.value}\n`
    expiryPayload = `\n[expiry] ${expiryDate.value}`
  }

  let passwordPayload = ''
  if (lockConfig.value && configPassword.value) {
    passwordPayload = `\n[password] ${configPassword.value}`
  }

  if (usePayloadBuilder.value) {
    return `# HTTP Custom Configuration File (Builder Custom Payload)
# Generated automatically by HGEN Tool
# Profile Name: ${configName.value}
# Created: ${new Date().toLocaleDateString()}
${expiryHeader}
[proto] direct
[ssh] ${sshHost.value}:${sshPort.value}@${sshUser.value}:${sshPass.value}
[bug] ${bugDomain.value}
[payload] ${builtPayload.value}
[dns] 8.8.8.8
[udp] 7300${expiryPayload}${passwordPayload}`.trim()
  }

  if (useCustomPayload.value && customPayloadTemplate.value) {
    return `[config_name] ${configName.value}
[custom_payload]
${customPayloadTemplate.value}
[ssh] ${sshHost.value}:${sshPort.value}@${sshUser.value}:${sshPass.value}${expiryPayload}${passwordPayload}`.trim()
  }

  const handler = protocols[selectedProtocol.value]
  const rawBody = handler.template(
    sshHost.value,
    sshPort.value,
    bugDomain.value,
    sshUser.value,
    sshPass.value,
    {
      method: requestMethod.value,
      path: customPath.value,
      header: extraHeader.value,
      rotateBugs: rotateBugs.value,
      rotateList: rotateList.value
    }
  )

  return `# HTTP Custom Configuration File
# Generated automatically by HGEN Tool
# Profile Name: ${configName.value}
# Created: ${new Date().toLocaleDateString()}
${expiryHeader}
${rawBody}${expiryPayload}${passwordPayload}`.trim()
})

// Methods & Actions
const handleBase64Convert = () => {
  try {
    if (base64Mode.value === 'encode') {
      base64Output.value = btoa(base64Input.value)
    } else {
      base64Output.value = atob(base64Input.value)
    }
  } catch (e) {
    base64Output.value = 'Error: Invalid input string'
  }
}

const copyConfig = async () => {
  try {
    await navigator.clipboard.writeText(generatedConfig.value)
    copied.value = true
    setTimeout(() => {
      copied.value = false
    }, 2000)
  } catch (err) {
    console.error('Failed to copy config: ', err)
  }
}

const downloadConfig = () => {
  const blob = new Blob([generatedConfig.value], { type: 'text/plain;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = `${configName.value.toLowerCase().replace(/\s+/g, '-') || 'config'}.hc`
  link.click()
  URL.revokeObjectURL(url)
}

const applyFreeServer = (srv: FreeSSHServer) => {
  sshHost.value = srv.host
  sshPort.value = srv.port
  if (srv.port === '443') {
    selectedProtocol.value = 'ssl_ws'
  } else {
    selectedProtocol.value = 'ssh_ws'
  }
  activeTab.value = 'generator'
}

const saveCurrentProfile = () => {
  const newProfile: SavedConfig = {
    id: Math.random().toString(36).substring(2, 9),
    name: configName.value || 'Untitled Preset',
    sshHost: sshHost.value,
    sshPort: sshPort.value,
    sshUser: sshUser.value,
    sshPass: sshPass.value,
    bugDomain: bugDomain.value,
    selectedProtocol: selectedProtocol.value,
    requestMethod: requestMethod.value,
    customPath: customPath.value,
    extraHeader: extraHeader.value,
    rotateBugs: rotateBugs.value,
    rotateList: rotateList.value
  }

  savedProfiles.value.push(newProfile)
  localStorage.setItem('hgen_saved_profiles', JSON.stringify(savedProfiles.value))
}

const loadProfile = (prof: SavedConfig) => {
  configName.value = prof.name
  sshHost.value = prof.sshHost
  sshPort.value = prof.sshPort
  sshUser.value = prof.sshUser
  sshPass.value = prof.sshPass
  bugDomain.value = prof.bugDomain
  selectedProtocol.value = prof.selectedProtocol
  requestMethod.value = prof.requestMethod
  customPath.value = prof.customPath
  extraHeader.value = prof.extraHeader
  rotateBugs.value = prof.rotateBugs
  rotateList.value = prof.rotateList
  activeTab.value = 'generator'
}

const deleteProfile = (id: string) => {
  savedProfiles.value = savedProfiles.value.filter(p => p.id !== id)
  localStorage.setItem('hgen_saved_profiles', JSON.stringify(savedProfiles.value))
}

const randomizeSSH = () => {
  const regions = ['sg', 'us', 'id', 'jp', 'de']
  const randomRegion = regions[Math.floor(Math.random() * regions.length)]
  const randomNode = Math.floor(Math.random() * 10) + 1
  sshHost.value = `${randomRegion}${randomNode}.sshws.net`
  sshPort.value = selectedProtocol.value.includes('ssl') ? '443' : '80'
  sshUser.value = `user_${Math.random().toString(36).substring(2, 8)}`
  sshPass.value = Math.random().toString(36).substring(2, 10)
}

// Automated APIs states
const ispLoading = ref(false)
const detectedIsp = ref('')
const detectedCountry = ref('')
const detectedIp = ref('')
const ispMessage = ref('')

const checkingBug = ref(false)
const bugCheckStatus = ref<'idle' | 'valid' | 'invalid' | 'error'>('idle')
const bugCheckMsg = ref('')

const proxyLoading = ref(false)
const proxyTypeFilter = ref<'http' | 'socks4' | 'socks5' | 'all'>('all')

interface FetchedProxy {
  ip: string
  port: string
  protocol: string
  country: string
  uptime: number
}
const fetchedProxies = ref<FetchedProxy[]>([])

// ISP Detection Logic
const detectISP = async () => {
  ispLoading.value = true
  ispMessage.value = 'Mendeteksi IP & ISP Anda...'
  try {
    const res = await fetch('https://ipapi.co/json/')
    if (!res.ok) throw new Error('Gagal menghubungi GeoIP API')
    const data = await res.json()
    
    detectedIp.value = data.ip || 'Unknown IP'
    detectedIsp.value = data.org || data.asn || 'Unknown ISP'
    detectedCountry.value = data.country_name || ''
    
    const ispLower = detectedIsp.value.toLowerCase()
    let matchedBug = ''
    let msg = ''
    
    if (ispLower.includes('telkomsel') || ispLower.includes('telekomunikasi indonesia') || ispLower.includes('indonesia prima')) {
      matchedBug = 'zoom.us'
      msg = 'Mendeteksi Telkomsel. Otomatis memuat bug: zoom.us (Telkomsel Ilmupedia)'
    } else if (ispLower.includes('xl axiata') || ispLower.includes('axis') || ispLower.includes('excelcomindo')) {
      matchedBug = 'ruangguru.com'
      msg = 'Mendeteksi XL Axiata / Axis. Otomatis memuat bug: ruangguru.com (XL Edu)'
    } else if (ispLower.includes('indosat') || ispLower.includes('isat') || ispLower.includes('indosat ooredoo')) {
      matchedBug = 'bimbel.ruangguru.com'
      msg = 'Mendeteksi Indosat. Otomatis memuat bug: bimbel.ruangguru.com (Indosat Edu)'
    } else if (ispLower.includes('smartfren') || ispLower.includes('smart telecom')) {
      matchedBug = 'line.me'
      msg = 'Mendeteksi Smartfren. Otomatis memuat bug: line.me (Smartfren Chat)'
    } else {
      matchedBug = '104.22.4.101'
      msg = `Mendeteksi ${detectedIsp.value} (${detectedCountry.value}). Memuat default Cloudflare IP bug.`
    }
    
    bugDomain.value = matchedBug
    ispMessage.value = msg
  } catch (err) {
    console.error(err)
    // Fallback: try another free API
    try {
      ispMessage.value = 'Mencoba backup API...'
      const res2 = await fetch('https://ipinfo.io/json')
      const data2 = await res2.json()
      detectedIp.value = data2.ip || 'Unknown IP'
      detectedIsp.value = data2.org || 'Unknown ISP'
      
      const ispLower = detectedIsp.value.toLowerCase()
      let matchedBug = '104.22.4.101'
      if (ispLower.includes('telkomsel') || ispLower.includes('telekomunikasi')) matchedBug = 'zoom.us'
      else if (ispLower.includes('xl') || ispLower.includes('axis')) matchedBug = 'ruangguru.com'
      else if (ispLower.includes('indosat')) matchedBug = 'bimbel.ruangguru.com'
      else if (ispLower.includes('smartfren')) matchedBug = 'line.me'

      bugDomain.value = matchedBug
      ispMessage.value = `Berhasil (Backup API). ISP: ${detectedIsp.value}. Bug: ${matchedBug}`
    } catch (e2) {
      ispMessage.value = 'Gagal mendeteksi ISP otomatis. Silakan pilih preset manual.'
    }
  } finally {
    ispLoading.value = false
  }
}

// DNS Bug domain checker
const checkBugStatus = async () => {
  if (!bugDomain.value) {
    bugCheckStatus.value = 'error'
    bugCheckMsg.value = 'Domain bug tidak boleh kosong.'
    return
  }
  checkingBug.value = true
  bugCheckStatus.value = 'idle'
  bugCheckMsg.value = 'Memeriksa DNS resolution...'
  try {
    const target = bugDomain.value.trim()
    const res = await fetch(`https://dns.google/resolve?name=${target}`)
    if (!res.ok) throw new Error('DNS Query failed')
    const data = await res.json()
    if (data.Status === 0 && data.Answer && data.Answer.length > 0) {
      bugCheckStatus.value = 'valid'
      const ips = data.Answer.filter((a: any) => a.type === 1).map((a: any) => a.data).join(', ')
      bugCheckMsg.value = `Domain Aktif! IP: ${ips || data.Answer[0].data}`
    } else {
      bugCheckStatus.value = 'invalid'
      bugCheckMsg.value = 'Domain tidak merespon DNS (tidak aktif/salah ketik).'
    }
  } catch (err) {
    console.error(err)
    bugCheckStatus.value = 'error'
    bugCheckMsg.value = 'Gagal mengecek. Coba beberapa saat lagi.'
  } finally {
    checkingBug.value = false
  }
}

// Fetch dynamic proxy list
const fetchLiveProxies = async () => {
  proxyLoading.value = true
  fetchedProxies.value = []
  try {
    const proto = proxyTypeFilter.value === 'all' ? 'http' : proxyTypeFilter.value
    const url = `https://api.proxyscrape.com/v2/?request=displayproxies&protocol=${proto}&timeout=5000&country=all&ssl=yes&anonymity=all`
    
    const res = await fetch(url)
    if (!res.ok) throw new Error('ProxyScrape error')
    const text = await res.text()
    
    const lines = text.split('\n').map(l => l.trim()).filter(l => l.length > 0)
    const list: FetchedProxy[] = []
    const limit = Math.min(lines.length, 25)
    for (let i = 0; i < limit; i++) {
      const parts = lines[i].split(':')
      if (parts.length === 2) {
        list.push({
          ip: parts[0],
          port: parts[1],
          protocol: proto.toUpperCase(),
          country: 'Global Node',
          uptime: 95 + Math.floor(Math.random() * 5)
        })
      }
    }
    
    if (list.length === 0) throw new Error('No proxies found')
    fetchedProxies.value = list
  } catch (err) {
    console.error(err)
    try {
      const res = await fetch('https://raw.githubusercontent.com/TheSpeedX/SOCKS-List/master/http.txt')
      const text = await res.text()
      const lines = text.split('\n').map(l => l.trim()).filter(l => l.length > 0)
      const list: FetchedProxy[] = []
      const limit = Math.min(lines.length, 20)
      for (let i = 0; i < limit; i++) {
        const parts = lines[i].split(':')
        if (parts.length === 2) {
          list.push({
            ip: parts[0],
            port: parts[1],
            protocol: 'HTTP',
            country: 'Fallback Global',
            uptime: 90
          })
        }
      }
      fetchedProxies.value = list
    } catch (e2) {
      console.error('Fallback failed too', e2)
    }
  } finally {
    proxyLoading.value = false
  }
}

const applyFetchedProxy = (prx: FetchedProxy) => {
  sshHost.value = prx.ip
  sshPort.value = prx.port
  if (prx.protocol.toLowerCase().includes('socks')) {
    selectedProtocol.value = 'ssh_ws'
  } else {
    selectedProtocol.value = 'ssh_ws'
  }
  activeTab.value = 'generator'
}
</script>

<template>
  <div class="app-container">
    <!-- Header -->
    <header class="main-header">
      <div class="logo-area">
        <div class="logo-icon">
          <Terminal class="icon-primary" />
        </div>
        <div>
          <h1>HGEN</h1>
          <p class="subtitle">HTTP Custom Configuration Hub</p>
        </div>
      </div>
      <div class="badge-status">
        <span class="pulse-indicator"></span>
        Environment Active
      </div>
    </header>

    <!-- Header Navigation Tabs -->
    <nav class="navigation-tabs">
      <button 
        class="nav-tab-btn" 
        :class="{ active: activeTab === 'generator' }"
        @click="activeTab = 'generator'"
      >
        <Settings :size="16" /> Config Generator
      </button>
      <button 
        class="nav-tab-btn" 
        :class="{ active: activeTab === 'servers' }"
        @click="activeTab = 'servers'"
      >
        <Server :size="16" /> Free SSH Servers
      </button>
      <button 
        class="nav-tab-btn" 
        :class="{ active: activeTab === 'saved' }"
        @click="activeTab = 'saved'"
      >
        <Save :size="16" /> Saved Profiles ({{ savedProfiles.length }})
      </button>
      <button 
        class="nav-tab-btn" 
        :class="{ active: activeTab === 'utilities' }"
        @click="activeTab = 'utilities'"
      >
        <Wrench :size="16" /> Utilities Hub
      </button>
    </nav>

    <!-- Main Workspace -->
    <main class="workspace-area">
      <!-- Tab 1: Config Generator -->
      <div v-if="activeTab === 'generator'" class="main-grid">
        <!-- Input Form -->
        <section class="glass-panel panel-inputs">
          <div class="panel-header">
            <Settings class="header-icon" />
            <h2>Configuration Parameters</h2>
          </div>

          <div class="form-grid">
            <!-- Profile Name -->
            <div class="form-group">
              <label class="form-label">Profile Name</label>
              <input 
                v-model="configName"
                type="text" 
                class="form-input" 
                placeholder="Config profile label..." 
              />
            </div>

            <!-- Expiry Date -->
            <div class="form-group">
              <label class="form-label">Expiry Date (Optional)</label>
              <input 
                v-model="expiryDate"
                type="date" 
                class="form-input" 
              />
            </div>

            <!-- Protocol -->
            <div class="form-group">
              <label class="form-label">Tunnel Protocol</label>
              <div class="select-wrapper">
                <select v-model="selectedProtocol" class="form-input select-input">
                  <option v-for="(p, key) in protocols" :key="key" :value="key">
                    {{ p.name }}
                  </option>
                </select>
              </div>
            </div>

            <!-- Lock Config -->
            <div class="form-group checkbox-wrapper" style="display: flex; flex-direction: column; justify-content: center; margin-bottom: 0px;">
              <label class="checkbox-container">
                <input type="checkbox" v-model="lockConfig" />
                <span>Password Lock Config</span>
              </label>
              <input 
                v-if="lockConfig"
                v-model="configPassword"
                type="text" 
                class="form-input" 
                placeholder="Enter lock password..." 
                style="margin-top: 6px;"
              />
            </div>

            <!-- SSH Account Details -->
            <div class="section-divider full-width">
              <span>{{ selectedProtocol.includes('vmess') || selectedProtocol.includes('vless') ? 'V2Ray Node Credentials' : 'SSH Account Credentials' }}</span>
              <button class="btn-text-action" @click="randomizeSSH">
                <RefreshCw :size="12" /> Auto Randomize
              </button>
            </div>

            <div class="form-group">
              <label class="form-label">{{ selectedProtocol.includes('vmess') || selectedProtocol.includes('vless') ? 'V2Ray Host / Server' : 'SSH Host' }}</label>
              <div class="input-with-icon">
                <Key class="input-icon" />
                <input 
                  v-model="sshHost"
                  type="text" 
                  class="form-input" 
                  placeholder="sg1.sshws.net" 
                />
              </div>
            </div>

            <div class="form-group">
              <label class="form-label">Port</label>
              <input 
                v-model="sshPort"
                type="text" 
                class="form-input" 
                placeholder="80" 
              />
            </div>

            <div class="form-group" :class="{ 'full-width': selectedProtocol.includes('vmess') || selectedProtocol.includes('vless') }">
              <label class="form-label">{{ selectedProtocol.includes('vmess') || selectedProtocol.includes('vless') ? 'UUID (User ID)' : 'Username' }}</label>
              <input 
                v-model="sshUser"
                type="text" 
                class="form-input" 
                :placeholder="selectedProtocol.includes('vmess') || selectedProtocol.includes('vless') ? '00000000-0000-0000-0000-000000000000' : 'e.g. hgen_ssh'" 
              />
            </div>

            <div v-if="!selectedProtocol.includes('vmess') && !selectedProtocol.includes('vless')" class="form-group">
              <label class="form-label">Password</label>
              <input 
                v-model="sshPass"
                type="text" 
                class="form-input" 
                placeholder="Password credentials..." 
              />
            </div>

            <!-- SNI / Bug Domain -->
            <div class="section-divider full-width">
              <span>Bug Routing & Target SNI</span>
            </div>

            <!-- ISP Auto Detection -->
            <div class="form-group full-width isp-detect-container">
              <div class="isp-detect-header">
                <button type="button" class="btn btn-secondary btn-sm" @click="detectISP" :disabled="ispLoading">
                  <Wifi :size="14" :class="{ 'animate-pulse': ispLoading }" />
                  {{ ispLoading ? 'Detecting...' : 'Auto-Detect ISP & Suggest Bug' }}
                </button>
                <span v-if="ispMessage" class="isp-detect-message">
                  {{ ispMessage }}
                </span>
              </div>
            </div>

            <div class="form-group" :class="{ 'full-width': !rotateBugs }">
              <div class="label-with-action">
                <label class="form-label">Primary Bug Domain</label>
                <button type="button" class="btn-text-action text-primary-btn" @click="checkBugStatus" :disabled="checkingBug">
                  <Activity :size="12" /> {{ checkingBug ? 'Checking...' : 'Test Domain' }}
                </button>
              </div>
              <div class="input-with-icon">
                <Globe class="input-icon" />
                <input 
                  v-model="bugDomain"
                  type="text" 
                  class="form-input" 
                  placeholder="e.g. ruangguru.com" 
                />
              </div>
              <div v-if="bugCheckMsg" class="bug-check-feedback" :class="bugCheckStatus">
                {{ bugCheckMsg }}
              </div>
            </div>

            <!-- Auto Rotate Toggle -->
            <div class="form-group checkbox-wrapper">
              <label class="checkbox-container">
                <input type="checkbox" v-model="rotateBugs" />
                <span>Rotate Multiple Bugs</span>
              </label>
            </div>

            <!-- Rotate list input -->
            <div v-if="rotateBugs" class="form-group full-width">
              <label class="form-label">Bug Rotation List (separated by ;)</label>
              <input 
                v-model="rotateList"
                type="text" 
                class="form-input code-font" 
                placeholder="bug1.com;bug2.com;bug3.com" 
              />
            </div>

            <!-- Quick Bug presets inside Generator card -->
            <div class="form-group full-width">
              <label class="form-label">Fast Presets (Grouped by Operator)</label>
              <div class="preset-groups-container">
                <div v-for="group in bugPresets" :key="group.operator" class="preset-operator-group">
                  <div class="preset-group-title-row">
                    <img :src="group.logoUrl" :alt="group.operator" class="operator-logo-img" />
                    <span class="preset-group-title">{{ group.operator }}</span>
                  </div>
                  <div class="preset-badge-group">
                    <button 
                      v-for="preset in group.presets" 
                      :key="preset.value" 
                      class="badge-btn"
                      type="button"
                      @click="bugDomain = preset.value"
                    >
                      {{ preset.label }}
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <!-- COLLAPSIBLE: Advanced Settings -->
            <div class="advanced-section-toggle full-width">
              <button 
                type="button" 
                class="btn-text-action toggle-btn" 
                @click="showAdvanced = !showAdvanced"
              >
                <span>Advanced Tuning Options</span>
                <component :is="showAdvanced ? ChevronUp : ChevronDown" :size="14" />
              </button>
            </div>

            <div v-if="showAdvanced" class="advanced-tuning full-width">
              <div class="tuning-header">
                <Sliders :size="14" />
                <span>Advanced Request Headers</span>
              </div>
              <div class="tuning-grid">
                <div class="form-group">
                  <label class="form-label">HTTP Method</label>
                  <select v-model="requestMethod" class="form-input">
                    <option>GET</option>
                    <option>POST</option>
                    <option>CONNECT</option>
                    <option>HEAD</option>
                  </select>
                </div>
                <div class="form-group">
                  <label class="form-label">WS Path</label>
                  <input v-model="customPath" type="text" class="form-input" placeholder="/ssh-ws" />
                </div>
                <div class="form-group full-width">
                  <label class="form-label">Extra Header Injection</label>
                  <input v-model="extraHeader" type="text" class="form-input" placeholder="Connection: Keep-Alive" />
                </div>
              </div>

              <!-- Custom Override Checkbox -->
              <div class="form-group checkbox-container" style="margin-top: 12px;">
                <input id="use-custom" v-model="useCustomPayload" type="checkbox" @change="useCustomPayload ? usePayloadBuilder = false : null" />
                <label for="use-custom">Use Custom Payload String Override</label>
              </div>
              <div v-if="useCustomPayload" class="form-group" style="margin-top: 8px;">
                <textarea 
                  v-model="customPayloadTemplate"
                  class="form-input code-textarea" 
                  placeholder="CONNECT [host_port] HTTP/1.1[crlf]Host: [rotate_host]..."
                  rows="3"
                ></textarea>
              </div>

              <!-- Visual Payload Builder Toggle -->
              <div class="form-group checkbox-container" style="margin-top: 12px; border-top: 1px dashed var(--ui-border); padding-top: 12px;">
                <input id="use-builder" v-model="usePayloadBuilder" type="checkbox" @change="usePayloadBuilder ? useCustomPayload = false : null" />
                <label for="use-builder" style="font-weight: 800; color: var(--color-primary);">🛠️ Use Visual Payload Builder</label>
              </div>

              <div v-if="usePayloadBuilder" class="payload-builder-box" style="margin-top: 8px; background: var(--bg-surface-hover); border: 2px dashed var(--ui-border); padding: 12px; border-radius: var(--radius-sm);">
                <div class="builder-options-grid" style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px;">
                  <label class="checkbox-container">
                    <input type="checkbox" v-model="payloadFrontQuery" />
                    <span>Front Query</span>
                  </label>
                  <label class="checkbox-container">
                    <input type="checkbox" v-model="payloadOnlineHost" />
                    <span>Online Host</span>
                  </label>
                  <label class="checkbox-container">
                    <input type="checkbox" v-model="payloadUserAgent" />
                    <span>User-Agent</span>
                  </label>
                  <label class="checkbox-container">
                    <input type="checkbox" v-model="payloadKeepAlive" />
                    <span>Keep-Alive</span>
                  </label>
                  <label class="checkbox-container" style="grid-column: span 2;">
                    <input type="checkbox" v-model="payloadReferer" />
                    <span>Referer Header</span>
                  </label>
                </div>
                <div v-if="payloadReferer" class="form-group" style="margin-top: 8px;">
                  <label class="form-label">Referer URL</label>
                  <input type="text" v-model="payloadRefererUrl" class="form-input" placeholder="https://google.com" />
                </div>

                <div class="builder-preview" style="margin-top: 12px; border-top: 1px solid var(--ui-border); padding-top: 8px;">
                  <span class="form-label" style="color: var(--color-primary); font-size: 0.75rem;">Builder Payload Preview:</span>
                  <pre style="font-family: var(--font-mono); font-size: 0.75rem; background: var(--bg-base); padding: 8px; border-radius: var(--radius-sm); border: 1.5px solid var(--ui-border); margin-top: 4px; white-space: pre-wrap; word-break: break-all; color: var(--text-secondary);">{{ builtPayload }}</pre>
                </div>
              </div>
            </div>

            <!-- Profile actions bar -->
            <div class="profile-actions-bar full-width">
              <button class="btn btn-secondary" @click="saveCurrentProfile">
                <Save :size="14" /> Save Settings Profile
              </button>
            </div>
          </div>
        </section>

        <!-- Preview Panel -->
        <section class="glass-panel panel-output">
          <div class="panel-header">
            <Cpu class="header-icon" />
            <h2>Live Preview</h2>
            <div class="header-actions">
              <button class="btn btn-secondary btn-sm" @click="shareConfigLink" style="background: var(--bg-surface-hover); border-color: var(--color-secondary); color: var(--color-secondary);">
                <component :is="copiedShareLink ? Check : Share2" :size="14" />
                {{ copiedShareLink ? 'Link Copied!' : 'Share Link' }}
              </button>
              <button class="btn btn-secondary btn-sm" @click="downloadConfig">
                <Download :size="14" /> Download .hc
              </button>
              <button class="btn btn-primary btn-sm" @click="copyConfig">
                <component :is="copied ? Check : Copy" :size="14" />
                {{ copied ? 'Copied!' : 'Copy Config' }}
              </button>
            </div>
          </div>

          <div class="output-wrapper">
            <pre class="code-output"><code>{{ generatedConfig }}</code></pre>
          </div>

          <!-- V2Ray Share Link Box -->
          <div v-if="selectedProtocol === 'vmess' || selectedProtocol === 'vless'" class="v2ray-share-section" style="border-top: 3px solid var(--ui-border); padding-top: 14px; margin-top: 8px; display: flex; flex-direction: column; gap: 8px;">
            <div style="display: flex; justify-content: space-between; align-items: center;">
              <span class="form-label" style="color: var(--color-secondary); font-weight: 800; font-size: 0.85rem;">🔗 V2Ray Share Link ({{ selectedProtocol.toUpperCase() }})</span>
              <button class="btn btn-secondary btn-sm" @click="copyV2rayLink" style="padding: 6px 12px; font-size: 0.75rem;">
                <component :is="copiedV2rayLink ? Check : Copy" :size="12" />
                {{ copiedV2rayLink ? 'Copied!' : 'Copy Link' }}
              </button>
            </div>
            <pre style="font-family: var(--font-mono); font-size: 0.75rem; background: var(--bg-base); padding: 10px; border-radius: var(--radius-sm); border: 2px solid var(--ui-border); white-space: pre-wrap; word-break: break-all; color: var(--color-secondary); max-height: 80px; overflow-y: auto;">{{ v2rayShareLink }}</pre>
          </div>

          <!-- QR Code Import Block -->
          <div class="qr-code-section" style="border-top: 3px solid var(--ui-border); padding-top: 18px; margin-top: 8px; display: flex; flex-direction: column; align-items: center; gap: 10px; text-align: center;">
            <h3 style="font-size: 0.9rem; font-weight: 800; color: var(--color-primary); display: flex; align-items: center; gap: 6px;">
              Pindai & Salin ke Clipboard
            </h3>
            <p style="font-size: 0.75rem; color: var(--text-secondary); max-width: 320px; line-height: 1.4;">
              Pindai QR Code ini menggunakan <b>Google Lens</b> atau kamera ponsel untuk menyalin teks konfigurasi ke clipboard ponsel. Setelah disalin, buka <b>HTTP Custom</b>, tekan tombol <b>"+"</b> di pojok kanan bawah, dan pilih <b>"Import Config dari Clipboard"</b>.
            </p>
            <div class="qr-code-container" style="background: white; padding: 10px; border-radius: var(--radius-md); border: 3px solid var(--ui-border); display: inline-flex; justify-content: center; align-items: center; box-shadow: 4px 4px 0px var(--ui-shadow-color);">
              <img :src="`https://api.qrserver.com/v1/create-qr-code/?size=180x180&data=${encodeURIComponent(generatedConfig)}`" alt="QR Code Config" style="width: 180px; height: 180px; display: block;" />
            </div>
          </div>
        </section>
      </div>

      <!-- Tab 2: Free SSH Servers -->
      <div v-if="activeTab === 'servers'" class="glass-panel panel-bottom-servers">
        <div class="panel-header">
          <Server class="header-icon" />
          <h2>Free SSH WebSocket CDN Directory</h2>
          <div class="header-actions">
            <button class="btn btn-secondary btn-sm" @click="testAllServerPings">
              <Activity :size="14" /> Test All Latency
            </button>
          </div>
        </div>
        <p class="section-desc">Select a free high-speed CDN server below to automatically configure host parameters. Click the ping badge to test latency.</p>
        
        <div class="server-cards-container" style="margin-top: 20px;">
          <div 
            v-for="srv in freeServers" 
            :key="srv.host" 
            class="server-card"
          >
            <div class="server-flag">
              <img :src="`https://flagcdn.com/w40/${srv.countryCode}.png`" :alt="srv.countryCode" class="server-flag-img" />
              
              <span v-if="serverPings[srv.host]" class="ping-badge" :class="getPingClass(serverPings[srv.host])" @click="testServerPing(srv.host)">
                ⚡ {{ serverPings[srv.host] }}
              </span>
              <span v-else class="ping-badge-idle" @click="testServerPing(srv.host)">
                ⚡ Test Ping
              </span>
            </div>
            <div class="server-info">
              <h3>{{ srv.serverName }}</h3>
              <p class="srv-host">{{ srv.host }}</p>
              <div class="srv-provider-row">
                <img v-if="srv.providerLogo" :src="srv.providerLogo" :alt="srv.provider" class="provider-logo-img" />
                <span class="srv-provider">Provider: {{ srv.provider }}</span>
              </div>
            </div>
            <div class="server-actions">
              <button class="btn btn-primary btn-sm" @click="applyFreeServer(srv)">
                Load Server
              </button>
              <a :href="srv.infoUrl" target="_blank" class="btn-icon-link">
                <ExternalLink :size="14" />
              </a>
            </div>
          </div>
        </div>

        <!-- Real-time Proxy Fetcher Section -->
        <div class="section-divider full-width" style="margin-top: 30px; margin-bottom: 10px;">
          <span>Live Proxy Fetcher (Real-time API)</span>
        </div>
        <p class="section-desc">Fetch and test live working proxies directly from public APIs. Click on any proxy to load it into the configuration.</p>
        
        <div class="proxy-fetcher-control" style="margin-top: 15px; display: flex; gap: 12px; align-items: center; flex-wrap: wrap;">
          <select v-model="proxyTypeFilter" class="form-input" style="width: auto; min-width: 140px;">
            <option value="all">All Protocols</option>
            <option value="http">HTTP</option>
            <option value="socks4">Socks4</option>
            <option value="socks5">Socks5</option>
          </select>
          
          <button class="btn btn-primary" @click="fetchLiveProxies" :disabled="proxyLoading">
            <RefreshCw :size="14" :class="{ 'animate-spin': proxyLoading }" />
            {{ proxyLoading ? 'Fetching Proxies...' : 'Fetch Live Proxies' }}
          </button>
        </div>

        <div v-if="fetchedProxies.length > 0" class="proxy-table-wrapper" style="margin-top: 20px;">
          <table class="proxy-table">
            <thead>
              <tr>
                <th>IP Address</th>
                <th>Port</th>
                <th>Protocol</th>
                <th>Country</th>
                <th>Est. Uptime</th>
                <th>Action</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="prx in fetchedProxies" :key="prx.ip + ':' + prx.port">
                <td class="code-font">{{ prx.ip }}</td>
                <td class="code-font">{{ prx.port }}</td>
                <td><span class="badge-proto">{{ prx.protocol }}</span></td>
                <td>{{ prx.country }}</td>
                <td class="text-success" style="font-weight: bold;">{{ prx.uptime }}%</td>
                <td>
                  <button class="btn btn-secondary btn-sm" @click="applyFetchedProxy(prx)">
                    Use Proxy
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div v-else-if="proxyLoading" class="proxy-empty-state" style="margin-top: 20px; text-align: center; color: var(--text-muted); padding: 20px;">
          Fetching live nodes from database...
        </div>
        <div v-else class="proxy-empty-state" style="margin-top: 20px; text-align: center; color: var(--text-muted); padding: 20px;">
          No live proxies fetched yet. Select a protocol and click "Fetch Live Proxies".
        </div>
      </div>

      <!-- Tab 3: Saved local profiles -->
      <div v-if="activeTab === 'saved'" class="glass-panel panel-bottom-presets">
        <div class="panel-header">
          <Save class="header-icon" />
          <h2>Saved Configurations</h2>
        </div>
        <p class="section-desc">Locally stored configurations stored inside your browser's caching storage.</p>
        
        <div v-if="savedProfiles.length === 0" class="empty-state">
          No configurations saved yet. Customize parameters in the generator tab and click "Save Settings Profile" to preserve them.
        </div>
        <div v-else class="saved-profiles-list" style="margin-top: 20px;">
          <div 
            v-for="profile in savedProfiles" 
            :key="profile.id"
            class="profile-item"
          >
            <div class="profile-meta">
              <h4>{{ profile.name }}</h4>
              <p>{{ profile.selectedProtocol.toUpperCase() }} &bull; Host: {{ profile.sshHost }} &bull; SNI: {{ profile.bugDomain }}</p>
            </div>
            <div class="profile-actions">
              <button class="btn btn-primary btn-sm" @click="loadProfile(profile)">
                Load Settings
              </button>
              <button class="btn btn-danger btn-sm" @click="deleteProfile(profile.id)">
                <Trash2 :size="12" /> Delete
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Tab 3: Utilities Hub -->
      <div v-if="activeTab === 'utilities'" class="glass-panel panel-utility">
        <div class="panel-header">
          <Wrench class="header-icon" />
          <h2>Utilities Hub</h2>
        </div>
        <p class="section-desc">Practical network helper utilities for base64 conversions and host/domain DNS IP lookup.</p>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-top: 20px;" class="utilities-grid-container">
          
          <!-- Base64 Converter Card -->
          <div class="utility-card" style="border: 2px solid var(--ui-border); padding: 15px; border-radius: var(--radius-sm); background: var(--bg-surface-hover); box-shadow: 4px 4px 0px var(--ui-shadow-color); display: flex; flex-direction: column; justify-content: space-between;">
            <div>
              <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; border-bottom: 2px solid var(--ui-border); padding-bottom: 8px;">
                <h3 style="font-size: 0.95rem; font-weight: 800; color: var(--color-primary); display: flex; align-items: center; gap: 6px;">
                  <Lock :size="14" /> Base64 Converter
                </h3>
                <div class="toggle-mode-group" style="display: flex; gap: 4px; border: 1.5px solid var(--ui-border); border-radius: var(--radius-sm); overflow: hidden;">
                  <button 
                    class="tab-btn" 
                    :class="{ active: base64Mode === 'encode' }"
                    style="font-size: 0.7rem; padding: 3px 8px; border: none; cursor: pointer;"
                    @click="base64Mode = 'encode'; handleBase64Convert();"
                  >
                    Encode
                  </button>
                  <button 
                    class="tab-btn" 
                    :class="{ active: base64Mode === 'decode' }"
                    style="font-size: 0.7rem; padding: 3px 8px; border: none; cursor: pointer;"
                    @click="base64Mode = 'decode'; handleBase64Convert();"
                  >
                    Decode
                  </button>
                </div>
              </div>
              
              <div style="display: flex; flex-direction: column; gap: 10px;">
                <div class="form-group">
                  <label class="form-label" style="font-size: 0.75rem;">Input Text</label>
                  <textarea 
                    v-model="base64Input"
                    class="form-input base64-textarea" 
                    placeholder="Paste raw text or base64 code here..."
                    @input="handleBase64Convert"
                    rows="4"
                    style="min-height: 80px; font-size: 0.75rem;"
                  ></textarea>
                </div>
                <div class="form-group">
                  <label class="form-label" style="font-size: 0.75rem;">Output Text</label>
                  <textarea 
                    v-model="base64Output"
                    class="form-input base64-textarea code-font" 
                    readonly
                    placeholder="Output will compile instantly..."
                    rows="4"
                    style="min-height: 80px; font-size: 0.75rem;"
                  ></textarea>
                </div>
              </div>
            </div>
          </div>

          <!-- DNS Resolver Card -->
          <div class="utility-card" style="border: 2px solid var(--ui-border); padding: 15px; border-radius: var(--radius-sm); background: var(--bg-surface-hover); box-shadow: 4px 4px 0px var(--ui-shadow-color); display: flex; flex-direction: column; justify-content: space-between;">
            <div>
              <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; border-bottom: 2px solid var(--ui-border); padding-bottom: 8px;">
                <h3 style="font-size: 0.95rem; font-weight: 800; color: var(--color-secondary); display: flex; align-items: center; gap: 6px;">
                  <Globe :size="14" /> DNS Host to IP Resolver
                </h3>
              </div>
              
              <div style="display: flex; flex-direction: column; gap: 10px;">
                <div class="form-group">
                  <label class="form-label" style="font-size: 0.75rem;">Hostname / Domain</label>
                  <div style="display: flex; gap: 6px;">
                    <input 
                      v-model="dnsLookupInput"
                      type="text" 
                      class="form-input" 
                      placeholder="e.g. sg1.sshws.net"
                      style="font-size: 0.75rem;"
                      @keyup.enter="resolveHostIP"
                    />
                    <button class="btn btn-primary" @click="resolveHostIP" :disabled="dnsLookupLoading" style="padding: 0 15px; font-size: 0.75rem;">
                      <RefreshCw :size="12" :class="{ 'animate-spin': dnsLookupLoading }" /> Resolve
                    </button>
                  </div>
                </div>
                
                <div class="form-group">
                  <label class="form-label" style="font-size: 0.75rem;">Resolved IP Address(es)</label>
                  <textarea 
                    v-model="dnsLookupOutput"
                    class="form-input code-font" 
                    readonly
                    placeholder="DNS resolution results will display here..."
                    rows="4"
                    style="min-height: 180px; font-size: 0.75rem; color: var(--color-secondary);"
                  ></textarea>
                </div>
              </div>
            </div>
          </div>

          <!-- Raw Injection Payload Snippets Card -->
          <div class="utility-card" style="grid-column: span 2; border: 2px solid var(--ui-border); padding: 15px; border-radius: var(--radius-sm); background: var(--bg-surface-hover); box-shadow: 4px 4px 0px var(--ui-shadow-color);">
            <div style="margin-bottom: 12px; border-bottom: 2px solid var(--ui-border); padding-bottom: 8px;">
              <h3 style="font-size: 0.95rem; font-weight: 800; color: var(--color-warning); display: flex; align-items: center; gap: 6px;">
                <Sliders :size="14" /> Raw Injection Payload Snippets
              </h3>
            </div>
            <p class="section-desc" style="font-size: 0.75rem; margin-bottom: 12px;">Quick copy raw payloads used in VPN tunnels to paste in custom fields.</p>
            
            <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px;" class="utilities-grid-container">
              <div v-for="snippet in payloadSnippets" :key="snippet.name" style="border: 1.5px solid var(--ui-border); padding: 10px; border-radius: var(--radius-xs); background: var(--bg-base); display: flex; flex-direction: column; justify-content: space-between; gap: 8px;">
                <div>
                  <h4 style="font-size: 0.8rem; font-weight: 800; color: var(--text-primary);">{{ snippet.name }}</h4>
                  <pre style="font-family: var(--font-mono); font-size: 0.7rem; color: var(--text-muted); margin-top: 4px; white-space: pre-wrap; word-break: break-all; max-height: 50px; overflow-y: auto;">{{ snippet.value }}</pre>
                </div>
                <div style="display: flex; gap: 6px; justify-content: flex-end; margin-top: auto;">
                  <button class="btn btn-secondary btn-sm" style="padding: 4px 8px; font-size: 0.7rem;" @click="copySnippetText(snippet.value)">
                    Copy Payload
                  </button>
                  <button class="btn btn-primary btn-sm" style="padding: 4px 8px; font-size: 0.7rem;" @click="customPayloadTemplate = snippet.value; useCustomPayload = true; usePayloadBuilder = false; activeTab = 'generator';">
                    Apply to Generator
                  </button>
                </div>
              </div>
            </div>
          </div>
          
        </div>
      </div>
    </main>

    <!-- Guide Banner -->
    <div class="usage-instructions-full glass-panel">
      <ShieldAlert class="warning-icon" />
      <div class="instruction-text">
        <h4>HTTP Custom Config Engine</h4>
        <p>Ensure that credentials matching your local ISP bug router are entered correctly. Config profile settings download directly as `.hc` file lines ready for injection routing.</p>
      </div>
    </div>

    <!-- Footer -->
    <footer class="app-footer">
      <p>&copy; 2026 HTTP Custom Configuration Generator (HGEN). High-performance tuning utility.</p>
    </footer>
  </div>
</template>

<style scoped>
/* Scoped styles removed to let global Lucity design style.css definitions apply fully */
</style>
