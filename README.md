# Microservices Blog App

A modern blog application built using microservices architecture with Docker and Kubernetes orchestration.

## Overview

This project demonstrates a scalable blog platform built with Node.js microservices, featuring independent services for posts, comments, moderation, and querying. The application uses an event-driven architecture to communicate between services and is containerized with Docker and orchestrated using Kubernetes.

## Architecture

The application follows a microservices architecture pattern with the following key components:

### Services

1. **Client Service** (Port 3000)
   - React-based frontend application
   - Enables users to create posts and add comments
   - Communicates with backend services via HTTP APIs

2. **Posts Service** (Port 4000)
   - Handles blog post creation and retrieval
   - Maintains a registry of all posts
   - Publishes `PostCreated` events when new posts are added
   - **Route:** `POST /posts/create`, `GET /posts`

3. **Comments Service** (Port 4001)
   - Manages comments for individual posts
   - Stores comments with moderation status (pending/approved/rejected)
   - Publishes `CommentCreated` and `CommentUpdated` events
   - Listens to `CommentModerated` events for status updates
   - **Route:** `GET /posts/:id/comments`, `POST /posts/:id/comments`

4. **Query Service** (Port 4002)
   - Provides an optimized read model for the application
   - Maintains a denormalized view of posts with their comments
   - Synchronizes state by replaying events from the event bus on startup
   - Rebuilds data structure based on event stream
   - **Route:** `GET /posts`

5. **Event Bus Service** (Port 4005)
   - Central message broker for all inter-service communication
   - Stores all events for replay capability
   - Routes events to all interested services
   - Ensures reliable event delivery across services

6. **Moderation Service** (Port 4003)
   - Reviews comments for inappropriate content (currently filters comments containing "orange")
   - Publishes `CommentModerated` events with approval/rejection status
   - Listens to `CommentCreated` events

## Event Flow

```
User Action → Frontend (Client)
     ↓
  API Request to Service
     ↓
  Service Creates Event
     ↓
  Event Posted to Event Bus
     ↓
  Event Bus Distributes to All Services
     ↓
  Services Update Their Data & Publish New Events (if needed)
```

### Key Events

- **PostCreated:** Triggered when a new blog post is created
- **CommentCreated:** Triggered when a new comment is added to a post
- **CommentModerated:** Triggered by moderation service with approval/rejection status
- **CommentUpdated:** Triggered when comment status changes after moderation

## Technology Stack

### Frontend
- **React** (v18.3.1) - UI framework
- **Axios** - HTTP client for API communication
- **React Scripts** - Build tooling

### Backend Services
- **Node.js** - Runtime environment
- **Express.js** (v4.19.2) - Web framework
- **Axios** (v1.7.5) - HTTP client for inter-service communication
- **Body-parser** - Middleware for parsing JSON request bodies
- **CORS** - Cross-origin resource sharing middleware
- **Nodemon** (v3.1.4) - Development auto-restart tool

### Infrastructure
- **Docker** - Container runtime
- **Kubernetes** - Container orchestration
- **Skaffold** - Development workflow tool

## Project Structure

```
Microservices-Blog-App/
├── client/                 # React frontend application
│   ├── src/
│   ├── public/
│   └── package.json
├── posts/                  # Posts microservice
│   ├── index.js
│   ├── Dockerfile
│   └── package.json
├── comments/              # Comments microservice
│   ├── index.js
│   ├── Dockerfile
│   └── package.json
├── moderation/            # Moderation microservice
│   ├── index.js
│   ├── Dockerfile
│   └── package.json
├── query/                 # Query/Read model service
│   ├── index.js
│   ├── Dockerfile
│   └── package.json
├── event-bus/             # Event bus/Message broker
│   ├── index.js
│   ├── Dockerfile
│   └── package.json
├── infra/
│   └── k8s/              # Kubernetes manifests
├── skaffold.yaml         # Skaffold configuration
└── README.md
```

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- Docker
- Kubernetes cluster (or Minikube for local development)
- Skaffold

### Local Development with Skaffold

