# WebAX Distributed Chat Example

This example demonstrates how to build a peer-to-peer chat application using the WebAX distributed communication system. The application allows users to chat directly with each other without a central server, using WebRTC for communication.

## Features

- Peer-to-peer messaging using WebRTC
- Distributed content storage using the WebAX Distributed Hash System (DHS)
- Offline message queuing and synchronization
- End-to-end encryption for secure communication
- User discovery through a signaling server

## Project Structure

```
distributed-chat/
├── app/                      # Source code
│   ├── src/                  # Source code
│   │   ├── index.ts          # Main entry point
│   │   ├── components/       # UI components
│   │   │   ├── ChatView.tsx  # Chat interface
│   │   │   ├── MessageList.tsx # Message list
│   │   │   ├── UserList.tsx  # User list
│   │   │   └── MessageInput.tsx # Message input
│   │   ├── state/            # State management
│   │   │   ├── actions.ts    # State actions
│   │   │   ├── reducers.ts   # State reducers
│   │   │   └── selectors.ts  # State selectors
│   │   ├── comlink/          # Distributed communication
│   │   │   ├── connection.ts # WebRTC connection
│   │   │   ├── messages.ts   # Message handling
│   │   │   └── storage.ts    # Distributed storage
│   │   └── utils/            # Utility functions
│   │       ├── crypto.ts     # Encryption utilities
│   │       └── time.ts       # Time utilities
│   ├── environment/          # Environment configuration
│   ├── tests/                # Test files
│   └── assets/               # Static assets
├── cicda/                    # CI/CD configuration
│   ├── build/                # Build configuration
│   ├── environment/          # Environment configuration for CI/CD
│   └── tests/                # Test runners and configuration
├── develop/                  # Developer tools
│   └── environment/          # Environment configuration for development
├── dist/                     # Distribution files (generated)
├── docs/                     # Documentation
├── package.json              # Package configuration
└── README.md                 # Project overview
```

## Implementation Details

### WebRTC Connection Management

The application uses the WebAX distributed communication system to establish WebRTC connections between peers:

```typescript
// src/comlink/connection.ts
import { createConnection } from '@net.webax/comlink-core';

export async function connectToPeer(peerId: string) {
  // Create a connection to the peer
  const connection = await createConnection(peerId, {
    signaling: 'wss://signaling.webax.net',
    iceServers: [
      { urls: 'stun:stun.webax.net:3478' },
      {
        urls: 'turn:turn.webax.net:3478',
        username: 'webax',
        credential: 'webax'
      }
    ]
  });

  // Set up event handlers
  connection.on('open', () => {
    console.log(`Connected to peer: ${peerId}`);
  });

  connection.on('close', () => {
    console.log(`Disconnected from peer: ${peerId}`);
  });

  connection.on('error', (error) => {
    console.error(`Connection error: ${error.message}`);
  });

  return connection;
}
```

### Message Handling

Messages are sent and received through the WebRTC data channel:

```typescript
// src/comlink/messages.ts
import { Connection } from '@net.webax/comlink-core';
import { encrypt, decrypt } from '../utils/crypto';

export interface Message {
  id: string;
  sender: string;
  recipient: string;
  content: string;
  timestamp: number;
  encrypted: boolean;
}

export async function sendMessage(
  connection: Connection,
  message: Message
) {
  // Encrypt the message if needed
  let messageToSend = message;
  if (message.encrypted) {
    messageToSend = {
      ...message,
      content: await encrypt(message.content, message.recipient)
    };
  }

  // Send the message
  connection.send({
    type: 'chat-message',
    data: messageToSend
  });

  // Store the message locally
  await storeMessage(messageToSend);
}

export async function receiveMessage(
  message: Message
) {
  // Decrypt the message if needed
  let receivedMessage = message;
  if (message.encrypted) {
    receivedMessage = {
      ...message,
      content: await decrypt(message.content)
    };
  }

  // Store the message locally
  await storeMessage(receivedMessage);

  return receivedMessage;
}

async function storeMessage(message: Message) {
  // Store the message in the distributed hash system
  const dhs = await import('@net.webax/comlink-dhs');
  await dhs.put(`message:${message.id}`, message);
}
```

### Distributed Storage

The application uses the WebAX Distributed Hash System (DHS) to store and retrieve messages:

