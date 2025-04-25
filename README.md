# WebAXD (Web Application Experience Distribution)

WebAXD is the distribution layer for the WebAX ecosystem, providing a comprehensive abstraction for distributing web applications and their assets. It supports multiple distribution models, including direct-to-service, peer-to-peer, and distributed service approaches.

## Overview

WebAXD provides a unified API for distributing and consuming web applications, regardless of the underlying distribution mechanism. This allows developers to focus on building their applications without worrying about the complexities of distribution.

## Key Features

- **Multiple Distribution Models**: Support for direct-to-service, peer-to-peer, and distributed service models
- **Unified API**: A single API for all distribution models
- **Seamless Integration**: Integrates with WebAXP and other WebAX components
- **Offline Support**: Applications can be distributed and consumed offline
- **Security**: End-to-end encryption and content verification
- **Scalability**: Scales from small applications to large-scale deployments

## Architecture

WebAXD consists of several key components:

### 1. Distribution Core

The core distribution engine that provides the unified API and manages the distribution process.

### 2. Distribution Adapters

Adapters for different distribution models:

- **Direct Adapter**: For direct-to-service distribution (CDN, static hosting)
- **P2P Adapter**: For peer-to-peer distribution using WebRTC
- **Distributed Adapter**: For distributed service distribution using a mesh network

### 3. Content Management

Tools for managing content distribution:

- **Content Packaging**: Packaging applications for distribution
- **Content Versioning**: Managing versions of distributed content
- **Content Discovery**: Finding and retrieving distributed content

### 4. Security Layer

Security features for distributed content:

- **Content Encryption**: Encrypting content for secure distribution
- **Content Verification**: Verifying the integrity of distributed content
- **Access Control**: Controlling access to distributed content

## Integration with WebAX Ecosystem

WebAXD integrates with other WebAX components:

- **WebAXP**: WebAXP applications can be distributed through WebAXD
- **WebAXPDN**: The WebAXP Distribution Network is built on WebAXD
- **jcule**: jcule can use WebAXD for distributing and consuming business logic
- **jellicule**: jellicule components can be distributed through WebAXD

## Usage

### Distributing an Application

```javascript
import { distribute } from '@net.webax/webaxd';

// Distribute an application
const distributionId = await distribute({
  // Source directory or files
  source: './dist',
  
  // Distribution options
  options: {
    // Distribution model
    model: 'p2p', // 'direct', 'p2p', or 'distributed'
    
    // Content options
    content: {
      // Content name
      name: 'my-app',
      
      // Content version
      version: '1.0.0',
      
      // Content description
      description: 'My WebAXP Application',
    },
    
    // Security options
    security: {
      // Enable encryption
      encrypt: true,
      
      // Access control
      access: 'public', // 'public', 'private', or 'restricted'
    },
  },
});

console.log(`Application distributed: webaxd://${distributionId}`);
```

### Consuming a Distributed Application

```javascript
import { consume } from '@net.webax/webaxd';

// Consume a distributed application
const app = await consume('webaxd://distribution-id');

// Mount the application
app.mount('#app-container');
```

## WebAXPDN Integration

WebAXD serves as the foundation for WebAXPDN, the WebAXP Distribution Network. WebAXPDN provides a specialized distribution network for WebAXP applications, built on the WebAXD infrastructure.

See the [WebAXPDN documentation](../WebAXPDN/README.md) for more details.

## Configuration

WebAXD can be configured through the WebAXP configuration file:

```javascript
// webaxp.config.js
module.exports = {
  // WebAXD configuration
  webaxd: {
    // Default distribution model
    model: 'p2p',
    
    // P2P configuration
    p2p: {
      // Signaling server
      signaling: 'wss://signaling.webax.net',
      
      // ICE servers
      iceServers: [
        { urls: 'stun:stun.webax.net:3478' },
        {
          urls: 'turn:turn.webax.net:3478',
          username: 'webax',
          credential: 'webax',
        },
      ],
      
      // Maximum number of peers
      maxPeers: 10,
    },
    
    // Distributed configuration
    distributed: {
      // Bootstrap nodes
      bootstrapNodes: [
        'webaxd://bootstrap-node-1',
        'webaxd://bootstrap-node-2',
      ],
      
      // Replication factor
      replicationFactor: 3,
    },
    
    // Security configuration
    security: {
      // Default encryption
      encrypt: true,
      
      // Default access control
      access: 'public',
    },
  },
};
```

## CLI

WebAXD includes a command-line interface for distributing and consuming applications:

```bash
# Distribute an application
webaxd distribute ./dist --model p2p

# Consume a distributed application
webaxd consume webaxd://distribution-id --output ./app

# List distributed applications
webaxd list

# Get information about a distributed application
webaxd info webaxd://distribution-id
```

## API Reference

See the [API Reference](./docs/api.md) for detailed documentation of the WebAXD API.

## Examples

See the [Examples](./examples) directory for example applications using WebAXD.

## Contributing

Contributions are welcome! Please see the [Contributing Guide](./CONTRIBUTING.md) for more details.

## License

MIT
