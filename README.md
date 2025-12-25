# Tapir Mini-Apps

A collection of mini-apps for the [Tapir](https://github.com/alnicko-lab/tapir) wearable device.

## What is Tapir?

Tapir is an open-source wearable device with:
- 390×450 pixel display (32×18 character terminal mode)
- 12 RGB LED keys
- BLE connectivity
- Powered by SiFli SF32LB52 chip

## How Mini-Apps Work

Mini-apps are HTML/JavaScript applications that run inside the **Tapir Runtime** Android app. They communicate with the device through a JavaScript bridge (`window.tapir`).

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Mini-App      │     │  Tapir Runtime  │     │  Tapir Device   │
│   (WebView)     │◄───►│  (React Native) │◄───►│   (Firmware)    │
│                 │     │                 │ BLE │                 │
│  window.tapir   │     │  Native Bridge  │     │  32×18 Terminal │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Available Mini-Apps

### 🔔 Pager (`pager.html`)

A notification pager with world clocks.

**Features:**
- Shows current time in Osaka (JST) and Kyiv (EET)
- Displays last 3 phone notifications
- Auto-updates the device display
- Real-time terminal preview

**Screenshot:**
```
TAPIR PAGER              14:32
================================
OSAKA 22:32         KYIV  15:32
--------------------------------
Messages
Mom
Don't forget to call grandma!
................................
Slack
#general
Meeting in 10 minutes
................................
Gmail
GitHub
New pull request from dependabot
================================
```

## Bridge API

Mini-apps have access to `window.tapir`:

### Terminal Display

```javascript
// Send screen buffer to device (32×18 chars)
const buffer = btoa("Hello World..."); // base64 encoded
await window.tapir.terminal(buffer, 32, 18);

// Clear the terminal
await window.tapir.terminalClear();
```

### LED Control

```javascript
// Set LED color (index 0-11, RGB 0-255)
await window.tapir.led(0, 255, 0, 0); // Red
await window.tapir.led(5, 0, 255, 0); // Green
```

### Device Info

```javascript
const info = await window.tapir.device.info();
// {
//   connected: true,
//   deviceId: "AA:BB:CC:DD:EE:FF",
//   mtu: 512,
//   terminalCols: 32,
//   terminalRows: 18
// }
```

### AI Chat (Proxied)

```javascript
// Uses API key stored in Tapir Runtime (never exposed to mini-app)
const result = await window.tapir.ai.chat("What's the weather?");
console.log(result.text);
```

### Storage (Sandboxed)

```javascript
// Each mini-app has isolated storage
await window.tapir.storage.set('key', 'value');
const value = await window.tapir.storage.get('key');
```

### Events

```javascript
// Button presses from device
window.tapir.on('button', (data) => {
  console.log(`Button ${data.id} ${data.event}`); // "down" or "up"
});

// Phone notifications (requires Android permission)
window.tapir.on('notification', (data) => {
  console.log(`${data.app}: ${data.title} - ${data.text}`);
});

// Connection state changes
window.tapir.on('connection', (data) => {
  console.log(`Connection: ${data.state}`); // "connected", "disconnected"
});
```

## Running Mini-Apps

### Option 1: Load from URL

1. Host your mini-app on a web server (or use GitHub Pages)
2. Open Tapir Runtime → Mini-App screen
3. Enter the URL

### Option 2: Load from local file

1. Copy HTML to your phone
2. Use `file://` URL in Mini-App screen

### Option 3: Embed in Tapir Runtime

Add your mini-app HTML to the Tapir Runtime source code.

## Development

### Testing without Device

Mini-apps work in any browser for UI development. The `window.tapir` bridge won't be available, but you can mock it:

```javascript
if (!window.tapir) {
  window.tapir = {
    terminal: async (b, c, r) => console.log('Terminal:', c, 'x', r),
    led: async (i, r, g, b) => console.log('LED:', i, r, g, b),
    device: { info: async () => ({ connected: false }) },
    on: (e, cb) => console.log('Registered listener:', e),
  };
}
```

### Terminal Dimensions

- **Columns:** 32
- **Rows:** 18
- **Total chars:** 576
- **Font:** Terminus 24 (12×24 pixels per char)
- **Screen:** 390×450 pixels

## Creating Your Own Mini-App

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Tapir App</title>
</head>
<body>
  <h1>My App</h1>
  <button onclick="sendHello()">Send to Device</button>
  
  <script>
    const COLS = 32;
    const ROWS = 18;
    
    function createBuffer(text) {
      let buffer = text.padEnd(COLS * ROWS, ' ');
      return btoa(buffer.substring(0, COLS * ROWS));
    }
    
    async function sendHello() {
      if (!window.tapir) {
        alert('Not in Tapir Runtime');
        return;
      }
      
      const buffer = createBuffer('Hello from my mini-app!');
      await window.tapir.terminal(buffer, COLS, ROWS);
    }
  </script>
</body>
</html>
```

## License

MIT License - See [LICENSE](LICENSE)

## Links

- [Tapir Firmware](https://github.com/alnicko-lab/tapir-testing-fw)
- [Tapir Runtime (React Native)](https://github.com/alnicko-lab/tapir-rn)

