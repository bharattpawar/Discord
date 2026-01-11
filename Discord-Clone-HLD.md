# Discord Clone - High-Level Design (HLD) Document

## Project Overview

A scalable, real-time communication platform built with free and open-source technologies. This document outlines the complete architecture, data flow, and system design for a Discord-like application.

---

## 1. Tech Stack (Free & Open Source)

### Frontend
- **Framework**: React.js with Hooks
- **State Management**: Redux or Context API
- **Styling**: Tailwind CSS
- **Real-time Communication**: Socket.io Client
- **UI Components**: React Bootstrap or Material-UI (free tier)

### Backend
- **Runtime**: Node.js
- **Framework**: Express.js
- **Real-time Engine**: Socket.io
- **Authentication**: JWT + Bcrypt
- **File Upload**: Multer (for local storage)

### Database
- **Primary**: MongoDB (free tier via MongoDB Atlas or local)
- **Cache**: Redis (optional, for scaling)
- **Search**: MongoDB Full-Text Search

### Infrastructure
- **Deployment**: Docker + Docker Compose
- **Hosting**: Free tier (Render, Railway, Vercel)
- **Version Control**: Git + GitHub

---

## 2. High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER (Web/Mobile)                    │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   React UI   │  │  Redux Store │  │ Socket.io    │             │
│  │ Components   │  │  (State Mgmt)│  │  Client      │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│         │                  │                  │                    │
│         └──────────────────┼──────────────────┘                    │
│                            │                                       │
│                     WebSocket Connection                           │
│                            │                                       │
└────────────────────────────┼───────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Load Balancer  │ (Nginx/HAProxy)
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼──────┐     ┌───────▼──────┐     ┌───────▼──────┐
│  Chat Server │     │  Chat Server │     │  Chat Server │
│  Instance 1  │     │  Instance 2  │     │  Instance N  │
│              │     │              │     │              │
│ ┌──────────┐ │     │ ┌──────────┐ │     │ ┌──────────┐ │
│ │Socket.io │ │     │ │Socket.io │ │     │ │Socket.io │ │
│ │ Server   │ │     │ │ Server   │ │     │ │ Server   │ │
│ └──────────┘ │     │ └──────────┘ │     │ └──────────┘ │
│ ┌──────────┐ │     │ ┌──────────┐ │     │ ┌──────────┐ │
│ │Express   │ │     │ │Express   │ │     │ │Express   │ │
│ │API       │ │     │ │API       │ │     │ │API       │ │
│ └──────────┘ │     │ └──────────┘ │     │ └──────────┘ │
└───────┬──────┘     └───────┬──────┘     └───────┬──────┘
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                    ┌────────▼──────────┐
                    │  Data Layer       │
                    ├────────┬──────────┤
                    │        │          │
              ┌─────▼─┐  ┌──▼──┐  ┌──┐ │
              │MongoDB│  │Redis│  │S3│ │
              └───────┘  └─────┘  └──┘ │
              ├─────────────────────────┤
              │ • Messages             │
              │ • Users & Profiles     │
              │ • Servers & Channels   │
              │ • Sessions & Cache     │
              │ • Media Files          │
              └────────────────────────┘

