# Backend Architecture: Discord Clone

This presentation provides a detailed overview of the backend architecture for the Discord clone application.

## 1. Backend Overview

The backend is built using a modern, scalable, and maintainable architecture. It leverages a set of powerful and popular technologies to provide a robust and real-time communication platform.

### High-Level Architecture

The backend follows a classic three-tier architecture:

1.  **Presentation Layer**: This layer is responsible for handling incoming HTTP requests and real-time connections. It includes the Express server, API routes, and Socket.IO for real-time communication.

2.  **Business Logic Layer**: This layer contains the core application logic. It includes controllers that process requests, interact with the database, and perform business operations.

3.  **Data Access Layer**: This layer is responsible for interacting with the database. It uses Prisma as an ORM to provide a type-safe and intuitive way to access the database.

### Key Technologies

-   **Framework**: Express.js
-   **Database**: Prisma (ORM)
-   **Authentication**: `bcryptjs` and `jsonwebtoken` (JWT)
-   **Real-time**: `socket.io`
-   **Session Management**: `express-session` with Redis
-   **Security**: `cors`, `csurf`, `cookie-parser`
-   **Validation**: `zod`
-   **Language**: TypeScript

## 2. Project Structure

The backend codebase is organized into a modular and easy-to-navigate structure. This separation of concerns makes the application easier to understand, maintain, and scale.

```
/apps/server/
├── src/
│   ├── config/         # Configuration files (Express, Sockets, DB)
│   ├── controllers/    # Request handling and business logic
│   ├── middleware/     # Custom middleware (auth, error handling)
│   ├── routes/         # API endpoint definitions
│   ├── validators/     # Data validation schemas (Zod)
│   └── server.ts       # Application entry point
├── .env              # Environment variables
├── package.json        # Dependencies and scripts
└── tsconfig.json       # TypeScript configuration
```

### Key Directories

-   **`src/config`**: Contains the setup for Express (`express.ts`), Socket.IO (`socket.ts`), database connection (`database.ts`), and Redis (`redis.ts`).
-   **`src/controllers`**: Holds the core logic for handling API requests. For example, `auth.controller.ts` would manage user registration and login.
-   **`src/middleware`**: Includes functions that run before the main request handlers, such as checking for user authentication or handling errors.
-   **`src/routes`**: Defines the API endpoints. For instance, `auth.routes.ts` maps URLs like `/api/auth/login` to the corresponding controller functions.
-   **`src/validators`**: Contains schemas for validating incoming data, ensuring data integrity before it's processed.
-   **`server.ts`**: The main file that bootstraps and starts the Express server, initializes configurations, and mounts the API routes.

## 3. Configuration

The `src/config` directory is the central hub for setting up the application's core components. Each file has a distinct responsibility, ensuring a clean and organized configuration.

### `database.ts`

-   **Purpose**: Manages the connection to the PostgreSQL database using Prisma.
-   **Key Functions**:
    -   `connectDB()`: Establishes a connection to the database when the server starts.
    -   `disconnectDB()`: Closes the database connection gracefully.

### `express.ts`

-   **Purpose**: Configures the Express application by setting up all necessary middleware.
-   **Key Middleware**:
    -   `cors`: Enables Cross-Origin Resource Sharing to allow requests from the frontend.
    -   `express.json()` and `express.urlencoded()`: Parses incoming request bodies.
    -   `cookie-parser`: Parses cookies attached to the client request.
    -   `express-session`: Manages user sessions, using `connect-redis` to store session data in Redis for persistence.
    -   `csurf`: Provides CSRF protection to secure the application from cross-site request forgery attacks.

### `redis.ts`

-   **Purpose**: Initializes and manages the connection to the Redis server.
-   **Key Features**:
    -   Establishes a connection to Redis using `ioredis`.
    -   Handles connection events (`connect`, `error`, `close`) to ensure the application can gracefully handle Redis availability.

### `socket.ts`

-   **Purpose**: Configures the Socket.IO server for real-time communication.
-   **Key Features**:
    -   **Authentication**: Implements middleware to authenticate socket connections.
    -   **Event Handling**: Defines event listeners for various real-time actions, such as:
        -   `join-servers`: Allows a user to join rooms for each server they are a member of.
        -   `join-channel`: Allows a user to join a specific channel room.
        -   `send-message`: Handles incoming messages, saves them to the database, and broadcasts them to the appropriate channel.
        -   `typing-start` and `typing-stop`: Manages typing indicators.

