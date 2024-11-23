# Creating an MVP for Direct File Transfer with NAT Traversal

## Question to ChatGPT (o1-preview)
"Okay that is great. However we are indeed on different networks, both behind NATs. Overcoming this is part of the MVP in my opinion. It should be very easy to connect"

## Answer
Understood! Overcoming NAT traversal is indeed crucial when both peers are behind NATs and on different networks. Let's enhance the MVP to include NAT traversal so that you and your friend can easily connect and transfer large files directly.

## Solution Overview

To enable direct communication between two peers behind NATs, we'll use WebRTC (Web Real-Time Communication) technology, which includes built-in NAT traversal mechanisms using:

- ICE (Interactive Connectivity Establishment)
- STUN (Session Traversal Utilities for NAT)
- TURN (Traversal Using Relays around NAT)

For our Python implementation, we'll use the aiortc library, an asynchronous implementation of WebRTC and ORTC.

## Key Components

- **aiortc Library**: Provides WebRTC and data channel support in Python.
- **Signaling Server**: A minimal server for exchanging session descriptions (SDP) and ICE candidates between peers.
- **Peer Scripts**: Both peers run a script that handles file sending and receiving over a WebRTC data channel.

## Implementation Steps

1. Install Dependencies
2. Set Up a Signaling Server
3. Implement the Peer Script
4. Run the Application

### 1. Install Dependencies

First, install the necessary Python packages:

```bash	
pip install aiortc aiohttp asyncio
```

### 2. Set Up a Signaling Server

The signaling server is used to exchange connection information between peers. We'll create a simple server using `aiohttp`.
```python
# signaling_server.py

import asyncio
from aiohttp import web

connected_peers = {}

async def index(request):
    return web.Response(text="WebRTC Signaling Server is running.")

async def offer(request):
    params = await request.json()
    peer_id = params["peer_id"]
    offer = params["offer"]

    # Store the offer for the peer
    connected_peers[peer_id] = {"offer": offer}
    return web.Response(text="Offer received.")

async def answer(request):
    params = await request.json()
    peer_id = params["peer_id"]
    answer = params["answer"]

    if peer_id in connected_peers:
        connected_peers[peer_id]["answer"] = answer
        return web.Response(text="Answer received.")
    else:
        return web.Response(status=404, text="Peer not found.")

async def get_offer(request):
    peer_id = request.query.get("peer_id")
    if peer_id in connected_peers and "offer" in connected_peers[peer_id]:
        return web.json_response({"offer": connected_peers[peer_id]["offer"]})
    else:
        return web.Response(status=404, text="Offer not found.")

async def get_answer(request):
    peer_id = request.query.get("peer_id")
    if peer_id in connected_peers and "answer" in connected_peers[peer_id]:
        return web.json_response({"answer": connected_peers[peer_id]["answer"]})
    else:
        return web.Response(status=404, text="Answer not found.")

app = web.Application()
app.add_routes([
    web.get('/', index),
    web.post('/offer', offer),
    web.post('/answer', answer),
    web.get('/offer', get_offer),
    web.get('/answer', get_answer),
])

if __name__ == '__main__':
    web.run_app(app, host='0.0.0.0', port=8080)
```

Notes:

    This server allows peers to post and retrieve offers and answers.
    In a production environment, you should implement authentication and clean up old session data.

3. Implement the Peer Script