```

---

## 3. Core Components & Their Responsibilities

### 3.1 Frontend Components

#### User Authentication Module
- User registration and login
- JWT token management
- Profile management
- Friend list management

#### Server Management Module
- Create/edit/delete servers
- Server settings and customization
- Server member management
- Role assignment

#### Channel Management Module
- Create/edit/delete text and voice channels
- Channel permissions and roles
- Channel categories

#### Real-time Messaging Module
- Message sending/receiving via WebSocket
- Message editing/deletion
- Emoji reactions
- File attachments display

#### User Presence Module
- Online/offline status
- Last seen timestamp
- User activity tracking

### 3.2 Backend Components

#### Authentication Service
- User registration/login
- JWT token generation and validation
- Password hashing with Bcrypt
- Session management

#### Server Service
- Server creation and management
- Member management
- Server permissions

#### Channel Service
- Channel CRUD operations
- Channel permissions
- Channel-member relationships

#### Message Service
- Message storage and retrieval
- Message broadcasting
- Message history with pagination

#### User Service
- User profile management
- Friend management
- User presence tracking

#### Socket.io Manager
- WebSocket connection handling
- Event broadcasting
- Room management for channels
- Presence updates

#### File Service
- File upload handling
- File validation
- File storage management

---

## 4. Data Models (MongoDB Schema)

### User Schema
```javascript
{
  _id: ObjectId,
  username: String (unique),
  email: String (unique),
  password: String (hashed),
  avatar: String (URL),
  status: String (online/offline/idle),
  lastSeen: Timestamp,
  friends: [ObjectId], // Array of user IDs
  blockedUsers: [ObjectId],
  createdAt: Timestamp,
  updatedAt: Timestamp
}
```

### Server Schema
```javascript
{
  _id: ObjectId,
  name: String,
  description: String,
  ownerId: ObjectId,
  icon: String (URL),
  members: [
    {
      userId: ObjectId,
      role: String (admin/moderator/member),
      joinedAt: Timestamp
    }
  ],
  channels: [ObjectId], // Array of channel IDs
  inviteCode: String (unique),
  createdAt: Timestamp,
  updatedAt: Timestamp
}
```

### Channel Schema
```javascript
{
  _id: ObjectId,
  serverId: ObjectId,
  name: String,
  description: String,
  type: String (text/voice/announcement),
  categoryId: ObjectId, // Optional, for organizing channels
  permissions: [
    {
      roleId: ObjectId,
      allowedActions: [String] // send_messages, read_messages, etc.
    }
  ],
  createdAt: Timestamp,
  updatedAt: Timestamp
}
```

### Message Schema
```javascript
{
  _id: ObjectId,
  channelId: ObjectId,
  serverId: ObjectId,
  senderId: ObjectId,
  content: String,
  attachments: [
    {
      url: String,
      type: String (image/video/file),
      size: Number
    }
  ],
  reactions: [
    {
      emoji: String,
      userIds: [ObjectId]
    }
  ],
  editedAt: Timestamp,
  deletedAt: Timestamp,
  createdAt: Timestamp
}
```

### Conversation Schema (DMs)
```javascript
{
  _id: ObjectId,
  participants: [ObjectId], // Array of user IDs
  lastMessage: ObjectId, // Reference to message ID
  messages: [ObjectId], // Array of message IDs
  createdAt: Timestamp,
  updatedAt: Timestamp
}
```

### Presence Schema (Redis)
```
Key: presence:{userId}
Value: {
  status: "online" | "offline" | "idle",
  lastActive: Timestamp,
  currentChannel: ObjectId
}
TTL: 30 minutes (auto-expire if no heartbeat)
```

---

## 5. Real-Time Message Flow

### 5.1 User Sends a Message

```
┌──────────────────────────────────────────────────────────────┐
│ CLIENT SIDE                                                  │
├──────────────────────────────────────────────────────────────┤
│ 1. User types message in input field                        │
│ 2. User clicks "Send" button                                │
│ 3. Redux dispatches action: sendMessage({content, file})    │
│ 4. React component shows message as "pending"               │
│ 5. Socket.io emits: socket.emit('message:send', payload)   │
└──────────────────────────────────────────────────────────────┘
                           │
                    WebSocket Transmission
                           │
┌──────────────────────────────────────────────────────────────┐
│ SERVER SIDE                                                  │
├──────────────────────────────────────────────────────────────┤
│ 6. Socket.io server receives 'message:send' event           │
│ 7. Validate message and user permissions                    │
│ 8. Create message document with unique ID                   │
│ 9. Save to MongoDB message collection                       │
│ 10. Emit: socket.to(channelId).emit('message:new', msg)    │
│ 11. Send acknowledgment to sender: socket.emit('ack')      │
└──────────────────────────────────────────────────────────────┘
                           │
                    WebSocket Transmission
                           │
┌──────────────────────────────────────────────────────────────┐
│ ALL CONNECTED CLIENTS IN CHANNEL                            │
├──────────────────────────────────────────────────────────────┤
│ 12. Socket.io client receives 'message:new' event           │
│ 13. Redux updates messages array in store                   │
│ 14. React re-renders MessageList component                  │
│ 15. Message appears in real-time for all users              │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 User Presence Update

