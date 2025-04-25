# WebAX Distributed Communication Specification

## Overview

This specification defines how WebAX applications communicate with each other using a WebRTC-based distributed communication system. This system enables peer-to-peer content sharing, distributed application deployment, and direct communication between users without intermediary servers.

## Core Concepts

### 1. Distributed Hash System

The WebAX Distributed Hash System (DHS) is a content-addressable storage system that uses cryptographic hashes to identify and verify content. Content is split into chunks and distributed across the network, similar to BitTorrent.

### 2. WebRTC Communication Layer

WebRTC provides the foundation for peer-to-peer connections between WebAX applications. It enables:
- Direct browser-to-browser connections
- Data channels for content exchange
- Secure, encrypted communication

### 3. Distribution Protocol

The distribution protocol defines how content is shared across the network:
- Content chunking and reassembly
- Piece selection and distribution algorithms
- Content verification and integrity checking

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   WebAX Application                      │
│                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐│
│  │  Business     │  │  UI           │  │  Application  ││
│  │  Logic (jcule)│  │  (jellicule)  │  │  Logic        ││
│  └───────────────┘  └───────────────┘  └───────────────┘│
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                WebAX Communication Layer                 │
│                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐│
│  │  Distributed  │  │  WebRTC       │  │  Content      ││
│  │  Hash System  │  │  Manager      │  │  Distribution ││
│  └───────────────┘  └───────────────┘  └───────────────┘│
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                   Network Layer                          │
│                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐│
│  │  WebRTC       │  │  Signaling    │  │  STUN/TURN    ││
│  │  Connections  │  │  Server       │  │  Servers      ││
│  └───────────────┘  └───────────────┘  └───────────────┘│
└─────────────────────────────────────────────────────────┘
```

## Distributed Hash System (DHS)

### Content Addressing

All content in the DHS is identified by its cryptographic hash:

```javascript
// Content identifier
{
  hash: 'sha256-hash-of-content',
  type: 'application/javascript',
  size: 1024,
  pieces: 4
}
```

### Content Chunking

Content is split into fixed-size chunks (16KB by default) to optimize network transfer:

```javascript
function splitIntoPieces(content) {
  const pieces = new Map();
  const data = content.data;
  for (let i = 0; i < data.length; i += PIECE_SIZE) {
    pieces.set(i / PIECE_SIZE, data.slice(i, i + PIECE_SIZE));
  }
  return pieces;
}
```

### Content Verification

Content integrity is verified using cryptographic hashes:

```javascript
async function verifyContent(content, expectedHash) {
  const actualHash = await calculateHash(content);
  return actualHash === expectedHash;
}
```

## WebRTC Communication Layer

### Peer Discovery

1. Client connects to a signaling server
2. Client registers with a unique peer ID
3. Server provides a list of active peers
4. Client establishes WebRTC connections with peers
5. After initial connection, content discovery happens directly between peers

### Connection Establishment

```javascript
async function connectToPeer(peerId) {
  // Create peer connection
  const peerConnection = new RTCPeerConnection(config);
  
  // Create data channel
  const dataChannel = peerConnection.createDataChannel('data');
  
  // Set up event handlers
  dataChannel.onmessage = handleMessage;
  
  // Create and send offer
  const offer = await peerConnection.createOffer();
  await peerConnection.setLocalDescription(offer);
  
  // Send offer to peer via signaling server
  signaling.send({
    type: 'offer',
    to: peerId,
    from: localPeerId,
    offer
  });
  
  // Return connection
  return { peerConnection, dataChannel };
}
```

### Message Types

| Message Type | Description | Payload |
|--------------|-------------|---------|
| `register` | Register with signaling server | `{ peerId }` |
| `offer` | WebRTC connection offer | `{ to, from, offer }` |
| `answer` | WebRTC connection answer | `{ to, from, answer }` |
| `ice-candidate` | ICE candidate for connection | `{ to, from, candidate }` |
| `content-metadata` | Metadata about content | `{ contentId, metadata }` |
| `content-piece` | A piece of content | `{ contentId, pieceIndex, data }` |
| `request-piece` | Request for a specific piece | `{ contentId, pieceIndex }` |

## Distribution Protocol

### Content Distribution Process

1. **Content Publication**:
   ```
   Application publishes content → Content is chunked → Metadata is generated → Initial pieces are distributed to connected peers
   ```

2. **Content Propagation**:
   ```
   Peer receives piece → Stores piece locally → Updates piece availability → Shares piece with other peers
   ```

3. **Content Retrieval**:
   ```
   Application requests content → Discovers which peers have pieces → Requests missing pieces → Assembles complete content
   ```

### Piece Selection Strategy

When sharing content with new peers, a rarest-first strategy is used to maximize distribution efficiency:

```javascript
function selectPieceToSend(contentId, peerAvailability) {
  // Find rarest pieces
  const pieceAvailability = new Map();
  for (const [peerId, pieces] of peerAvailability.entries()) {
    for (const pieceIndex of pieces) {
      pieceAvailability.set(
        pieceIndex, 
        (pieceAvailability.get(pieceIndex) || 0) + 1
      );
    }
  }
  
  // Sort by availability (ascending)
  const sortedPieces = Array.from(pieceAvailability.entries())
    .sort((a, b) => a[1] - b[1])
    .map(entry => entry[0]);
  
  // Return rarest piece
  return sortedPieces[0] || 0;
}
```

## Integration with WebAX Applications

### Serving the `/dist` Folder

The `/dist` folder of WebAX applications can be served through the DHS:

1. **Publishing**:
   ```javascript
   // Publish the dist folder
   const contentId = await webax.dhs.publishDirectory('/dist');
   
   // Share the content ID
   console.log(`Application published: webax://${contentId}`);
   ```

2. **Consuming**:
   ```javascript
   // Load an application from the DHS
   const app = await webax.dhs.loadApplication('webax://hash-of-dist-folder');
   
   // Mount the application
   app.mount('#app-container');
   ```

### Direct Communication Between Users

Users can communicate directly using the WebRTC layer:

```javascript
// Connect to a specific user
const connection = await webax.connect('user-id');