```typescript
// src/comlink/storage.ts
import { init, put, get, list } from '@net.webax/comlink-dhs';
import { Message } from './messages';

// Initialize the DHS
export async function initStorage() {
  await init({
    storageQuota: 10 * 1024 * 1024, // 10MB
    maxPeers: 10
  });
}

// Store a message
export async function storeMessage(message: Message) {
  await put(`message:${message.id}`, message);
}

// Get a message
export async function getMessage(messageId: string) {
  return await get(`message:${messageId}`);
}

// Get all messages
export async function getAllMessages() {
  const keys = await list('message:');
  const messages = await Promise.all(
    keys.map(key => get(key))
  );
  return messages.sort((a, b) => a.timestamp - b.timestamp);
}
```

### State Management

The application uses jcule for state management:

```typescript
// src/state/actions.ts
import { createAction } from '@net.webax/jcule';
import { Message } from '../comlink/messages';

export const sendMessage = createAction<Message>('SEND_MESSAGE');
export const receiveMessage = createAction<Message>('RECEIVE_MESSAGE');
export const connectToPeer = createAction<string>('CONNECT_TO_PEER');
export const disconnectFromPeer = createAction<string>('DISCONNECT_FROM_PEER');
```

```typescript
// src/state/reducers.ts
import { createReducer } from '@net.webax/jcule';
import { Message } from '../comlink/messages';

interface ChatState {
  messages: Message[];
  connections: string[];
  users: {
    id: string;
    name: string;
    online: boolean;
  }[];
}

const initialState: ChatState = {
  messages: [],
  connections: [],
  users: []
};

export const chatReducer = createReducer(initialState, {
  'RECEIVE_MESSAGE': (state, action) => {
    return {
      ...state,
      messages: [...state.messages, action.payload]
    };
  },
  'CONNECT_TO_PEER': (state, action) => {
    return {
      ...state,
      connections: [...state.connections, action.payload],
      users: state.users.map(user => 
        user.id === action.payload 
          ? { ...user, online: true } 
          : user
      )
    };
  },
  'DISCONNECT_FROM_PEER': (state, action) => {
    return {
      ...state,
      connections: state.connections.filter(id => id !== action.payload),
      users: state.users.map(user => 
        user.id === action.payload 
          ? { ...user, online: false } 
          : user
      )
    };
  }
});
```

### UI Components

The application uses jellicule components for the UI:

```tsx
// src/components/ChatView.tsx
import React from 'react';
import { useStore } from '@net.webax/jcule';
import { MessageList } from './MessageList';
import { UserList } from './UserList';
import { MessageInput } from './MessageInput';

export function ChatView() {
  const { state } = useStore();
  
  return (
    <div className="chat-view">
      <div className="chat-sidebar">
        <UserList users={state.users} />
      </div>
      <div className="chat-main">
        <MessageList messages={state.messages} />
        <MessageInput />
      </div>
    </div>
  );
}
```

```tsx
// src/components/MessageInput.tsx
import React, { useState } from 'react';
import { useStore } from '@net.webax/jcule';
import { sendMessage } from '../state/actions';
import { v4 as uuidv4 } from 'uuid';

export function MessageInput() {
  const { state, dispatch } = useStore();
  const [message, setMessage] = useState('');
  
  const handleSend = () => {
    if (!message.trim()) return;
    
    // Create a new message
    const newMessage = {
      id: uuidv4(),
      sender: 'current-user-id', // Replace with actual user ID
      recipient: 'selected-user-id', // Replace with selected user ID
      content: message,
      timestamp: Date.now(),
      encrypted: true
    };
    
    // Dispatch the action
    dispatch(sendMessage(newMessage));
    
    // Clear the input
    setMessage('');
  };
  
  return (
    <div className="message-input">
      <input
        type="text"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Type a message..."
      />
      <button onClick={handleSend}>Send</button>
    </div>
  );
}
```

## Running the Example

1. Install dependencies:

```bash
npm install
```

2. Start the development server:

```bash
npm run dev
```

3. Open your browser and navigate to http://localhost:3000

4. Share your peer ID with another user to start chatting

## Deployment

The application can be deployed as a Progressive Web App (PWA) or distributed through the WebAX Distributed Hash System (DHS):

```bash
# Build the application
npm run build

# Publish to DHS
npx webaxp dhs publish
```

After publishing, users can access the application using the DHS URL:

```
webax://[content-hash]
```

## Conclusion

This example demonstrates how to build a peer-to-peer chat application using the WebAX distributed communication system. The application showcases the key features of the system, including WebRTC communication, distributed storage, and offline support.

By following this example, developers can create their own distributed applications using the WebAX ecosystem.
