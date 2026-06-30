## ADDED Requirements

### Requirement: Generate SDP Offer
The system SHALL generate an SDP offer as the "room creator" and encode it as a portable "invite code" string.

#### Scenario: Create invite code
- **WHEN** user clicks "Create Room" and enters a nickname
- **THEN** the system SHALL create an RTCPeerConnection, generate an SDP offer, base64-encode the SDP (as `btoa(JSON.stringify({sdp, type, roomName, version}))`), and display the resulting string as the "Invite Code"

#### Scenario: Display invite code
- **WHEN** the invite code is generated
- **THEN** the system SHALL display the invite code in a clearly labeled text area with a "Copy" button

### Requirement: Parse and Accept Offer
The system SHALL accept a pasted invite code, decode the SDP offer, create a local peer connection, and generate an SDP answer.

#### Scenario: Join room by pasting invite code
- **WHEN** user pastes an invite code into the "Join Room" input and clicks "Join"
- **THEN** the system SHALL base64-decode the code, extract the SDP offer, set it as remote description on a new PeerConnection, generate an SDP answer, and display the answer as a "Reply Code"

#### Scenario: Invalid invite code format
- **WHEN** user pastes text that is not valid base64 or does not contain valid SDP JSON
- **THEN** the system SHALL display an error message "Invalid invite code" and SHALL NOT attempt to create a connection

### Requirement: Complete Handshake with Answer
The room creator SHALL accept a reply code to complete the WebRTC handshake.

#### Scenario: Apply answer to complete connection
- **WHEN** the room creator pastes a reply code from a joiner
- **THEN** the system SHALL base64-decode the code, set the SDP answer as remote description, and complete the WebRTC handshake

### Requirement: Multi-Peer Signaling
When a new peer joins an existing room with multiple members, the system SHALL facilitate pairwise signaling with each existing member.

#### Scenario: Join room with multiple peers
- **WHEN** a new peer's invite code is processed in a room with N existing peers
- **THEN** the system SHALL create N separate peer connections, one for each existing peer