// Send a message
connection.send({
  type: 'chat-message',
  content: 'Hello, world!'
});

// Receive messages
connection.on('message', (message) => {
  console.log('Received:', message);
});
```

### Application Updates

Applications can be updated through the DHS:

```javascript
// Check for updates
const updates = await webax.dhs.checkForUpdates('webax://hash-of-dist-folder');

// Apply updates if available
if (updates) {
  await webax.dhs.applyUpdates(updates);
  location.reload();
}
```

## API Reference

### WebAX DHS API

```javascript
// Initialize the DHS
const dhs = await webax.dhs.init({
  signaling: 'wss://signaling.webax.net',
  maxPeers: 10,
  storageQuota: 50 * 1024 * 1024 // 50MB
});

// Publish content
const contentId = await dhs.publish(content);

// Retrieve content
const content = await dhs.get(contentId);

// Check if content exists locally
const exists = await dhs.has(contentId);

// List all available content
const contentList = await dhs.list();

// Subscribe to content updates
dhs.subscribe(contentId, (updatedContent) => {
  console.log('Content updated:', updatedContent);
});

// Unsubscribe from content updates
dhs.unsubscribe(contentId);
```

### WebAX Connection API

```javascript
// Connect to a peer
const connection = await webax.connect(peerId);

// Send data
connection.send(data);

// Receive data
connection.on('data', (data) => {
  console.log('Received data:', data);
});

// Close connection
connection.close();

// Check connection status
const status = connection.status;
```

## Security Considerations

### Content Integrity

All content in the DHS is verified using cryptographic hashes to ensure integrity.

### Encryption

Sensitive content can be encrypted before distribution:

```javascript
// Encrypt content
const encryptedContent = await webax.crypto.encrypt(content, {
  recipients: ['recipient-public-key']
});

// Publish encrypted content
const contentId = await dhs.publish(encryptedContent);

// Retrieve and decrypt content
const encryptedContent = await dhs.get(contentId);
const content = await webax.crypto.decrypt(encryptedContent);
```

### Trust Model

The WebAX distributed communication system operates on a "trust but verify" model:

- Content is accepted from any peer
- Content integrity is verified cryptographically
- Applications can implement additional trust layers
- Malicious peers can be blocked at the application level

## Implementation Considerations

### Browser Compatibility

The distributed communication system requires browsers that support:

- WebRTC (RTCPeerConnection, RTCDataChannel)
- IndexedDB
- Service Workers
- Web Crypto API

This includes recent versions of Chrome, Firefox, Safari, and Edge.

### Network Constraints

- NAT traversal may require TURN servers in restrictive networks
- Maximum concurrent peer connections depend on device capabilities
- Mobile devices may have more limited connection capabilities

### Storage Limitations

- IndexedDB storage limits vary by browser (typically 50MB-1GB)
- Users can configure maximum storage allocation
- Least recently used content is pruned when storage limits are reached

## Conclusion

The WebAX Distributed Communication Specification provides a comprehensive framework for peer-to-peer communication between WebAX applications. By leveraging WebRTC and a distributed hash system, WebAX applications can share content, communicate directly, and operate in a decentralized manner.

This approach enables a new generation of web applications that are more resilient, efficient, and user-centric, reducing reliance on centralized infrastructure while improving performance and user experience.
