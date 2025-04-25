# WebAX Distributed Communication Architecture

This document provides a visual representation of the WebAX distributed communication architecture, showing how the various components interact to enable peer-to-peer communication and content distribution.

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        WebAX Application Layer                           │
│                                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌───────────┐ │
│  │  WebAXP       │  │  jcule        │  │  jellicule    │  │  WebAX    │ │
│  │  Framework    │  │  Business     │  │  UI           │  │  Lib      │ │
│  │               │  │  Logic        │  │  Components   │  │           │ │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘  └─────┬─────┘ │
└──────────┼─────────────────┼─────────────────┼───────────────────┼──────┘
           │                 │                 │                   │
           ▼                 ▼                 ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     WebAX Communication Layer                           │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                      Comlink Core                                 │  │
│  │                                                                   │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐  │  │
│  │  │  Connection   │  │  Message      │  │  Event                │  │  │
│  │  │  Manager      │  │  Handler      │  │  Bus                  │  │  │
│  │  └───────────────┘  └───────────────┘  └───────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                 Distributed Hash System (DHS)                     │  │
│  │                                                                   │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐  │  │
│  │  │  Content      │  │  Piece        │  │  Storage              │  │  │
│  │  │  Manager      │  │  Manager      │  │  Manager              │  │  │
│  │  └───────────────┘  └───────────────┘  └───────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                      Security Layer                               │  │
│  │                                                                   │  │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────┐  │  │
│  │  │  Encryption   │  │  Identity     │  │  Access               │  │  │
│  │  │  Manager      │  │  Manager      │  │  Control              │  │  │
│  │  └───────────────┘  └───────────────┘  └───────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          Browser APIs                                    │
│                                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌───────────┐ │
│  │  WebRTC       │  │  IndexedDB    │  │  Service      │  │  Web      │ │
│  │  API          │  │  API          │  │  Worker API   │  │  Crypto   │ │
│  └───────────────┘  └───────────────┘  └───────────────┘  └───────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          Network Layer                                   │
│                                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌───────────┐ │
│  │  WebRTC       │  │  Signaling    │  │  STUN/TURN    │  │  HTTP     │ │
│  │  Connections  │  │  Server       │  │  Servers      │  │  Fallback │ │
│  └───────────────┘  └───────────────┘  └───────────────┘  └───────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## Component Interactions

### Content Publishing Flow

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│          │         │          │         │          │         │          │
│  WebAXP  │         │  Comlink │         │   DHS    │         │  Peers   │
│          │         │   Core   │         │          │         │          │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                     │                    │                    │
     │ Publish /dist       │                    │                    │
     │ ──────────────────> │                    │                    │
     │                     │                    │                    │
     │                     │ Split into chunks  │                    │
     │                     │ ──────────────────>│                    │
     │                     │                    │                    │
     │                     │                    │ Generate metadata  │
     │                     │                    │ ─────────────────┐ │
     │                     │                    │                  │ │
     │                     │                    │ <────────────────┘ │
     │                     │                    │                    │
     │                     │                    │ Store locally      │
     │                     │                    │ ─────────────────┐ │
     │                     │                    │                  │ │
     │                     │                    │ <────────────────┘ │
     │                     │                    │                    │
     │                     │                    │ Announce content   │
     │                     │                    │ ──────────────────>│
     │                     │                    │                    │
     │                     │                    │                    │ Request pieces
     │                     │                    │ <───────────────── │
     │                     │                    │                    │
     │                     │                    │ Send pieces        │
     │                     │                    │ ──────────────────>│
     │                     │                    │                    │
     │                     │ Return content ID  │                    │
     │                     │ <──────────────────│                    │
     │                     │                    │                    │
     │ Return content ID   │                    │                    │
     │ <─────────────────── │                    │                    │
     │                     │                    │                    │
```

### Content Consumption Flow

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│          │         │          │         │          │         │          │
│  WebAXP  │         │  Comlink │         │   DHS    │         │  Peers   │
│          │         │   Core   │         │          │         │          │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                     │                    │                    │
     │ Load content        │                    │                    │
     │ ──────────────────> │                    │                    │
     │                     │                    │                    │
     │                     │ Request content    │                    │
     │                     │ ──────────────────>│                    │
     │                     │                    │                    │
     │                     │                    │ Check local cache  │
     │                     │                    │ ─────────────────┐ │
     │                     │                    │                  │ │
     │                     │                    │ <────────────────┘ │
     │                     │                    │                    │
     │                     │                    │ Query peers        │
     │                     │                    │ ──────────────────>│
     │                     │                    │                    │
     │                     │                    │ Respond with       │
     │                     │                    │ available pieces   │
     │                     │                    │ <────────────────── │
     │                     │                    │                    │
     │                     │                    │ Request missing    │
     │                     │                    │ pieces             │
     │                     │                    │ ──────────────────>│
     │                     │                    │                    │
     │                     │                    │ Send pieces        │
     │                     │                    │ <────────────────── │
     │                     │                    │                    │
     │                     │                    │ Assemble content   │
     │                     │                    │ ─────────────────┐ │
     │                     │                    │                  │ │
     │                     │                    │ <────────────────┘ │
     │                     │                    │                    │
     │                     │                    │ Verify integrity   │
     │                     │                    │ ─────────────────┐ │
     │                     │                    │                  │ │
     │                     │                    │ <────────────────┘ │
     │                     │                    │                    │
     │                     │ Return content     │                    │
     │                     │ <──────────────────│                    │
     │                     │                    │                    │
     │ Return content      │                    │                    │
     │ <─────────────────── │                    │                    │
     │                     │                    │                    │
```

