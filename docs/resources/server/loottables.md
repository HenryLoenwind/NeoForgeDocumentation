# Loot Tables

Loot tables are logic files which dictate what should happen when various actions or scenarios occur. Although the vanilla system deals purely with item generation, the system can be expanded to perform any number of defined actions.

## Data-Driven Tables

Most loot tables within vanilla are data driven via JSON. This means that a mod is not necessary to create a new loot table, only a [Data pack][datapack]. A full list on how to create and put these loot tables within the mod's `resources` folder can be found on the [Minecraft Wiki][wiki].

## Using a Loot Table

A loot table is referenced by its `ResourceKey<LootTable>` which points to `data/<namespace>/loot_tables/<path>.json`. The `LootTable` associated with the reference can be obtained using `ReloadableServerRegistries.Holder#getLootTable`, where `ReloadableServerRegistries.Holder` can be obtained via `MinecraftServer#reloadableRegistries`.

A loot table is always generated with given parameters. The `LootParams` contains the level the table is generated in, luck for better generation, the `LootContextParam`s which define scenario context, and any dynamic information that should occur on activation. The `LootParams` can be created using the constructor of the `LootParams.Builder` builder, and built via `LootParams.Builder#create` by passing in the `LootContextParamSet`.

A loot table may also have some context. The `LootContext` takes in the built `LootParams` and can set some random seeded instance. The context is created via the builder `LootContext.Builder` and built using `LootContext.Builder#create` by passing in a optional `ResourceLocation` representing the random instance to use.

A `LootTable` can be used to generate `ItemStack`s using one of the available methods which may take in a `LootParams` or a `LootContext`:

Method              | Description
:---:               | :---
`getRandomItemsRaw` | Consumes the items generated by the loot table. Do not use this method unless you don't want [loot modifiers][glm] to apply.
`getRandomItems`    | Returns the items generated by the loot table.
`fill`              | Fills a container with the generated loot table.

:::note
Loot tables were built for generating items, so the methods expect some handling for the `ItemStack`s.
:::

## Additional Features

NeoForge provides some additional behavior to loot tables for greater control of the system.

### `LootTableLoadEvent`

`LootTableLoadEvent` is an [event] which is fired whenever a loot table is loaded. If the event is canceled, then an empty loot table will be loaded instead.

:::info
Do **not** modify a loot table's drops through this event. Those modifications should be done using [global loot modifiers][glm].
:::

### Loot Pool Names

Loot pools can be named using the `name` key. Any non-named loot pool will be the hash code of the pool prefixed by `custom#`.

```json5
// For some loot pool
{
    "name": "example_pool", // Pool will be named 'example_pool'
    "rolls": {
        // ...
    },
    "entries": {
        // ...
    }
}
```

### Additional Context Parameters

NeoForge extends certain parameter sets to account for missing contexts which may be applicable. `LootContextParamSets#CHEST` now allows for a `LootContextParams#KILLER_ENTITY` as chest minecarts are entities which can be broken (or 'killed'). `LootContextParamSets#FISHING` also allows for a `LootContextParams#KILLER_ENTITY` since the fishing hook is also an entity which is retracted (or 'killed') when the player retrieves it.

### Multiple Items on Smelting

When using the `SmeltItemFunction`, a smelted recipe will now return the actual number of items from the result instead of a single smelted item (e.g. if a smelting recipe returns 3 items and there are 3 drops, then the result would be 9 smelted items instead of 3).

### Loot Table Id Condition

NeoForge adds an additional `LootItemCondition` which allows certain items to generate for a specific table. This is typically used within [global loot modifiers][glm].

```json5
// In some loot pool or pool entry
{
    "conditions": [
        {
            "condition": "neoforge:loot_table_id",
            // Will apply when the loot table is for dirt
            "loot_table_id": "minecraft:blocks/dirt"
        }
    ]
}
```

### Can Item Perform Ability Condition

NeoForge adds an additional `LootItemCondition` which checks whether the given `LootContextParams#TOOL` can perform the specified `ItemAbility`.

```json5
// In some loot pool or pool entry
{
    "conditions": [
        {
            "condition": "neoforge:can_item_perform_ability",
            // Will apply when the tool can strip a log like an axe
            "action": "axe_strip"
        }
    ]
}
```

[datapack]: https://minecraft.wiki/w/Data_pack
[wiki]: https://minecraft.wiki/w/Loot_table
[event]: ../../concepts/events.md#registering-an-event-handler
[glm]: ./glm.md