## 4. API Endpoints and Logic

The backend exposes a RESTful API for managing users, servers, and messages. The API is divided into three main resources: `auth`, `servers`, and `messages`.

### Authentication (`/api/auth`)

-   **`POST /register`**: Registers a new user. It hashes the password, creates a new user in the database, and starts a new session.
-   **`POST /login`**: Authenticates a user. It verifies the email and password, and if successful, creates a session.
-   **`POST /logout`**: Logs out the user by destroying the current session.
-   **`GET /me`**: Retrieves the profile of the currently authenticated user.
-   **`PATCH /profile`**: Updates the profile of the currently authenticated user.

### Servers (`/api/servers`)

-   **`POST /`**: Creates a new server. The user who creates the server is automatically assigned as the owner.
-   **`GET /`**: Retrieves a list of servers that the current user is a member of.
-   **`GET /:serverId`**: Retrieves the details of a specific server, including its members and channels.
-   **`POST /join`**: Allows a user to join a server using an invite code.
-   **`POST /:serverId/leave`**: Allows a user to leave a server.
-   **`DELETE /:serverId`**: Deletes a server. This action is restricted to the server owner.
-   **`PATCH /:serverId`**: Updates the details of a server. This action is restricted to the server owner.
-   **`POST /:serverId/invite`**: Regenerates the invite code for a server. This action is restricted to the server owner.

### Messages (`/api`)

-   **`POST /channels/:channelId/messages`**: Sends a message to a specific channel.
-   **`GET /channels/:channelId/messages`**: Retrieves the messages from a specific channel, with pagination support.
-   **`DELETE /messages/:messageId`**: Deletes a message. This action is restricted to the author of the message.

## 5. Security and Data Validation

The application employs a multi-layered approach to security and data integrity, using custom middleware and validation schemas to protect the API.

### Middleware

-   **`auth.middleware.ts`**: This middleware protects routes by checking for a valid user session. If a session exists, it attaches the user's ID to the request object, making it available to downstream handlers. Routes that require authentication use this middleware to ensure only logged-in users can access them.

-   **`error.middleware.ts`**: This file provides a centralized error-handling mechanism. It catches errors thrown anywhere in the application, logs them, and sends a standardized error response to the client. This prevents sensitive error details from being leaked and ensures a consistent error format.

-   **`validate.middleware.ts`**: This middleware integrates `zod` for request validation. It takes a `zod` schema and validates the request's body, query parameters, and URL parameters against it. If validation fails, it sends a detailed 400 Bad Request response, preventing invalid data from reaching the controllers.

### Data Validation (`/validators`)

-   **Purpose**: This directory contains all the `zod` schemas used for validating incoming data. Each schema defines the expected shape and type of data for a specific API endpoint.
-   **Examples**:
    -   `auth.validator.ts`: Defines schemas for user registration, login, and profile updates, ensuring that fields like email, username, and password meet specific criteria (e.g., minimum length, valid email format).
    -   `server.validator.ts`: Contains schemas for creating and joining servers, validating data like server names and invite codes.
    -   `message.validator.ts`: Provides schemas for sending messages, ensuring that message content is not empty and does not exceed the maximum length.

## 6. Real-time Features: Voice/Video Calls & Direct Messaging

The application includes a rich set of real-time features, enabling seamless communication between users.

### Direct Messaging (DMs)

-   **Database Schema**: A `Conversation` model represents a direct message thread between users, and a `DirectMessage` model stores the messages within these conversations.
-   **Socket.IO Rooms**: Each conversation has its own Socket.IO room (e.g., `conversation:<conversationId>`). When a user opens a DM, their socket joins this room to receive real-time messages.
-   **API Endpoints**:
    -   `POST /api/conversations`: Starts a new conversation with another user.
    -   `GET /api/conversations`: Gets the current user's conversation list.
    -   `POST /api/conversations/:conversationId/messages`: Sends a direct message.

### Audio/Video Calls with WebRTC

WebRTC is used for peer-to-peer, real-time voice and video communication directly in the browser. The server's role is primarily for signaling—coordinating the connection between users.