### Direct Communication Flow

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│          │         │          │         │          │         │          │
│  jcule   │         │  Comlink │         │ Signaling│         │  Peer    │
│          │         │   Core   │         │  Server  │         │          │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                     │                    │                    │
     │ Connect to peer     │                    │                    │
     │ ──────────────────> │                    │                    │
     │                     │                    │                    │
     │                     │ Register           │                    │
     │                     │ ──────────────────>│                    │
     │                     │                    │                    │
     │                     │ Send offer         │                    │
     │                     │ ──────────────────>│                    │
     │                     │                    │                    │
     │                     │                    │ Forward offer      │
     │                     │                    │ ──────────────────>│
     │                     │                    │                    │
     │                     │                    │ Send answer        │
     │                     │                    │ <────────────────── │
     │                     │                    │                    │
     │                     │ Receive answer     │                    │
     │                     │ <──────────────────│                    │
     │                     │                    │                    │
     │                     │ Exchange ICE       │                    │
     │                     │ candidates         │                    │
     │                     │ ──────────────────>│                    │
     │                     │                    │ Forward ICE        │
     │                     │                    │ candidates         │
     │                     │                    │ ──────────────────>│
     │                     │                    │                    │
     │                     │ Establish direct   │                    │
     │                     │ connection         │                    │
     │                     │ <─────────────────────────────────────> │
     │                     │                    │                    │
     │                     │ Connection         │                    │
     │                     │ established        │                    │
     │                     │ ─────────────────┐ │                    │
     │                     │                  │ │                    │
     │                     │ <────────────────┘ │                    │
     │                     │                    │                    │
     │ Connection ready    │                    │                    │
     │ <─────────────────── │                    │                    │
     │                     │                    │                    │
     │ Send message        │                    │                    │
     │ ──────────────────> │                    │                    │
     │                     │                    │                    │
     │                     │ Encrypt message    │                    │
     │                     │ ─────────────────┐ │                    │
     │                     │                  │ │                    │
     │                     │ <────────────────┘ │                    │
     │                     │                    │                    │
     │                     │ Send message       │                    │
     │                     │ ──────────────────────────────────────> │
     │                     │                    │                    │
     │                     │                    │                    │ Decrypt message
     │                     │                    │                    │ ─────────────┐
     │                     │                    │                    │              │
     │                     │                    │                    │ <────────────┘
     │                     │                    │                    │
```

## Network Topology

The WebAX distributed communication system uses a mesh network topology, where each peer connects directly to multiple other peers:

```
                    ┌─────────┐
                    │         │
                 ┌──┤  Peer A ├──┐
                 │  │         │  │
                 │  └─────────┘  │
                 │               │
                 ▼               ▼
          ┌─────────┐     ┌─────────┐
          │         │     │         │
       ┌──┤  Peer B ├─────┤  Peer C ├──┐
       │  │         │     │         │  │
       │  └─────────┘     └─────────┘  │
       │                               │
       ▼                               ▼
┌─────────┐                     ┌─────────┐
│         │                     │         │
│  Peer D ├─────────────────────┤  Peer E │
│         │                     │         │
└─────────┘                     └─────────┘
```

Each peer maintains connections to a subset of the network, typically 5-10 other peers. This approach provides:

1. **Resilience**: The network continues to function even if some peers disconnect
2. **Scalability**: The network can grow to thousands of peers without overloading any single peer
3. **Efficiency**: Content can be retrieved from multiple peers simultaneously

## Security Architecture

The WebAX distributed communication system includes multiple security layers:

```
┌─────────────────────────────────────────────────────────┐
│                  Application Security                    │
│                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐│
│  │  Access       │  │  Content      │  │  User         ││
│  │  Control      │  │  Moderation   │  │  Management   ││
│  └───────────────┘  └───────────────┘  └───────────────┘│
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  Content Security                        │
│                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐│
│  │  Content      │  │  Content      │  │  Content      ││
│  │  Encryption   │  │  Signing      │  │  Verification ││
│  └───────────────┘  └───────────────┘  └───────────────┘│
└─────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  Network Security                        │
│                                                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐│
│  │  WebRTC       │  │  Peer         │  │  Connection   ││
│  │  Encryption   │  │  Verification │  │  Monitoring   ││
│  └───────────────┘  └───────────────┘  └───────────────┘│
└─────────────────────────────────────────────────────────┘
```

## Conclusion

The WebAX distributed communication architecture provides a comprehensive framework for peer-to-peer communication and content distribution. By leveraging WebRTC and a distributed hash system, WebAX applications can communicate directly, share content, and operate in a decentralized manner.

This architecture enables a new generation of web applications that are more resilient, efficient, and user-centric, reducing reliance on centralized infrastructure while improving performance and user experience.
