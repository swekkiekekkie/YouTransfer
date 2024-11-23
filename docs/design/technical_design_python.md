# Technical Design

## 1. Introduction

This document outlines the design of a direct file transfer application, which facilitates secure, peer-to-peer file transfers between users. The application aims to overcome challenges such as NAT traversal and ensure cross-platform compatibility. For more details on the project goals, refer to the Project Overview. This document is inspired by the Functional Requirements Document and focuses on the design aspects of the system.

## 2. System Architecture

The application is designed with a clear separation between the client-side GUI and the signaling server. This modular approach enhances both the maintainability and scalability of the system.

### 2.1 Client-Side Architecture

The client-side application is built using modern web technologies to ensure a responsive and user-friendly interface. The key technologies used in the client-side are:
- **Tkinter**: A standard GUI toolkit for Python, providing a simple and effective way to create desktop applications.
- **PyQt**: An alternative to Tkinter, offering more advanced UI components and features.
- **Type Hints**: Used throughout the codebase to improve code quality and maintainability.

#### Network Protocol Libraries
- **TCP/UDP**: Using Python's built-in `socket` module for network communication.
- **QUIC Protocol**: Experimental support via libraries like `aioquic`.
- **WebRTC**: Using `aiortc` for peer-to-peer connections and data channels.

#### Security Libraries
- **TLS/SSL**: Python's built-in `ssl` module for secure communication.
- **Certificate Management**: `cryptography` library for handling PKI and X.509 certificates.
- **Cryptography**: Python's `hashlib` and `cryptography` libraries for data integrity checks.

#### Client Codebase
The client codebase is located in the `/client/` directory, its structure is as follows:
- `/client/src/pages/`: Contains the different views of the application.
- `/client/src/components/`: Reusable UI components.
- `/client/src/store/`: State management and data handling.
- `/client/src/main/`: Main process scripts for application lifecycle management.

### 2.2 Signaling Server Architecture

The signaling server is designed to handle the exchange of connection information between peers. The key technologies used in the signaling server are:
- **Python**: A versatile programming language with robust networking support.
- **aiohttp**: An asynchronous web framework for creating RESTful endpoints for signaling.

#### NAT Traversal Libraries
- **STUN**: `pystun3` library for client implementation.
- **TURN**: `pytun` or similar libraries for server implementation.
- **ICE**: Integrated via `aiortc` for WebRTC support.

#### File Transfer Libraries
- **Chunking**: Python's `os` and `io` modules for file handling and chunk management.
- **Resume Capability**: Custom implementation using HTTP range requests and file metadata.
- **Data Integrity**: `hashlib` for checksums and verification.

The signaling server codebase is located in the `/server/` directory.

#### Signaling Server Design
The signaling server facilitates the exchange of session descriptions and ICE candidates between peers. It is implemented using `aiohttp`, providing endpoints for offer and answer exchanges.

#### Security and Access Control
The signaling server implements basic security measures, such as authentication tokens, to ensure that only authorized peers can exchange connection information.

### 3.1 Motivation for Separation
The separation of client-side and server components is primarily motivated by NAT traversal requirements:
- **NAT Traversal**: A central signaling server is required to facilitate peer discovery and connection establishment between clients behind NAT routers.
- **Connection Negotiation**: The server enables WebRTC offer/answer exchange between peers that cannot directly communicate initially.
- **ICE Framework**: Server assists with Interactive Connectivity Establishment (ICE) by relaying ICE candidates between peers.
- **STUN/TURN Support**: Server provides STUN/TURN services to help peers establish direct connections through NAT.

### 3.1 Client-Side Implementation
- **File Transfer Logic**: Implemented using WebRTC data channels for peer-to-peer file transfers.
- **NAT Traversal**: Utilizes `aiortc` for STUN and TURN protocols.
- **User Interface**: Built with Tkinter or PyQt for a responsive and intuitive user experience.

### 3.2 Signaling Server Implementation
- **Endpoints**: Provides RESTful endpoints for offer and answer exchanges.
- **Security**: Implements basic authentication to secure signaling exchanges.

## 4. Conclusion
This technical design provides a comprehensive framework for developing a secure and efficient direct file transfer application. By leveraging existing protocols and libraries, the application can overcome common networking challenges and deliver a seamless user experience. 