1. **Clone the repository**
   ```bash
   git clone https://github.com/imgirish07/Microservices-Blog-App.git
   cd Microservices-Blog-App
   ```

2. **Install dependencies for each service**
   ```bash
   # Frontend
   cd client && npm install && cd ..
   
   # Backend services
   cd posts && npm install && cd ..
   cd comments && npm install && cd ..
   cd moderation && npm install && cd ..
   cd query && npm install && cd ..
   cd event-bus && npm install && cd ..
   ```

3. **Start with Skaffold**
   ```bash
   skaffold dev
   ```

   Skaffold will:
   - Build Docker images for all services
   - Deploy to your Kubernetes cluster
   - Set up file synchronization for live code reload
   - Stream logs from all services

4. **Access the application**
   - Frontend: `http://localhost:3000`

### Running Services Individually (Development)

Each service can be run independently with nodemon for auto-restart:

```bash
# Terminal 1 - Event Bus
cd event-bus && npm start

# Terminal 2 - Posts Service
cd posts && npm start

# Terminal 3 - Comments Service
cd comments && npm start

# Terminal 4 - Query Service
cd query && npm start

# Terminal 5 - Moderation Service
cd moderation && npm start

# Terminal 6 - Client (from client directory)
cd client && npm start
```

## API Endpoints

### Posts Service
- `GET /posts` - Retrieve all posts
- `POST /posts/create` - Create a new post

### Comments Service
- `GET /posts/:id/comments` - Get comments for a post
- `POST /posts/:id/comments` - Add a comment to a post

### Query Service
- `GET /posts` - Get denormalized posts with comments

### Event Bus
- `POST /events` - Publish an event
- `GET /events` - Retrieve all events

### Moderation Service
- `POST /events` - Receive and process events

## How It Works

1. **User creates a post** via the React frontend
2. **Posts Service** stores the post and emits a `PostCreated` event
3. **Event Bus** receives the event and distributes it to all services
4. **Query Service** receives the event and updates its read model
5. **User adds a comment** to the post via frontend
6. **Comments Service** stores the comment with "pending" status and emits `CommentCreated`
7. **Event Bus** routes the event to the Moderation Service
8. **Moderation Service** reviews the comment and emits `CommentModerated` with approval/rejection
9. **Comments Service** receives moderation result and emits `CommentUpdated`
10. **Query Service** updates comment status in its read model
11. **Frontend** fetches updated data to display to the user

## Docker Images

Built images are pushed to Docker Hub under the `imgirish07/` namespace:
- `imgirish07/client`
- `imgirish07/posts`
- `imgirish07/comments`
- `imgirish07/moderation`
- `imgirish07/query`
- `imgirish07/event-bus`

## Kubernetes Deployment

Kubernetes manifests are located in the `infra/k8s/` directory and define:
- Service deployments for each microservice
- ClusterIP services for inter-service communication
- Frontend service (typically LoadBalancer or NodePort)

Deploy to Kubernetes:
```bash
kubectl apply -f infra/k8s/
```

## Development Considerations

- **Service Discovery:** Uses Kubernetes service names for inter-service communication (e.g., `posts-clusterip-srv:4000`)
- **Event Replay:** The Query Service replays all events on startup to rebuild its state
- **Moderation Logic:** Currently simple (filters "orange" keyword), can be extended
- **State Storage:** All data is in-memory; no persistent database in current implementation
- **Error Handling:** Basic error logging; can be enhanced with structured logging

## Future Enhancements

- Add persistent database (MongoDB, PostgreSQL) for data storage
- Implement proper authentication and authorization
- Add more sophisticated moderation algorithms
- Implement distributed tracing and monitoring
- Add comprehensive error handling and retry logic
- Implement API rate limiting
- Add unit and integration tests
- Implement health checks and readiness probes
- Add CI/CD pipeline

## License

ISC

## Author

[imgirish07](https://github.com/imgirish07)

## Repository

[Microservices-Blog-App](https://github.com/imgirish07/Microservices-Blog-App)
