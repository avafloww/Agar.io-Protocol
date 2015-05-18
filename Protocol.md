# Agar.io Protocol

## Data Types
Agar.io uses standard JavaScript DataView data types, for the most part. *Note that in contrast to standard DataView behavior, all multi-byte values are encoded across the wire as little endian.* This table is a reference of some of the commonly used data types.

| Data Type | Description 
|-----------|-----------
| uint8     | Unsigned 1 byte integer
| uint16    | Unsigned 2 byte integer
| uint32    | Unsigned 4 byte integer
| float64   | Signed 8 byte floating point value
| string    | UTF-16 string (2 bytes per character)

Each packet starts with a uint8 containing the packet ID.

## Packet 0: Set Nickname
Sets the player's nickname.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | string    | Nickname

## Packet 1: Spectate
Puts the player in spectator mode.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

## Packet 16: Mouse Move
Sent when the player's mouse moves.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | float64   | Mouse X on canvas (absolute position?)
| 9        | float64   | Mouse Y on canvas (absolute position?)
| 17       | uint32    | Unknown, always 0 in vanilla

## Packet 17: Split
Splits the player's cell.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

## Packet 18: Key Q Pressed
Sent when the player presses Q. The use for this packet (or the capture of the Q key) is unknown.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

## Packet 19: Key Q Released
Sent when the player releases Q. The use for this packet (or the capture of the Q key) is unknown.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

## Packet 21: Eject Mass
Ejects mass from the player's cell.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID

## Packet 255: Reset Connection
Called at the beginning of a connection.

| Position | Data Type | Description
|----------|-----------|-----------------
| 0        | uint8     | Packet ID
| 1        | uint32    | Unknown, always 1 in vanilla

