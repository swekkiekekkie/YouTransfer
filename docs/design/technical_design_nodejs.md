# Technical Design

## 1. Introduction

This document outlines the design of a direct file transfer application, which facilitates secure, peer-to-peer file transfers between users. The application aims to overcome challenges such as NAT traversal and ensure cross-platform compatibility. For more details on the project goals, refer to the Project Overview. This document is inspired by the Functional Requirements Document and focuses on the design aspects of the system.

## 2. System Architecture

The application is designed with a clear separation between the client-side GUI and the signaling server. This modular approach enhances both the maintainability and scalability of the system.

### 2.1 Client-Side Architecture

The client-side application is built using modern web technologies to ensure a responsive and user-friendly interface. The key technologies used in the client-side are:
- React: A JavaScript library for building user interfaces.
- TypeScript: A statically typed superset of JavaScript that improves code quality and maintainability.
- Electron: A framework for building cross-platform desktop applications with web technologies.
- Redux Toolkit: A library for managing application state and caching data fetched from the backend.
- Material-UI: A popular React UI framework that provides pre-designed components.

#### Network Protocol Libraries
- TCP/UDP: Using Node.js built-in 'net' and 'dgram' modules with TypeScript support via @types/node
- QUIC Protocol: Experimental support via node-quic and http3 libraries
- WebRTC: Using wrtc and simple-peer libraries for peer-to-peer connections

#### Security Libraries
- TLS/SSL: Node.js built-in 'tls' module with TypeScript support
- Certificate Management: node-forge for PKI and X.509 certificates
- Cryptography: Node.js crypto module and crypto-js for data integrity checks

#### Pages
Electron pages are used to define the different views of the application. Each page is associated with a TypeScript file in the /client/src/pages/ directory. More information about what pages are available can be found in the User Interface Design document.

#### Main Process
The main process in Electron handles the lifecycle of the application, including window creation and inter-process communication. It is located in the /client/src/main/ directory.

#### Client Codebase
The client codebase is located in the `/client/` directory, its structure is as follows:
- `/client/src/pages/`: Contains the different views of the application.
- `/client/src/components/`: Reusable UI components.
- `/client/src/store/`: Redux store configuration and slices.
- `/client/src/main/`: Main process scripts for Electron.

### 2.2 Signaling Server Architecture

The signaling server is designed to handle the exchange of connection information between peers. The key technologies used in the signaling server are:
- Node.js: A JavaScript runtime built on Chrome's V8 JavaScript engine.
- TypeScript: Ensures type safety and enhances code maintainability.
- Express: A minimal and flexible Node.js web application framework, used to create RESTful endpoints for signaling.

#### NAT Traversal Libraries
- STUN: node-stun and stun libraries for client implementation
- TURN: node-turn for server implementation
- ICE: Integrated via WebRTC libraries (wrtc and simple-peer)

#### File Transfer Libraries
- Chunking: Node.js fs and stream modules, with read-chunk for chunk management
- Resume Capability: resumable.js for stable uploads and HTTP range requests
- Data Integrity: crypto module and crypto-js for checksums and verification

The signaling server codebase is located in the `/server/` directory.

#### Signaling Server Design
The signaling server facilitates the exchange of session descriptions and ICE candidates between peers. It is implemented using Express and aiohttp, providing endpoints for offer and answer exchanges.

#### Security and Access Control
The signaling server implements basic security measures, such as authentication tokens, to ensure that only authorized peers can exchange connection information.

### 3.1 Motivation for Separation
The separation of client-side and server components is primarily motivated by NAT traversal requirements:
- NAT Traversal: A central signaling server is required to facilitate peer discovery and connection establishment between clients behind NAT routers
- Connection Negotiation: The server enables WebRTC offer/answer exchange between peers that cannot directly communicate initially
- ICE Framework: Server assists with Interactive Connectivity Establishment (ICE) by relaying ICE candidates between peers
- STUN/TURN Support: Server provides STUN/TURN services to help peers establish direct connections through NAT

### 3.1 Client-Side Implementation
- File Transfer Logic: Implemented using WebRTC data channels for peer-to-peer file transfers.
- NAT Traversal: Utilizes WebRTC libraries for STUN and TURN protocols.
- User Interface: Built with React and Material-UI for a responsive and intuitive user experience.

### 3.2 Signaling Server Implementation
- Endpoints: Provides RESTful endpoints for offer and answer exchanges.
- Security: Implements basic authentication to secure signaling exchanges.

## 4. Conclusion
This technical design provides a comprehensive framework for developing a secure and efficient direct file transfer application. By leveraging existing protocols and libraries, the application can overcome common networking challenges and deliver a seamless user experience.