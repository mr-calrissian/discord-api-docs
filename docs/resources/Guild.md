# Guild Resource

Guilds in Discord represent an isolated collection of users and channels, and are often referred to as "servers" in the UI.

### Guild Object

###### Guild Structure

| Field                         | Type                                                                                                         | Description                                                                                                                               |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| id                            | snowflake                                                                                                    | guild id                                                                                                                                  |
| name                          | string                                                                                                       | guild name (2-100 characters, excluding trailing and leading whitespace)                                                                  |
| icon                          | ?string                                                                                                      | [icon hash](#DOCS_REFERENCE/image-formatting)                                                                                             |
| splash                        | ?string                                                                                                      | [splash hash](#DOCS_REFERENCE/image-formatting)                                                                                           |
| discovery_splash              | ?string                                                                                                      | [discovery splash hash](#DOCS_REFERENCE/image-formatting); only present for guilds with the "DISCOVERABLE" feature                        |
| owner?                        | boolean                                                                                                      | true if [the user](#DOCS_RESOURCES_USER/get-current-user-guilds) is the owner of the guild                                                |
| owner_id                      | snowflake                                                                                                    | id of owner                                                                                                                               |
| permissions?                  | integer                                                                                                      | total permissions for [the user](#DOCS_RESOURCES_USER/get-current-user-guilds) in the guild (excludes overrides)                          |
| region                        | [voice region](#DOCS_RESOURCES_VOICE/voice-region-object)                                                    | voice region id for the guild                                                                                                             |
| afk_channel_id                | ?snowflake                                                                                                   | id of afk channel                                                                                                                         |
| afk_timeout                   | integer                                                                                                      | afk timeout in seconds                                                                                                                    |
| embed_enabled?                | boolean                                                                                                      | true if the server widget is enabled (deprecated, replaced with `widget_enabled`)                                                         |
| embed_channel_id?             | ?snowflake                                                                                                   | the channel id that the widget will generate an invite to, or `null` if set to no invite (deprecated, replaced with `widget_channel_id`)  |
| verification_level            | [verification level](#DOCS_RESOURCES_GUILD/guild-object-verification-level)                                  | verification level required for the guild                                                                                                 |
| default_message_notifications | [default message notification level](#DOCS_RESOURCES_GUILD/guild-object-default-message-notification-level)  | default message notification level                                                                                                        |
| explicit_content_filter       | [explicit content filter level](#DOCS_RESOURCES_GUILD/guild-object-explicit-content-filter-level)            | explicit content filter level                                                                                                             |
| roles                         | array of [role](#DOCS_TOPICS_PERMISSIONS/role-object) objects                                                | roles in the guild                                                                                                                        |
| emojis                        | array of [emoji](#DOCS_RESOURCES_EMOJI/emoji-object) objects                                                 | custom guild emojis                                                                                                                       |
| features                      | array of [guild feature](#DOCS_RESOURCES_GUILD/guild-object-guild-features) strings                          | enabled guild features                                                                                                                    |
| mfa_level                     | [MFA level](#DOCS_RESOURCES_GUILD/guild-object-mfa-level)                                                    | required MFA Level for the guild                                                                                                          |
| application_id                | ?snowflake                                                                                                   | application id of the guild creator if it is bot-created                                                                                  |
| widget_enabled?               | boolean                                                                                                      | true if the server widget is enabled                                                                                                      |
| widget_channel_id?            | ?snowflake                                                                                                   | the channel id that the widget will generate an invite to, or `null` if set to no invite                                                  |
| system_channel_id             | ?snowflake                                                                                                   | the id of the channel where guild notices such as welcome messages and boost events are posted                                            |
| system_channel_flags          | [system channel flags](#DOCS_RESOURCES_GUILD/guild-object-system-channel-flags)                              | system channel flags                                                                                                                      |
| rules_channel_id              | ?snowflake                                                                                                   | the id of the channel where guilds with the "PUBLIC" feature can display rules and/or guidelines                                          |
| joined_at? \*                 | ISO8601 timestamp                                                                                            | when this guild was joined at                                                                                                             |
| large? \*                     | boolean                                                                                                      | true if this is considered a large guild                                                                                                  |
| unavailable? \*               | boolean                                                                                                      | true if this guild is unavailable due to an outage                                                                                        |
| member_count? \*              | integer                                                                                                      | total number of members in this guild                                                                                                     |
| voice_states? \*              | array of partial [voice state](#DOCS_RESOURCES_VOICE/voice-state-object) objects                             | states of members currently in voice channels; lacks the `guild_id` key                                                                   |
| members? \*                   | array of [guild member](#DOCS_RESOURCES_GUILD/guild-member-object) objects                                   | users in the guild                                                                                                                        |
| channels? \*                  | array of [channel](#DOCS_RESOURCES_CHANNEL/channel-object) objects                                           | channels in the guild                                                                                                                     |
| presences? \*                 | array of partial [presence](#DOCS_TOPICS_GATEWAY/presence-update) objects                                    | presences of the members in the guild, will only include non-offline members if the size is greater than `large threshold`                |
| max_presences?                | ?integer                                                                                                     | the maximum number of presences for the guild (the default value, currently 25000, is in effect when `null` is returned)                  |
| max_members?                  | integer                                                                                                      | the maximum number of members for the guild                                                                                               |
| vanity_url_code               | ?string                                                                                                      | the vanity url code for the guild                                                                                                         |
| description                   | ?string                                                                                                      | the description for the guild, if the guild is discoverable                                                                               |
| banner                        | ?string                                                                                                      | [banner hash](#DOCS_REFERENCE/image-formatting)                                                                                           |
| premium_tier                  | [premium tier](#DOCS_RESOURCES_GUILD/guild-object-premium-tier)                                              | server Boost level                                                                                                                        |
| premium_subscription_count?   | integer                                                                                                      | the number of boosts this guild currently has                                                                                             |
| preferred_locale              | string                                                                                                       | the preferred locale of a guild with the "PUBLIC" feature; used in server discovery and notices from Discord; defaults to "en-US"         |
| public_updates_channel_id     | ?snowflake                                                                                                   | the id of the channel where admins and moderators of guilds with the "PUBLIC" feature receive notices from Discord                        |
| max_video_channel_users?      | integer                                                                                                      | the maximum amount of users in a video channel                                                                                            |
| approximate_member_count?     | integer                                                                                                      | approximate number of members in this guild, returned from the `GET /guild/<id>` endpoint when `with_counts` is `true`                    |
| approximate_presence_count?   | integer                                                                                                      | approximate number of non-offline members in this guild, returned from the `GET /guild/<id>` endpoint when `with_counts` is `true`        |

** \* These fields are only sent within the [GUILD_CREATE](#DOCS_TOPICS_GATEWAY/guild-create) event **

###### Default Message Notification Level

| Value (integer) | Key           |
| --------------- | ------------- |
| 0               | ALL_MESSAGES  |
| 1               | ONLY_MENTIONS |

###### Explicit Content Filter Level

| Value (integer) | Level                 | 
| --------------- | --------------------- | 
| 0               | DISABLED              |
| 1               | MEMBERS_WITHOUT_ROLES |
| 2               | ALL_MEMBERS           |

###### MFA Level

| Value (integer) | Level    |
| --------------- | -------- |
| 0               | NONE     |
| 1               | ELEVATED |

###### Verification Level

| Value (integer) | Level     | Description                                                                |
| --------------- | --------- | -------------------------------------------------------------------------- |
| 0               | NONE      | unrestricted                                                               |
| 1               | LOW       | must have verified email on account                                        |
| 2               | MEDIUM    | must be registered on Discord for longer than 5 minutes                    |
| 3               | HIGH      | (╯°□°）╯︵ ┻━┻ - must be a member of the server for longer than 10 minutes |
| 4               | VERY_HIGH | ┻━┻ ミヽ(ಠ 益 ಠ)ﾉ彡 ┻━┻ - must have a verified phone number                |

###### Premium Tier

| Value (integer) | Level  |
| --------------- | ------ |
| 0               | NONE   |
| 1               | TIER_1 |
| 2               | TIER_2 |
| 3               | TIER_3 |

###### System Channel Flags

| Value (bitfield) | Name                           | Description                         |
| ---------------- | ------------------------------ | ----------------------------------- |
| 1 << 0           | SUPPRESS_JOIN_NOTIFICATIONS    | Suppress member join notifications  |
| 1 << 1           | SUPPRESS_PREMIUM_SUBSCRIPTIONS | Suppress server boost notifications |

###### Guild Feature

| Value (string)         | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| INVITE_SPLASH          | guild has access to set an invite splash background                             |
| VIP_REGIONS            | guild has access to set 384kbps bitrate in voice (previously VIP voice servers) |
| VANITY_URL             | guild has access to set a vanity URL                                            |
| VERIFIED               | guild is verified                                                               |
| PARTNERED              | guild is partnered                                                              |
| PUBLIC                 | guild is public                                                                 |
| COMMERCE               | guild has access to use commerce features (i.e. create store channels)          |
| NEWS                   | guild has access to create news channels                                        |
| DISCOVERABLE           | guild is able to be discovered in the directory                                 |
| FEATURABLE             | guild is able to be featured in the directory                                   |
| ANIMATED_ICON          | guild has access to set an animated guild icon                                  |
| BANNER                 | guild has access to set a guild banner image                                    |
| PUBLIC_DISABLED        | guild cannot be public                                                          |
| WELCOME_SCREEN_ENABLED | guild has enabled the welcome screen                                            |

###### Example Guild

```json
{
  "id": "197038439483310086",
  "name": "Discord Testers",
  "icon": "f64c482b807da4f539cff778d174971c",
  "description": "The official place to report Discord Bugs!",
  "splash": null,
  "discovery_splash": null,
  "features": [
    "ANIMATED_ICON",
    "VERIFIED",
    "NEWS",
    "VANITY_URL",
    "DISCOVERABLE",
    "MORE_EMOJI",
    "INVITE_SPLASH",
    "BANNER",
    "PUBLIC"
  ],
  "emojis": [],
  "banner": "9b6439a7de04f1d26af92f84ac9e1e4a",
  "owner_id": "73193882359173120",
  "application_id": null,
  "region": "us-west",
  "afk_channel_id": null,
  "afk_timeout": 300,
  "system_channel_id": null,
  "widget_enabled": true,
  "widget_channel_id": null,
  "verification_level": 3,
  "roles": [],
  "default_message_notifications": 1,
  "mfa_level": 1,
  "explicit_content_filter": 2,
  "max_presences": 40000,
  "max_members": 250000,
  "vanity_url_code": "discord-testers",
  "premium_tier": 3,
  "premium_subscription_count": 33,
  "system_channel_flags": 0,
  "preferred_locale": "en-US",
  "rules_channel_id": "441688182833020939",
  "public_updates_channel_id": "281283303326089216",
  "embed_enabled": true,
  "embed_channel_id": null
}
```

### Unavailable Guild Object

A partial [guild](#DOCS_RESOURCES_GUILD/guild-object) object. Represents an Offline Guild, or a Guild whose information has not been provided through [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) events during the Gateway connect.

###### Example Unavailable Guild

```json
{
  "id": "41771983423143937",
  "unavailable": true
}
```

### Guild Preview Object

###### Guild Preview Structure

| Field                      | Type                                                                                | Description                                               |
| -------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------- |
| id                         | snowflake                                                                           | guild id                                                  |
| name                       | string                                                                              | guild name (2-100 characters)                             |
| icon                       | ?string                                                                             | [icon hash](#DOCS_REFERENCE/image-formatting)             |
| splash                     | ?string                                                                             | [splash hash](#DOCS_REFERENCE/image-formatting)           |
| discovery_splash           | ?string                                                                             | [discovery splash hash](#DOCS_REFERENCE/image-formatting) |
| emojis                     | array of [emoji](#DOCS_RESOURCES_EMOJI/emoji-object) objects                        | custom guild emojis                                       |
| features                   | array of [guild feature](#DOCS_RESOURCES_GUILD/guild-object-guild-features) strings | enabled guild features                                    |
| approximate_member_count   | integer                                                                             | approximate number of members in this guild               |
| approximate_presence_count | integer                                                                             | approximate number of online members in this guild        |
| description                | ?string                                                                             | the description for the guild                             |

###### Example Guild Preview

```json
{
  "id": "197038439483310086",
  "name": "Discord Testers",
  "icon": "f64c482b807da4f539cff778d174971c",
  "splash": null,
  "discovery_splash": null,
  "emojis": [],
  "features": [
    "DISCOVERABLE",
    "VANITY_URL",
    "ANIMATED_ICON",
    "INVITE_SPLASH",
    "NEWS",
    "PUBLIC",
    "BANNER",
    "VERIFIED",
    "MORE_EMOJI"
  ],
  "approximate_member_count": 60814,
  "approximate_presence_count": 20034,
  "description": "The official place to report Discord Bugs!"
}
```

### Guild Widget Object

###### Guild Widget Structure

| Field      | Type       | Description                  |
| ---------- | ---------- | ---------------------------- |
| enabled    | boolean    | whether the widget is enabled |
| channel_id | ?snowflake | the widget channel id         |

###### Example Guild Widget

```json
{
  "enabled": true,
  "channel_id": "41771983444115456"
}
```

### Guild Member Object

###### Guild Member Structure

| Field          | Type                                            | Description                                                                                                                |
| -------------- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| user?          | [user](#DOCS_RESOURCES_USER/user-object) object | the user this guild member represents                                                                                      |
| nick           | ?string                                         | this users guild nickname                                                                                                  |
| roles          | array of snowflakes                             | array of [role](#DOCS_TOPICS_PERMISSIONS/role-object) object ids                                                           |
| joined_at      | ISO8601 timestamp                               | when the user joined the guild                                                                                             |
| premium_since? | ?ISO8601 timestamp                              | when the user started [boosting](https://support.discord.com/hc/en-us/articles/360028038352-Server-Boosting-) the guild |
| deaf           | boolean                                         | whether the user is deafened in voice channels                                                                             |
| mute           | boolean                                         | whether the user is muted in voice channels                                                                                |

> info
> The field `user` won't be included in the member object attached to `MESSAGE_CREATE` and `MESSAGE_UPDATE` gateway events.

###### Example Guild Member

```json
{
  "user": {},
  "nick": "NOT API SUPPORT",
  "roles": [],
  "joined_at": "2015-04-26T06:26:56.936000+00:00",
  "deaf": false,
  "mute": false
}
```

### Integration Object

###### Integration Structure

| Field               | Type                                                                                                 | Description                                                                     |
| ------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| id                  | snowflake                                                                                            | integration id                                                                  |
| name                | string                                                                                               | integration name                                                                |
| type                | string                                                                                               | integration type (twitch, youtube, etc)                                         |
| enabled             | boolean                                                                                              | is this integration enabled                                                     |
| syncing             | boolean                                                                                              | is this integration syncing                                                     |
| role_id             | snowflake                                                                                            | id that this integration uses for "subscribers"                                 |
| enable_emoticons?   | boolean                                                                                              | whether emoticons should be synced for this integration (twitch only currently) |
| expire_behavior     | [integration expire behavior](#DOCS_RESOURCES_GUILD/integration-object-integration-expire-behaviors) | the behavior of expiring subscribers                                            |
| expire_grace_period | integer                                                                                              | the grace period (in days) before expiring subscribers                          |
| user                | [user](#DOCS_RESOURCES_USER/user-object) object                                                      | user for this integration                                                       |
| account             | [account](#DOCS_RESOURCES_GUILD/integration-account-object) object                                   | integration account information                                                 |
| synced_at           | ISO8601 timestamp                                                                                    | when this integration was last synced                                           |

###### Integration Expire Behavior

| Value (integer) | Name        |
| --------------- | ----------- |
| 0               | Remove role |
| 1               | Kick        |

### Integration Account Object

###### Integration Account Structure

| Field | Type   | Description         |
| ----- | ------ | ------------------- |
| id    | string | id of the account   |
| name  | string | name of the account |

### Ban Object

###### Ban Structure

| Field  | Type                                            | Description            |
| ------ | ----------------------------------------------- | ---------------------- |
| reason | ?string                                         | the reason for the ban |
| user   | [user](#DOCS_RESOURCES_USER/user-object) object | the banned user        |

###### Example Ban

```json
{
  "reason": "mentioning b1nzy",
  "user": {
    "username": "Mason",
    "discriminator": "9999",
    "id": "53908099506183680",
    "avatar": "a_bab14f271d565501444b2ca3be944b25"
  }
}
```

## Create Guild % POST /guilds

Create a new guild. Returns a [guild](#DOCS_RESOURCES_GUILD/guild-object) object on success. Fires a [Guild Create](#DOCS_TOPICS_GATEWAY/guild-create) Gateway event.

> warn
> This endpoint can be used only by bots in less than 10 guilds.

###### JSON Params

| Field                          | Type                                                                       | Description                                                                                                 |
| ------------------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| name                           | string                                                                     | name of the guild (2-100 characters)                                                                        |
| region?                        | string                                                                     | [voice region](#DOCS_RESOURCES_VOICE/voice-region-object) id                                                |
| icon?                          | [image data](#DOCS_REFERENCE/image-data)                                   | base64 128x128 image for the guild icon                                                                     |
| verification_level?            | integer                                                                    | [verification level](#DOCS_RESOURCES_GUILD/guild-object-verification-level)                                 |
| default_message_notifications? | integer                                                                    | default [message notification level](#DOCS_RESOURCES_GUILD/guild-object-default-message-notification-level) |
| explicit_content_filter?       | integer                                                                    | [explicit content filter level](#DOCS_RESOURCES_GUILD/guild-object-explicit-content-filter-level)           |
| roles?                         | array of [role](#DOCS_TOPICS_PERMISSIONS/role-object) objects              | new guild roles                                                                                             |
| channels?                      | array of partial [channel](#DOCS_RESOURCES_CHANNEL/channel-object) objects | new guild's channels                                                                                        |
| afk_channel_id?                | snowflake                                                                  | id for afk channel                                                                                          |
| afk_timeout?                   | integer                                                                    | afk timeout in seconds                                                                                      |
| system_channel_id?             | snowflake                                                                  | the id of the channel where guild notices such as welcome messages and boost events are posted              |

> warn
> When using the `roles` parameter, the first member of the array is used to change properties of the guild's `@everyone` role. If you are trying to bootstrap a guild with additional roles, keep this in mind.

> info
> When using the `roles` parameter, the required `id` field within each role object is an integer placeholder, and will be replaced by the API upon consumption. Its purpose is to allow you to [overwrite](#DOCS_RESOURCES_CHANNEL/overwrite-object) a role's permissions in a channel when also passing in channels with the channels array.

> warn
> When using the `channels` parameter, the `position` field is ignored, and none of the default channels are created.

> info
> When using the `channels` parameter, the `id` field within each channel object may be set to an integer placeholder, and will be replaced by the API upon consumption. Its purpose is to allow you to create `GUILD_CATEGORY` channels by setting the `parent_id` field on any children to the category's `id` field. Category channels must be listed before any children.

###### Example Partial Channel Object

```json
{
  "name": "naming-things-is-hard",
  "type": 0
}
```

###### Example Category Channel

```json
{
  "name": "my-category",
  "type": 4,
  "id": 1
}
{
  "name": "naming-things-is-hard",
  "type": 0,
  "id": 2,
  "parent_id": 1
}
```

## Get Guild % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}

Returns the [guild](#DOCS_RESOURCES_GUILD/guild-object) object for the given id. If `with_counts` is set to `true`, this endpoint will also return `approximate_member_count` and `approximate_presence_count` for the guild.

###### Query String Params

| Field        | Type    | Description                                                                   | Required | Default |
| ------------ | ------- | ----------------------------------------------------------------------------- | -------- | ------- |
| with_counts? | boolean | when `true`, will return approximate member and presence counts for the guild | false    | false   |

###### Example Response

```json
{
  "id": "2909267986263572999",
  "name": "Mason's Test Server",
  "icon": "389030ec9db118cb5b85a732333b7c98",
  "description": null,
  "splash": "75610b05a0dd09ec2c3c7df9f6975ea0",
  "discovery_splash": null,
  "approximate_member_count": 2,
  "approximate_presence_count": 2,
  "features": ["INVITE_SPLASH", "VANITY_URL", "COMMERCE", "BANNER", "NEWS", "VERIFIED", "VIP_REGIONS"],
  "emojis": [
    {
      "name": "ultrafastparrot",
      "roles": [],
      "id": "393564762228785161",
      "require_colons": true,
      "managed": false,
      "animated": true,
      "available": true
    }
  ],
  "banner": "5c3cb8d1bc159937fffe7e641ec96ca7",
  "owner_id": "53908232506183680",
  "application_id": null,
  "region": "us-east",
  "afk_channel_id": null,
  "afk_timeout": 300,
  "system_channel_id": null,
  "widget_enabled": true,
  "widget_channel_id": "639513352485470208",
  "verification_level": 0,
  "roles": [
    {
      "id": "2909267986263572999",
      "name": "@everyone",
      "permissions": 49794752,
      "position": 0,
      "color": 0,
      "hoist": false,
      "managed": false,
      "mentionable": false
    }
  ],
  "default_message_notifications": 1,
  "mfa_level": 0,
  "explicit_content_filter": 0,
  "max_presences": null,
  "max_members": 250000,
  "max_video_channel_users": 25,
  "vanity_url_code": "no",
  "premium_tier": 0,
  "premium_subscription_count": 0,
  "system_channel_flags": 0,
  "preferred_locale": "en-US",
  "rules_channel_id": null,
  "public_updates_channel_id": null,
  "embed_enabled": true,
  "embed_channel_id": "639513352485470999"
}
```

## Get Guild Preview % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/preview

Returns the [guild preview](#DOCS_RESOURCES_GUILD/guild-preview-object) object for the given id, even if the user is not in the guild.

> info
> This endpoint is only for Public guilds

## Modify Guild % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}

Modify a guild's settings. Requires the `MANAGE_GUILD` permission. Returns the updated [guild](#DOCS_RESOURCES_GUILD/guild-object) object on success. Fires a [Guild Update](#DOCS_TOPICS_GATEWAY/guild-update) Gateway event.

> info
> All parameters to this endpoint are optional

###### JSON Params

| Field                         | Type                                      | Description                                                                                                              |
| ----------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| name                          | string                                    | guild name                                                                                                               |
| region                        | ?string                                   | guild [voice region](#DOCS_RESOURCES_VOICE/voice-region-object) id                                                       |
| verification_level            | ?integer                                  | [verification level](#DOCS_RESOURCES_GUILD/guild-object-verification-level)                                              |
| default_message_notifications | ?integer                                  | default [message notification level](#DOCS_RESOURCES_GUILD/guild-object-default-message-notification-level)              |
| explicit_content_filter       | ?integer                                  | [explicit content filter level](#DOCS_RESOURCES_GUILD/guild-object-explicit-content-filter-level)                        |
| afk_channel_id                | ?snowflake                                | id for afk channel                                                                                                       |
| afk_timeout                   | integer                                   | afk timeout in seconds                                                                                                   |
| icon                          | ?[image data](#DOCS_REFERENCE/image-data) | base64 1024x1024 png/jpeg/gif image for the guild icon (can be animated gif when the server has `ANIMATED_ICON` feature) |
| owner_id                      | snowflake                                 | user id to transfer guild ownership to (must be owner)                                                                   |
| splash                        | ?[image data](#DOCS_REFERENCE/image-data) | base64 16:9 png/jpeg image for the guild splash (when the server has `INVITE_SPLASH` feature)                            |
| banner                        | ?[image data](#DOCS_REFERENCE/image-data) | base64 16:9 png/jpeg image for the guild banner (when the server has `BANNER` feature)                                   |
| system_channel_id             | ?snowflake                                | the id of the channel where guild notices such as welcome messages and boost events are posted                           |
| rules_channel_id              | ?snowflake                                | the id of the channel where "PUBLIC" guilds display rules and/or guidelines                                              |
| public_updates_channel_id     | ?snowflake                                | the id of the channel where admins and moderators of "PUBLIC" guilds receive notices from Discord                        |
| preferred_locale              | ?string                                   | the preferred locale of a "PUBLIC" guild used in server discovery and notices from Discord; defaults to "en-US"          |

## Delete Guild % DELETE /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}

Delete a guild permanently. User must be owner. Returns `204 No Content` on success. Fires a [Guild Delete](#DOCS_TOPICS_GATEWAY/guild-delete) Gateway event.

## Get Guild Channels % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/channels

Returns a list of guild [channel](#DOCS_RESOURCES_CHANNEL/channel-object) objects.

## Create Guild Channel % POST /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/channels

Create a new [channel](#DOCS_RESOURCES_CHANNEL/channel-object) object for the guild. Requires the `MANAGE_CHANNELS` permission. Returns the new [channel](#DOCS_RESOURCES_CHANNEL/channel-object) object on success. Fires a [Channel Create](#DOCS_TOPICS_GATEWAY/channel-create) Gateway event.

> info
> All parameters to this endpoint are optional excluding 'name'

###### JSON Params

| Field                 | Type                                                                   | Description                                                                                                                                                                     |
| --------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name                  | string                                                                 | channel name (2-100 characters)                                                                                                                                                 |
| type                  | integer                                                                | the [type of channel](#DOCS_RESOURCES_CHANNEL/channel-object-channel-types)                                                                                                     |
| topic                 | string                                                                 | channel topic (0-1024 characters)                                                                                                                                               |
| bitrate               | integer                                                                | the bitrate (in bits) of the voice channel (voice only)                                                                                                                         |
| user_limit            | integer                                                                | the user limit of the voice channel (voice only)                                                                                                                                |
| rate_limit_per_user   | integer                                                                | amount of seconds a user has to wait before sending another message (0-21600); bots, as well as users with the permission `manage_messages` or `manage_channel`, are unaffected |
| position              | integer                                                                | sorting position of the channel                                                                                                                                                 |
| permission_overwrites | array of [overwrite](#DOCS_RESOURCES_CHANNEL/overwrite-object) objects | the channel's permission overwrites                                                                                                                                             |
| parent_id             | snowflake                                                              | id of the parent category for a channel                                                                                                                                         |
| nsfw                  | boolean                                                                | whether the channel is nsfw                                                                                                                                                     |

## Modify Guild Channel Positions % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/channels

Modify the positions of a set of [channel](#DOCS_RESOURCES_CHANNEL/channel-object) objects for the guild. Requires `MANAGE_CHANNELS` permission. Returns a 204 empty response on success. Fires multiple [Channel Update](#DOCS_TOPICS_GATEWAY/channel-update) Gateway events.

> info
> Only channels to be modified are required, with the minimum being a swap between at least two channels.

This endpoint takes a JSON array of parameters in the following format:

###### JSON Params

| Field    | Type      | Description                     |
| -------- | --------- | ------------------------------- |
| id       | snowflake | channel id                      |
| position | ?integer  | sorting position of the channel |

## Get Guild Member % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/{user.id#DOCS_RESOURCES_USER/user-object}

Returns a [guild member](#DOCS_RESOURCES_GUILD/guild-member-object) object for the specified user.

## List Guild Members % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members

Returns a list of [guild member](#DOCS_RESOURCES_GUILD/guild-member-object) objects that are members of the guild.

> warn
> In the future, this endpoint will be restricted in line with our [Privileged Intents](#DOCS_TOPICS_GATEWAY/privileged-intents)

> info
> All parameters to this endpoint are optional

###### Query String Params

| Field | Type      | Description                              | Default |
| ----- | --------- | ---------------------------------------- | ------- |
| limit | integer   | max number of members to return (1-1000) | 1       |
| after | snowflake | the highest user id in the previous page | 0       |

## Add Guild Member % PUT /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/{user.id#DOCS_RESOURCES_USER/user-object}

Adds a user to the guild, provided you have a valid oauth2 access token for the user with the `guilds.join` scope. Returns a 201 Created with the [guild member](#DOCS_RESOURCES_GUILD/guild-member-object) as the body, or 204 No Content if the user is already a member of the guild. Fires a [Guild Member Add](#DOCS_TOPICS_GATEWAY/guild-member-add) Gateway event.

> info
> All parameters to this endpoint except for `access_token` are optional.

> info
> The Authorization header must be a Bot token (belonging to the same application used for authorization), and the bot must be a member of the guild with `CREATE_INSTANT_INVITE` permission.

###### JSON Params

| Field        | Type                | Description                                                                                                              | Permission       |
| ------------ | ------------------- | ------------------------------------------------------------------------------------------------------------------------ | ---------------- |
| access_token | string              | an oauth2 access token granted with the `guilds.join` to the bot's application for the user you want to add to the guild |                  |
| nick         | string              | value to set users nickname to                                                                                           | MANAGE_NICKNAMES |
| roles        | array of snowflakes | array of role ids the member is assigned                                                                                 | MANAGE_ROLES     |
| mute         | boolean             | whether the user is muted in voice channels                                                                              | MUTE_MEMBERS     |
| deaf         | boolean             | whether the user is deafened in voice channels                                                                           | DEAFEN_MEMBERS   |

## Modify Guild Member % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/{user.id#DOCS_RESOURCES_USER/user-object}

Modify attributes of a [guild member](#DOCS_RESOURCES_GUILD/guild-member-object). Returns a 204 empty response on success. Fires a [Guild Member Update](#DOCS_TOPICS_GATEWAY/guild-member-update) Gateway event. If the `channel_id` is set to null, this will force the target user to be disconnected from voice.

> info
> All parameters to this endpoint are optional and nullable. When moving members to channels, the API user _must_ have permissions to both connect to the channel and have the `MOVE_MEMBERS` permission.

###### JSON Params

| Field      | Type                | Description                                                                                            | Permission       |
| ---------- | ------------------- | ------------------------------------------------------------------------------------------------------ | ---------------- |
| nick       | string              | value to set users nickname to                                                                         | MANAGE_NICKNAMES |
| roles      | array of snowflakes | array of role ids the member is assigned                                                               | MANAGE_ROLES     |
| mute       | boolean             | whether the user is muted in voice channels. Will throw a 400 if the user is not in a voice channel    | MUTE_MEMBERS     |
| deaf       | boolean             | whether the user is deafened in voice channels. Will throw a 400 if the user is not in a voice channel | DEAFEN_MEMBERS   |
| channel_id | snowflake           | id of channel to move user to (if they are connected to voice)                                         | MOVE_MEMBERS     |

## Modify Current User Nick % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/@me/nick

Modifies the nickname of the current user in a guild. Returns a 200 with the nickname on success. Fires a [Guild Member Update](#DOCS_TOPICS_GATEWAY/guild-member-update) Gateway event.

###### JSON Params

| Field | Type    | Description                    | Permission      |
| ----- | ------- | ------------------------------ | --------------- |
| ?nick | ?string | value to set users nickname to | CHANGE_NICKNAME |

## Add Guild Member Role % PUT /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/{user.id#DOCS_RESOURCES_USER/user-object}/roles/{role.id#DOCS_TOPICS_PERMISSIONS/role-object}

Adds a role to a [guild member](#DOCS_RESOURCES_GUILD/guild-member-object). Requires the `MANAGE_ROLES` permission. Returns a 204 empty response on success. Fires a [Guild Member Update](#DOCS_TOPICS_GATEWAY/guild-member-update) Gateway event.

## Remove Guild Member Role % DELETE /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/{user.id#DOCS_RESOURCES_USER/user-object}/roles/{role.id#DOCS_TOPICS_PERMISSIONS/role-object}

Removes a role from a [guild member](#DOCS_RESOURCES_GUILD/guild-member-object). Requires the `MANAGE_ROLES` permission. Returns a 204 empty response on success. Fires a [Guild Member Update](#DOCS_TOPICS_GATEWAY/guild-member-update) Gateway event.

## Remove Guild Member % DELETE /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/members/{user.id#DOCS_RESOURCES_USER/user-object}

Remove a member from a guild. Requires `KICK_MEMBERS` permission. Returns a 204 empty response on success. Fires a [Guild Member Remove](#DOCS_TOPICS_GATEWAY/guild-member-remove) Gateway event.

## Get Guild Bans % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/bans

Returns a list of [ban](#DOCS_RESOURCES_GUILD/ban-object) objects for the users banned from this guild. Requires the `BAN_MEMBERS` permission.

## Get Guild Ban % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/bans/{user.id#DOCS_RESOURCES_USER/user-object}

Returns a [ban](#DOCS_RESOURCES_GUILD/ban-object) object for the given user or a 404 not found if the ban cannot be found. Requires the `BAN_MEMBERS` permission.

## Create Guild Ban % PUT /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/bans/{user.id#DOCS_RESOURCES_USER/user-object}

Create a guild ban, and optionally delete previous messages sent by the banned user. Requires the `BAN_MEMBERS` permission. Returns a 204 empty response on success. Fires a [Guild Ban Add](#DOCS_TOPICS_GATEWAY/guild-ban-add) Gateway event.

###### Query String Params

| Field                | Type    | Description                                 |
| -------------------- | ------- | ------------------------------------------- |
| delete-message-days? | integer | number of days to delete messages for (0-7) |
| reason?              | string  | reason for the ban                          |

## Remove Guild Ban % DELETE /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/bans/{user.id#DOCS_RESOURCES_USER/user-object}

Remove the ban for a user. Requires the `BAN_MEMBERS` permissions. Returns a 204 empty response on success. Fires a [Guild Ban Remove](#DOCS_TOPICS_GATEWAY/guild-ban-remove) Gateway event.

## Get Guild Roles % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/roles

Returns a list of [role](#DOCS_TOPICS_PERMISSIONS/role-object) objects for the guild.

## Create Guild Role % POST /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/roles

Create a new [role](#DOCS_TOPICS_PERMISSIONS/role-object) for the guild. Requires the `MANAGE_ROLES` permission. Returns the new [role](#DOCS_TOPICS_PERMISSIONS/role-object) object on success. Fires a [Guild Role Create](#DOCS_TOPICS_GATEWAY/guild-role-create) Gateway event. All JSON params are optional.

###### JSON Params

| Field       | Type    | Description                                                    | Default                        |
| ----------- | ------- | -------------------------------------------------------------- | ------------------------------ |
| name        | string  | name of the role                                               | "new role"                     |
| permissions | integer | bitwise value of the enabled/disabled permissions              | @everyone permissions in guild |
| color       | integer | RGB color value                                                | 0                              |
| hoist       | boolean | whether the role should be displayed separately in the sidebar | false                          |
| mentionable | boolean | whether the role should be mentionable                         | false                          |

## Modify Guild Role Positions % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/roles

Modify the positions of a set of [role](#DOCS_TOPICS_PERMISSIONS/role-object) objects for the guild. Requires the `MANAGE_ROLES` permission. Returns a list of all of the guild's [role](#DOCS_TOPICS_PERMISSIONS/role-object) objects on success. Fires multiple [Guild Role Update](#DOCS_TOPICS_GATEWAY/guild-role-update) Gateway events.

This endpoint takes a JSON array of parameters in the following format:

###### JSON Params

| Field     | Type      | Description                  |
| --------- | --------- | ---------------------------- |
| id        | snowflake | role                         |
| ?position | ?integer  | sorting position of the role |

## Modify Guild Role % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/roles/{role.id#DOCS_TOPICS_PERMISSIONS/role-object}

Modify a guild role. Requires the `MANAGE_ROLES` permission. Returns the updated [role](#DOCS_TOPICS_PERMISSIONS/role-object) on success. Fires a [Guild Role Update](#DOCS_TOPICS_GATEWAY/guild-role-update) Gateway event.

> info
> All parameters to this endpoint are optional and nullable.

###### JSON Params

| Field       | Type    | Description                                                    |
| ----------- | ------- | -------------------------------------------------------------- |
| name        | string  | name of the role                                               |
| permissions | integer | bitwise value of the enabled/disabled permissions              |
| color       | integer | RGB color value                                                |
| hoist       | boolean | whether the role should be displayed separately in the sidebar |
| mentionable | boolean | whether the role should be mentionable                         |

## Delete Guild Role % DELETE /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/roles/{role.id#DOCS_TOPICS_PERMISSIONS/role-object}

Delete a guild role. Requires the `MANAGE_ROLES` permission. Returns a 204 empty response on success. Fires a [Guild Role Delete](#DOCS_TOPICS_GATEWAY/guild-role-delete) Gateway event.

## Get Guild Prune Count % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/prune

Returns an object with one 'pruned' key indicating the number of members that would be removed in a prune operation. Requires the `KICK_MEMBERS` permission. 

By default, prune will not remove users with roles. You can optionally include specific roles in your prune by providing the `include_roles` parameter. Any inactive user that has a subset of the provided role(s) will be counted in the prune and users with additional roles will not.

###### Query String Params

| Field         | Type                | Description                                   | Default |
| ------------- | ------------------- | --------------------------------------------- | ------- |
| days          | integer             | number of days to count prune for (1 or more) | 7       |
| include_roles | array of snowflakes | role(s) to include                            | none    | 

## Begin Guild Prune % POST /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/prune

Begin a prune operation. Requires the `KICK_MEMBERS` permission. Returns an object with one 'pruned' key indicating the number of members that were removed in the prune operation. For large guilds it's recommended to set the `compute_prune_count` option to `false`, forcing 'pruned' to `null`. Fires multiple [Guild Member Remove](#DOCS_TOPICS_GATEWAY/guild-member-remove) Gateway events.

By default, prune will not remove users with roles. You can optionally include specific roles in your prune by providing the `include_roles` parameter. Any inactive user that has a subset of the provided role(s) will be included in the prune and users with additional roles will not.

###### Query String Params

| Field               | Type                | Description                                                | Default |
| ------------------- | ------------------- | ---------------------------------------------------------- | ------- |
| days                | integer             | number of days to prune (1 or more)                        | 7       |
| compute_prune_count | boolean             | whether 'pruned' is returned, discouraged for large guilds | true    |
| include_roles       | array of snowflakes | role(s) to include                                         | none    | 

## Get Guild Voice Regions % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/regions

Returns a list of [voice region](#DOCS_RESOURCES_VOICE/voice-region-object) objects for the guild. Unlike the similar `/voice` route, this returns VIP servers when the guild is VIP-enabled.

## Get Guild Invites % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/invites

Returns a list of [invite](#DOCS_RESOURCES_INVITE/invite-object) objects (with [invite metadata](#DOCS_RESOURCES_INVITE/invite-metadata-object)) for the guild. Requires the `MANAGE_GUILD` permission.

## Get Guild Integrations % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/integrations

Returns a list of [integration](#DOCS_RESOURCES_GUILD/integration-object) objects for the guild. Requires the `MANAGE_GUILD` permission.

## Create Guild Integration % POST /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/integrations

Attach an [integration](#DOCS_RESOURCES_GUILD/integration-object) object from the current user to the guild. Requires the `MANAGE_GUILD` permission. Returns a 204 empty response on success. Fires a [Guild Integrations Update](#DOCS_TOPICS_GATEWAY/guild-integrations-update) Gateway event.

###### JSON Params

| Field | Type      | Description          |
| ----- | --------- | -------------------- |
| type  | string    | the integration type |
| id    | snowflake | the integration id   |

## Modify Guild Integration % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/integrations/{integration.id#DOCS_RESOURCES_GUILD/integration-object}

Modify the behavior and settings of an [integration](#DOCS_RESOURCES_GUILD/integration-object) object for the guild. Requires the `MANAGE_GUILD` permission. Returns a 204 empty response on success. Fires a [Guild Integrations Update](#DOCS_TOPICS_GATEWAY/guild-integrations-update) Gateway event.

> info
> All parameters to this endpoint are optional and nullable.

###### JSON Params

| Field               | Type    | Description                                                                                                                                                                        |
| ------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| expire_behavior     | integer | the behavior when an integration subscription lapses (see the [integration expire behaviors](#DOCS_RESOURCES_GUILD/integration-object-integration-expire-behaviors) documentation) |
| expire_grace_period | integer | period (in days) where the integration will ignore lapsed subscriptions                                                                                                            |
| enable_emoticons    | boolean | whether emoticons should be synced for this integration (twitch only currently)                                                                                                    |

## Delete Guild Integration % DELETE /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/integrations/{integration.id#DOCS_RESOURCES_GUILD/integration-object}

Delete the attached [integration](#DOCS_RESOURCES_GUILD/integration-object) object for the guild. Requires the `MANAGE_GUILD` permission. Returns a 204 empty response on success. Fires a [Guild Integrations Update](#DOCS_TOPICS_GATEWAY/guild-integrations-update) Gateway event.

## Sync Guild Integration % POST /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/integrations/{integration.id#DOCS_RESOURCES_GUILD/integration-object}/sync

Sync an integration. Requires the `MANAGE_GUILD` permission. Returns a 204 empty response on success.

## Get Guild Widget % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/widget

Returns the [guild widget](#DOCS_RESOURCES_GUILD/guild-widget-object) object. Requires the `MANAGE_GUILD` permission.

## Get Guild Embed % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/embed

Same as above, but this endpoint is deprecated.

## Modify Guild Widget % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/widget

Modify a [guild widget](#DOCS_RESOURCES_GUILD/guild-widget-object) object for the guild. All attributes may be passed in with JSON and modified. Requires the `MANAGE_GUILD` permission. Returns the updated [guild widget](#DOCS_RESOURCES_GUILD/guild-widget-object) object.

## Modify Guild Embed % PATCH /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/embed

Same as above, but this endpoint is deprecated.

## Get Guild Vanity URL % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/vanity-url

Returns a partial [invite](#DOCS_RESOURCES_INVITE/invite-object) object for guilds with that feature enabled. Requires the `MANAGE_GUILD` permission. `code` will be null if a vanity url for the guild is not set.

###### Example Partial Invite Object

```json
{
  "code": "abc",
  "uses": 12
}
```

## Get Guild Widget Image % GET /guilds/{guild.id#DOCS_RESOURCES_GUILD/guild-object}/widget.png

Returns a PNG image widget for the guild. Requires no permissions or authentication.
The same documentation also applies to `embed.png`.

> info
> All parameters to this endpoint are optional.

###### Query String Params

| Field | Type   | Description                                    | Default |
| ----- | ------ | ---------------------------------------------- | ------- |
| style | string | style of the widget image returned (see below) | shield  |

###### Widget Style Options

| Value (string) | Description                                                                                                                                                    | Example                                                                                 |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| shield         | shield style widget with Discord icon and guild members online count                                                                                           | [Example](https://discord.com/api/guilds/81384788765712384/widget.png?style=shield)  |
| banner1        | large image with guild icon, name and online count. "POWERED BY DISCORD" as the footer of the widget                                                           | [Example](https://discord.com/api/guilds/81384788765712384/widget.png?style=banner1) |
| banner2        | smaller widget style with guild icon, name and online count. Split on the right with Discord logo                                                              | [Example](https://discord.com/api/guilds/81384788765712384/widget.png?style=banner2) |
| banner3        | large image with guild icon, name and online count. In the footer, Discord logo on the left and "Chat Now" on the right                                        | [Example](https://discord.com/api/guilds/81384788765712384/widget.png?style=banner3) |
| banner4        | large Discord logo at the top of the widget. Guild icon, name and online count in the middle portion of the widget and a "JOIN MY SERVER" button at the bottom | [Example](https://discord.com/api/guilds/81384788765712384/widget.png?style=banner4) |
