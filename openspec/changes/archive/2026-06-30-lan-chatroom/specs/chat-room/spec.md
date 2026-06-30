## ADDED Requirements

### Requirement: Create Room
The system SHALL allow a user to create a new chat room by entering a nickname.

#### Scenario: Create new room
- **WHEN** user enters a nickname and clicks "Create Room"
- **THEN** the system SHALL generate an invite code and display it, with the user listed as the room creator

### Requirement: Join Room
The system SHALL allow a user to join an existing room by pasting an invite code and entering a nickname.

#### Scenario: Join room by invite code
- **WHEN** user pastes a valid invite code, enters a nickname, and clicks "Join"
- **THEN** the system SHALL establish peer connections with existing members and display the chat interface

### Requirement: Display Member List
The system SHALL display a list of all currently connected members with their nicknames and connection status.

#### Scenario: Show online members
- **WHEN** the chat interface is active
- **THEN** the system SHALL display a member list showing each participant's nickname with a connection status indicator (connected/disconnected)

#### Scenario: Member joins
- **WHEN** a new peer successfully establishes a connection
- **THEN** the system SHALL add the new member to the member list and SHALL broadcast a system message "[nickname] joined the room"

#### Scenario: Member leaves
- **WHEN** a peer disconnects
- **THEN** the system SHALL remove the member from the list and SHALL broadcast a system message "[nickname] left the room"

### Requirement: Nickname Uniqueness
The system SHALL warn if multiple peers share the same nickname.

#### Scenario: Duplicate nickname warning
- **WHEN** a peer connects with a nickname that matches an existing member
- **THEN** the system SHALL display a notice in the chat area (non-blocking) indicating the duplicate nickname
