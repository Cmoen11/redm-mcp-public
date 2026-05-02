# Example prompts

Things to ask your agent once `redm-mcp` is installed. The server advertises itself via MCP `instructions` so most agents will pick the right tool automatically — these are sanity-check prompts.

## Native lookup (exact)

> What is `0x09C28F828EE674FA`?

> Look up `BOOST_PLAYER_HORSE_SPEED_FOR_TIME` and explain the parameters.

> Found `Citizen.InvokeNative(0xAEEF48CDF5B6CE7C, ...)` in some code — what does it do?

## Behavior search (semantic)

> How do I teleport a player in RedM?

> What's the right way to spawn a horse and have a ped mount it?

> I need to apply a combat style to a ped — find the relevant natives and VORP/RSGCore helpers.

## Framework / inventory

> Show me how to add an item to a player's inventory using VORP.

> What RSGCore exports exist for getting the current player object?

> How do I run a parameterized query with oxmysql?

## Community data (rdr3_discoveries)

> Find the hash for `weapon_pistol_volcanic`.

> List a few `a_c_bear_*` ped models.

> What `CPED_CONFIG_FLAG_*` values control whether a ped flees?

## Debugging

> `Citizen.InvokeNative(0x...)` is returning nil — look it up and tell me if it has a return value or needs a `Citizen.ResultAsInteger()`-style cast.

> I'm calling `SET_ENTITY_COORDS` and the ped won't move — what are the parameters and common gotchas?
