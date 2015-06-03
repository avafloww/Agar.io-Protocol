# Agar.io Protocol

Note that a lot of the documentation here is just guesswork for the time being. As work progresses on Ogar, my open-source Agar.io server implementation (creative name, I know), this documentation will be updated further.

## Data Types
Agar.io uses standard JavaScript DataView data types, for the most part. *Note that in contrast to standard DataView behavior, all multi-byte values are encoded across the wire as little endian.* This table is a reference of some of the commonly used data types.

| Data Type | Description 
|-----------|-----------
| uint8     | Unsigned 1 byte integer (byte)
| uint16    | Unsigned 2 byte integer (short)
| uint32    | Unsigned 4 byte integer (int)
| float32   | Signed 4 byte floating point value (float)
| float64   | Signed 8 byte floating point value (double)
| string    | Null-terminated UTF-16 string (2 bytes per character)
| boolean   | uint8 where 0 = false, 1 = true

Each packet starts with a uint8 containing the packet ID.

## Clientbound Packets
### Packet 16: Update Nodes
Sent to the client by the server to update information about one or more nodes. Nodes to be destroyed are placed at the beginning of the node data list.

| Position | Data Type     | Description
|----------|---------------|-----------------
| 0        | uint8         | Packet ID
| 1        | uint16        | Number of nodes to be destroyed
| 3...?    | Node Data     | Data for all nodes
| ?        | uint32        | Always 0; terminates the node data listing
| ?        | uint16        | Always 0; discarded by the client
| ?        | uint32        | Number of nodes marked for destroying
| ?...?    | uint32        | Node ID of each destroyed node

#### Node Data
Each visible node is described by the following data. This data repeats n times at the end of the Update Nodes packet, where n is the number specified by position 1 in the packet (number of nodes).

| Offset | Data Type | Description
|--------|-----------|-------------------
| 0      | uint32    | Node ID
| 4      | uint16    | X position
| 6      | uint16    | Y position
| 8      | uint16    | Radius of node
| 10     | uint8     | Color (Red component)
| 11     | uint8     | Color (Green component)
| 12     | uint8     | Color (Blue component)
| 13     | uint8     | Flags - see below
|        |           | Skip a specific number of bytes based on the flags field. See below.
| ?      | string    | Node name
| ?      | uint16    | End of string

The flags field is 1 byte in length, and is a bitfield. If no flag that specifies the offset is set, 0 bytes will be skipped. Here's a table describing the known behaviors of setting specific flags:

| Bit | Behavior
|-----|------------------
| 1   | If set, the node is a virus
| 2   | Advance offset after flags by 4 bytes
| 5   | Agitated Virus
| 4   | Advance offset after flags by 8 bytes
| 8   | Advance offset after flags by 16 bytes

Node data that is marked for destruction has a simpler format:

| Offset | Data Type | Description
|--------|-----------|-------------------
| 0      | uint32    | Node ID of killing cell
| 4      | uint32    | Node ID of killed cell

### Packet 17: Update Position and Size
Updates the position and size of the player. Used when spectating.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | float32   | X position
| 5        | float32   | Y position
| 9        | float32   | Zoom factor of client

### Packet 20: Clear All Nodes
Clears all nodes off of the player's screen.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 32: Add Node
Adds a node to the player's screen. Nodes that are added by this packet are centered on by the client's camera. Probably used when splitting cells/spawning in.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Node ID

### Packet 49: Update Leaderboard (FFA)
Updates the leaderboard on the client's screen.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | The following repeats the number of times specified by this field.
| ?        | uint32    | Node ID
| ?        | string    | Node name

### Packet 50: Update Leaderboard (Team)
Updates the leaderboard on the client's screen.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Amount of teams
| ?        | float32   | Team data

### Packet 64: Set Border
Sets the map border.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | float64   | Left position
| 9        | float64   | Top position
| 17       | float64   | Right position
| 25       | float64   | Bottom position

## Serverbound Packets
### Packet 0: Set Nickname
Sets the player's nickname.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | string    | Nickname

### Packet 1: Spectate
Puts the player in spectator mode.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 16: Mouse Move
Sent when the player's mouse moves.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | float64   | Absolute mouse X on canvas
| 9        | float64   | Absolute mouse Y on canvas
| 17       | uint32    | Unknown, always 0 in vanilla

### Packet 17: Split
Splits the player's cell.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 18: Key Q Pressed
Sent when the player presses Q. The use for this packet (or the capture of the Q key) is unknown.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 19: Key Q Released
Sent when the player releases Q. The use for this packet (or the capture of the Q key) is unknown.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 21: Eject Mass
Ejects mass from the player's cell.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 255: Reset Connection
Called at the beginning of a connection.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Unknown, always 1 in vanilla