```
┌────────────────────────────────────┐
│ Client Heartbeat (every 10 seconds)│
└────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────┐
│ socket.emit('presence:update',     │
│  {userId, status, currentChannel}) │
└────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────┐
│ Server updates Redis:              │
│ presence:{userId} = data + TTL     │
└────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────┐
│ Broadcast to channel:              │
│ socket.to(channel)                 │
│  .emit('user:status-change', data) │
└────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────┐
│ All users see updated status       │
│ (online/offline indicators)        │
└────────────────────────────────────┘
```

---

## 6. API Endpoints (REST + WebSocket Events)

### REST API Endpoints

#### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `GET /api/auth/me` - Get current user
- `POST /api/auth/refresh-token` - Refresh JWT token

#### Users
- `GET /api/users/:userId` - Get user profile
- `PUT /api/users/:userId` - Update user profile
- `GET /api/users/:userId/friends` - Get user's friends
- `POST /api/users/:userId/friends/:friendId` - Add friend
- `DELETE /api/users/:userId/friends/:friendId` - Remove friend

#### Servers
- `POST /api/servers` - Create server
- `GET /api/servers` - Get user's servers
- `GET /api/servers/:serverId` - Get server details
- `PUT /api/servers/:serverId` - Update server
- `DELETE /api/servers/:serverId` - Delete server
- `POST /api/servers/:serverId/members` - Add member to server
- `GET /api/servers/:serverId/members` - Get server members
- `DELETE /api/servers/:serverId/members/:userId` - Remove member

#### Channels
- `POST /api/servers/:serverId/channels` - Create channel
- `GET /api/servers/:serverId/channels` - Get server channels
- `GET /api/channels/:channelId` - Get channel details
- `PUT /api/channels/:channelId` - Update channel
- `DELETE /api/channels/:channelId` - Delete channel

#### Messages
- `GET /api/channels/:channelId/messages` - Get messages (with pagination)
- `POST /api/channels/:channelId/messages` - Send message
- `PUT /api/messages/:messageId` - Edit message
- `DELETE /api/messages/:messageId` - Delete message

#### Direct Messages (DMs)
- `GET /api/conversations` - Get all conversations
- `GET /api/conversations/:userId` - Get conversation with user
- `POST /api/conversations/:userId/messages` - Send DM

### WebSocket Events

#### Client → Server
```javascript
// Messages
socket.emit('message:send', {channelId, content, attachments})
socket.emit('message:edit', {messageId, content})
socket.emit('message:delete', {messageId})
socket.emit('message:react', {messageId, emoji})

// Presence
socket.emit('presence:update', {status, currentChannel})
socket.emit('typing:start', {channelId})
socket.emit('typing:stop', {channelId})

// Rooms
socket.emit('channel:join', {channelId})
socket.emit('channel:leave', {channelId})

// Direct Messages
socket.emit('dm:send', {userId, content})
```

#### Server → Client
```javascript
// Message events
socket.on('message:new', (message))
socket.on('message:updated', (message))
socket.on('message:deleted', (messageId))
socket.on('message:reaction-added', (messageId, emoji, userId))

// Presence events
socket.on('user:online', (userId))
socket.on('user:offline', (userId))
socket.on('user:status-changed', (userId, status))

// Typing indicators
socket.on('typing:active', ({channelId, userId}))
socket.on('typing:inactive', ({channelId, userId}))

// Notifications
socket.on('notification:new', (notification))

// Room events
socket.on('channel:user-joined', (userId))
socket.on('channel:user-left', (userId))
```

---

## 7. Authentication Flow

