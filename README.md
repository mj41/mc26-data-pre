# mc26-data — Minecraft 26.3-pre-3

Game data of Minecraft: Java Edition 26.3-pre-3 (protocol 1073742159, data version
5019, Java 25), extracted from Mojang's server jar
(`74b30963f532fa08c5f32311cc7caa0de41bf29e`) by [mc26](https://github.com/mj41/mc26) `190a89967ae4` on
2026-10-03T15:42:17Z. Details in `_meta.json`.

| file | content |
|---|---|
| `registries.json` | every registry with numeric ids (Mojang's data generator report) |
| `blocks.json`, `block_properties.json` | block states and their properties |
| `items.json` | per-item stack size and name |
| `entities.json`, `block_entities.json`, `biomes.json` | entity dimensions, block entity types, biome order |
| `packets.json`, `packet_schema.json` | packet ids per state and the typed wire layout of every packet |
| `components.json`, `component_schema.json` | item data component types and their wire schema |
| `commands.json`, `datapack.json`, `version.json` | command tree, built-in datapack list, version facts |
| `nbt_schema.json`, `save_schema.json` | the NBT shape of the registries sent at configuration time and of the save formats (rendered as `docs/registries.md` and `docs/save.md`) |
| `lang/*.json` | every language file |
| `docs/*.md` | the same, for a reader: [protocol.md](docs/protocol.md) (the frame, the primitives, the node kinds), [packets.md](docs/packets.md), [components.md](docs/components.md), [registries.md](docs/registries.md) |

JSON and its documentation, no code; one branch per Minecraft version, tags `v0.<YYN>.<n>` pin an
extraction. The README on `main` lists the branches.