Each peer will use the same script but in different modes (sender or receiver).
```python
# file_transfer.py

import argparse
import asyncio
import json
import os
import uuid
import aiohttp
from aiortc import RTCIceCandidate, RTCPeerConnection, RTCSessionDescription, DataChannel

SIGNALING_SERVER = 'http://<signaling_server_ip>:8080'

async def run_sender(file_path, peer_id):
    async with aiohttp.ClientSession() as session:
        # Create peer connection
        pc = RTCPeerConnection()

        channel = pc.createDataChannel('filetransfer')

        # File metadata
        file_name = os.path.basename(file_path)
        file_size = os.path.getsize(file_path)

        @channel.on('open')
        def on_open():
            print('Channel opened. Sending file metadata.')
            # Send file metadata first
            metadata = json.dumps({'file_name': file_name, 'file_size': file_size})
            channel.send(metadata)

            # Send file data
            asyncio.ensure_future(send_file(file_path, channel))

        @channel.on('close')
        def on_close():
            print('Channel closed.')
            asyncio.get_event_loop().stop()

        # Create offer
        await pc.setLocalDescription(await pc.createOffer())

        # Send offer to signaling server
        await session.post(f'{SIGNALING_SERVER}/offer', json={
            'peer_id': peer_id,
            'offer': pc.localDescription.sdp
        })

        # Wait for answer
        while True:
            async with session.get(f'{SIGNALING_SERVER}/answer', params={'peer_id': peer_id}) as resp:
                if resp.status == 200:
                    data = await resp.json()
                    answer = data['answer']
                    break
                await asyncio.sleep(1)

        await pc.setRemoteDescription(RTCSessionDescription(answer, 'answer'))

        # Keep the script running
        await asyncio.Event().wait()

async def run_receiver(save_path, peer_id):
    async with aiohttp.ClientSession() as session:
        # Create peer connection
        pc = RTCPeerConnection()

        incoming_channel = None

        @pc.on('datachannel')
        def on_datachannel(channel):
            nonlocal incoming_channel
            incoming_channel = channel

            @channel.on('message')
            async def on_message(message):
                if isinstance(message, str):
                    # Receive file metadata
                    metadata = json.loads(message)
                    file_name = metadata['file_name']
                    file_size = metadata['file_size']
                    full_path = os.path.join(save_path, file_name)
                    print(f"Receiving file: {file_name} ({file_size} bytes)")
                    channel.file = open(full_path, 'wb')
                    channel.received_bytes = 0
                else:
                    # Receive file data
                    channel.file.write(message)
                    channel.received_bytes += len(message)
                    print(f"Received {channel.received_bytes}/{file_size} bytes", end='\r')
                    if channel.received_bytes >= file_size:
                        channel.file.close()
                        print(f"\nFile received and saved as {full_path}")
                        await channel.close()
                        asyncio.get_event_loop().stop()

        # Get offer from signaling server
        while True:
            async with session.get(f'{SIGNALING_SERVER}/offer', params={'peer_id': peer_id}) as resp:
                if resp.status == 200:
                    data = await resp.json()
                    offer = data['offer']
                    break
                await asyncio.sleep(1)

        await pc.setRemoteDescription(RTCSessionDescription(offer, 'offer'))

        # Create answer
        await pc.setLocalDescription(await pc.createAnswer())

        # Send answer to signaling server
        await session.post(f'{SIGNALING_SERVER}/answer', json={
            'peer_id': peer_id,
            'answer': pc.localDescription.sdp
        })

        # Keep the script running
        await asyncio.Event().wait()

async def send_file(file_path, channel: DataChannel):
    with open(file_path, 'rb') as f:
        while True:
            data = f.read(16384)  # Read in chunks
            if not data:
                break
            channel.send(data)
    print("File transfer complete.")
    await channel.close()
    asyncio.get_event_loop().stop()

def main():
    parser = argparse.ArgumentParser(description="P2P File Transfer using WebRTC")
    parser.add_argument('mode', choices=['send', 'receive'], help='Mode: send or receive')
    parser.add_argument('--file', help='File to send')
    parser.add_argument('--save', default='.', help='Directory to save received file')
    parser.add_argument('--peer-id', help='Unique ID for the peer connection')
    parser.add_argument('--signaling-server', default=SIGNALING_SERVER, help='Signaling server URL')

    args = parser.parse_args()

    if args.peer_id:
        peer_id = args.peer_id
    else:
        peer_id = str(uuid.uuid4())
        print(f"Generated peer ID: {peer_id}")

    global SIGNALING_SERVER
    SIGNALING_SERVER = args.signaling_server

    if args.mode == 'send':
        if not args.file:
            print("Please specify a file to send using --file")
            return
        asyncio.run(run_sender(args.file, peer_id))
    elif args.mode == 'receive':
        asyncio.run(run_receiver(args.save, peer_id))

if __name__ == '__main__':
    main()
```

