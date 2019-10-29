# Granular and shared settings in Riot

Occasionaly it is desired that from a product perspective a setting be persisted
across all Riot clients but *not* in the Matrix specification. Settings like this
are often product-specific and serve no bearing on the protocol.

Settings are simply arbitrary flags, configuration, or shared information that one
or more Riot clients need to be aware of. For example, a setting could be whether
or not a user wants to enable a certain feature (boolean) or which room IDs the user
has visited recently (array, for breadcrumbs).

This specification does not replace the existing setting structures in the Riot
clients, but instead builds upon what is already present to allow for shared settings
across the different clients. This system *should not* be used for settings which
are only applicable to a single platform.

This specification does not replace settings which are formally specified in Matrix.
For example, the user's choice of identity server is a Matrix "setting", not a setting
that would be specified here.

## Architecture

Cross-platform settings are stored in three places: room state, room account data, and
account data. For platforms which support granular settings (like riot-web), the outer
stages such as per-device, per-room-per-device, and config are kept where they are and
can optionally be used. The default values are defined for each setting here.

*Note*: A lot of this is based upon riot-web's [granular settings](https://github.com/matrix-org/matrix-react-sdk/blob/develop/docs/settings.md)
implementation.

The levels are:
* `room` - Settings stored in room state which affect all users of that room.
* `room-account` - Affects the logged in user for that room only.
* `account` - Affects the logged in user for all rooms.

The preference of each level is defined individually by each setting, though typically
it will be `room-account` before `account` before `room`.

All settings are namespaced under `im.vector.setting` as the event type/account data key.
The content of that event/account data is the setting value.

## Reading values for settings

Clients *should* cache the values of settings where possible, and update their caches when
the values change. However, there should be no problem with trying to get the value fresh
from the server each time.

## Writing values for settings

Simply change the appropriate event. For `room`-level settings, this means changing the
content of the state event. For `room-account` and `account` settings this means setting
the relevant account data.

## Settings

The settings all Riot platforms should be aware of or support are:

### Breadcrumbs

This is the list of most recently viewed rooms for showing in breadcrumbs. Because the user
will typically only be using one device at a time, collisions are rare. When collisions do
happen, we accept the fact that data might be lost and the "recent rooms" might be inaccurate.

**Event type**: `im.vector.setting.breadcrumbs`

**Levels**: Only `account`

**Content**:
```json
{
    "recent_rooms": [
        "!roomid:example.org",
        "!otherroom:example.org",
    ]
}
```

**Default**:
```json
{
    "recent_rooms": []
}
```

The first item of `recent_rooms` is the most recent. Clients should trim the array to 20 entries
when updating the array. Each time the user opens/views a room, it should be prepended to the
array.

### Selecting "no provisioning" for integration managers

When the user doesn't want to use an integration manager for provisioning things, this setting
should be set to `false`. This should have no bearing on whether or not widgets work, but when
`false` should hide the sticker picker button and integration manager buttons.

**Event type**: `im.vector.setting.integration_provisioning`

**Levels**: Only `account`

**Content**:
```json
{
    "enabled": false
}
```

**Default**:
```json
{
    "enabled": true
}
```

### Tracking which widgets the user has allowed to load

When the user has accepted that a widget can load, that option should be stored in the user's
settings. Other clients should respect this and not show the prompt if the user has already
allowed the widget.

Because account data doesn't support state keys, the widget ID is embedded into the event type.

**Event type**: `im.vector.setting.widget.<widget ID>`

**Levels**: Only `room-account`

**Content**:
```json
{
    "allowed": true
}
```

**Default**:
```json
{
    "allowed": false
}
```
