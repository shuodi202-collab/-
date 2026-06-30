## ADDED Requirements

### Requirement: Create PeerConnection
The system SHALL create an RTCPeerConnection with appropriate ICE servers (STUN) when initiating or accepting a connection.

#### Scenario: Create connection with STUN servers
- **WHEN** user creates a new room or joins an existing room
- **THEN** the system SHALL create an RTCPeerConnection with `stun:stun.l.google.com:19302` and `stun:stun1.l.google.com:19302` as ICE servers

### Requirement: Manage DataChannel
The system SHALL create a reliable, ordered RTCDataChannel named `chat` for each peer connection to transmit messages.

#### Scenario: Open DataChannel on new connection
- **WHEN** a new RTCPeerConnection is established
- **THEN** the system SHALL open an RTCDataChannel labeled `chat` with `ordered: true` and `negotiated: false`

#### Scenario: Receive DataChannel from remote peer
- **WHEN** a remote peer opens a DataChannel
- **THEN** the system SHALL listen to the `ondatachannel` event and register the channel for message sending

### Requirement: Maintain Mesh Topology
The system SHALL maintain a peer-to-peer mesh where each participant connects directly to every other participant.

#### Scenario: New peer joins existing room of 3
- **WHEN** a fourth peer joins a room with 3 existing peers
- **THEN** the new peer SHALL establish a PeerConnection with each of the 3 existing peers, and each existing peer SHALL create a new PeerConnection for the new peer

#### Scenario: Peer disconnects
- **WHEN** a peer disconnects (connection state changes to `disconnected` or `failed`)
- **THEN** the system SHALL remove that peer from the member list and SHALL clean up all associated PeerConnection resources

### Requirement: Connection State Feedback
The system SHALL expose real-time connection state changes (connecting, connected, disconnected, failed) for each peer in the UI.

#### Scenario: Display connection states
- **WHEN** a peer's connection state changes
- **THEN** the UI SHALL display the updated state next to the peer's name (e.g., "connecting...", "connected", "disconnected")

### Requirement: Automatic Reconnection
The system SHALL attempt to re-establish a failed peer connection once.

#### Scenario: Handle ICE restart
- **WHEN** a connection fails and the peer is still in the member list
- **THEN** the system SHALL attempt an ICE restart once, and if it fails again SHALL mark the peer as disconnected