Notes:

    Peer ID: A unique identifier to match the sender and receiver. Both peers need to use the same peer_id.
    Signaling Server IP: Replace `<signaling_server_ip>` with the IP address where the signaling server is running.
    Data Channels: We use WebRTC data channels for transferring file data.

### 4. Run the Application

#### Step 1: Start the Signaling Server

On a machine with a public IP address (or accessible by both peers), run:

```bash
python signaling_server.py
```

#### Step 2: Determine the Peer ID

    Agree with your friend on a unique peer_id (e.g., a UUID or a shared secret).

#### Step 3: Sender (Peer A) - Send the File

On the sender's machine:

```bash
python file_transfer.py send --file path/to/large_file.zip --peer-id YOUR_PEER_ID --signaling-server http://<signaling_server_ip>:8080
```

Replace:

    `path/to/large_file.zip` with the path to the file you want to send.
    `YOUR_PEER_ID` with the agreed-upon peer ID.
    `<signaling_server_ip>` with the signaling server's IP address.

#### Step 4: Receiver (Peer B) - Receive the File

On the receiver's machine:

```bash
python file_transfer.py receive --save path/to/save/directory --peer-id YOUR_PEER_ID --signaling-server http://<signaling_server_ip>:8080
```

Replace:

    `path/to/save/directory` with the directory where you want to save the received file.
    `YOUR_PEER_ID` with the same peer ID used by the sender.
    `<signaling_server_ip>` with the signaling server's IP address.

## How It Works

    Signaling Phase:
        The sender creates a WebRTC offer and sends it to the signaling server along with the peer_id.
        The receiver periodically polls the signaling server for an offer matching the peer_id.
        Upon receiving the offer, the receiver creates an answer and sends it back to the signaling server.
        The sender retrieves the answer from the signaling server.

    Connection Establishment:
        The peers establish a direct connection using ICE, which handles NAT traversal.
        They exchange ICE candidates automatically via the signaling server.

    Data Transfer:
        The sender opens a data channel and sends file metadata (name and size).
        The sender reads the file in chunks and sends it over the data channel.
        The receiver writes the incoming data to a file.

## Important Considerations

1. NAT Traversal

    ICE Mechanism: WebRTC's ICE protocol tries multiple methods to establish a direct connection between peers.
    STUN Servers: aiortc uses Google's public STUN servers by default. For better reliability, consider setting up your own STUN server or specifying alternative servers.

2. Signaling Server Accessibility

    The signaling server must be accessible to both peers over the internet.
    If the server is behind a NAT, you need to set up port forwarding or host it on a VPS/cloud service.

3. Security

    Encryption: WebRTC data channels are encrypted using DTLS (Datagram Transport Layer Security).
    Authentication: Currently, any peer knowing the peer_id can connect. For enhanced security:
        Use a secure method to share the peer_id.
        Implement authentication tokens or secure the signaling server endpoints.

4. Firewalls

    Ensure that firewalls on both peers allow outbound UDP traffic and do not block the ports used by ICE.

## Testing the Application

    Test Locally:
        Run both the sender and receiver scripts on the same machine with different peer IDs to test functionality.

    Test Across Networks:
        Have the sender and receiver on different networks (e.g., home and mobile hotspot) to test NAT traversal.

## Enhancements for Production

    Persistent Signaling: Use WebSockets instead of HTTP polling for signaling to reduce latency.
    Error Handling: Add robust error handling and retry mechanisms.
    Progress Reporting: Implement better progress indicators and possibly resume capabilities.
    File Integrity: Include checksum verification to ensure file integrity after transfer.
    Multiple Files: Extend the application to support multiple file transfers or entire directories.
    User Interface: Develop a GUI using frameworks like Tkinter or PyQt for user-friendly interaction.

## Conclusion

By leveraging WebRTC and aiortc, we've created an MVP that overcomes NAT traversal issues, allowing for direct file transfers between two peers behind NATs. This solution provides encrypted, peer-to-peer communication without the need for external services like WeTransfer.

Feel free to ask if you need further assistance or have questions about specific parts of the implementation.