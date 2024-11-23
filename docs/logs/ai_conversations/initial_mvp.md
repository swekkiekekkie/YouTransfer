# Creating an MVP for Direct File Transfer

## Question to ChatGPT (o1-preview)
"Excellent, create an MVP for the project."

## Overview
Let's implement a Minimum Viable Product (MVP) of a direct file transfer application using Python. This MVP will focus on the core functionality: transferring large files directly between two computers over a network.

The MVP consists of two Python scripts:
- **Server Script**: Runs on the sender's machine and waits for a connection from the client to send a specified file
- **Client Script**: Runs on the receiver's machine, connects to the server, and receives the file

We'll use Python's built-in socket library for network communication and keep the implementation straightforward for clarity.

## Implementation

### Server Script (file_server.py)

The server script will:
- Listen on a specified port for incoming connections
- Accept a connection from a client
- Send a specified file to the connected client in chunks