-   **Signaling Server**: The existing Socket.IO server acts as the signaling server, passing messages (like offers, answers, and ICE candidates) between users to establish a direct peer-to-peer connection.
-   **Socket.IO Events for Signaling**:
    -   `join-call`: A user signals their intent to join a call in a specific channel or DM.
    -   `offer`: A user sends a connection offer to another user in the call.
    -   `answer`: The receiving user sends back an answer to the offer.
    -   `ice-candidate`: Both users exchange network information (ICE candidates) to find the best path for the peer-to-peer connection.
-   **STUN/TURN Servers**: For users behind complex firewalls (NAT), STUN/TURN servers are used to facilitate the connection.

## 7. Next Steps: Scaling for a Massive User Base

With a feature-complete application, the next challenge is to ensure the architecture can handle a massive, growing user base. The following strategies are critical for scaling to millions of concurrent users.

### Horizontal Scaling

-   **Load Balancer**: Instead of a single server instance, the application is deployed across multiple machines. A load balancer (like Nginx or a cloud provider's load balancer) distributes incoming traffic evenly across these instances.
-   **Sticky Sessions**: For Socket.IO to work across multiple instances, sticky sessions are configured on the load balancer. This ensures a user's socket connection is always routed to the same server instance. The `socket.io-redis-adapter` is also used to broadcast events across all instances.

### Database Scaling

-   **Read Replicas**: To handle a high volume of read operations, read replicas of the main database are created. Read-heavy queries are directed to these replicas, reducing the load on the primary database.
-   **Database Sharding**: For extremely large datasets, the database is sharded (partitioned) horizontally. This means splitting the data across multiple database servers, with each server holding a subset of the data.

### Handling Mass Messaging and Abuse

-   **Rate Limiting**: To prevent spam and abuse, a rate limiter is implemented. This restricts the number of requests a user can make to the API within a certain time frame (e.g., 10 messages per minute). Redis is ideal for implementing an efficient rate limiter.
-   **Message Queues**: For non-critical, intensive operations (like sending notifications or processing media), a message queue (like RabbitMQ or AWS SQS) is used. Instead of processing these tasks immediately, they are added to a queue and processed by a separate fleet of worker services. This prevents the main API server from being blocked and improves responsiveness.

## 8. Advanced Scalability and Architectural Patterns

For massive scale, like that of Discord, more advanced techniques are employed to ensure reliability, low latency, and high availability.

### Caching Layer

-   **Purpose**: To reduce database load and improve response times for frequently accessed data.
-   **Implementation**: A distributed cache like Redis or Memcached is used to store data that is read often but not updated frequently. Examples include user profiles, server details, and channel information. When a request for this data comes in, the application first checks the cache. If the data is present (a cache hit), it's returned immediately. If not (a cache miss), the data is fetched from the database, stored in the cache, and then returned.

### Content Delivery Network (CDN)

-   **Purpose**: To serve static assets (like user avatars, images, and videos) quickly to users around the world.
-   **Implementation**: A CDN like Cloudflare or Amazon CloudFront is used to cache static content in edge locations geographically closer to users. When a user requests an asset, it's served from the nearest edge location, resulting in significantly lower latency.

### Microservices Architecture

-   **Concept**: Instead of a single monolithic backend, the application is broken down into smaller, independent services. Each service is responsible for a specific business capability (e.g., a `UserService`, a `MessageService`, a `PresenceService`).
-   **Benefits**:
    -   **Independent Scaling**: Each service can be scaled independently based on its specific load.
    -   **Improved Fault Isolation**: If one service fails, it doesn't bring down the entire application.
    -   **Technology Flexibility**: Different services can be written in different languages or use different technologies best suited for their task.

### Observability: Logging, Monitoring, and Alerting

-   **Purpose**: To gain deep insights into the health and performance of the system, enabling proactive problem detection and resolution.
-   **Components**:
    -   **Structured Logging**: All services generate structured logs (e.g., in JSON format) that can be easily collected, searched, and analyzed by a centralized logging platform like the ELK Stack (Elasticsearch, Logstash, Kibana) or Datadog.
    -   **Metrics and Monitoring**: Key performance indicators (KPIs) like CPU usage, memory consumption, request latency, and error rates are collected using a tool like Prometheus and visualized in dashboards using Grafana.
    -   **Alerting**: Automated alerts are set up (e.g., with Prometheus Alertmanager) to notify the engineering team of any critical issues, such as a spike in error rates or a service becoming unresponsive.
