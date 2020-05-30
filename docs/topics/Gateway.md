# Gateways

Gateways are Discord's form of real-time communication over secure WebSockets. Clients will receive events and data over the gateway they are connected to and send data over the REST API. The API for interacting with Gateways is complex and fairly unforgiving, therefore it's highly recommended you read _all_ of the following documentation before writing a custom implementation.

The Discord Gateway has a versioning system separate from the HTTP APIs. The documentation herein is only for the latest version in the following table, unless otherwise specified.

Important note: Not all event fields are documented, in particular, fields prefixed with an underscore are considered _internal fields_ and should not be relied on. We may change the format at any time.

###### Gateway Versions

| Version | Status       |
| ------- | ------------ |
| 6       | Available    |
| 5       | Discontinued |
| 4       | Discontinued |

## Payloads

###### Gateway Payload Structure

| Field | Type                    | Description                                                                     |
| ----- | ----------------------- | ------------------------------------------------------------------------------- |
| op    | integer                 | [opcode](#DOCS_TOPICS_OPCODES_AND_STATUS_CODES/gateway-opcodes) for the payload |
| d     | ?mixed (any JSON value) | event data                                                                      |
| s     | ?integer \*             | sequence number, used for resuming sessions and heartbeats                      |
| t     | ?string \*              | the event name for this payload                                                 |

\* `s` and `t` are `null` when `op` is not `0` (Gateway Dispatch Opcode).

### Sending Payloads

Packets sent from the client to the Gateway API are encapsulated within a [gateway payload object](#DOCS_TOPICS_GATEWAY/sending-payloads-example-gateway-dispatch) and must have the proper opcode and data object set. The payload object can then be serialized in the format of choice (see [ETF/JSON](#DOCS_TOPICS_GATEWAY/etfjson)), and sent over the websocket. Payloads to the gateway are limited to a maximum of 4096 bytes sent, going over this will cause a connection termination with error code 4002.

###### Example Gateway Dispatch

```json
{
  "op": 0,
  "d": {},
  "s": 42,
  "t": "GATEWAY_EVENT_NAME"
}
```

### Receiving Payloads

Receiving payloads with the Gateway API is slightly more complex than sending. When using the JSON encoding with compression enabled, the Gateway has the option of sending payloads as compressed JSON binaries using zlib, meaning your library _must_ detect (see [RFC1950 2.2](https://tools.ietf.org/html/rfc1950#section-2.2)) and decompress these payloads before attempting to parse them. The gateway does not implement a shared compression context between messages sent.

## Encoding and Compression

#### ETF/JSON

When initially creating and handshaking connections to the Gateway, a user can choose whether they wish to communicate over plain-text JSON or binary [ETF](https://erlang.org/doc/apps/erts/erl_ext_dist.html). When using ETF, the client must not send compressed messages to the server. Note that Snowflake IDs are transmitted as 64-bit integers over ETF, but are transmitted as strings over JSON. See [erlpack](https://github.com/discord/erlpack) for an implementation example.

#### Payload Compression

When using JSON encoding with payload compression enabled (`compress: true` in identify), the Gateway may optionally send zlib-compressed payloads (see [RFC1950 2.2](https://tools.ietf.org/html/rfc1950#section-2.2)). Your library _must_ detect and decompress these payloads to plain-text JSON before attempting to parse them. If you are using payload compression, the gateway does not implement a shared compression context between messages sent. Payload compression will be disabled if you use transport compression (see below).

#### Transport Compression

Currently the only available transport compression option is `zlib-stream`. You will need to run all received packets through a shared zlib context, as seen in the example below. Every connection to the gateway should use its own unique zlib context.

###### Transport Compression Example

```python
# Z_SYNC_FLUSH suffix
ZLIB_SUFFIX = b'\x00\x00\xff\xff'
# initialize a buffer to store chunks
buffer = bytearray()
# create a zlib inflation context to run chunks through
inflator = zlib.decompressobj()

# ...
def on_websocket_message(msg):
  # always push the message data to your cache
  buffer.extend(msg)

  # check if the last four bytes are equal to ZLIB_SUFFIX
  if len(msg) < 4 or msg[-4:] != ZLIB_SUFFIX:
    return

  # if the message *does* end with ZLIB_SUFFIX,
  # get the full message by decompressing the buffers
  # NOTE: the message is utf-8 encoded.
  msg = inflator.decompress(buffer)
  buffer = bytearray()

  # here you can treat `msg` as either JSON or ETF encoded,
  # depending on your `encoding` param
```

## Connecting to the Gateway

### Connecting

###### Gateway URL Params

| Field     | Type    | Description                                   | Accepted Values                                                            |
| --------- | ------- | --------------------------------------------- | -------------------------------------------------------------------------- |
| v         | integer | Gateway Version to use                        | 6 (see [Gateway versions](#DOCS_TOPICS_GATEWAY/gateways-gateway-versions)) |
| encoding  | string  | The encoding of received gateway packets      | 'json' or 'etf'                                                            |
| compress? | string  | The (optional) compression of gateway packets | 'zlib-stream'                                                              |

The first step in establishing connectivity to the gateway is requesting a valid websocket endpoint from the API. This can be done through either the [Get Gateway](#DOCS_TOPICS_GATEWAY/get-gateway) or the [Get Gateway Bot](#DOCS_TOPICS_GATEWAY/get-gateway-bot) endpoint.

With the resulting payload, you can now open a websocket connection to the "url" (or endpoint) specified. Generally, it is a good idea to explicitly pass the gateway version and encoding. For example, we may connect to `wss://gateway.discord.gg/?v=6&encoding=json`.

Once connected, the client should immediately receive an [Opcode 10 Hello](#DOCS_TOPICS_GATEWAY/hello) payload, with information on the connection's heartbeat interval:

###### Example Gateway Hello

```json
{
  "op": 10,
  "d": {
    "heartbeat_interval": 45000
  }
}
```

### Heartbeating

The client should now begin sending [Opcode 1 Heartbeat](#DOCS_TOPICS_GATEWAY/heartbeat) payloads every `heartbeat_interval` milliseconds, until the connection is eventually closed or terminated. This OP code is also bidirectional. The gateway may request a heartbeat from you in some situations, and you should send a heartbeat back to the gateway as you normally would.

> info
> In the event of a service outage where you stay connected to the gateway, you should continue to heartbeat and receive ACKs. The gateway will eventually respond and issue a session once it's able to.

Clients can detect zombied or failed connections by listening for [Opcode 11 Heartbeat ACK](#DOCS_TOPICS_GATEWAY/heartbeating-example-gateway-heartbeat-ack):

###### Example Gateway Heartbeat ACK

```json
{
  "op": 11
}
```

If a client does not receive a heartbeat ack between its attempts at sending heartbeats, it should immediately terminate the connection with a non-1000 close code, reconnect, and attempt to resume.

### Identifying

Next, the client is expected to send an [Opcode 2 Identify](#DOCS_TOPICS_GATEWAY/identify):

###### Example Gateway Identify

This is a minimal `IDENTIFY` payload. `IDENTIFY` supports additional optional fields for other session properties, such as payload compression, or an initial presence state. See the [Identify Structure](#DOCS_TOPICS_GATEWAY/identify) for a more complete example of all options you can pass in.

```json
{
  "op": 2,
  "d": {
    "token": "my_token",
    "properties": {
      "$os": "linux",
      "$browser": "my_library",
      "$device": "my_library"
    }
  }
}
```

If the payload is valid, the gateway will respond with a [Ready](#DOCS_TOPICS_GATEWAY/ready) event. Your client is now considered in a "connected" state. Clients are limited to 1 identify every 5 seconds; if they exceed this limit, the gateway will respond with an [Opcode 9 Invalid Session](#DOCS_TOPICS_GATEWAY/invalid-session). It is important to note that although the ready event contains a large portion of the required initial state, some information (such as guilds and their members) is sent asynchronously (see [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) event).

> warn
> Clients are limited to 1000 `IDENTIFY` calls to the websocket in a 24-hour period. This limit is global and across all shards, but does not include `RESUME` calls. Upon hitting this limit, all active sessions for the bot will be terminated, the bot's token will be reset, and the owner will receive an email notification. It's up to the owner to update their application with the new token.

## Resuming

The Internet is a scary place. Disconnections happen, especially with persistent connections. Due to Discord's architecture, this is a semi-regular event and should be expected and handled. Discord has a process for "resuming" (or reconnecting) a connection that allows the client to replay any lost events from the last sequence number they received in the exact same way they would receive them normally.

Your client should store the `session_id` from the [Ready](#DOCS_TOPICS_GATEWAY/ready), and the sequence number of the last event it received. When your client detects that it has been disconnected, it should completely close the connection and open a new one (following the same strategy as [Connecting](#DOCS_TOPICS_GATEWAY/connecting)). Once the new connection has been opened, the client should send a [Gateway Resume](#DOCS_TOPICS_GATEWAY/resume):

###### Example Gateway Resume

```json
{
  "op": 6,
  "d": {
    "token": "my_token",
    "session_id": "session_id_i_stored",
    "seq": 1337
  }
}
```

If successful, the gateway will respond by replaying all missed events in order, finishing with a [Resumed](#DOCS_TOPICS_GATEWAY/resumed) event to signal replay has finished, and all subsequent events are new. It's also possible that your client cannot reconnect in time to resume, in which case the client will receive a [Opcode 9 Invalid Session](#DOCS_TOPICS_GATEWAY/invalid-session) and is expected to wait a random amount of time—between 1 and 5 seconds—then send a fresh [Opcode 2 Identify](#DOCS_TOPICS_GATEWAY/identify).

### Disconnections

If the gateway ever issues a disconnect to your client, it will provide a close event code that you can use to properly handle the disconnection. A full list of these close codes can be found in the [Response Codes](#DOCS_TOPICS_OPCODES_AND_STATUS_CODES/gateway-close-event-codes) documentation.

## Gateway Intents

> info
> Intents are optionally supported on the v6 gateway. They will become mandatory in a future gateway version.

Maintaining a stateful application can be difficult when it comes to the amount of data you're expected to process, especially at scale. Gateway Intents are a system to help you lower that computational burden.

When [identifying](#DOCS_TOPICS_GATEWAY/identifying) to the gateway, you can specify an `intents` parameter which allows you to conditionally subscribe to pre-defined "intents", groups of events defined by Discord. If you do not specify a certain intent, you will not receive any of the gateway events that are batched into that group. The valid intents are:

### List of Intents

```
GUILDS (1 << 0)
  - GUILD_CREATE
  - GUILD_UPDATE
  - GUILD_DELETE
  - GUILD_ROLE_CREATE
  - GUILD_ROLE_UPDATE
  - GUILD_ROLE_DELETE
  - CHANNEL_CREATE
  - CHANNEL_UPDATE
  - CHANNEL_DELETE
  - CHANNEL_PINS_UPDATE

GUILD_MEMBERS (1 << 1)
  - GUILD_MEMBER_ADD
  - GUILD_MEMBER_UPDATE
  - GUILD_MEMBER_REMOVE

GUILD_BANS (1 << 2)
  - GUILD_BAN_ADD
  - GUILD_BAN_REMOVE

GUILD_EMOJIS (1 << 3)
  - GUILD_EMOJIS_UPDATE

GUILD_INTEGRATIONS (1 << 4)
  - GUILD_INTEGRATIONS_UPDATE

GUILD_WEBHOOKS (1 << 5)
  - WEBHOOKS_UPDATE

GUILD_INVITES (1 << 6)
  - INVITE_CREATE
  - INVITE_DELETE

GUILD_VOICE_STATES (1 << 7)
  - VOICE_STATE_UPDATE

GUILD_PRESENCES (1 << 8)
  - PRESENCE_UPDATE

GUILD_MESSAGES (1 << 9)
  - MESSAGE_CREATE
  - MESSAGE_UPDATE
  - MESSAGE_DELETE
  - MESSAGE_DELETE_BULK

GUILD_MESSAGE_REACTIONS (1 << 10)
  - MESSAGE_REACTION_ADD
  - MESSAGE_REACTION_REMOVE
  - MESSAGE_REACTION_REMOVE_ALL
  - MESSAGE_REACTION_REMOVE_EMOJI

GUILD_MESSAGE_TYPING (1 << 11)
  - TYPING_START

DIRECT_MESSAGES (1 << 12)
  - CHANNEL_CREATE
  - MESSAGE_CREATE
  - MESSAGE_UPDATE
  - MESSAGE_DELETE
  - CHANNEL_PINS_UPDATE

DIRECT_MESSAGE_REACTIONS (1 << 13)
  - MESSAGE_REACTION_ADD
  - MESSAGE_REACTION_REMOVE
  - MESSAGE_REACTION_REMOVE_ALL
  - MESSAGE_REACTION_REMOVE_EMOJI

DIRECT_MESSAGE_TYPING (1 << 14)
  - TYPING_START
```

### Caveats

Any [events not defined in an intent](#DOCS_TOPICS_GATEWAY/commands-and-events-gateway-events) are considered "passthrough" and will always be sent to you.

[Guild Member Update](#DOCS_TOPICS_GATEWAY/guild-member-update) is sent for current-user updates regardless of whether the `GUILD_MEMBERS` intent is set.

[Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) and [Request Guild Members](#DOCS_TOPICS_GATEWAY/request-guild-members) are uniquely affected by intents. See these sections for more information.

If you specify an `intent` value in your `IDENTIFY` payload that is *invalid*, the socket will close with a [`4013` close code](#DOCS_TOPICS_OPCODES_AND_STATUS_CODES/gateway-gateway-close-event-codes). An invalid intent is one that is not meaningful and not documented above.

If you specify an `intent` value in your `IDENTIFY` payload that is *disallowed*, the socket will close with a [`4014` close code](#DOCS_TOPICS_OPCODES_AND_STATUS_CODES/gateway-gateway-close-event-codes). A disallowed intent is one which you have not enabled for your bot or one that your bot is not whitelisted to use.

### Privileged Intents

Some intents are defined as "Privileged" due to the sensitive nature of the data. Those intents are:

- `GUILD_PRESENCES`
- `GUILD_MEMBERS`

In order to specify these intents in your `IDENTIFY` payload, you must first go to your application in the Developer Portal and enable the toggle for the Privileged Intents you wish to use.

In the future, access to Privileged Intents will come with restrictions and limitations. [Read more here.](https://github.com/discord/discord-api-docs/issues/1363)

## Rate Limiting

Clients are allowed 120 events every 60 seconds, meaning you can send on average at a rate of up to 2 events per second. Clients who surpass this limit are immediately disconnected from the Gateway, and similarly to the HTTP API, repeat offenders will have their API access revoked. Clients are also limited to one gateway connection per 5 seconds. If you hit this limit, the Gateway will respond with an [Opcode 9 Invalid Session](#DOCS_TOPICS_GATEWAY/invalid-session).

## Tracking State

Most of a client's state is provided during the initial [Ready](#DOCS_TOPICS_GATEWAY/ready) event and the [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) events that immediately follow. As objects are further created/updated/deleted, other events are sent to notify the client of these changes and to provide the new or updated data. To avoid excessive API calls, Discord expects clients to locally cache as many _relevant_ object states as possible, and to update them as gateway events are received.

An example of state tracking can be found with member status caching. When initially connecting to the gateway, the client receives information regarding the online status of guild members (online, idle, dnd, offline). To keep this state updated, a client must track and parse [Presence Update](#DOCS_TOPICS_GATEWAY/presence-update) events as they are received, and apply the provided data to the cached member objects.

For larger bots, client state can grow to be quite large. We recommend only storing objects in memory that are needed for a bot's operation. Many bots, for example, just respond to user input through chat commands. These bots may only need to keep guild information (like guild/channel roles and permissions) in memory, since [MESSAGE_CREATE](#DOCS_TOPICS_GATEWAY/message-create) and [MESSAGE_UPDATE](#DOCS_TOPICS_GATEWAY/message-update) events have the full member object available.

## Guild Subscriptions

> info
> While not deprecated, Guild Subscriptions have been superceded by [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents). You may continue to use guild subscriptions, but we recommend migrating to intents for even better results.

Presence and typing events get dispatched from guilds that your bot is a member of. For many bots, these events are not useful and can be frequent and expensive to process at scale. Because of this, we allow bots to opt out of guild subscriptions by setting `guild_subscriptions` to `false` when [Identify](#DOCS_TOPICS_GATEWAY/identify)ing.

## Guild Availability

When connecting to the gateway as a bot user, guilds that the bot is a part of will start out as unavailable. Don't fret! The gateway will automatically attempt to reconnect on your behalf. As guilds become available to you, you will receive [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) events.

## Sharding

As bots grow and are added to an increasing number of guilds, some developers may find it necessary to break or split portions of their bots operations into separate logical processes. As such, Discord gateways implement a method of user-controlled guild sharding which allows for splitting events across a number of gateway connections. Guild sharding is entirely user controlled, and requires no state-sharing between separate connections to operate.

To enable sharding on a connection, the user should send the `shard` array in the [Identify](#DOCS_TOPICS_GATEWAY/identify) payload. The first item in this array should be the zero-based integer value of the current shard, while the second represents the total number of shards. DMs will only be sent to shard 0. To calculate what events will be sent to what shard, the following formula can be used:

###### Sharding Formula

```python
(guild_id >> 22) % num_shards == shard_id
```

As an example, if you wanted to split the connection between three shards, you'd use the following values for `shard` for each connection: `[0, 3]`, `[1, 3]`, and `[2, 3]`. Note that only the first shard (`[0, 3]`) would receive DMs.

Note that `num_shards` does not relate to, or limit, the total number of potential sessions—it is only used for *routing* traffic. As such, sessions do not have to be identified in an evenly distributed manner when sharding. You can establish multiple sessions with the same `[shard_id, num_shards]`, or sessions with different `num_shards` values. This allows you to create sessions that will handle more or less traffic than others for more fine-tuned load balancing, or orchestrate "zero-downtime" scaling/updating by handing off traffic to a new deployment of sessions with a higher or lower `num_shards` count that are prepared in parallel.

## Sharding for Very Large Bots

If you own a bot that is in over 250,000 guilds, there are some additional considerations you must take around sharding. Please file a support-ticket to get moved to the sharding for big bots, when you reach this amount of servers. You can contact the discord support using [https://dis.gd/contact](https://dis.gd/contact).

The number of shards you run must be a multiple of a fixed number we will determine when reaching out to you. If you attempt to start your bot with an invalid number of shards, your websocket connection will close with a 4010 Invalid Shard opcode. The gateway bot bootstrap endpoint will return the correct amount of shards, so if you're already using this endpoint to determine your number of shards, you shouldn't require any further changes.

The session start limit for these bots will also be increased from 1000 to 2000 per day. Finally, the [Get Current User Guilds](#DOCS_RESOURCES_USER/get-current-user-guilds) endpoint will no longer return results for your bot. We will be creating a new endpoint that is more shard-aware to iterate through your bot's guilds if needed.

## Commands and Events

Commands are requests made to the gateway socket by a client.

###### Gateway Commands

| name                                                                | description                                                  |
| ------------------------------------------------------------------- | ------------------------------------------------------------ |
| [Identify](#DOCS_TOPICS_GATEWAY/identify)                           | triggers the initial handshake with the gateway              |
| [Resume](#DOCS_TOPICS_GATEWAY/resume)                               | resumes a dropped gateway connection                         |
| [Heartbeat](#DOCS_TOPICS_GATEWAY/heartbeat)                         | maintains an active gateway connection                       |
| [Request Guild Members](#DOCS_TOPICS_GATEWAY/request-guild-members) | requests members for a guild                                 |
| [Update Voice State](#DOCS_TOPICS_GATEWAY/update-voice-state)       | joins, moves, or disconnects the client from a voice channel |
| [Update Status](#DOCS_TOPICS_GATEWAY/update-status)                 | updates a client's presence                                  |

Events are payloads sent over the socket to a client that correspond to events in Discord.

###### Gateway Events

| name                                                                                | description                                                                                                                      |
| ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| [Hello](#DOCS_TOPICS_GATEWAY/hello)                                                 | defines the heartbeat interval                                                                                                   |
| [Ready](#DOCS_TOPICS_GATEWAY/ready)                                                 | contains the initial state information                                                                                           |
| [Resumed](#DOCS_TOPICS_GATEWAY/resumed)                                             | response to [Resume](#DOCS_TOPICS_GATEWAY/resume)                                                                                |
| [Reconnect](#DOCS_TOPICS_GATEWAY/reconnect)                                         | server is going away, client should reconnect to gateway and resume                                                              |
| [Invalid Session](#DOCS_TOPICS_GATEWAY/invalid-session)                             | failure response to [Identify](#DOCS_TOPICS_GATEWAY/identify) or [Resume](#DOCS_TOPICS_GATEWAY/resume) or invalid active session |
| [Channel Create](#DOCS_TOPICS_GATEWAY/channel-create)                               | new channel created                                                                                                              |
| [Channel Update](#DOCS_TOPICS_GATEWAY/channel-update)                               | channel was updated                                                                                                              |
| [Channel Delete](#DOCS_TOPICS_GATEWAY/channel-delete)                               | channel was deleted                                                                                                              |
| [Channel Pins Update](#DOCS_TOPICS_GATEWAY/channel-pins-update)                     | message was pinned or unpinned                                                                                                   |
| [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create)                                   | lazy-load for unavailable guild, guild became available, or user joined a new guild                                              |
| [Guild Update](#DOCS_TOPICS_GATEWAY/guild-update)                                   | guild was updated                                                                                                                |
| [Guild Delete](#DOCS_TOPICS_GATEWAY/guild-delete)                                   | guild became unavailable, or user left/was removed from a guild                                                                  |
| [Guild Ban Add](#DOCS_TOPICS_GATEWAY/guild-ban-add)                                 | user was banned from a guild                                                                                                     |
| [Guild Ban Remove](#DOCS_TOPICS_GATEWAY/guild-ban-remove)                           | user was unbanned from a guild                                                                                                   |
| [Guild Emojis Update](#DOCS_TOPICS_GATEWAY/guild-emojis-update)                     | guild emojis were updated                                                                                                        |
| [Guild Integrations Update](#DOCS_TOPICS_GATEWAY/guild-integrations-update)         | guild integration was updated                                                                                                    |
| [Guild Member Add](#DOCS_TOPICS_GATEWAY/guild-member-add)                           | new user joined a guild                                                                                                          |
| [Guild Member Remove](#DOCS_TOPICS_GATEWAY/guild-member-remove)                     | user was removed from a guild                                                                                                    |
| [Guild Member Update](#DOCS_TOPICS_GATEWAY/guild-member-update)                     | guild member was updated                                                                                                         |
| [Guild Members Chunk](#DOCS_TOPICS_GATEWAY/guild-members-chunk)                     | response to [Request Guild Members](#DOCS_TOPICS_GATEWAY/request-guild-members)                                                  |
| [Guild Role Create](#DOCS_TOPICS_GATEWAY/guild-role-create)                         | guild role was created                                                                                                           |
| [Guild Role Update](#DOCS_TOPICS_GATEWAY/guild-role-update)                         | guild role was updated                                                                                                           |
| [Guild Role Delete](#DOCS_TOPICS_GATEWAY/guild-role-delete)                         | guild role was deleted                                                                                                           |
| [Invite Create](#DOCS_TOPICS_GATEWAY/invite-create)                                 | invite to a channel was created                                                                                                  |
| [Invite Delete](#DOCS_TOPICS_GATEWAY/invite-delete)                                 | invite to a channel was deleted                                                                                                  |
| [Message Create](#DOCS_TOPICS_GATEWAY/message-create)                               | message was created                                                                                                              |
| [Message Update](#DOCS_TOPICS_GATEWAY/message-update)                               | message was edited                                                                                                               |
| [Message Delete](#DOCS_TOPICS_GATEWAY/message-delete)                               | message was deleted                                                                                                              |
| [Message Delete Bulk](#DOCS_TOPICS_GATEWAY/message-delete-bulk)                     | multiple messages were deleted at once                                                                                           |
| [Message Reaction Add](#DOCS_TOPICS_GATEWAY/message-reaction-add)                   | user reacted to a message                                                                                                        |
| [Message Reaction Remove](#DOCS_TOPICS_GATEWAY/message-reaction-remove)             | user removed a reaction from a message                                                                                           |
| [Message Reaction Remove All](#DOCS_TOPICS_GATEWAY/message-reaction-remove-all)     | all reactions were explicitly removed from a message                                                                             |
| [Message Reaction Remove Emoji](#DOCS_TOPICS_GATEWAY/message-reaction-remove-emoji) | all reactions for a given emoji were explicitly removed from a message                                                           |
| [Presence Update](#DOCS_TOPICS_GATEWAY/presence-update)                             | user was updated                                                                                                                 |
| [Typing Start](#DOCS_TOPICS_GATEWAY/typing-start)                                   | user started typing in a channel                                                                                                 |
| [User Update](#DOCS_TOPICS_GATEWAY/user-update)                                     | properties about the user changed                                                                                                |
| [Voice State Update](#DOCS_TOPICS_GATEWAY/voice-state-update)                       | someone joined, left, or moved a voice channel                                                                                   |
| [Voice Server Update](#DOCS_TOPICS_GATEWAY/voice-server-update)                     | guild's voice server was updated                                                                                                 |
| [Webhooks Update](#DOCS_TOPICS_GATEWAY/webhooks-update)                             | guild channel webhook was created, update, or deleted                                                                            |

### Event Names

Event names are in standard constant form, fully upper-cased and replacing all spaces with underscores. For instance, [Channel Create](#DOCS_TOPICS_GATEWAY/channel-create) would be `CHANNEL_CREATE` and [Voice State Update](#DOCS_TOPICS_GATEWAY/voice-state-update) would be `VOICE_STATE_UPDATE`. Within the following documentation, they have been left in standard English form to aid in readability.

#### Identify

Used to trigger the initial handshake with the gateway.

###### Identify Structure

| Field                | Type                                                                                                  | Description                                                                                                                    | Default |
| -------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ------- |
| token                | string                                                                                                | authentication token                                                                                                           | -       |
| properties           | [identify connection properties](#DOCS_TOPICS_GATEWAY/identify-identify-connection-properties) object | information about the client and how it's connecting                                                                           | -       |
| compress?            | boolean                                                                                               | whether this connection supports compression of packets                                                                        | false   |
| large_threshold?     | integer                                                                                               | value between 50 and 250, total number of members where the gateway will stop sending offline members in the guild member list | 50      |
| shard?               | array of two integers (shard_id, num_shards)                                                          | used for [Guild Sharding](#DOCS_TOPICS_GATEWAY/sharding)                                                                       | -       |
| presence?            | [gateway status update](#DOCS_TOPICS_GATEWAY/update-status) object                                    | presence structure for initial presence information                                                                            | -       |
| guild_subscriptions? | boolean                                                                                               | enables dispatching of guild subscription events (presence and typing events)                                                  | true    |
| intents?             | integer                                                                                               | the [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents) you wish to receive                                                | -       |

###### Identify Connection Properties

| Field     | Type   | Description           |
| --------- | ------ | --------------------- |
| \$os      | string | your operating system |
| \$browser | string | your library name     |
| \$device  | string | your library name     |

###### Example Identify

```json
{
  "op": 2,
  "d": {
    "token": "my_token",
    "properties": {
      "$os": "linux",
      "$browser": "disco",
      "$device": "disco"
    },
    "compress": true,
    "large_threshold": 250,
    "guild_subscriptions": false,
    "shard": [0, 1],
    "presence": {
      "game": {
        "name": "Cards Against Humanity",
        "type": 0
      },
      "status": "dnd",
      "since": 91879201,
      "afk": false
    },
    // This intent represents 1 << 0 for GUILDS, 1 << 1 for GUILD_MEMBERS, and 1 << 2 for GUILD_BANS
    // This connection will only receive the events defined in those three intents
    "intents": 7
  }
}
```

#### Resume

Used to replay missed events when a disconnected client resumes.

###### Resume Structure

| Field      | Type    | Description                   |
| ---------- | ------- | ----------------------------- |
| token      | string  | session token                 |
| session_id | string  | session id                    |
| seq        | integer | last sequence number received |

###### Example Resume

```json
{
  "op": 6,
  "d": {
    "token": "randomstring",
    "session_id": "evenmorerandomstring",
    "seq": 1337
  }
}
```

#### Heartbeat

Used to maintain an active gateway connection. Must be sent every `heartbeat_interval` milliseconds after the [Opcode 10 Hello](#DOCS_TOPICS_GATEWAY/hello) payload is received. The inner `d` key is the last sequence number—`s`—received by the client. If you have not yet received one, send `null`.

###### Example Heartbeat

```
{
	"op": 1,
	"d": 251
}
```

#### Request Guild Members

Used to request all members for a guild or a list of guilds. When initially connecting, the gateway will only send offline members if a guild has less than the `large_threshold` members (value in the [Gateway Identify](#DOCS_TOPICS_GATEWAY/identify)). If a client wishes to receive additional members, they need to explicitly request them via this operation. The server will send [Guild Members Chunk](#DOCS_TOPICS_GATEWAY/guild-members-chunk) events in response with up to 1000 members per chunk until all members that match the request have been sent.

If you are using [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents), there are some significant changes to this command to be mindful of:

- `GUILD_PRESENCES` intent is required to set `presences = true`. Otherwise, it will always be false
- `GUILD_MEMBERS` intent is required to request the entire member list—`(query=‘’, limit=0<=n)`
- You will be limited to requesting 1 `guild_id`
- Requesting a prefix will return a maximum of 100 members
- Requesting `user_ids` will continue to be limited to returning 100 members

###### Guild Request Members Structure

| Field      | Type                             | Description                                                                                                                           | Required                   |
| ---------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| guild_id   | snowflake or array of snowflakes | id of the guild(s) to get members for                                                                                                 | true                       |
| query?     | string                           | string that username starts with, or an empty string to return all members                                                            | one of query or user_ids   |
| limit      | integer                          | maximum number of members to send matching the `query`; a limit of `0` can be used with an empty string `query` to return all members | true when specifying query |
| presences? | boolean                          | used to specify if we want the presences of the matched members                                                                       | false                      |
| user_ids?  | snowflake or array of snowflakes | used to specify which users you wish to fetch                                                                                         | one of query or user_ids   |
| nonce?     | string                           | nonce to identify the [Guild Members Chunk](#DOCS_TOPICS_GATEWAY/guild-members-chunk) response                                        | false                      |

###### Guild Request Members

```json
{
  "op": 8,
  "d": {
    "guild_id": "41771983444115456",
    "query": "",
    "limit": 0
  }
}
```

#### Update Voice State

Sent when a client wants to join, move, or disconnect from a voice channel.

###### Gateway Voice State Update Structure

| Field      | Type       | Description                                                          |
| ---------- | ---------- | -------------------------------------------------------------------- |
| guild_id   | snowflake  | id of the guild                                                      |
| channel_id | ?snowflake | id of the voice channel client wants to join (null if disconnecting) |
| self_mute  | boolean    | is the client muted                                                  |
| self_deaf  | boolean    | is the client deafened                                               |

###### Example Gateway Voice State Update

```json
{
  "op": 4,
  "d": {
    "guild_id": "41771983423143937",
    "channel_id": "127121515262115840",
    "self_mute": false,
    "self_deaf": false
  }
}
```

#### Update Status

Sent by the client to indicate a presence or status update.

###### Gateway Status Update Structure

| Field  | Type                                                     | Description                                                                                 |
| ------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| since  | ?integer                                                 | unix time (in milliseconds) of when the client went idle, or null if the client is not idle |
| game   | ?[activity](#DOCS_TOPICS_GATEWAY/activity-object) object | null, or the user's new activity                                                            |
| status | string                                                   | the user's new [status](#DOCS_TOPICS_GATEWAY/update-status-status-types)                    |
| afk    | boolean                                                  | whether or not the client is afk                                                            |

###### Status Types

| Value (string) | Description                    |
| -------------- | ------------------------------ |
| online         | Online                         |
| dnd            | Do Not Disturb                 |
| idle           | AFK                            |
| invisible      | Invisible and shown as offline |
| offline        | Offline                        |

###### Example Gateway Status Update

```json
{
  "op": 3,
  "d": {
    "since": 91879201,
    "game": {
      "name": "Save the Oxford Comma",
      "type": 0
    },
    "status": "online",
    "afk": false
  }
}
```

### Connecting and Resuming

#### Hello

Sent on connection to the websocket. Defines the heartbeat interval that the client should heartbeat to.

###### Hello Structure

| Field              | Type    | Description                                                     |
| ------------------ | ------- | --------------------------------------------------------------- |
| heartbeat_interval | integer | the interval (in milliseconds) the client should heartbeat with |

###### Example Hello

```json
{
  "op": 10,
  "d": {
    "heartbeat_interval": 45000
  }
}
```

#### Ready

The ready event is dispatched when a client has completed the initial handshake with the gateway (for new sessions). The ready event can be the largest and most complex event the gateway will send, as it contains all the state required for a client to begin interacting with the rest of the platform.

`guilds` are the guilds of which your bot is a member. They start out as unavailable when you connect to the gateway. As they become available, your bot will be notified via [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) events. `private_channels` will be an empty array. As bots receive private messages, they will be notified via [Channel Create](#DOCS_TOPICS_GATEWAY/channel-create) events.

###### Ready Event Fields

| Field            | Type                                                                                 | Description                                                                                                   |
| ---------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| v                | integer                                                                              | [gateway version](#DOCS_TOPICS_GATEWAY/gateways-gateway-versions)                                             |
| user             | [user](#DOCS_RESOURCES_USER/user-object) object                                      | information about the user including email                                                                    |
| private_channels | array                                                                                | empty array                                                                                                   |
| guilds           | array of [Unavailable Guild](#DOCS_RESOURCES_GUILD/unavailable-guild-object) objects | the guilds the user is in                                                                                     |
| session_id       | string                                                                               | used for resuming connections                                                                                 |
| shard?           | array of two integers (shard_id, num_shards)                                         | the [shard information](#DOCS_TOPICS_GATEWAY/sharding) associated with this session, if sent when identifying |

#### Resumed

The resumed event is dispatched when a client has sent a [resume payload](#DOCS_TOPICS_GATEWAY/resume) to the gateway (for resuming existing sessions).

#### Reconnect

The reconnect event is dispatched when a client should reconnect to the gateway (and resume their existing session, if they have one). This event usually occurs during deploys to migrate sessions gracefully off old hosts.

#### Invalid Session

Sent to indicate one of at least three different situations:

- the gateway could not initialize a session after receiving an [Opcode 2 Identify](#DOCS_TOPICS_GATEWAY/identify)
- the gateway could not resume a previous session after receiving an [Opcode 6 Resume](#DOCS_TOPICS_GATEWAY/resume)
- the gateway has invalidated an active session and is requesting client action

The inner `d` key is a boolean that indicates whether the session may be resumable. See [Connecting](#DOCS_TOPICS_GATEWAY/connecting) and [Resuming](#DOCS_TOPICS_GATEWAY/resuming) for more information.

###### Example Gateway Invalid Session

```json
{
  "op": 9,
  "d": false
}
```

### Channels

#### Channel Create

Sent when a new channel is created, relevant to the current user. The inner payload is a [channel](#DOCS_RESOURCES_CHANNEL/channel-object) object.

#### Channel Update

Sent when a channel is updated. The inner payload is a [channel](#DOCS_RESOURCES_CHANNEL/channel-object) object. This is not sent when the field `last_message_id` is altered. To keep track of the last_message_id changes, you must listen for [Message Create](#DOCS_TOPICS_GATEWAY/message-create) events.

#### Channel Delete

Sent when a channel relevant to the current user is deleted. The inner payload is a [channel](#DOCS_RESOURCES_CHANNEL/channel-object) object.

#### Channel Pins Update

Sent when a message is pinned or unpinned in a text channel. This is not sent when a pinned message is deleted.

###### Channel Pins Update Event Fields

| Field               | Type              | Description                                                 |
| ------------------- | ----------------- | ----------------------------------------------------------- |
| guild_id?           | snowflake         | the id of the guild                                         |
| channel_id          | snowflake         | the id of the channel                                       |
| last_pin_timestamp? | ISO8601 timestamp | the time at which the most recent pinned message was pinned |

### Guilds

#### Guild Create

This event can be sent in three different scenarios:

1.  When a user is initially connecting, to lazily load and backfill information for all unavailable guilds sent in the [Ready](#DOCS_TOPICS_GATEWAY/ready) event.
2.  When a Guild becomes available again to the client.
3.  When the current user joins a new Guild.

The inner payload is a [guild](#DOCS_RESOURCES_GUILD/guild-object) object, with all the extra fields specified.

> warn
> If you are using [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents), members and presences returned in this event will only contain your bot and users in voice channels unless you specify the `GUILD_PRESENCES` intent.

#### Guild Update

Sent when a guild is updated. The inner payload is a [guild](#DOCS_RESOURCES_GUILD/guild-object) object.

#### Guild Delete

Sent when a guild becomes unavailable during a guild outage, or when the user leaves or is removed from a guild. The inner payload is an [unavailable guild](#DOCS_RESOURCES_GUILD/unavailable-guild-object) object. If the `unavailable` field is not set, the user was removed from the guild.

#### Guild Ban Add

Sent when a user is banned from a guild.

###### Guild Ban Add Event Fields

| Field    | Type                                              | Description     |
| -------- | ------------------------------------------------- | --------------- |
| guild_id | snowflake                                         | id of the guild |
| user     | a [user](#DOCS_RESOURCES_USER/user-object) object | the banned user |

#### Guild Ban Remove

Sent when a user is unbanned from a guild.

###### Guild Ban Remove Event Fields

| Field    | Type                                              | Description       |
| -------- | ------------------------------------------------- | ----------------- |
| guild_id | snowflake                                         | id of the guild   |
| user     | a [user](#DOCS_RESOURCES_USER/user-object) object | the unbanned user |

#### Guild Emojis Update

Sent when a guild's emojis have been updated.

###### Guild Emojis Update Event Fields

| Field    | Type      | Description                                           |
| -------- | --------- | ----------------------------------------------------- |
| guild_id | snowflake | id of the guild                                       |
| emojis   | array     | array of [emojis](#DOCS_RESOURCES_EMOJI/emoji-object) |

#### Guild Integrations Update

Sent when a guild integration is updated.

###### Guild Integrations Update Event Fields

| Field    | Type      | Description                                     |
| -------- | --------- | ----------------------------------------------- |
| guild_id | snowflake | id of the guild whose integrations were updated |

#### Guild Member Add

> warn
> If using [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents), the `GUILD_MEMBERS` intent will be required to receive this event.

Sent when a new user joins a guild. The inner payload is a [guild member](#DOCS_RESOURCES_GUILD/guild-member-object) object with an extra `guild_id` key:

###### Guild Member Add Extra Fields

| Field    | Type      | Description     |
| -------- | --------- | --------------- |
| guild_id | snowflake | id of the guild |

#### Guild Member Remove

> warn
> If using [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents), the `GUILD_MEMBERS` intent will be required to receive this event.

Sent when a user is removed from a guild (leave/kick/ban).

###### Guild Member Remove Event Fields

| Field    | Type                                              | Description              |
| -------- | ------------------------------------------------- | ------------------------ |
| guild_id | snowflake                                         | the id of the guild      |
| user     | a [user](#DOCS_RESOURCES_USER/user-object) object | the user who was removed |

#### Guild Member Update

> warn
> If using [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents), the `GUILD_MEMBERS` intent will be required to receive this event.

Sent when a guild member is updated. This will also fire when the user object of a guild member changes.

###### Guild Member Update Event Fields

| Field          | Type                                              | Description                                                                                                                 |
| -------------- | ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| guild_id       | snowflake                                         | the id of the guild                                                                                                         |
| roles          | array of snowflakes                               | user role ids                                                                                                               |
| user           | a [user](#DOCS_RESOURCES_USER/user-object) object | the user                                                                                                                    |
| nick?          | ?string                                           | nickname of the user in the guild                                                                                           |
| premium_since? | ?ISO8601 timestamp                                | when the user starting [boosting](https://support.discord.com/hc/en-us/articles/360028038352-Server-Boosting-) the guild |

#### Guild Members Chunk

Sent in response to [Guild Request Members](#DOCS_TOPICS_GATEWAY/request-guild-members).
You can use the `chunk_index` and `chunk_count` to calculate how many chunks are left for your request.

###### Guild Members Chunk Event Fields

| Field       | Type                                                                       | Description                                                                                        |
| ----------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| guild_id    | snowflake                                                                  | the id of the guild                                                                                |
| members     | array of [guild member](#DOCS_RESOURCES_GUILD/guild-member-object) objects | set of guild members                                                                               |
| chunk_index | integer                                                                    | the chunk index in the expected chunks for this response (0 <= chunk\_index < chunk\_count)        |
| chunk_count | integer                                                                    | the total number of expected chunks for this response                                              |
| not_found?  | array                                                                      | if passing an invalid id to `REQUEST_GUILD_MEMBERS`, it will be returned here                      |
| presences?  | array of [presence](#DOCS_TOPICS_GATEWAY/presence) objects                 | if passing true to `REQUEST_GUILD_MEMBERS`, presences of the returned members will be here         |
| nonce?      | string                                                                     | the nonce used in the [Guild Members Request](#DOCS_TOPICS_GATEWAY/request-guild-members)          |

#### Guild Role Create

Sent when a guild role is created.

###### Guild Role Create Event Fields

| Field    | Type                                                  | Description         |
| -------- | ----------------------------------------------------- | ------------------- |
| guild_id | snowflake                                             | the id of the guild |
| role     | a [role](#DOCS_TOPICS_PERMISSIONS/role-object) object | the role created    |

#### Guild Role Update

Sent when a guild role is updated.

###### Guild Role Update Event Fields

| Field    | Type                                                  | Description         |
| -------- | ----------------------------------------------------- | ------------------- |
| guild_id | snowflake                                             | the id of the guild |
| role     | a [role](#DOCS_TOPICS_PERMISSIONS/role-object) object | the role updated    |

#### Guild Role Delete

Sent when a guild role is deleted.

###### Guild Role Delete Event Fields

| Field    | Type      | Description     |
| -------- | --------- | --------------- |
| guild_id | snowflake | id of the guild |
| role_id  | snowflake | id of the role  |

### Invites

### Invite Create

Sent when a new invite to a channel is created.

###### Invite Create Event Fields

| Field             | Type                                                    | Description                                                                                                        |
| ----------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| channel_id        | snowflake                                               | the channel the invite is for                                                                                      |
| code              | string                                                  | the unique invite [code](#DOCS_RESOURCES_INVITE/invite-object)                                                     |
| created_at        | integer                                                 | unix timestamp of the time at which the invite was created                                                                           |
| guild_id?         | snowflake                                               | the guild of the invite                                                                                            |
| inviter?          | [user](#DOCS_RESOURCES_USER/user-object) object         | the user that created the invite                                                                                   |
| max_age           | integer                                                 | how long the invite is valid for (in seconds)                                                                      |
| max_uses          | integer                                                 | the maximum number of times the invite can be used                                                                 |
| target_user?      | partial [user](#DOCS_RESOURCES_USER/user-object) object | the target user for this invite                                                                                    |
| target_user_type? | integer                                                 | the [type of user target](#DOCS_RESOURCES_INVITE/invite-object-target-user-types) for this invite                  |
| temporary         | boolean                                                 | whether or not the invite is temporary (invited users will be kicked on disconnect unless they're assigned a role) |
| uses              | integer                                                 | how many times the invite has been used (always will be 0)                                                         |

### Invite Delete

Sent when an invite is deleted.

###### Invite Delete Event Fields

| Field      | Type      | Description                                                    |
| ---------- | --------- | -------------------------------------------------------------- |
| channel_id | snowflake | the channel of the invite                                      |
| guild_id?  | snowflake | the guild of the invite                                        |
| code       | string    | the unique invite [code](#DOCS_RESOURCES_INVITE/invite-object) |

### Messages

#### Message Create

Sent when a message is created. The inner payload is a [message](#DOCS_RESOURCES_CHANNEL/message-object) object.

#### Message Update

Sent when a message is updated. The inner payload is a [message](#DOCS_RESOURCES_CHANNEL/message-object) object.

> warn
> Unlike creates, message updates may contain only a subset of the full message object payload (but will always contain an id and channel_id).

#### Message Delete

Sent when a message is deleted.

###### Message Delete Event Fields

| Field      | Type      | Description           |
| ---------- | --------- | --------------------- |
| id         | snowflake | the id of the message |
| channel_id | snowflake | the id of the channel |
| guild_id?  | snowflake | the id of the guild   |

#### Message Delete Bulk

Sent when multiple messages are deleted at once.

###### Message Delete Bulk Event Fields

| Field      | Type                | Description             |
| ---------- | ------------------- | ----------------------- |
| ids        | array of snowflakes | the ids of the messages |
| channel_id | snowflake           | the id of the channel   |
| guild_id?  | snowflake           | the id of the guild     |

#### Message Reaction Add

Sent when a user adds a reaction to a message.

###### Message Reaction Add Event Fields

| Field      | Type                                                         | Description                                                                                                     |
| ---------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| user_id    | snowflake                                                    | the id of the user                                                                                              |
| channel_id | snowflake                                                    | the id of the channel                                                                                           |
| message_id | snowflake                                                    | the id of the message                                                                                           |
| guild_id?  | snowflake                                                    | the id of the guild                                                                                             |
| member?    | [member](#DOCS_RESOURCES_GUILD/guild-member-object) object   | the member who reacted if this happened in a guild                                                              |
| emoji      | partial [emoji](#DOCS_RESOURCES_EMOJI/emoji-object) object | the emoji used to react - [example](#DOCS_RESOURCES_EMOJI/emoji-object-gateway-reaction-standard-emoji-example) |

#### Message Reaction Remove

Sent when a user removes a reaction from a message.

###### Message Reaction Remove Event Fields

| Field      | Type                                                         | Description                                                                                                     |
| ---------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| user_id    | snowflake                                                    | the id of the user                                                                                              |
| channel_id | snowflake                                                    | the id of the channel                                                                                           |
| message_id | snowflake                                                    | the id of the message                                                                                           |
| guild_id?  | snowflake                                                    | the id of the guild                                                                                             |
| emoji      | partial [emoji](#DOCS_RESOURCES_EMOJI/emoji-object) object | the emoji used to react - [example](#DOCS_RESOURCES_EMOJI/emoji-object-gateway-reaction-standard-emoji-example) |

#### Message Reaction Remove All

Sent when a user explicitly removes all reactions from a message.

###### Message Reaction Remove All Event Fields

| Field      | Type      | Description           |
| ---------- | --------- | --------------------- |
| channel_id | snowflake | the id of the channel |
| message_id | snowflake | the id of the message |
| guild_id?  | snowflake | the id of the guild   |

#### Message Reaction Remove Emoji

Sent when a bot removes all instances of a given emoji from the reactions of a message.

###### Message Reaction Remove Emoji

| Field      | Type                                                       | Description                |
| ---------- | ---------------------------------------------------------- | -------------------------- |
| channel_id | snowflake                                                  | the id of the channel      |
| guild_id?  | snowflake                                                  | the id of the guild        |
| message_id | snowflake                                                  | the id of the message      |
| emoji      | partial [emoji](#DOCS_RESOURCES_EMOJI/emoji-object) object | the emoji that was removed |

### Presence

#### Presence Update

> warn
> If you are using [Gateway Intents](#DOCS_TOPICS_GATEWAY/gateway-intents), you _must_ specify the `GUILD_PRESENCES` intent in order to receive Presence Update events

A user's presence is their current state on a guild. This event is sent when a user's presence or info, such as name or avatar, is updated.

> warn
> The user object within this event can be partial, the only field which must be sent is the `id` field, everything else is optional. Along with this limitation, no fields are required, and the types of the fields are **not** validated. Your client should expect any combination of fields and types within this event.

###### Presence Update Event Fields

| Field          | Type                                                              | Description                                                                                                                |
| -------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| user           | [user](#DOCS_RESOURCES_USER/user-object) object                   | the user presence is being updated for                                                                                     |
| roles          | array of snowflakes                                               | roles this user is in                                                                                                      |
| game           | ?[activity](#DOCS_TOPICS_GATEWAY/activity-object) object          | null, or the user's current activity                                                                                       |
| guild_id       | snowflake                                                         | id of the guild                                                                                                            |
| status         | string                                                            | either "idle", "dnd", "online", or "offline"                                                                               |
| activities     | array of [activity](#DOCS_TOPICS_GATEWAY/activity-object) objects | user's current activities                                                                                                  |
| client_status  | [client_status](#DOCS_TOPICS_GATEWAY/client-status-object) object | user's platform-dependent status                                                                                           |
| premium_since? | ?ISO8601 timestamp                                                | when the user started [boosting](https://support.discord.com/hc/en-us/articles/360028038352-Server-Boosting-) the guild |
| nick?          | ?string                                                           | this users guild nickname (if one is set)                                                                                  |

#### Client Status Object

Active sessions are indicated with an "online", "idle", or "dnd" string per platform. If a user is offline or invisible, the corresponding field is not present.

| Field    | Type   | Description                                                                           |
| -------- | ------ | ------------------------------------------------------------------------------------- |
| desktop? | string | the user's status set for an active desktop (Windows, Linux, Mac) application session |
| mobile?  | string | the user's status set for an active mobile (iOS, Android) application session         |
| web?     | string | the user's status set for an active web (browser, bot account) application session    |

#### Activity Object

###### Activity Structure

| Field           | Type                                                                                   | Description                                                                                                               |
| --------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| name            | string                                                                                 | the activity's name                                                                                                       |
| type            | integer                                                                                | [activity type](#DOCS_TOPICS_GATEWAY/activity-object-activity-types)                                                      |
| url?            | ?string                                                                                | stream url, is validated when type is 1                                                                                   |
| created_at      | integer                                                                                | unix timestamp of when the activity was added to the user's session                                                       |
| timestamps?     | [activity timestamps](#DOCS_TOPICS_GATEWAY/activity-object-activity-timestamps) object | unix timestamps for start and/or end of the game                                                                          |
| application_id? | snowflake                                                                              | application id for the game                                                                                               |
| details?        | ?string                                                                                | what the player is currently doing                                                                                        |
| state?          | ?string                                                                                | the user's current party status                                                                                           |
| emoji?          | ?[activity emoji](#DOCS_TOPICS_GATEWAY/activity-object-activity-emoji) object          | the emoji used for a custom status                                                                                        |
| party?          | [activity party](#DOCS_TOPICS_GATEWAY/activity-object-activity-party) object           | information for the current party of the player                                                                           |
| assets?         | [activity assets](#DOCS_TOPICS_GATEWAY/activity-object-activity-assets) object         | images for the presence and their hover texts                                                                             |
| secrets?        | [activity secrets](#DOCS_TOPICS_GATEWAY/activity-object-activity-secrets) object       | secrets for Rich Presence joining and spectating                                                                          |
| instance?       | boolean                                                                                | whether or not the activity is an instanced game session                                                                  |
| flags?          | integer                                                                                | [activity flags](#DOCS_TOPICS_GATEWAY/activity-object-activity-flags) `OR`d together, describes what the payload includes |

> info
> Bots are only able to send `name`, `type`, and optionally `url`.

###### Activity Types

| Values (integer)  | Name      | Format              | Example                   |
| ----------------- | --------- | ------------------- | ------------------------- |
| 0                 | Game      | Playing {name}      | "Playing Rocket League"   |
| 1                 | Streaming | Streaming {details} | "Streaming Rocket League" |
| 2                 | Listening | Listening to {name} | "Listening to Spotify"    |
| 4                 | Custom    | {emoji} {name}      | ":smiley: I am cool"      |

> info
> The streaming type currently only supports Twitch and YouTube. Only `https://twitch.tv/` and `https://youtube.com/` urls will work.

###### Activity Timestamps

| Field  | Type    | Description                                              |
| ------ | ------- | -------------------------------------------------------- |
| start? | integer | unix time (in milliseconds) of when the activity started |
| end?   | integer | unix time (in milliseconds) of when the activity ends    |

###### Activity Emoji

| Field     | Type      | Description                    |
| --------- | --------- | ------------------------------ |
| name      | string    | the name of the emoji          |
| id?       | snowflake | the id of the emoji            |
| animated? | boolean   | whether this emoji is animated |

###### Activity Party

| Field | Type                                           | Description                                       |
| ----- | ---------------------------------------------- | ------------------------------------------------- |
| id?   | string                                         | the id of the party                               |
| size? | array of two integers (current_size, max_size) | used to show the party's current and maximum size |

###### Activity Assets

| Field        | Type   | Description                                                       |
| ------------ | ------ | ----------------------------------------------------------------- |
| large_image? | string | the id for a large asset of the activity, usually a snowflake     |
| large_text?  | string | text displayed when hovering over the large image of the activity |
| small_image? | string | the id for a small asset of the activity, usually a snowflake     |
| small_text?  | string | text displayed when hovering over the small image of the activity |

###### Activity Secrets

| Field     | Type   | Description                               |
| --------- | ------ | ----------------------------------------- |
| join?     | string | the secret for joining a party            |
| spectate? | string | the secret for spectating a game          |
| match?    | string | the secret for a specific instanced match |

###### Activity Flags

| Value (bitfield) | Name         | 
| ---------------- | ------------ |
| 1 << 0           | INSTANCE     |
| 1 << 1           | JOIN         |
| 1 << 2           | SPECTATE     |
| 1 << 3           | JOIN_REQUEST |
| 1 << 4           | SYNC         |
| 1 << 5           | PLAY         |

###### Example Activity

```json
{
  "details": "24H RL Stream for Charity",
  "state": "Rocket League",
  "name": "Twitch",
  "type": 1,
  "url": "https://www.twitch.tv/discordapp"
}
```

###### Example Activity with Rich Presence

```json
{
  "name": "Rocket League",
  "type": 0,
  "application_id": "379286085710381999",
  "state": "In a Match",
  "details": "Ranked Duos: 2-1",
  "timestamps": {
    "start": 15112000660000
  },
  "party": {
    "id": "9dd6594e-81b3-49f6-a6b5-a679e6a060d3",
    "size": [2, 2]
  },
  "assets": {
    "large_image": "351371005538729000",
    "large_text": "DFH Stadium",
    "small_image": "351371005538729111",
    "small_text": "Silver III"
  },
  "secrets": {
    "join": "025ed05c71f639de8bfaa0d679d7c94b2fdce12f",
    "spectate": "e7eb30d2ee025ed05c71ea495f770b76454ee4e0",
    "match": "4b2fdce12f639de8bfa7e3591b71a0d679d7c93f"
  }
}
```

> warn
> Clients may only update their game status 5 times per 20 seconds.

#### Typing Start

Sent when a user starts typing in a channel.

###### Typing Start Event Fields

| Field      | Type                                                       | Description                                               |
| ---------- | ---------------------------------------------------------- | --------------------------------------------------------- |
| channel_id | snowflake                                                  | id of the channel                                         |
| guild_id?  | snowflake                                                  | id of the guild                                           |
| user_id    | snowflake                                                  | id of the user                                            |
| timestamp  | integer                                                    | unix time (in seconds) of when the user started typing    |
| member?    | [member](#DOCS_RESOURCES_GUILD/guild-member-object) object | the member who started typing if this happened in a guild |

#### User Update

Sent when properties about the user change. Inner payload is a [user](#DOCS_RESOURCES_USER/user-object) object.

### Voice

#### Voice State Update

Sent when someone joins/leaves/moves voice channels. Inner payload is a [voice state](#DOCS_RESOURCES_VOICE/voice-state-object) object.

#### Voice Server Update

Sent when a guild's voice server is updated. This is sent when initially connecting to voice, and when the current voice instance fails over to a new server.

###### Voice Server Update Event Fields

| Field    | Type      | Description                               |
| -------- | --------- | ----------------------------------------- |
| token    | string    | voice connection token                    |
| guild_id | snowflake | the guild this voice server update is for |
| endpoint | string    | the voice server host                     |

###### Example Voice Server Update Payload

```json
{
  "token": "my_token",
  "guild_id": "41771983423143937",
  "endpoint": "smart.loyal.discord.gg"
}
```

### Webhooks

#### Webhooks Update

Sent when a guild channel's webhook is created, updated, or deleted.

###### Webhook Update Event Fields

| Field      | Type      | Description       |
| ---------- | --------- | ----------------- |
| guild_id   | snowflake | id of the guild   |
| channel_id | snowflake | id of the channel |

## Get Gateway % GET /gateway

> info
> This endpoint does not require authentication.

Returns an object with a single valid WSS URL, which the client can use for [Connecting](#DOCS_TOPICS_GATEWAY/connecting). Clients **should** cache this value and only call this endpoint to retrieve a new URL if they are unable to properly establish a connection using the cached version of the URL.

###### Example Response

```json
{
  "url": "wss://gateway.discord.gg/"
}
```

## Get Gateway Bot % GET /gateway/bot

> warn
> This endpoint requires authentication using a valid bot token.

Returns an object based on the information in [Get Gateway](#DOCS_TOPICS_GATEWAY/get-gateway), plus additional metadata that can help during the operation of large or [sharded](#DOCS_TOPICS_GATEWAY/sharding) bots. Unlike the [Get Gateway](#DOCS_TOPICS_GATEWAY/get-gateway), this route should not be cached for extended periods of time as the value is not guaranteed to be the same per-call, and changes as the bot joins/leaves guilds.

###### JSON Response

| Field               | Type                                                                          | Description                                                                              |
| ------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| url                 | string                                                                        | The WSS URL that can be used for connecting to the gateway                               |
| shards              | integer                                                                       | The recommended number of [shards](#DOCS_TOPICS_GATEWAY/sharding) to use when connecting |
| session_start_limit | [session_start_limit](#DOCS_TOPICS_GATEWAY/session-start-limit-object) object | Information on the current session start limit                                           |

###### Example Response

```json
{
  "url": "wss://gateway.discord.gg/",
  "shards": 9,
  "session_start_limit": {
    "total": 1000,
    "remaining": 999,
    "reset_after": 14400000
  }
}
```

## Session Start Limit Object

###### Session Start Limit Structure

| Field       | Type    | Description                                                        |
| ----------- | ------- | ------------------------------------------------------------------ |
| total       | integer | The total number of session starts the current user is allowed     |
| remaining   | integer | The remaining number of session starts the current user is allowed |
| reset_after | integer | The number of milliseconds after which the limit resets            |
