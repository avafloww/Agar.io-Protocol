# Agar.io Protocol

This file contains information about the client-server connection under protocols 4, 5, 6, 7 and 8.

## Data Types
Agar.io uses standard JavaScript DataView data types, for the most part. *Note that in contrast to standard DataView behavior, all multi-byte values are encoded across the wire as little endian.* This table is a reference of some of the commonly used data types.

| Data Type | Description 
|-----------|-----------
| uint8     | Unsigned 1 byte integer (byte)
| uint16    | Unsigned 2 byte integer (short)
| uint32    | Unsigned 4 byte integer (int)
| float32   | Signed 4 byte floating point value (float)
| float64   | Signed 8 byte floating point value (double)
| string    | Null-terminated UTF-16 (before protocol 6) or UTF-8 (after protocol 6)
| boolean   | uint32 where 0 = false, 1 = true

Each packet starts with a uint8 containing the packet ID.

## Clientbound Packets
### Packet 16: Update Nodes
Sent to the client by the server to update information about one or more nodes. Nodes to be destroyed are placed at the beginning of the node data list.

| Position | Data Type     | Description
|----------|---------------|-----------------
| 0        | uint8         | Packet ID
| 1        | uint16        | Number of nodes to be destroyed
| 2...?    | Destruct Data | Nodes' ID marked for destruction
| ?...?    | Node Data     | Data for all nodes
| ?        | uint32        | Always 0; terminates the node data listing
| ?        | uint16        | Always 0; discarded by the client
| ?        | uint32/uint16 | Number of nodes marked for destroying (uint16 at protocols 6+)
| ?...?    | Destruct Data | Node ID of each destroyed node (uint32)

Node data that is marked for destruction has a simple format:

| Offset | Data Type | Description
|--------|-----------|-------------------
| 0      | uint32    | Node ID of killing cell
| 4      | uint32    | Node ID of killed cell

#### Node Data
Each visible node is described by the following data. This data repeats n times at the end of the Update Nodes packet, where n is the number specified by position 1 in the packet (number of nodes). Nodes that are stationary (like food) are only sent **once** to the client. In additon, the name field of each node is sent **once**.

##### Protocol version 4/5
| Offset | Data Type  | Description
|--------|------------|-------------------
| 0      | uint32     | Node ID
| 4      | int16/int32| X position (int32 after late protocol 5)
| 8      | int16/int32| Y position (int32 after late protocol 5)
| 12     | uint16     | Radius of node
| 14     | uint8      | Color (Red component)
| 15     | uint8      | Color (Green component)
| 16     | uint8      | Color (Blue component)
| 17     | uint8      | Flags - see below
|        |            | Skip a specific number of bytes based on the flags field. See below.
| ?      | string     | Node name
| ?      | uint16     | End of string

##### Protocol versions 6+
| Offset | Data Type | Description
|--------|-----------|-------------------
| 0      | uint32    | Node ID
| 4      | int32     | X position
| 8      | int32     | Y position
| 12     | uint16    | Radius of node
| 17     | uint8     | Flags - see below
| 14     | uint8     | Red component (flags determined)
| 15     | uint8     | Green component (flags determined)
| 16     | uint8     | Blue component (flags determined)
| 17     | string    | Skin name (flags determined)
| ?      | uint8     | End of string
| ?      | string    | Node name (flags determined)
| ?      | uint8     | End of string

The flags field is 1 byte in length, and is a bitfield. Here's a table describing the known behaviors of setting specific flags:

##### Protocols older than v6
| Bit | Behavior
|-----|------------------
| 0   | If set, the node is a virus
| 1   | Advance offset after flags by 4 bytes
| 2   | Advance offset after flags by 8 bytes
| 3   | Advance offset after flags by 16 bytes
| 4   | Agitated Virus
##### Protocols v6 and newer
| Bit | Behavior
|-----|------------------
| 1   | Is virus
| 2   | Color present
| 4   | Skin present
| 8   | Name present
| 16  | Agitated cell
| 32  | Is ejected cell

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

### Packet 21: Draw Line
Draws a line from all the player cells to the specified position.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint16    | X position
| 3        | uint16    | Y position

### Packet 32: Add Node
Adds a node to the player's screen. Nodes that are added by this packet are centered on by the client's camera. Probably used when splitting cells/spawning in.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Node ID

### Packet 49: Update Leaderboard (FFA)
Updates the leaderboard on the client's screen.

#### Protocols before v6
| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | The following repeats the number of times specified by this field.
| ?        | uint32    | One of player's cell node ID
| ?        | string    | Player's name
#### Protocols after v6
| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | The following repeats the number of times specified by this field.
| ?        | boolean   | Is me flag (0 - no, 1 - yes)
| ?        | string    | Player's name

### Packet 50: Update Leaderboard (Team)
Updates the leaderboard on the client's screen. Team score is the percentage of the total mass in game that the team has.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Amount of teams
| ?        | float32   | Team score

### Packet 64: Set Border
Sets the map border.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | float64   | Left position
| 9        | float64   | Top position
| 17       | float64   | Right position
| 25       | float64   | Bottom position

### Packet 255: Compressed packet
Seen as an envelope of compressed packets 16 (Update nodes) and 64 (Set border). The compression uses LZ4 algorithm to compress data.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1...?    | bytes     | Compressed data

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
X and Y positions' data types:
- float64 at protocol 4
- int16 at early protocol 5
- int32 at late protocol 5 and newer

| Position | Data Type            | Description
|----------|----------------------|-----------------
| 0        | uint8                | Packet ID
| 1        | protocol-dependent   | Absolute mouse X on canvas
| ?        | protocol-dependent   | Absolute mouse Y on canvas
| ?        | uint32               | Node ID of the cell to control. Earlier used to move only specified cells.

### Packet 17: Split
Splits the player's cell.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 18: Key Q Pressed
Sent when the player presses Q.  
In early iterations, was used to bring your cells closer together after spreading them apart. Now is used to enable free-roam mode while spectating.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 19: Key Q Released
Sent when the player releases Q. See packet 18.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 21: Eject Mass
Ejects mass from the player's cell.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

### Packet 80: Send Token
Sent at the beginning of a connection, after packet 255.  
Used to authenticate with a one-use token issued by the load balancer.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | string    | Token. 8 characters, alpha-numeric, and some symbols. (uint8)

### Packet 254: Reset Connection 1
Sent at the beginning of a connection, before packet 255.
The server is obligated to send ClearNodes and SetBorder packets to ensure client doesn't drop the connection.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Protocol version. Currently 8 as of vanilla 7/2/2016.

### Packet 255: Reset Connection 2
Sent at the beginning of a connection, after packet 254.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Protocol version. Currently 2838145714 as of 7/2/2016. Unsure if changes.

