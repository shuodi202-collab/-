## ADDED Requirements

### Requirement: Send Text Messages
The system SHALL allow a user to send text messages to all connected peers.

#### Scenario: Send message
- **WHEN** user types text in the message input field and presses Enter or clicks "Send"
- **THEN** the system SHALL send the message via DataChannel to all connected peers and SHALL display it in the local message list as sent by the current user

#### Scenario: Empty message not sent
- **WHEN** user clicks "Send" with an empty or whitespace-only message
- **THEN** the system SHALL NOT send the message

### Requirement: Receive Text Messages
The system SHALL display messages received from connected peers in real-time.

#### Scenario: Receive and display message
- **WHEN** a message is received via DataChannel from a peer
- **THEN** the system SHALL display the message in the message list showing: sender nickname, message text, and timestamp

### Requirement: Message Format
Messages SHALL be transmitted as JSON strings with a defined structure.

#### Scenario: Message JSON structure
- **WHEN** any message is sent or received
- **THEN** the message payload SHALL follow this structure: `{ type: "text", from: "<nickname>", text: "<content>", id: "<uuid>", timestamp: <unix_ms> }`

### Requirement: System Messages
The system SHALL display non-chat system events (join, leave, connection changes) in the message list.

#### Scenario: Display system message
- **WHEN** a user joins or leaves, or a connection state changes
- **THEN** the system SHALL display a non-editable system message in the message list with a distinct visual style (e.g., gray italic, centered)

### Requirement: Message History
The system SHALL persist the last 200 messages in localStorage so they survive page refreshes.

#### Scenario: Persist messages
- **WHEN** the page is refreshed
- **THEN** the system SHALL reload the last 200 messages from localStorage and display them in the message list

### Requirement: Message Scroll Behavior
The system SHALL auto-scroll to the newest message when new messages arrive, unless the user has scrolled up to read history.

#### Scenario: Auto-scroll to latest
- **WHEN** a new message arrives and the user is at the bottom of the scroll area (within 50px of bottom)
- **THEN** the system SHALL auto-scroll to show the newest message

#### Scenario: Stay on history when scrolled up
- **WHEN** a new message arrives and the user has scrolled up more than 50px from the bottom
- **THEN** the system SHALL NOT auto-scroll, and SHALL show a "New messages" indicator