```
┌─────────────────────────────────────────────────────┐
│ 1. User enters credentials (email, password)        │
└────────────────────┬────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────────┐
        │ POST /api/auth/login       │
        └────────────────┬───────────┘
                         │
                         ▼
        ┌────────────────────────────┐
        │ 2. Hash password with salt │
        │    Compare with DB hash    │
        └────────────────┬───────────┘
                         │
             ┌───────────┴───────────┐
             │                       │
          VALID                    INVALID
             │                       │
             ▼                       ▼
    ┌──────────────────┐   ┌──────────────┐
    │ 3. Generate JWT  │   │ Return Error │
    │ (Header.Payload  │   │ Status: 401  │
    │  .Signature)     │   └──────────────┘
    └────────┬─────────┘
             │
             ▼
    ┌────────────────────┐
    │ 4. Send JWT in     │
    │ response + Refresh │
    │ Token (HTTP-only)  │
    └────────┬───────────┘
             │
             ▼
    ┌────────────────────┐
    │ 5. Client stores   │
    │ JWT in memory/     │
    │ localStorage       │
    └────────┬───────────┘
             │
             ▼
    ┌────────────────────┐
    │ 6. Future requests │
    │ include JWT in     │
    │ Authorization      │
    │ header             │
    └────────────────────┘
```

---

## 8. Scalability Considerations

### Horizontal Scaling
- **Load Balancer**: Distribute connections across multiple server instances
- **Socket.io Adapter**: Use Redis adapter to sync events across server instances
- **Database Replication**: MongoDB replica set for high availability

### Database Optimization
- **Indexing**: Create indexes on frequently queried fields (userId, channelId, serverId)
- **Pagination**: Load messages in batches (default: 50 messages per request)
- **Archiving**: Archive old messages to separate collection/database
- **TTL Indexes**: Auto-delete temporary data (presence, sessions)

### Caching Strategy
- **Redis Cache**: Cache user profiles, server list, frequently accessed messages
- **Client-side Cache**: Redux store for local state management
- **CDN**: Use CDN for static assets and media files

### Performance Optimization
- **Message Queue**: Use Redis pub/sub for message broadcasting
- **Compression**: Gzip compression for HTTP responses
- **WebSocket Fallback**: Implement polling fallback if WebSocket fails
- **Lazy Loading**: Load messages on-demand, not all at once

---

## 9. Security Considerations

### Authentication & Authorization
- ✅ JWT with expiration (15-30 minutes)
- ✅ Refresh token rotation
- ✅ Password hashing (bcrypt, salt rounds: 10)
- ✅ Role-based access control (RBAC)

### Data Protection
- ✅ HTTPS/TLS encryption in transit
- ✅ Input sanitization (XSS prevention)
- ✅ SQL/NoSQL injection prevention (use parameterized queries)
- ✅ CORS configuration
- ✅ Rate limiting on API endpoints

### Privacy & Compliance
- ✅ User data encryption at rest (optional for sensitive data)
- ✅ Audit logs for admin actions
- ✅ User consent management (GDPR/privacy policy)
- ✅ Data retention policies

---

## 10. Project Folder Structure

```
discord-clone/
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Auth/
│   │   │   ├── Server/
│   │   │   ├── Channel/
│   │   │   ├── Message/
│   │   │   └── User/
│   │   ├── pages/
│   │   ├── services/ (API calls)
│   │   ├── redux/ (state management)
│   │   ├── styles/
│   │   ├── App.js
│   │   └── index.js
│   └── package.json
│
├── backend/
│   ├── config/
│   │   ├── database.js
│   │   ├── socketio.js
│   │   └── environment.js
│   ├── models/
│   │   ├── User.js
│   │   ├── Server.js
│   │   ├── Channel.js
│   │   ├── Message.js
│   │   └── Conversation.js
│   ├── routes/
│   │   ├── auth.js
│   │   ├── users.js
│   │   ├── servers.js
│   │   ├── channels.js
│   │   ├── messages.js
│   │   └── conversations.js
│   ├── controllers/
│   │   ├── authController.js
│   │   ├── userController.js
│   │   ├── serverController.js
│   │   ├── channelController.js
│   │   ├── messageController.js
│   │   └── conversationController.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── validation.js
│   ├── socket/
│   │   ├── events.js
│   │   └── handlers.js
│   ├── utils/
│   │   ├── logger.js
│   │   └── helpers.js
│   ├── server.js
│   └── package.json
│
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 11. Implementation Phases

### Phase 1: Foundation (Week 1-2)
- [ ] Project setup (React + Node.js)
- [ ] MongoDB connection
- [ ] Authentication system (register/login)
- [ ] JWT implementation
- [ ] User profile management

### Phase 2: Core Features (Week 3-4)
- [ ] Server creation and management
- [ ] Channel creation and management
- [ ] Basic REST API endpoints
- [ ] User friend list

### Phase 3: Real-Time Communication (Week 5-6)
- [ ] Socket.io setup
- [ ] Real-time message sending/receiving
- [ ] Presence tracking (online/offline)
- [ ] Typing indicators
- [ ] Message history retrieval

### Phase 4: Enhanced Features (Week 7-8)
- [ ] Message editing/deletion
- [ ] Emoji reactions
- [ ] File attachments
- [ ] Direct messaging (DMs)
- [ ] User notifications

### Phase 5: Optimization & Deployment (Week 9-10)
- [ ] Performance optimization
- [ ] Security hardening
- [ ] Error handling & logging
- [ ] Docker containerization
- [ ] Deployment to free hosting (Render/Railway)

---

## 12. Key Metrics & KPIs

- **Message Latency**: < 100ms
- **User Presence Update**: < 500ms
- **Server Response Time**: < 200ms
- **Database Query Time**: < 50ms
- **Connection Success Rate**: > 99%
- **Message Delivery Success**: > 99.9%
- **System Uptime**: > 99%

---

## 13. Common Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Message Broadcasting at Scale | Use Redis pub/sub with Socket.io adapter |
| Presence Management | Redis with TTL + Heartbeat mechanism |
| Message History Pagination | Implement cursor-based pagination |
| Duplicate Messages | Implement idempotency keys + deduplication |
| Connection Failures | WebSocket fallback with polling |
| Database Performance | Indexing, sharding, read replicas |
| Memory Leaks | Proper socket cleanup, connection pooling |
| Rate Limiting | Implement on API and WebSocket events |

---

## 14. Testing Strategy

### Unit Testing
- Test individual utility functions
- Test Redux actions/reducers
- Test API endpoint logic

### Integration Testing
- Test user authentication flow
- Test message sending workflow
- Test real-time updates

### Performance Testing
- Load testing with multiple concurrent users
- Message throughput testing
- Database query performance testing

### Security Testing
- SQL/NoSQL injection testing
- XSS vulnerability testing
- CSRF protection testing

---

## 15. Monitoring & Logging

### Key Metrics to Monitor
- CPU & Memory usage
- Database connection pool
- WebSocket connection count
- API response times
- Error rates
- Message queue depth

### Logging Strategy
- Application logs (Error, Info, Debug levels)
- Access logs (HTTP requests)
- Socket.io event logs
- Database query logs
- Security audit logs

---

## 16. Future Enhancements

- 🔊 Voice and video call integration (WebRTC)
- 📱 Mobile app (React Native)
- 🎮 Screen sharing
- 🌐 Multi-language support
- 🎨 Theme customization
- 🤖 AI-powered moderation
- 📊 Analytics dashboard
- 🔐 End-to-end encryption

---

## 17. Resources & References

### Documentation
- [Socket.io Docs](https://socket.io/docs/)
- [Express.js Guide](https://expressjs.com/)
- [MongoDB Manual](https://docs.mongodb.com/manual/)
- [React Docs](https://react.dev/)

### Free Hosting Platforms
- Render.com (Backend)
- Vercel (Frontend)
- MongoDB Atlas (Database)
- Cloudinary (Media Storage)

### Learning Resources
- System Design Interview Preparation
- Node.js Best Practices
- MongoDB Performance Tuning
- WebSocket Deep Dive

---

## 18. Quick Reference: Key Decisions

| Decision Point | Chosen Option | Reason |
|---|---|---|
| Real-time Protocol | WebSocket via Socket.io | Low latency, full-duplex, easy scaling |
| Database | MongoDB | Flexible schema, good for unstructured data |
| Authentication | JWT | Stateless, scalable, secure |
| Caching | Redis | Fast, in-memory, pub/sub support |
| File Storage | Local or S3 | Simple, free tier available |
| State Management | Redux | Predictable state flow, debugging tools |
| Deployment | Docker | Consistency, easy scaling |

---

**Document Last Updated**: January 2026  
**Project Status**: Planning Phase  
**Maintained By**: Development Team

