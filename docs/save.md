# Save formats

<!-- This file is a template: gen/docs/save.mc26tmpl.md, rendered by `mc26 docs`. -->

What a world of Minecraft 26.3-pre-2 keeps on disk, as `save_schema.json` describes
it. These formats have no codec: Mojang reads them key by key and writes them the same way, so
the description is read from the reader and the writer themselves, every key with the tag type
the accessor implies, whether it may be absent, and the default used when it is. A part of a
format that does have a codec (the blending data of a chunk, its tick lists, its palettes) is
described through that codec, in full.

The shapes read as on the registries page: a compound is a list of keys and what each holds, `?`
marks a key that may be absent, `id in <registry>` is a namespaced id as a string, `either` is
whichever side the tag fits, `list` a TAG_List. Where a list's elements would not all have the
same tag type, NBT writes every element as a compound and wraps a bare value under the empty
key, which is how a block-state palette holds a bare block id beside a state with properties.

## The container

A dimension's chunks are in region files, `r.<x>.<z>.mca` under its `region/` directory, one per
32×32 chunks; `entities/` and `poi/` use the same container. `nodes.json` carries it as data
(`region`): the file is a whole number of 4 KiB sectors; the first holds 1024 four-byte locations
indexed `x + z * 32` (the high three bytes the chunk's first sector, 0 when the chunk is absent,
the low byte how many sectors it spans), the second 1024 four-byte timestamps; a chunk is a
four-byte big-endian length counting what follows, a compression byte (1 gzip, 2 zlib, 3 none, 4
lz4; bit 0x80 set means the data is in `c.<x>.<z>.mcc` next to the file), then the NBT: one
named root compound with an empty name, unlike the network form, which has no name.

## Formats

`chunk` is a saved chunk of the `region/` files, `entities` the chunk of an `entities/` region
file, whose `Entities` list holds one compound per entity of the type its `id` names —
described by `entity/<id>`, one format per entity type, from the save chain of the type's
class: `Entity.load` and `Entity.save` read the keys every entity has, then each class of the
hierarchy adds its own, and a helper the tag is handed to is followed, an interface's default
method included (a villager's `Inventory`). `entity` is that common part alone. `player` is a
file under `players/data/`, the server-side player read like an entity and written by
`saveWithoutId`, the writer `PlayerDataStorage` calls: the file carries no `id`. `level` is
`level.dat`: the tag `LevelStorageAccess.saveDataTag` builds, whose `Data` is what
`PrimaryLevelData.createTag` writes — a compound a helper builds and returns is followed like
one it is handed, a list it fills is typed by what it adds to it, and a map codec stored
without a key puts its keys at that level (the data packs and the enabled features). Its
reader takes the `Dynamic` API and reads nothing the writer does not write. The world options
and the dimensions are not in `level.dat` since 26.1: they are a saved-data file of their own
(`WorldGenSettings`, below). A scalar's tag is the one the writer writes: a reader's `getIntOr`
accepts any numeric tag, and `Air` is a short on disk.

| entry | Java | shape |
|---|---|---|
| [`chunk`](#save-chunk) | `net.minecraft.world.level.chunk.storage.SerializableChunkData` | struct `SerializableChunkData` |
| [`entities`](#save-entities) | `net.minecraft.world.level.chunk.storage.EntityStorage` | struct `EntityStorage` |
| [`entity`](#save-entity) | `net.minecraft.world.entity.Entity` | struct `Entity` |
| [`entity/minecraft:acacia_boat`](#save-entity-minecraft-acacia_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:acacia_chest_boat`](#save-entity-minecraft-acacia_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:allay`](#save-entity-minecraft-allay) | `net.minecraft.world.entity.animal.allay.Allay` | struct `Allay` |
| [`entity/minecraft:area_effect_cloud`](#save-entity-minecraft-area_effect_cloud) | `net.minecraft.world.entity.AreaEffectCloud` | struct `AreaEffectCloud` |
| [`entity/minecraft:armadillo`](#save-entity-minecraft-armadillo) | `net.minecraft.world.entity.animal.armadillo.Armadillo` | struct `Armadillo` |
| [`entity/minecraft:armor_stand`](#save-entity-minecraft-armor_stand) | `net.minecraft.world.entity.decoration.ArmorStand` | struct `ArmorStand` |
| [`entity/minecraft:arrow`](#save-entity-minecraft-arrow) | `net.minecraft.world.entity.projectile.arrow.Arrow` | struct `Arrow` |
| [`entity/minecraft:axolotl`](#save-entity-minecraft-axolotl) | `net.minecraft.world.entity.animal.axolotl.Axolotl` | struct `Axolotl` |
| [`entity/minecraft:bamboo_chest_raft`](#save-entity-minecraft-bamboo_chest_raft) | `net.minecraft.world.entity.vehicle.boat.ChestRaft` | struct `ChestRaft` |
| [`entity/minecraft:bamboo_raft`](#save-entity-minecraft-bamboo_raft) | `net.minecraft.world.entity.vehicle.boat.Raft` | struct `Raft` |
| [`entity/minecraft:bat`](#save-entity-minecraft-bat) | `net.minecraft.world.entity.ambient.Bat` | struct `Bat` |
| [`entity/minecraft:bee`](#save-entity-minecraft-bee) | `net.minecraft.world.entity.animal.bee.Bee` | struct `Bee` |
| [`entity/minecraft:birch_boat`](#save-entity-minecraft-birch_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:birch_chest_boat`](#save-entity-minecraft-birch_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:blaze`](#save-entity-minecraft-blaze) | `net.minecraft.world.entity.monster.Blaze` | struct `Blaze` |
| [`entity/minecraft:block_display`](#save-entity-minecraft-block_display) | `net.minecraft.world.entity.Display$BlockDisplay` | struct `DisplayBlockDisplay` |
| [`entity/minecraft:bogged`](#save-entity-minecraft-bogged) | `net.minecraft.world.entity.monster.skeleton.Bogged` | struct `Bogged` |
| [`entity/minecraft:breeze`](#save-entity-minecraft-breeze) | `net.minecraft.world.entity.monster.breeze.Breeze` | struct `Breeze` |
| [`entity/minecraft:breeze_wind_charge`](#save-entity-minecraft-breeze_wind_charge) | `net.minecraft.world.entity.projectile.hurtingprojectile.windcharge.BreezeWindCharge` | struct `BreezeWindCharge` |
| [`entity/minecraft:camel`](#save-entity-minecraft-camel) | `net.minecraft.world.entity.animal.camel.Camel` | struct `Camel` |
| [`entity/minecraft:camel_husk`](#save-entity-minecraft-camel_husk) | `net.minecraft.world.entity.animal.camel.CamelHusk` | struct `CamelHusk` |
| [`entity/minecraft:cat`](#save-entity-minecraft-cat) | `net.minecraft.world.entity.animal.feline.Cat` | struct `Cat` |
| [`entity/minecraft:cave_spider`](#save-entity-minecraft-cave_spider) | `net.minecraft.world.entity.monster.spider.CaveSpider` | struct `CaveSpider` |
| [`entity/minecraft:cherry_boat`](#save-entity-minecraft-cherry_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:cherry_chest_boat`](#save-entity-minecraft-cherry_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:chest_minecart`](#save-entity-minecraft-chest_minecart) | `net.minecraft.world.entity.vehicle.minecart.MinecartChest` | struct `MinecartChest` |
| [`entity/minecraft:chicken`](#save-entity-minecraft-chicken) | `net.minecraft.world.entity.animal.chicken.Chicken` | struct `Chicken` |
| [`entity/minecraft:cod`](#save-entity-minecraft-cod) | `net.minecraft.world.entity.animal.fish.Cod` | struct `Cod` |
| [`entity/minecraft:command_block_minecart`](#save-entity-minecraft-command_block_minecart) | `net.minecraft.world.entity.vehicle.minecart.MinecartCommandBlock` | struct `MinecartCommandBlock` |
| [`entity/minecraft:copper_golem`](#save-entity-minecraft-copper_golem) | `net.minecraft.world.entity.animal.golem.CopperGolem` | struct `CopperGolem` |
| [`entity/minecraft:cow`](#save-entity-minecraft-cow) | `net.minecraft.world.entity.animal.cow.Cow` | struct `Cow` |
| [`entity/minecraft:creaking`](#save-entity-minecraft-creaking) | `net.minecraft.world.entity.monster.creaking.Creaking` | struct `Creaking` |
| [`entity/minecraft:creeper`](#save-entity-minecraft-creeper) | `net.minecraft.world.entity.monster.Creeper` | struct `Creeper` |
| [`entity/minecraft:cushion`](#save-entity-minecraft-cushion) | `net.minecraft.world.entity.decoration.Cushion` | struct `Cushion` |
| [`entity/minecraft:dark_oak_boat`](#save-entity-minecraft-dark_oak_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:dark_oak_chest_boat`](#save-entity-minecraft-dark_oak_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:dolphin`](#save-entity-minecraft-dolphin) | `net.minecraft.world.entity.animal.dolphin.Dolphin` | struct `Dolphin` |
| [`entity/minecraft:donkey`](#save-entity-minecraft-donkey) | `net.minecraft.world.entity.animal.equine.Donkey` | struct `Donkey` |
| [`entity/minecraft:dragon_fireball`](#save-entity-minecraft-dragon_fireball) | `net.minecraft.world.entity.projectile.hurtingprojectile.DragonFireball` | struct `DragonFireball` |
| [`entity/minecraft:drowned`](#save-entity-minecraft-drowned) | `net.minecraft.world.entity.monster.zombie.Drowned` | struct `Drowned` |
| [`entity/minecraft:egg`](#save-entity-minecraft-egg) | `net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownEgg` | struct `ThrownEgg` |
| [`entity/minecraft:elder_guardian`](#save-entity-minecraft-elder_guardian) | `net.minecraft.world.entity.monster.ElderGuardian` | struct `ElderGuardian` |
| [`entity/minecraft:end_crystal`](#save-entity-minecraft-end_crystal) | `net.minecraft.world.entity.boss.enderdragon.EndCrystal` | struct `EndCrystal` |
| [`entity/minecraft:ender_dragon`](#save-entity-minecraft-ender_dragon) | `net.minecraft.world.entity.boss.enderdragon.EnderDragon` | struct `EnderDragon` |
| [`entity/minecraft:ender_pearl`](#save-entity-minecraft-ender_pearl) | `net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownEnderpearl` | struct `ThrownEnderpearl` |
| [`entity/minecraft:enderman`](#save-entity-minecraft-enderman) | `net.minecraft.world.entity.monster.Enderman` | struct `Enderman` |
| [`entity/minecraft:endermite`](#save-entity-minecraft-endermite) | `net.minecraft.world.entity.monster.Endermite` | struct `Endermite` |
| [`entity/minecraft:evoker`](#save-entity-minecraft-evoker) | `net.minecraft.world.entity.monster.illager.Evoker` | struct `Evoker` |
| [`entity/minecraft:evoker_fangs`](#save-entity-minecraft-evoker_fangs) | `net.minecraft.world.entity.projectile.EvokerFangs` | struct `EvokerFangs` |
| [`entity/minecraft:experience_bottle`](#save-entity-minecraft-experience_bottle) | `net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownExperienceBottle` | struct `ThrownExperienceBottle` |
| [`entity/minecraft:experience_orb`](#save-entity-minecraft-experience_orb) | `net.minecraft.world.entity.ExperienceOrb` | struct `ExperienceOrb` |
| [`entity/minecraft:eye_of_ender`](#save-entity-minecraft-eye_of_ender) | `net.minecraft.world.entity.projectile.EyeOfEnder` | struct `EyeOfEnder` |
| [`entity/minecraft:falling_block`](#save-entity-minecraft-falling_block) | `net.minecraft.world.entity.item.FallingBlockEntity` | struct `FallingBlockEntity` |
| [`entity/minecraft:fireball`](#save-entity-minecraft-fireball) | `net.minecraft.world.entity.projectile.hurtingprojectile.LargeFireball` | struct `LargeFireball` |
| [`entity/minecraft:firework_rocket`](#save-entity-minecraft-firework_rocket) | `net.minecraft.world.entity.projectile.FireworkRocketEntity` | struct `FireworkRocketEntity` |
| [`entity/minecraft:fishing_bobber`](#save-entity-minecraft-fishing_bobber) | `net.minecraft.world.entity.projectile.FishingHook` | struct `FishingHook` |
| [`entity/minecraft:fox`](#save-entity-minecraft-fox) | `net.minecraft.world.entity.animal.fox.Fox` | struct `Fox` |
| [`entity/minecraft:frog`](#save-entity-minecraft-frog) | `net.minecraft.world.entity.animal.frog.Frog` | struct `Frog` |
| [`entity/minecraft:furnace_minecart`](#save-entity-minecraft-furnace_minecart) | `net.minecraft.world.entity.vehicle.minecart.MinecartFurnace` | struct `MinecartFurnace` |
| [`entity/minecraft:ghast`](#save-entity-minecraft-ghast) | `net.minecraft.world.entity.monster.Ghast` | struct `Ghast` |
| [`entity/minecraft:giant`](#save-entity-minecraft-giant) | `net.minecraft.world.entity.monster.Giant` | struct `Giant` |
| [`entity/minecraft:glow_item_frame`](#save-entity-minecraft-glow_item_frame) | `net.minecraft.world.entity.decoration.GlowItemFrame` | struct `GlowItemFrame` |
| [`entity/minecraft:glow_squid`](#save-entity-minecraft-glow_squid) | `net.minecraft.world.entity.animal.squid.GlowSquid` | struct `GlowSquid` |
| [`entity/minecraft:goat`](#save-entity-minecraft-goat) | `net.minecraft.world.entity.animal.goat.Goat` | struct `Goat` |
| [`entity/minecraft:guardian`](#save-entity-minecraft-guardian) | `net.minecraft.world.entity.monster.Guardian` | struct `Guardian` |
| [`entity/minecraft:happy_ghast`](#save-entity-minecraft-happy_ghast) | `net.minecraft.world.entity.animal.happyghast.HappyGhast` | struct `HappyGhast` |
| [`entity/minecraft:hoglin`](#save-entity-minecraft-hoglin) | `net.minecraft.world.entity.monster.hoglin.Hoglin` | struct `Hoglin` |
| [`entity/minecraft:hopper_minecart`](#save-entity-minecraft-hopper_minecart) | `net.minecraft.world.entity.vehicle.minecart.MinecartHopper` | struct `MinecartHopper` |
| [`entity/minecraft:horse`](#save-entity-minecraft-horse) | `net.minecraft.world.entity.animal.equine.Horse` | struct `Horse` |
| [`entity/minecraft:husk`](#save-entity-minecraft-husk) | `net.minecraft.world.entity.monster.zombie.Husk` | struct `Husk` |
| [`entity/minecraft:illusioner`](#save-entity-minecraft-illusioner) | `net.minecraft.world.entity.monster.illager.Illusioner` | struct `Illusioner` |
| [`entity/minecraft:interaction`](#save-entity-minecraft-interaction) | `net.minecraft.world.entity.Interaction` | struct `Interaction` |
| [`entity/minecraft:iron_golem`](#save-entity-minecraft-iron_golem) | `net.minecraft.world.entity.animal.golem.IronGolem` | struct `IronGolem` |
| [`entity/minecraft:item`](#save-entity-minecraft-item) | `net.minecraft.world.entity.item.ItemEntity` | struct `ItemEntity` |
| [`entity/minecraft:item_display`](#save-entity-minecraft-item_display) | `net.minecraft.world.entity.Display$ItemDisplay` | struct `DisplayItemDisplay` |
| [`entity/minecraft:item_frame`](#save-entity-minecraft-item_frame) | `net.minecraft.world.entity.decoration.ItemFrame` | struct `ItemFrame` |
| [`entity/minecraft:jungle_boat`](#save-entity-minecraft-jungle_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:jungle_chest_boat`](#save-entity-minecraft-jungle_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:leash_knot`](#save-entity-minecraft-leash_knot) | `net.minecraft.world.entity.decoration.LeashFenceKnotEntity` | struct `LeashFenceKnotEntity` |
| [`entity/minecraft:lightning_bolt`](#save-entity-minecraft-lightning_bolt) | `net.minecraft.world.entity.LightningBolt` | struct `LightningBolt` |
| [`entity/minecraft:lingering_potion`](#save-entity-minecraft-lingering_potion) | `net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownLingeringPotion` | struct `ThrownLingeringPotion` |
| [`entity/minecraft:llama`](#save-entity-minecraft-llama) | `net.minecraft.world.entity.animal.equine.Llama` | struct `Llama` |
| [`entity/minecraft:llama_spit`](#save-entity-minecraft-llama_spit) | `net.minecraft.world.entity.projectile.LlamaSpit` | struct `LlamaSpit` |
| [`entity/minecraft:magma_cube`](#save-entity-minecraft-magma_cube) | `net.minecraft.world.entity.monster.cubemob.MagmaCube` | struct `MagmaCube` |
| [`entity/minecraft:mangrove_boat`](#save-entity-minecraft-mangrove_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:mangrove_chest_boat`](#save-entity-minecraft-mangrove_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:mannequin`](#save-entity-minecraft-mannequin) | `net.minecraft.world.entity.decoration.Mannequin` | struct `Mannequin` |
| [`entity/minecraft:marker`](#save-entity-minecraft-marker) | `net.minecraft.world.entity.Marker` | struct `Marker` |
| [`entity/minecraft:minecart`](#save-entity-minecraft-minecart) | `net.minecraft.world.entity.vehicle.minecart.Minecart` | struct `Minecart` |
| [`entity/minecraft:mooshroom`](#save-entity-minecraft-mooshroom) | `net.minecraft.world.entity.animal.cow.MushroomCow` | struct `MushroomCow` |
| [`entity/minecraft:mule`](#save-entity-minecraft-mule) | `net.minecraft.world.entity.animal.equine.Mule` | struct `Mule` |
| [`entity/minecraft:nautilus`](#save-entity-minecraft-nautilus) | `net.minecraft.world.entity.animal.nautilus.Nautilus` | struct `Nautilus` |
| [`entity/minecraft:oak_boat`](#save-entity-minecraft-oak_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:oak_chest_boat`](#save-entity-minecraft-oak_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:ocelot`](#save-entity-minecraft-ocelot) | `net.minecraft.world.entity.animal.feline.Ocelot` | struct `Ocelot` |
| [`entity/minecraft:ominous_item_spawner`](#save-entity-minecraft-ominous_item_spawner) | `net.minecraft.world.entity.OminousItemSpawner` | struct `OminousItemSpawner` |
| [`entity/minecraft:painting`](#save-entity-minecraft-painting) | `net.minecraft.world.entity.decoration.painting.Painting` | struct `Painting` |
| [`entity/minecraft:pale_oak_boat`](#save-entity-minecraft-pale_oak_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:pale_oak_chest_boat`](#save-entity-minecraft-pale_oak_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:panda`](#save-entity-minecraft-panda) | `net.minecraft.world.entity.animal.panda.Panda` | struct `Panda` |
| [`entity/minecraft:parched`](#save-entity-minecraft-parched) | `net.minecraft.world.entity.monster.skeleton.Parched` | struct `Parched` |
| [`entity/minecraft:parrot`](#save-entity-minecraft-parrot) | `net.minecraft.world.entity.animal.parrot.Parrot` | struct `Parrot` |
| [`entity/minecraft:phantom`](#save-entity-minecraft-phantom) | `net.minecraft.world.entity.monster.Phantom` | struct `Phantom` |
| [`entity/minecraft:pig`](#save-entity-minecraft-pig) | `net.minecraft.world.entity.animal.pig.Pig` | struct `Pig` |
| [`entity/minecraft:piglin`](#save-entity-minecraft-piglin) | `net.minecraft.world.entity.monster.piglin.Piglin` | struct `Piglin` |
| [`entity/minecraft:piglin_brute`](#save-entity-minecraft-piglin_brute) | `net.minecraft.world.entity.monster.piglin.PiglinBrute` | struct `PiglinBrute` |
| [`entity/minecraft:pillager`](#save-entity-minecraft-pillager) | `net.minecraft.world.entity.monster.illager.Pillager` | struct `Pillager` |
| [`entity/minecraft:player`](#save-entity-minecraft-player) | `net.minecraft.world.entity.player.Player` | struct `Player` |
| [`entity/minecraft:polar_bear`](#save-entity-minecraft-polar_bear) | `net.minecraft.world.entity.animal.polarbear.PolarBear` | struct `PolarBear` |
| [`entity/minecraft:poplar_boat`](#save-entity-minecraft-poplar_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:poplar_chest_boat`](#save-entity-minecraft-poplar_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:pufferfish`](#save-entity-minecraft-pufferfish) | `net.minecraft.world.entity.animal.fish.Pufferfish` | struct `Pufferfish` |
| [`entity/minecraft:rabbit`](#save-entity-minecraft-rabbit) | `net.minecraft.world.entity.animal.rabbit.Rabbit` | struct `Rabbit` |
| [`entity/minecraft:ravager`](#save-entity-minecraft-ravager) | `net.minecraft.world.entity.monster.Ravager` | struct `Ravager` |
| [`entity/minecraft:salmon`](#save-entity-minecraft-salmon) | `net.minecraft.world.entity.animal.fish.Salmon` | struct `Salmon` |
| [`entity/minecraft:sheep`](#save-entity-minecraft-sheep) | `net.minecraft.world.entity.animal.sheep.Sheep` | struct `Sheep` |
| [`entity/minecraft:shulker`](#save-entity-minecraft-shulker) | `net.minecraft.world.entity.monster.Shulker` | struct `Shulker` |
| [`entity/minecraft:shulker_bullet`](#save-entity-minecraft-shulker_bullet) | `net.minecraft.world.entity.projectile.ShulkerBullet` | struct `ShulkerBullet` |
| [`entity/minecraft:silverfish`](#save-entity-minecraft-silverfish) | `net.minecraft.world.entity.monster.Silverfish` | struct `Silverfish` |
| [`entity/minecraft:skeleton`](#save-entity-minecraft-skeleton) | `net.minecraft.world.entity.monster.skeleton.Skeleton` | struct `Skeleton` |
| [`entity/minecraft:skeleton_horse`](#save-entity-minecraft-skeleton_horse) | `net.minecraft.world.entity.animal.equine.SkeletonHorse` | struct `SkeletonHorse` |
| [`entity/minecraft:slime`](#save-entity-minecraft-slime) | `net.minecraft.world.entity.monster.cubemob.Slime` | struct `Slime` |
| [`entity/minecraft:small_fireball`](#save-entity-minecraft-small_fireball) | `net.minecraft.world.entity.projectile.hurtingprojectile.SmallFireball` | struct `SmallFireball` |
| [`entity/minecraft:sniffer`](#save-entity-minecraft-sniffer) | `net.minecraft.world.entity.animal.sniffer.Sniffer` | struct `Sniffer` |
| [`entity/minecraft:snow_golem`](#save-entity-minecraft-snow_golem) | `net.minecraft.world.entity.animal.golem.SnowGolem` | struct `SnowGolem` |
| [`entity/minecraft:snowball`](#save-entity-minecraft-snowball) | `net.minecraft.world.entity.projectile.throwableitemprojectile.Snowball` | struct `Snowball` |
| [`entity/minecraft:spawner_minecart`](#save-entity-minecraft-spawner_minecart) | `net.minecraft.world.entity.vehicle.minecart.MinecartSpawner` | struct `MinecartSpawner` |
| [`entity/minecraft:spectral_arrow`](#save-entity-minecraft-spectral_arrow) | `net.minecraft.world.entity.projectile.arrow.SpectralArrow` | struct `SpectralArrow` |
| [`entity/minecraft:spider`](#save-entity-minecraft-spider) | `net.minecraft.world.entity.monster.spider.Spider` | struct `Spider` |
| [`entity/minecraft:splash_potion`](#save-entity-minecraft-splash_potion) | `net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownSplashPotion` | struct `ThrownSplashPotion` |
| [`entity/minecraft:spruce_boat`](#save-entity-minecraft-spruce_boat) | `net.minecraft.world.entity.vehicle.boat.Boat` | struct `Boat` |
| [`entity/minecraft:spruce_chest_boat`](#save-entity-minecraft-spruce_chest_boat) | `net.minecraft.world.entity.vehicle.boat.ChestBoat` | struct `ChestBoat` |
| [`entity/minecraft:squid`](#save-entity-minecraft-squid) | `net.minecraft.world.entity.animal.squid.Squid` | struct `Squid` |
| [`entity/minecraft:stray`](#save-entity-minecraft-stray) | `net.minecraft.world.entity.monster.skeleton.Stray` | struct `Stray` |
| [`entity/minecraft:strider`](#save-entity-minecraft-strider) | `net.minecraft.world.entity.monster.Strider` | struct `Strider` |
| [`entity/minecraft:sulfur_cube`](#save-entity-minecraft-sulfur_cube) | `net.minecraft.world.entity.monster.cubemob.SulfurCube` | struct `SulfurCube` |
| [`entity/minecraft:tadpole`](#save-entity-minecraft-tadpole) | `net.minecraft.world.entity.animal.frog.Tadpole` | struct `Tadpole` |
| [`entity/minecraft:text_display`](#save-entity-minecraft-text_display) | `net.minecraft.world.entity.Display$TextDisplay` | struct `DisplayTextDisplay` |
| [`entity/minecraft:tnt`](#save-entity-minecraft-tnt) | `net.minecraft.world.entity.item.PrimedTnt` | struct `PrimedTnt` |
| [`entity/minecraft:tnt_minecart`](#save-entity-minecraft-tnt_minecart) | `net.minecraft.world.entity.vehicle.minecart.MinecartTNT` | struct `MinecartTNT` |
| [`entity/minecraft:trader_llama`](#save-entity-minecraft-trader_llama) | `net.minecraft.world.entity.animal.equine.TraderLlama` | struct `TraderLlama` |
| [`entity/minecraft:trident`](#save-entity-minecraft-trident) | `net.minecraft.world.entity.projectile.arrow.ThrownTrident` | struct `ThrownTrident` |
| [`entity/minecraft:tropical_fish`](#save-entity-minecraft-tropical_fish) | `net.minecraft.world.entity.animal.fish.TropicalFish` | struct `TropicalFish` |
| [`entity/minecraft:turtle`](#save-entity-minecraft-turtle) | `net.minecraft.world.entity.animal.turtle.Turtle` | struct `Turtle` |
| [`entity/minecraft:vex`](#save-entity-minecraft-vex) | `net.minecraft.world.entity.monster.Vex` | struct `Vex` |
| [`entity/minecraft:villager`](#save-entity-minecraft-villager) | `net.minecraft.world.entity.npc.villager.Villager` | struct `Villager` |
| [`entity/minecraft:vindicator`](#save-entity-minecraft-vindicator) | `net.minecraft.world.entity.monster.illager.Vindicator` | struct `Vindicator` |
| [`entity/minecraft:wandering_trader`](#save-entity-minecraft-wandering_trader) | `net.minecraft.world.entity.npc.wanderingtrader.WanderingTrader` | struct `WanderingTrader` |
| [`entity/minecraft:warden`](#save-entity-minecraft-warden) | `net.minecraft.world.entity.monster.warden.Warden` | struct `Warden` |
| [`entity/minecraft:wind_charge`](#save-entity-minecraft-wind_charge) | `net.minecraft.world.entity.projectile.hurtingprojectile.windcharge.WindCharge` | struct `WindCharge` |
| [`entity/minecraft:witch`](#save-entity-minecraft-witch) | `net.minecraft.world.entity.monster.Witch` | struct `Witch` |
| [`entity/minecraft:wither`](#save-entity-minecraft-wither) | `net.minecraft.world.entity.boss.wither.WitherBoss` | struct `WitherBoss` |
| [`entity/minecraft:wither_skeleton`](#save-entity-minecraft-wither_skeleton) | `net.minecraft.world.entity.monster.skeleton.WitherSkeleton` | struct `WitherSkeleton` |
| [`entity/minecraft:wither_skull`](#save-entity-minecraft-wither_skull) | `net.minecraft.world.entity.projectile.hurtingprojectile.WitherSkull` | struct `WitherSkull` |
| [`entity/minecraft:wolf`](#save-entity-minecraft-wolf) | `net.minecraft.world.entity.animal.wolf.Wolf` | struct `Wolf` |
| [`entity/minecraft:zoglin`](#save-entity-minecraft-zoglin) | `net.minecraft.world.entity.monster.Zoglin` | struct `Zoglin` |
| [`entity/minecraft:zombie`](#save-entity-minecraft-zombie) | `net.minecraft.world.entity.monster.zombie.Zombie` | struct `Zombie` |
| [`entity/minecraft:zombie_horse`](#save-entity-minecraft-zombie_horse) | `net.minecraft.world.entity.animal.equine.ZombieHorse` | struct `ZombieHorse` |
| [`entity/minecraft:zombie_nautilus`](#save-entity-minecraft-zombie_nautilus) | `net.minecraft.world.entity.animal.nautilus.ZombieNautilus` | struct `ZombieNautilus` |
| [`entity/minecraft:zombie_villager`](#save-entity-minecraft-zombie_villager) | `net.minecraft.world.entity.monster.zombie.ZombieVillager` | struct `ZombieVillager` |
| [`entity/minecraft:zombified_piglin`](#save-entity-minecraft-zombified_piglin) | `net.minecraft.world.entity.monster.zombie.ZombifiedPiglin` | struct `ZombifiedPiglin` |
| [`level`](#save-level) | `net.minecraft.world.level.storage.LevelStorageSource$LevelStorageAccess` | struct `LevelStorageSourceLevelStorageAccess` |
| [`player`](#save-player) | `net.minecraft.server.level.ServerPlayer` | struct `ServerPlayer` |

<a id="save-chunk"></a>
### chunk

`net.minecraft.world.level.chunk.storage.SerializableChunkData` — read from its parse, write

- `Status`?: id in minecraft:chunk_status
- `xPos`? (default 0): `INT`
- `zPos`? (default 0): `INT`
- `LastUpdate`? (default 0): `LONG`
- `InhabitedTime`? (default 0): `LONG`
- `UpgradeData`?: an NBT tag
- `isLightOn`? (default false): `BOOL`
- `blending_data`?: compound `BlendingData$Packed`
  - `min_section`: `INT`
  - `max_section`: `INT`
  - `heights`?: list of `FLOAT`
- `below_zero_retrogen`?: compound `BelowZeroRetrogen`
  - `target_status`: id in minecraft:chunk_status
  - `missing_bedrock`?: `LONG_ARRAY`
- `Heightmaps`?: an NBT tag
- `block_ticks`?: list
  - each: compound `SavedTick`
    - `i`: id in minecraft:block
    - `x`: `INT`
    - `y`: `INT`
    - `z`: `INT`
    - `t`: `INT`
    - `p`: `INT`
- `fluid_ticks`?: list
  - each: compound `SavedTick`
    - `i`: id in minecraft:fluid
    - `x`: `INT`
    - `y`: `INT`
    - `z`: `INT`
    - `t`: `INT`
    - `p`: `INT`
- `PostProcessing`?: list of an NBT tag
- `entities`?: list of an NBT tag
- `block_entities`?: list of an NBT tag
- `structures`?: an NBT tag
- `sections`?: list
  - each: compound `SerializableChunkDatasections`
    - `Y`? (default 0): `BYTE`
    - `block_states`?: compound `PalettedContainerRO$PackedData`
      - `palette`: list
        - each: one of
          - either: id in minecraft:block
          - or: compound `BlockState`
            - `id`: id in minecraft:block
            - `properties`?: map of `STRING` to `STRING`
      - `data`?: `LONG_ARRAY`
    - `biomes`?: compound `PalettedContainerRO$PackedData`
      - `palette`: list of id in minecraft:worldgen/biome
      - `data`?: `LONG_ARRAY`
    - `BlockLight`?: `BYTE_ARRAY`
    - `SkyLight`?: `BYTE_ARRAY`
- `DataVersion`: `INT`
- `yPos`: `INT`

<a id="save-entities"></a>
### entities

`net.minecraft.world.level.chunk.storage.EntityStorage` — read from its storeEntities

- `DataVersion`: `INT`
- `Entities`?: list of an NBT tag
- `Position`: `INT_ARRAY`

<a id="save-entity"></a>
### entity

`net.minecraft.world.entity.Entity` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-acacia_boat"></a>
### entity/minecraft:acacia_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-acacia_chest_boat"></a>
### entity/minecraft:acacia_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-allay"></a>
### entity/minecraft:allay

`net.minecraft.world.entity.animal.allay.Allay` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Inventory`?: list
  - each: recursive `ItemStack`
    - the codec `ItemStack`, spelled out above
- `listener`?: compound `VibrationSystem$Data`
  - `event`?: compound `VibrationInfo`
    - `game_event`: id in minecraft:game_event
    - `distance`: `FLOAT`
    - `pos`: list of `DOUBLE`
    - `source`?: `UUID`
    - `projectile_owner`?: `UUID`
  - `selector`: compound `VibrationSelector`
    - `event`?: compound `VibrationInfo`
      - `game_event`: id in minecraft:game_event
      - `distance`: `FLOAT`
      - `pos`: list of `DOUBLE`
      - `source`?: `UUID`
      - `projectile_owner`?: `UUID`
    - `tick`: `LONG`
  - `event_delay`? (default 0): `INT`
- `DuplicationCooldown`? (default 0): `LONG`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-area_effect_cloud"></a>
### entity/minecraft:area_effect_cloud

`net.minecraft.world.entity.AreaEffectCloud` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Age`? (default 0): `INT`
- `Duration`? (default -1): `INT`
- `WaitTime`? (default 20): `INT`
- `ReapplicationDelay`? (default 20): `INT`
- `DurationOnUse`? (default 0): `INT`
- `RadiusOnUse`? (default 0.0): `FLOAT`
- `RadiusPerTick`? (default 0.0): `FLOAT`
- `Radius`? (default 3.0): `FLOAT`
- `custom_particle`?: compound, `type` (id in minecraft:particle_type) selects
  - `minecraft:block`: compound `Block`
    - `block_state`: one of
      - either: id in minecraft:block
      - or: compound `BlockState`
        - `id`: id in minecraft:block
        - `properties`?: map of `STRING` to `STRING`
  - `minecraft:block_marker`: compound `BlockMarker`
    - `block_state`: one of
      - either: id in minecraft:block
      - or: compound `BlockState`
        - `id`: id in minecraft:block
        - `properties`?: map of `STRING` to `STRING`
  - `minecraft:geyser`: compound `GeyserParticleOptions`
    - `water_blocks`: `INT`
  - `minecraft:geyser_base`: compound `GeyserBaseParticleOptions`
    - `water_blocks`: `INT`
    - `burst_impulse_base`: `FLOAT`
  - `minecraft:geyser_poof`: compound `GeyserBaseParticleOptions`
    - `water_blocks`: `INT`
    - `burst_impulse_base`: `FLOAT`
  - `minecraft:geyser_plume`: compound `GeyserParticleOptions`
    - `water_blocks`: `INT`
  - `minecraft:dragon_breath`: compound `DragonBreath`
    - `power`? (default create): `FLOAT`
  - `minecraft:dust`: compound `DustParticleOptions`
    - `color`: `INT`
    - `scale`: recursive `DustParticleOptions.SCALE`: `FLOAT`
  - `minecraft:dust_color_transition`: compound `DustColorTransitionOptions`
    - `from_color`: `INT`
    - `to_color`: `INT`
    - `scale`: recursive `DustColorTransitionOptions.SCALE`: `FLOAT`
  - `minecraft:effect`: compound `SpellParticleOption`
    - `color`? (default -1): `INT`
    - `power`? (default 1.0): `FLOAT`
  - `minecraft:entity_effect`: compound `EntityEffect`
    - `color`: `INT`
  - `minecraft:falling_dust`: compound `FallingDust`
    - `block_state`: one of
      - either: id in minecraft:block
      - or: compound `BlockState`
        - `id`: id in minecraft:block
        - `properties`?: map of `STRING` to `STRING`
  - `minecraft:tinted_leaves`: compound `TintedLeaves`
    - `color`: `INT`
  - `minecraft:sculk_charge`: compound `SculkChargeParticleOptions`
    - `roll`: `FLOAT`
  - `minecraft:flash`: compound `Flash`
    - `color`: `INT`
  - `minecraft:instant_effect`: compound `SpellParticleOption`
    - `color`? (default -1): `INT`
    - `power`? (default 1.0): `FLOAT`
  - `minecraft:item`: compound `Item`
    - `item`: compound `ItemStackTemplate`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
  - `minecraft:vibration`: compound `VibrationParticleOption`
    - `destination`: compound, `type` (id in minecraft:position_source_type) selects
      - (cases not described)
    - `arrival_in_ticks`: `INT`
  - `minecraft:trail`: compound `TrailParticleOption`
    - `target`: list of `DOUBLE`
    - `color`: `INT`
    - `duration`: `INT`
  - `minecraft:shriek`: compound `ShriekParticleOption`
    - `delay`: `INT`
  - `minecraft:dust_pillar`: compound `DustPillar`
    - `block_state`: one of
      - either: id in minecraft:block
      - or: compound `BlockState`
        - `id`: id in minecraft:block
        - `properties`?: map of `STRING` to `STRING`
  - `minecraft:block_crumble`: compound `BlockCrumble`
    - `block_state`: one of
      - either: id in minecraft:block
      - or: compound `BlockState`
        - `id`: id in minecraft:block
        - `properties`?: map of `STRING` to `STRING`
- `potion_contents`?: compound `PotionContents`
  - `potion`?: id in minecraft:potion
  - `custom_color`?: `INT`
  - `custom_effects`? (default []): list
    - each: compound `MobEffectInstance`
      - `id`: id in minecraft:mob_effect
      - the codec: compound `MobEffectInstance$Details`
        - `amplifier`? (default 0): `BYTE`
        - `duration`? (default 0): `INT`
        - `ambient`? (default false): `BOOL`
        - `show_particles`? (default true): `BOOL`
        - `show_icon`?: `BOOL`
        - `hidden_effect`?: a `MobEffectInstance.Details` again
  - `custom_name`?: `STRING`
- `potion_duration_scale`? (default 1.0): `FLOAT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-armadillo"></a>
### entity/minecraft:armadillo

`net.minecraft.world.entity.animal.armadillo.Armadillo` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `state`?: enum `Armadillo$ArmadilloState` (var int, ids idle/rolling/scared/unrolling: IDLE, ROLLING, SCARED, UNROLLING)
- `scute_time`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-armor_stand"></a>
### entity/minecraft:armor_stand

`net.minecraft.world.entity.decoration.ArmorStand` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `Invisible`? (default false): `BOOL`
- `Small`? (default false): `BOOL`
- `ShowArms`? (default false): `BOOL`
- `DisabledSlots`? (default 0): `INT`
- `NoBasePlate`? (default false): `BOOL`
- `Marker`? (default false): `BOOL`
- `Pose`?: compound `ArmorStand$ArmorStandPose`
  - `Head`? (default DEFAULT_HEAD_POSE): list of `FLOAT`
  - `Body`? (default DEFAULT_BODY_POSE): list of `FLOAT`
  - `LeftArm`? (default DEFAULT_LEFT_ARM_POSE): list of `FLOAT`
  - `RightArm`? (default DEFAULT_RIGHT_ARM_POSE): list of `FLOAT`
  - `LeftLeg`? (default DEFAULT_LEFT_LEG_POSE): list of `FLOAT`
  - `RightLeg`? (default DEFAULT_RIGHT_LEG_POSE): list of `FLOAT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-arrow"></a>
### entity/minecraft:arrow

`net.minecraft.world.entity.projectile.arrow.Arrow` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `life`? (default 0): `SHORT`
- `inBlockState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `shake`? (default 0): `BYTE`
- `inGround`? (default false): `BOOL`
- `damage`? (default 2.0): `DOUBLE`
- `pickup`?: `BYTE`
- `crit`? (default false): `BOOL`
- `PierceLevel`? (default 0): `BYTE`
- `SoundEvent`?: an NBT tag
- `item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `weapon`?: recursive `ItemStack`
  - the codec `ItemStack`, spelled out above
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-axolotl"></a>
### entity/minecraft:axolotl

`net.minecraft.world.entity.animal.axolotl.Axolotl` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Variant`?: `INT`
- `FromBucket`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-bamboo_chest_raft"></a>
### entity/minecraft:bamboo_chest_raft

`net.minecraft.world.entity.vehicle.boat.ChestRaft` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-bamboo_raft"></a>
### entity/minecraft:bamboo_raft

`net.minecraft.world.entity.vehicle.boat.Raft` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-bat"></a>
### entity/minecraft:bat

`net.minecraft.world.entity.ambient.Bat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `BatFlags`? (default 0): `BYTE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-bee"></a>
### entity/minecraft:bee

`net.minecraft.world.entity.animal.bee.Bee` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `HasNectar`? (default false): `BOOL`
- `HasStung`? (default false): `BOOL`
- `TicksSincePollination`? (default 0): `INT`
- `CannotEnterHiveTicks`? (default 0): `INT`
- `CropsGrownSincePollination`? (default 0): `INT`
- `hive_pos`?: `INT_ARRAY`
- `flower_pos`?: `INT_ARRAY`
- `anger_end_time`?: `LONG`
- `AngerTime`?: `INT`
- `id`: `STRING`
- `angry_at`?: an NBT tag
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-birch_boat"></a>
### entity/minecraft:birch_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-birch_chest_boat"></a>
### entity/minecraft:birch_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-blaze"></a>
### entity/minecraft:blaze

`net.minecraft.world.entity.monster.Blaze` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-block_display"></a>
### entity/minecraft:block_display

`net.minecraft.world.entity.Display$BlockDisplay` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `transformation`?: compound `Transformation`
  - `translation`: list of `FLOAT`
  - `left_rotation`: list of `FLOAT`
  - `scale`: list of `FLOAT`
  - `right_rotation`: list of `FLOAT`
- `interpolation_duration`? (default 0): `INT`
- `start_interpolation`? (default 0): `INT`
- `teleport_duration`? (default 0): `INT`
- `billboard`?: enum `Display$BillboardConstraints` (var int, ids fixed/vertical/horizontal/center: FIXED, VERTICAL, HORIZONTAL, CENTER)
- `view_range`? (default 1.0): `FLOAT`
- `shadow_radius`? (default 0.0): `FLOAT`
- `shadow_strength`? (default 1.0): `FLOAT`
- `width`? (default 0.0): `FLOAT`
- `height`? (default 0.0): `FLOAT`
- `glow_color_override`? (default -1): `INT`
- `brightness`?: compound `Brightness`
  - `block`: `INT`
  - `sky`: `INT`
- `block_state`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-bogged"></a>
### entity/minecraft:bogged

`net.minecraft.world.entity.monster.skeleton.Bogged` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `sheared`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-breeze"></a>
### entity/minecraft:breeze

`net.minecraft.world.entity.monster.breeze.Breeze` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-breeze_wind_charge"></a>
### entity/minecraft:breeze_wind_charge

`net.minecraft.world.entity.projectile.hurtingprojectile.windcharge.BreezeWindCharge` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `acceleration_power`? (default 0.1): `DOUBLE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-camel"></a>
### entity/minecraft:camel

`net.minecraft.world.entity.animal.camel.Camel` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `LastPoseTick`? (default 0): `LONG`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-camel_husk"></a>
### entity/minecraft:camel_husk

`net.minecraft.world.entity.animal.camel.CamelHusk` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `LastPoseTick`? (default 0): `LONG`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cat"></a>
### entity/minecraft:cat

`net.minecraft.world.entity.animal.feline.Cat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Sitting`? (default false): `BOOL`
- `variant`?: `IDENTIFIER`
- `sound_variant`?: an NBT tag
- `CollarColor`?: `BYTE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cave_spider"></a>
### entity/minecraft:cave_spider

`net.minecraft.world.entity.monster.spider.CaveSpider` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cherry_boat"></a>
### entity/minecraft:cherry_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cherry_chest_boat"></a>
### entity/minecraft:cherry_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-chest_minecart"></a>
### entity/minecraft:chest_minecart

`net.minecraft.world.entity.vehicle.minecart.MinecartChest` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-chicken"></a>
### entity/minecraft:chicken

`net.minecraft.world.entity.animal.chicken.Chicken` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `IsChickenJockey`? (default false): `BOOL`
- `EggLayTime`?: `INT`
- `variant`?: `IDENTIFIER`
- `sound_variant`?: an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cod"></a>
### entity/minecraft:cod

`net.minecraft.world.entity.animal.fish.Cod` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `FromBucket`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-command_block_minecart"></a>
### entity/minecraft:command_block_minecart

`net.minecraft.world.entity.vehicle.minecart.MinecartCommandBlock` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `Command`?: `STRING`
- `SuccessCount`? (default 0): `INT`
- `TrackOutput`? (default true): `BOOL`
- `UpdateLastExecution`? (default true): `BOOL`
- `LastExecution`? (default -1): `LONG`
- `id`: `STRING`
- `LastOutput`?: a text component
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-copper_golem"></a>
### entity/minecraft:copper_golem

`net.minecraft.world.entity.animal.golem.CopperGolem` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `next_weather_age`? (default -1): `LONG`
- `weather_state`?: enum `WeatheringCopper$WeatherState` (var int, ids unaffected/exposed/weathered/oxidized: UNAFFECTED, EXPOSED, WEATHERED, OXIDIZED)
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cow"></a>
### entity/minecraft:cow

`net.minecraft.world.entity.animal.cow.Cow` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `variant`?: `IDENTIFIER`
- `sound_variant`?: an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-creaking"></a>
### entity/minecraft:creaking

`net.minecraft.world.entity.monster.creaking.Creaking` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-creeper"></a>
### entity/minecraft:creeper

`net.minecraft.world.entity.monster.Creeper` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `powered`? (default false): `BOOL`
- `Fuse`? (default 30): `SHORT`
- `ExplosionRadius`? (default 3): `BYTE`
- `ignited`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-cushion"></a>
### entity/minecraft:cushion

`net.minecraft.world.entity.decoration.Cushion` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `block_pos`?: `INT_ARRAY`
- `color`?: enum `DyeColor` (var int, ids white/orange/magenta/light_blue/yellow/lime/pink/gray/light_gray/cyan/purple/blue/brown/green/red/black: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-dark_oak_boat"></a>
### entity/minecraft:dark_oak_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-dark_oak_chest_boat"></a>
### entity/minecraft:dark_oak_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-dolphin"></a>
### entity/minecraft:dolphin

`net.minecraft.world.entity.animal.dolphin.Dolphin` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `GotFish`? (default false): `BOOL`
- `Moistness`? (default 2400): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-donkey"></a>
### entity/minecraft:donkey

`net.minecraft.world.entity.animal.equine.Donkey` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `ChestedHorse`? (default false): `BOOL`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-dragon_fireball"></a>
### entity/minecraft:dragon_fireball

`net.minecraft.world.entity.projectile.hurtingprojectile.DragonFireball` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `acceleration_power`? (default 0.1): `DOUBLE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-drowned"></a>
### entity/minecraft:drowned

`net.minecraft.world.entity.monster.zombie.Drowned` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsBaby`? (default false): `BOOL`
- `CanBreakDoors`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-egg"></a>
### entity/minecraft:egg

`net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownEgg` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-elder_guardian"></a>
### entity/minecraft:elder_guardian

`net.minecraft.world.entity.monster.ElderGuardian` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-end_crystal"></a>
### entity/minecraft:end_crystal

`net.minecraft.world.entity.boss.enderdragon.EndCrystal` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `beam_target`?: `INT_ARRAY`
- `ShowBottom`? (default true): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-ender_dragon"></a>
### entity/minecraft:ender_dragon

`net.minecraft.world.entity.boss.enderdragon.EnderDragon` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `DragonPhase`?: `INT`
- `DragonDeathTime`? (default 0): `INT`
- `sitting_damage_received`? (default 0.0): `FLOAT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-ender_pearl"></a>
### entity/minecraft:ender_pearl

`net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownEnderpearl` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-enderman"></a>
### entity/minecraft:enderman

`net.minecraft.world.entity.monster.Enderman` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `carriedBlockState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `anger_end_time`?: `LONG`
- `AngerTime`?: `INT`
- `id`: `STRING`
- `angry_at`?: an NBT tag
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-endermite"></a>
### entity/minecraft:endermite

`net.minecraft.world.entity.monster.Endermite` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Lifetime`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-evoker"></a>
### entity/minecraft:evoker

`net.minecraft.world.entity.monster.illager.Evoker` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `patrol_target`?: `INT_ARRAY`
- `PatrolLeader`? (default false): `BOOL`
- `Patrolling`? (default false): `BOOL`
- `Wave`? (default 0): `INT`
- `CanJoinRaid`? (default false): `BOOL`
- `RaidId`?: `INT`
- `SpellTicks`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-evoker_fangs"></a>
### entity/minecraft:evoker_fangs

`net.minecraft.world.entity.projectile.EvokerFangs` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Warmup`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-experience_bottle"></a>
### entity/minecraft:experience_bottle

`net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownExperienceBottle` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-experience_orb"></a>
### entity/minecraft:experience_orb

`net.minecraft.world.entity.ExperienceOrb` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Health`? (default 5): `SHORT`
- `Age`? (default 0): `SHORT`
- `Value`? (default 0): `SHORT`
- `Count`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-eye_of_ender"></a>
### entity/minecraft:eye_of_ender

`net.minecraft.world.entity.projectile.EyeOfEnder` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-falling_block"></a>
### entity/minecraft:falling_block

`net.minecraft.world.entity.item.FallingBlockEntity` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `BlockState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `Time`? (default 0): `INT`
- `HurtEntities`?: `BOOL`
- `FallHurtAmount`? (default 0.0): `FLOAT`
- `FallHurtMax`? (default 40): `INT`
- `DropItem`? (default true): `BOOL`
- `TileEntityData`?: an NBT tag
- `CancelDrop`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-fireball"></a>
### entity/minecraft:fireball

`net.minecraft.world.entity.projectile.hurtingprojectile.LargeFireball` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `acceleration_power`? (default 0.1): `DOUBLE`
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `ExplosionPower`? (default 1): `BYTE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-firework_rocket"></a>
### entity/minecraft:firework_rocket

`net.minecraft.world.entity.projectile.FireworkRocketEntity` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Life`? (default 0): `INT`
- `LifeTime`? (default 0): `INT`
- `FireworksItem`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `ShotAtAngle`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-fishing_bobber"></a>
### entity/minecraft:fishing_bobber

`net.minecraft.world.entity.projectile.FishingHook` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-fox"></a>
### entity/minecraft:fox

`net.minecraft.world.entity.animal.fox.Fox` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Trusted`?: list of `UUID`
- `Sleeping`? (default false): `BOOL`
- `Type`?: enum `Fox$Variant` (var int, ids red/snow: RED, SNOW)
- `Sitting`? (default false): `BOOL`
- `Crouching`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-frog"></a>
### entity/minecraft:frog

`net.minecraft.world.entity.animal.frog.Frog` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `variant`?: `IDENTIFIER`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-furnace_minecart"></a>
### entity/minecraft:furnace_minecart

`net.minecraft.world.entity.vehicle.minecart.MinecartFurnace` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `PushX`?: `DOUBLE`
- `PushZ`?: `DOUBLE`
- `Fuel`? (default 0): `SHORT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-ghast"></a>
### entity/minecraft:ghast

`net.minecraft.world.entity.monster.Ghast` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `ExplosionPower`? (default 1): `BYTE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-giant"></a>
### entity/minecraft:giant

`net.minecraft.world.entity.monster.Giant` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-glow_item_frame"></a>
### entity/minecraft:glow_item_frame

`net.minecraft.world.entity.decoration.GlowItemFrame` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `block_pos`?: `INT_ARRAY`
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `ItemRotation`? (default 0): `BYTE`
- `ItemDropChance`? (default 1.0): `FLOAT`
- `Facing`?: `BYTE`
- `Invisible`? (default false): `BOOL`
- `Fixed`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-glow_squid"></a>
### entity/minecraft:glow_squid

`net.minecraft.world.entity.animal.squid.GlowSquid` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `DarkTicksRemaining`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-goat"></a>
### entity/minecraft:goat

`net.minecraft.world.entity.animal.goat.Goat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `IsScreamingGoat`? (default false): `BOOL`
- `HasLeftHorn`? (default true): `BOOL`
- `HasRightHorn`? (default true): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-guardian"></a>
### entity/minecraft:guardian

`net.minecraft.world.entity.monster.Guardian` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-happy_ghast"></a>
### entity/minecraft:happy_ghast

`net.minecraft.world.entity.animal.happyghast.HappyGhast` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `still_timeout`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-hoglin"></a>
### entity/minecraft:hoglin

`net.minecraft.world.entity.monster.hoglin.Hoglin` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `IsImmuneToZombification`? (default false): `BOOL`
- `TimeInOverworld`? (default 0): `INT`
- `CannotBeHunted`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-hopper_minecart"></a>
### entity/minecraft:hopper_minecart

`net.minecraft.world.entity.vehicle.minecart.MinecartHopper` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `Enabled`? (default true): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-horse"></a>
### entity/minecraft:horse

`net.minecraft.world.entity.animal.equine.Horse` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `Variant`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-husk"></a>
### entity/minecraft:husk

`net.minecraft.world.entity.monster.zombie.Husk` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsBaby`? (default false): `BOOL`
- `CanBreakDoors`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-illusioner"></a>
### entity/minecraft:illusioner

`net.minecraft.world.entity.monster.illager.Illusioner` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `patrol_target`?: `INT_ARRAY`
- `PatrolLeader`? (default false): `BOOL`
- `Patrolling`? (default false): `BOOL`
- `Wave`? (default 0): `INT`
- `CanJoinRaid`? (default false): `BOOL`
- `RaidId`?: `INT`
- `SpellTicks`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-interaction"></a>
### entity/minecraft:interaction

`net.minecraft.world.entity.Interaction` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `width`? (default 1.0): `FLOAT`
- `height`? (default 1.0): `FLOAT`
- `attack`?: compound `Interaction$PlayerAction`
  - `player`: `UUID`
  - `timestamp`: `LONG`
- `interaction`?: compound `Interaction$PlayerAction`
  - `player`: `UUID`
  - `timestamp`: `LONG`
- `response`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-iron_golem"></a>
### entity/minecraft:iron_golem

`net.minecraft.world.entity.animal.golem.IronGolem` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `PlayerCreated`? (default false): `BOOL`
- `anger_end_time`?: `LONG`
- `AngerTime`?: `INT`
- `id`: `STRING`
- `angry_at`?: an NBT tag
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-item"></a>
### entity/minecraft:item

`net.minecraft.world.entity.item.ItemEntity` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Health`? (default 5): `SHORT`
- `Age`? (default 0): `SHORT`
- `PickupDelay`? (default 0): `SHORT`
- `Owner`?: `UUID`
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-item_display"></a>
### entity/minecraft:item_display

`net.minecraft.world.entity.Display$ItemDisplay` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `transformation`?: compound `Transformation`
  - `translation`: list of `FLOAT`
  - `left_rotation`: list of `FLOAT`
  - `scale`: list of `FLOAT`
  - `right_rotation`: list of `FLOAT`
- `interpolation_duration`? (default 0): `INT`
- `start_interpolation`? (default 0): `INT`
- `teleport_duration`? (default 0): `INT`
- `billboard`?: enum `Display$BillboardConstraints` (var int, ids fixed/vertical/horizontal/center: FIXED, VERTICAL, HORIZONTAL, CENTER)
- `view_range`? (default 1.0): `FLOAT`
- `shadow_radius`? (default 0.0): `FLOAT`
- `shadow_strength`? (default 1.0): `FLOAT`
- `width`? (default 0.0): `FLOAT`
- `height`? (default 0.0): `FLOAT`
- `glow_color_override`? (default -1): `INT`
- `brightness`?: compound `Brightness`
  - `block`: `INT`
  - `sky`: `INT`
- `item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `item_display`?: enum `ItemDisplayContext` (var int, ids none/thirdperson_lefthand/thirdperson_righthand/firstperson_lefthand/firstperson_righthand/head/gui/ground/fixed/on_shelf: NONE, THIRD_PERSON_LEFT_HAND, THIRD_PERSON_RIGHT_HAND, FIRST_PERSON_LEFT_HAND, FIRST_PERSON_RIGHT_HAND, HEAD, GUI, GROUND, FIXED, ON_SHELF)
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-item_frame"></a>
### entity/minecraft:item_frame

`net.minecraft.world.entity.decoration.ItemFrame` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `block_pos`?: `INT_ARRAY`
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `ItemRotation`? (default 0): `BYTE`
- `ItemDropChance`? (default 1.0): `FLOAT`
- `Facing`?: `BYTE`
- `Invisible`? (default false): `BOOL`
- `Fixed`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-jungle_boat"></a>
### entity/minecraft:jungle_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-jungle_chest_boat"></a>
### entity/minecraft:jungle_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-leash_knot"></a>
### entity/minecraft:leash_knot

`net.minecraft.world.entity.decoration.LeashFenceKnotEntity` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-lightning_bolt"></a>
### entity/minecraft:lightning_bolt

`net.minecraft.world.entity.LightningBolt` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-lingering_potion"></a>
### entity/minecraft:lingering_potion

`net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownLingeringPotion` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-llama"></a>
### entity/minecraft:llama

`net.minecraft.world.entity.animal.equine.Llama` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Strength`? (default 0): `INT`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `ChestedHorse`? (default false): `BOOL`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `Variant`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-llama_spit"></a>
### entity/minecraft:llama_spit

`net.minecraft.world.entity.projectile.LlamaSpit` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-magma_cube"></a>
### entity/minecraft:magma_cube

`net.minecraft.world.entity.monster.cubemob.MagmaCube` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Size`? (default 0): `INT`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `wasOnGround`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-mangrove_boat"></a>
### entity/minecraft:mangrove_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-mangrove_chest_boat"></a>
### entity/minecraft:mangrove_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-mannequin"></a>
### entity/minecraft:mannequin

`net.minecraft.world.entity.decoration.Mannequin` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `profile`?: compound `ResolvableProfile`
  - either: compound `GameProfile`
    - `id`: `UUID`
    - `name`: `STRING`
    - `properties`? (default EMPTY): one of
      - either: map of `STRING` to list of `STRING`
      - or: list
        - each: compound `ExtraCodecs`
          - `name`: `STRING`
          - `value`: `STRING`
          - `signature`?: `STRING`
  - or: compound `ResolvableProfile$Partial`
    - `name`?: `STRING`
    - `id`?: `UUID`
    - `properties`? (default EMPTY): one of
      - either: map of `STRING` to list of `STRING`
      - or: list
        - each: compound `ExtraCodecs`
          - `name`: `STRING`
          - `value`: `STRING`
          - `signature`?: `STRING`
  - `texture`?: `IDENTIFIER`
  - `cape`?: `IDENTIFIER`
  - `elytra`?: `IDENTIFIER`
  - `model`?: enum `PlayerModelType` (var int, ids slim/wide: SLIM, WIDE)
- `hidden_layers`?: list of enum `PlayerModelPart` (var int, ids cape/jacket/left_sleeve/right_sleeve/left_pants_leg/right_pants_leg/hat: CAPE, JACKET, LEFT_SLEEVE, RIGHT_SLEEVE, LEFT_PANTS_LEG, RIGHT_PANTS_LEG, HAT)
- `main_hand`?: enum `HumanoidArm` (var int, ids left/right: LEFT, RIGHT)
- `pose`?: enum `Pose` (var int, ids standing/fall_flying/sleeping/swimming/spin_attack/crouching/long_jumping/dying/croaking/using_tongue/sitting/roaring/sniffing/emerging/digging/sliding/shooting/inhaling: STANDING, FALL_FLYING, SLEEPING, SWIMMING, SPIN_ATTACK, CROUCHING, LONG_JUMPING, DYING, CROAKING, USING_TONGUE, SITTING, ROARING, SNIFFING, EMERGING, DIGGING, SLIDING, SHOOTING, INHALING)
- `immovable`? (default false): `BOOL`
- `hide_description`? (default false): `BOOL`
- `description`?: a text component
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-marker"></a>
### entity/minecraft:marker

`net.minecraft.world.entity.Marker` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-minecart"></a>
### entity/minecraft:minecart

`net.minecraft.world.entity.vehicle.minecart.Minecart` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-mooshroom"></a>
### entity/minecraft:mooshroom

`net.minecraft.world.entity.animal.cow.MushroomCow` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Type`?: enum `MushroomCow$Variant` (var int, ids red/brown: RED, BROWN)
- `stew_effects`?: list
  - each: compound `SuspiciousStewEffects$Entry`
    - `id`: id in minecraft:mob_effect
    - `duration`? (default 160): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-mule"></a>
### entity/minecraft:mule

`net.minecraft.world.entity.animal.equine.Mule` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `ChestedHorse`? (default false): `BOOL`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-nautilus"></a>
### entity/minecraft:nautilus

`net.minecraft.world.entity.animal.nautilus.Nautilus` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Sitting`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-oak_boat"></a>
### entity/minecraft:oak_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-oak_chest_boat"></a>
### entity/minecraft:oak_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-ocelot"></a>
### entity/minecraft:ocelot

`net.minecraft.world.entity.animal.feline.Ocelot` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Trusting`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-ominous_item_spawner"></a>
### entity/minecraft:ominous_item_spawner

`net.minecraft.world.entity.OminousItemSpawner` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `spawn_item_after_ticks`? (default 0): `LONG`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-painting"></a>
### entity/minecraft:painting

`net.minecraft.world.entity.decoration.painting.Painting` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `facing`?: `BYTE`
- `block_pos`?: `INT_ARRAY`
- `variant`?: `IDENTIFIER`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-pale_oak_boat"></a>
### entity/minecraft:pale_oak_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-pale_oak_chest_boat"></a>
### entity/minecraft:pale_oak_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-panda"></a>
### entity/minecraft:panda

`net.minecraft.world.entity.animal.panda.Panda` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `MainGene`?: enum `Panda$Gene` (var int, ids normal/lazy/worried/playful/brown/weak/aggressive: NORMAL, LAZY, WORRIED, PLAYFUL, BROWN, WEAK, AGGRESSIVE)
- `HiddenGene`?: enum `Panda$Gene` (var int, ids normal/lazy/worried/playful/brown/weak/aggressive: NORMAL, LAZY, WORRIED, PLAYFUL, BROWN, WEAK, AGGRESSIVE)
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-parched"></a>
### entity/minecraft:parched

`net.minecraft.world.entity.monster.skeleton.Parched` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-parrot"></a>
### entity/minecraft:parrot

`net.minecraft.world.entity.animal.parrot.Parrot` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Sitting`? (default false): `BOOL`
- `Variant`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-phantom"></a>
### entity/minecraft:phantom

`net.minecraft.world.entity.monster.Phantom` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `anchor_pos`?: `INT_ARRAY`
- `size`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-pig"></a>
### entity/minecraft:pig

`net.minecraft.world.entity.animal.pig.Pig` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `variant`?: `IDENTIFIER`
- `sound_variant`?: an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-piglin"></a>
### entity/minecraft:piglin

`net.minecraft.world.entity.monster.piglin.Piglin` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsImmuneToZombification`? (default false): `BOOL`
- `TimeInOverworld`? (default 0): `INT`
- `IsBaby`? (default false): `BOOL`
- `CannotHunt`? (default false): `BOOL`
- `Inventory`?: list
  - each: recursive `ItemStack`
    - the codec `ItemStack`, spelled out above
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-piglin_brute"></a>
### entity/minecraft:piglin_brute

`net.minecraft.world.entity.monster.piglin.PiglinBrute` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsImmuneToZombification`? (default false): `BOOL`
- `TimeInOverworld`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-pillager"></a>
### entity/minecraft:pillager

`net.minecraft.world.entity.monster.illager.Pillager` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `patrol_target`?: `INT_ARRAY`
- `PatrolLeader`? (default false): `BOOL`
- `Patrolling`? (default false): `BOOL`
- `Wave`? (default 0): `INT`
- `CanJoinRaid`? (default false): `BOOL`
- `RaidId`?: `INT`
- `Inventory`?: list
  - each: recursive `ItemStack`
    - the codec `ItemStack`, spelled out above
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-player"></a>
### entity/minecraft:player

`net.minecraft.world.entity.player.Player` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `Inventory`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `SelectedItemSlot`? (default 0): `INT`
- `SleepTimer`? (default 0): `SHORT`
- `XpP`? (default 0.0): `FLOAT`
- `XpLevel`? (default 0): `INT`
- `XpTotal`? (default 0): `INT`
- `XpSeed`? (default 0): `INT`
- `Score`? (default 0): `INT`
- `foodLevel`? (default 20): `INT`
- `foodTickTimer`? (default 0): `INT`
- `foodSaturationLevel`? (default 5.0): `FLOAT`
- `foodExhaustionLevel`? (default 0.0): `FLOAT`
- `abilities`?: compound `Abilities$Packed`
  - `invulnerable`? (default false): `BOOL`
  - `flying`? (default false): `BOOL`
  - `mayfly`? (default false): `BOOL`
  - `instabuild`? (default false): `BOOL`
  - `mayBuild`? (default true): `BOOL`
  - `flySpeed`? (default 0.05): `FLOAT`
  - `walkSpeed`? (default 0.1): `FLOAT`
- `EnderItems`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `LastDeathLocation`?: compound `GlobalPos`
  - `dimension`: resource key in minecraft:dimension
  - `pos`: `INT_ARRAY`
- `id`: `STRING`
- `DataVersion`: `INT`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-polar_bear"></a>
### entity/minecraft:polar_bear

`net.minecraft.world.entity.animal.polarbear.PolarBear` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `anger_end_time`?: `LONG`
- `AngerTime`?: `INT`
- `id`: `STRING`
- `angry_at`?: an NBT tag
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-poplar_boat"></a>
### entity/minecraft:poplar_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-poplar_chest_boat"></a>
### entity/minecraft:poplar_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-pufferfish"></a>
### entity/minecraft:pufferfish

`net.minecraft.world.entity.animal.fish.Pufferfish` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `FromBucket`? (default false): `BOOL`
- `PuffState`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-rabbit"></a>
### entity/minecraft:rabbit

`net.minecraft.world.entity.animal.rabbit.Rabbit` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `RabbitType`?: `INT`
- `MoreCarrotTicks`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-ravager"></a>
### entity/minecraft:ravager

`net.minecraft.world.entity.monster.Ravager` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `patrol_target`?: `INT_ARRAY`
- `PatrolLeader`? (default false): `BOOL`
- `Patrolling`? (default false): `BOOL`
- `Wave`? (default 0): `INT`
- `CanJoinRaid`? (default false): `BOOL`
- `RaidId`?: `INT`
- `AttackTick`? (default 0): `INT`
- `StunTick`? (default 0): `INT`
- `RoarTick`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-salmon"></a>
### entity/minecraft:salmon

`net.minecraft.world.entity.animal.fish.Salmon` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `FromBucket`? (default false): `BOOL`
- `type`?: enum `Salmon$Variant` (var int, ids small/medium/large: SMALL, MEDIUM, LARGE)
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-sheep"></a>
### entity/minecraft:sheep

`net.minecraft.world.entity.animal.sheep.Sheep` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Sheared`? (default false): `BOOL`
- `Color`?: `BYTE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-shulker"></a>
### entity/minecraft:shulker

`net.minecraft.world.entity.monster.Shulker` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `AttachFace`?: `BYTE`
- `Peek`? (default 0): `BYTE`
- `Color`? (default 16): `BYTE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-shulker_bullet"></a>
### entity/minecraft:shulker_bullet

`net.minecraft.world.entity.projectile.ShulkerBullet` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Steps`? (default 0): `INT`
- `TXD`? (default 0.0): `DOUBLE`
- `TYD`? (default 0.0): `DOUBLE`
- `TZD`? (default 0.0): `DOUBLE`
- `Dir`?: `BYTE`
- `id`: `STRING`
- `Target`: `UUID`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-silverfish"></a>
### entity/minecraft:silverfish

`net.minecraft.world.entity.monster.Silverfish` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-skeleton"></a>
### entity/minecraft:skeleton

`net.minecraft.world.entity.monster.skeleton.Skeleton` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-skeleton_horse"></a>
### entity/minecraft:skeleton_horse

`net.minecraft.world.entity.animal.equine.SkeletonHorse` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `SkeletonTrap`? (default false): `BOOL`
- `SkeletonTrapTime`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-slime"></a>
### entity/minecraft:slime

`net.minecraft.world.entity.monster.cubemob.Slime` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Size`? (default 0): `INT`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `wasOnGround`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-small_fireball"></a>
### entity/minecraft:small_fireball

`net.minecraft.world.entity.projectile.hurtingprojectile.SmallFireball` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `acceleration_power`? (default 0.1): `DOUBLE`
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-sniffer"></a>
### entity/minecraft:sniffer

`net.minecraft.world.entity.animal.sniffer.Sniffer` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-snow_golem"></a>
### entity/minecraft:snow_golem

`net.minecraft.world.entity.animal.golem.SnowGolem` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Pumpkin`? (default true): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-snowball"></a>
### entity/minecraft:snowball

`net.minecraft.world.entity.projectile.throwableitemprojectile.Snowball` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-spawner_minecart"></a>
### entity/minecraft:spawner_minecart

`net.minecraft.world.entity.vehicle.minecart.MinecartSpawner` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `Delay`? (default 20): `SHORT`
- `SpawnData`?: compound `SpawnData`
  - `entity`: an NBT tag
  - `custom_spawn_rules`?: compound `SpawnData$CustomSpawnRules`
    - `block_light_limit`? (default LIGHT_RANGE): either `INT` or list of `INT`
    - `sky_light_limit`? (default LIGHT_RANGE): either `INT` or list of `INT`
  - `equipment`?: compound `EquipmentTable`
    - `loot_table`: resource key in minecraft:loot_table
    - `slot_drop_chances`? (default {}): either `FLOAT` or map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `SpawnPotentials`?: list
  - each: compound `Weighted`
    - `data`: compound `SpawnData`
      - `entity`: an NBT tag
      - `custom_spawn_rules`?: compound `SpawnData$CustomSpawnRules`
        - `block_light_limit`? (default LIGHT_RANGE): either `INT` or list of `INT`
        - `sky_light_limit`? (default LIGHT_RANGE): either `INT` or list of `INT`
      - `equipment`?: compound `EquipmentTable`
        - `loot_table`: resource key in minecraft:loot_table
        - `slot_drop_chances`? (default {}): either `FLOAT` or map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
    - `weight`: `INT`
- `MinSpawnDelay`? (default 200): `SHORT`
- `MaxSpawnDelay`? (default 800): `SHORT`
- `SpawnCount`? (default 4): `SHORT`
- `MaxNearbyEntities`? (default 6): `SHORT`
- `RequiredPlayerRange`? (default 16): `SHORT`
- `SpawnRange`? (default 4): `SHORT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-spectral_arrow"></a>
### entity/minecraft:spectral_arrow

`net.minecraft.world.entity.projectile.arrow.SpectralArrow` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `life`? (default 0): `SHORT`
- `inBlockState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `shake`? (default 0): `BYTE`
- `inGround`? (default false): `BOOL`
- `damage`? (default 2.0): `DOUBLE`
- `pickup`?: `BYTE`
- `crit`? (default false): `BOOL`
- `PierceLevel`? (default 0): `BYTE`
- `SoundEvent`?: an NBT tag
- `item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `weapon`?: recursive `ItemStack`
  - the codec `ItemStack`, spelled out above
- `Duration`? (default 200): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-spider"></a>
### entity/minecraft:spider

`net.minecraft.world.entity.monster.spider.Spider` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-splash_potion"></a>
### entity/minecraft:splash_potion

`net.minecraft.world.entity.projectile.throwableitemprojectile.ThrownSplashPotion` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `Item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-spruce_boat"></a>
### entity/minecraft:spruce_boat

`net.minecraft.world.entity.vehicle.boat.Boat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-spruce_chest_boat"></a>
### entity/minecraft:spruce_chest_boat

`net.minecraft.world.entity.vehicle.boat.ChestBoat` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `leash`?: either field or `INT_ARRAY`
- `LootTable`?: resource key in minecraft:loot_table
- `LootTableSeed`? (default 0): `LONG`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-squid"></a>
### entity/minecraft:squid

`net.minecraft.world.entity.animal.squid.Squid` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-stray"></a>
### entity/minecraft:stray

`net.minecraft.world.entity.monster.skeleton.Stray` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-strider"></a>
### entity/minecraft:strider

`net.minecraft.world.entity.monster.Strider` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-sulfur_cube"></a>
### entity/minecraft:sulfur_cube

`net.minecraft.world.entity.monster.cubemob.SulfurCube` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `pickup_timer`? (default 0): `INT`
- `from_bucket`? (default false): `BOOL`
- `fuse`? (default -1): `INT`
- `Size`? (default 0): `INT`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `wasOnGround`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-tadpole"></a>
### entity/minecraft:tadpole

`net.minecraft.world.entity.animal.frog.Tadpole` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `FromBucket`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-text_display"></a>
### entity/minecraft:text_display

`net.minecraft.world.entity.Display$TextDisplay` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `transformation`?: compound `Transformation`
  - `translation`: list of `FLOAT`
  - `left_rotation`: list of `FLOAT`
  - `scale`: list of `FLOAT`
  - `right_rotation`: list of `FLOAT`
- `interpolation_duration`? (default 0): `INT`
- `start_interpolation`? (default 0): `INT`
- `teleport_duration`? (default 0): `INT`
- `billboard`?: enum `Display$BillboardConstraints` (var int, ids fixed/vertical/horizontal/center: FIXED, VERTICAL, HORIZONTAL, CENTER)
- `view_range`? (default 1.0): `FLOAT`
- `shadow_radius`? (default 0.0): `FLOAT`
- `shadow_strength`? (default 1.0): `FLOAT`
- `width`? (default 0.0): `FLOAT`
- `height`? (default 0.0): `FLOAT`
- `glow_color_override`? (default -1): `INT`
- `brightness`?: compound `Brightness`
  - `block`: `INT`
  - `sky`: `INT`
- `line_width`? (default 200): `INT`
- `text_opacity`? (default -1): `BYTE`
- `background`? (default 1073741824): `INT`
- `alignment`?: enum `Display$TextDisplay$Align` (var int, ids center/left/right: CENTER, LEFT, RIGHT)
- `text`?: a text component
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-tnt"></a>
### entity/minecraft:tnt

`net.minecraft.world.entity.item.PrimedTnt` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `fuse`? (default 80): `SHORT`
- `block_state`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `explosion_power`? (default 4.0): `FLOAT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-tnt_minecart"></a>
### entity/minecraft:tnt_minecart

`net.minecraft.world.entity.vehicle.minecart.MinecartTNT` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `DisplayState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `DisplayOffset`?: `INT`
- `FlippedRotation`? (default false): `BOOL`
- `HasTicked`? (default false): `BOOL`
- `fuse`? (default -1): `INT`
- `explosion_power`? (default 4.0): `FLOAT`
- `explosion_speed_factor`? (default 1.0): `FLOAT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-trader_llama"></a>
### entity/minecraft:trader_llama

`net.minecraft.world.entity.animal.equine.TraderLlama` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `Strength`? (default 0): `INT`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `ChestedHorse`? (default false): `BOOL`
- `Items`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `Variant`?: `INT`
- `DespawnDelay`? (default 47999): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-trident"></a>
### entity/minecraft:trident

`net.minecraft.world.entity.projectile.arrow.ThrownTrident` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `life`? (default 0): `SHORT`
- `inBlockState`?: one of
  - either: id in minecraft:block
  - or: compound `BlockState`
    - `id`: id in minecraft:block
    - `properties`?: map of `STRING` to `STRING`
- `shake`? (default 0): `BYTE`
- `inGround`? (default false): `BOOL`
- `damage`? (default 2.0): `DOUBLE`
- `pickup`?: `BYTE`
- `crit`? (default false): `BOOL`
- `PierceLevel`? (default 0): `BYTE`
- `SoundEvent`?: an NBT tag
- `item`?: recursive `ItemStack`
  - the codec: compound `ItemStack`
    - `id`: id in minecraft:item
    - `count`? (default 1): `INT`
    - `components`? (default EMPTY): an NBT tag
- `weapon`?: recursive `ItemStack`
  - the codec `ItemStack`, spelled out above
- `DealtDamage`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-tropical_fish"></a>
### entity/minecraft:tropical_fish

`net.minecraft.world.entity.animal.fish.TropicalFish` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `FromBucket`? (default false): `BOOL`
- `Variant`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-turtle"></a>
### entity/minecraft:turtle

`net.minecraft.world.entity.animal.turtle.Turtle` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `home_pos`?: `INT_ARRAY`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `has_egg`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-vex"></a>
### entity/minecraft:vex

`net.minecraft.world.entity.monster.Vex` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `bound_pos`?: `INT_ARRAY`
- `life_ticks`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-villager"></a>
### entity/minecraft:villager

`net.minecraft.world.entity.npc.villager.Villager` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `Offers`?: field
- `Inventory`?: list
  - each: recursive `ItemStack`
    - the codec `ItemStack`, spelled out above
- `VillagerData`?: compound `VillagerData`
  - `type`: id in minecraft:villager_type
  - `profession`: id in minecraft:villager_profession
  - `level`? (default 1): `INT`
- `VillagerDataFinalized`? (default false): `BOOL`
- `FoodLevel`? (default 0): `BYTE`
- `Gossips`?: list
  - each: compound `GossipContainer$GossipEntry`
    - `Target`: `UUID`
    - `Type`: enum `GossipType` (var int, ids major_negative/minor_negative/minor_positive/major_positive/trading: MAJOR_NEGATIVE, MINOR_NEGATIVE, MINOR_POSITIVE, MAJOR_POSITIVE, TRADING)
    - `Value`: `INT`
- `Xp`? (default 0): `INT`
- `LastRestock`? (default 0): `LONG`
- `LastGossipDecay`? (default 0): `LONG`
- `RestocksToday`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-vindicator"></a>
### entity/minecraft:vindicator

`net.minecraft.world.entity.monster.illager.Vindicator` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `patrol_target`?: `INT_ARRAY`
- `PatrolLeader`? (default false): `BOOL`
- `Patrolling`? (default false): `BOOL`
- `Wave`? (default 0): `INT`
- `CanJoinRaid`? (default false): `BOOL`
- `RaidId`?: `INT`
- `Johnny`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-wandering_trader"></a>
### entity/minecraft:wandering_trader

`net.minecraft.world.entity.npc.wanderingtrader.WanderingTrader` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `Offers`?: field
- `Inventory`?: list
  - each: recursive `ItemStack`
    - the codec `ItemStack`, spelled out above
- `DespawnDelay`? (default 0): `INT`
- `wander_target`?: `INT_ARRAY`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-warden"></a>
### entity/minecraft:warden

`net.minecraft.world.entity.monster.warden.Warden` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `anger`?: an NBT tag
- `listener`?: compound `VibrationSystem$Data`
  - `event`?: compound `VibrationInfo`
    - `game_event`: id in minecraft:game_event
    - `distance`: `FLOAT`
    - `pos`: list of `DOUBLE`
    - `source`?: `UUID`
    - `projectile_owner`?: `UUID`
  - `selector`: compound `VibrationSelector`
    - `event`?: compound `VibrationInfo`
      - `game_event`: id in minecraft:game_event
      - `distance`: `FLOAT`
      - `pos`: list of `DOUBLE`
      - `source`?: `UUID`
      - `projectile_owner`?: `UUID`
    - `tick`: `LONG`
  - `event_delay`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-wind_charge"></a>
### entity/minecraft:wind_charge

`net.minecraft.world.entity.projectile.hurtingprojectile.windcharge.WindCharge` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `acceleration_power`? (default 0.1): `DOUBLE`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-witch"></a>
### entity/minecraft:witch

`net.minecraft.world.entity.monster.Witch` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `patrol_target`?: `INT_ARRAY`
- `PatrolLeader`? (default false): `BOOL`
- `Patrolling`? (default false): `BOOL`
- `Wave`? (default 0): `INT`
- `CanJoinRaid`? (default false): `BOOL`
- `RaidId`?: `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-wither"></a>
### entity/minecraft:wither

`net.minecraft.world.entity.boss.wither.WitherBoss` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Invul`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-wither_skeleton"></a>
### entity/minecraft:wither_skeleton

`net.minecraft.world.entity.monster.skeleton.WitherSkeleton` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-wither_skull"></a>
### entity/minecraft:wither_skull

`net.minecraft.world.entity.projectile.hurtingprojectile.WitherSkull` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `LeftOwner`? (default false): `BOOL`
- `HasBeenShot`? (default false): `BOOL`
- `can_break`?: list
  - each: compound `BlockPredicate`
    - `blocks`?: set of minecraft:block (a tag or ids)
    - `state`?: compound of
      - keys: `STRING`
      - values: one of
        - either: `STRING`
        - or: compound `StatePropertiesPredicate$RangedMatcher`
          - `min`?: `STRING`
          - `max`?: `STRING`
    - `nbt`?: an NBT tag
    - `components`? (default EMPTY): map of id in minecraft:data_component_type to an NBT tag
    - `predicates`? (default {}): map of either id in minecraft:data_component_predicate_type or id in minecraft:data_component_type to an NBT tag
- `acceleration_power`? (default 0.1): `DOUBLE`
- `dangerous`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-wolf"></a>
### entity/minecraft:wolf

`net.minecraft.world.entity.animal.wolf.Wolf` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Sitting`? (default false): `BOOL`
- `variant`?: `IDENTIFIER`
- `CollarColor`?: `BYTE`
- `anger_end_time`?: `LONG`
- `AngerTime`?: `INT`
- `sound_variant`?: an NBT tag
- `id`: `STRING`
- `angry_at`?: an NBT tag
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-zoglin"></a>
### entity/minecraft:zoglin

`net.minecraft.world.entity.monster.Zoglin` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsBaby`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-zombie"></a>
### entity/minecraft:zombie

`net.minecraft.world.entity.monster.zombie.Zombie` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsBaby`? (default false): `BOOL`
- `CanBreakDoors`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-zombie_horse"></a>
### entity/minecraft:zombie_horse

`net.minecraft.world.entity.animal.equine.ZombieHorse` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `EatingHaystack`? (default false): `BOOL`
- `Bred`? (default false): `BOOL`
- `Temper`? (default 0): `INT`
- `Tame`? (default false): `BOOL`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-zombie_nautilus"></a>
### entity/minecraft:zombie_nautilus

`net.minecraft.world.entity.animal.nautilus.ZombieNautilus` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `Age`? (default 0): `INT`
- `ForcedAge`? (default 0): `INT`
- `AgeLocked`? (default false): `BOOL`
- `InLove`? (default 0): `INT`
- `Sitting`? (default false): `BOOL`
- `variant`?: `IDENTIFIER`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-zombie_villager"></a>
### entity/minecraft:zombie_villager

`net.minecraft.world.entity.monster.zombie.ZombieVillager` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsBaby`? (default false): `BOOL`
- `CanBreakDoors`? (default false): `BOOL`
- `VillagerData`?: compound `VillagerData`
  - `type`: id in minecraft:villager_type
  - `profession`: id in minecraft:villager_profession
  - `level`? (default 1): `INT`
- `VillagerDataFinalized`? (default false): `BOOL`
- `Offers`?: field
- `Gossips`?: list
  - each: compound `GossipContainer$GossipEntry`
    - `Target`: `UUID`
    - `Type`: enum `GossipType` (var int, ids major_negative/minor_negative/minor_positive/major_positive/trading: MAJOR_NEGATIVE, MINOR_NEGATIVE, MINOR_POSITIVE, MAJOR_POSITIVE, TRADING)
    - `Value`: `INT`
- `ConversionTime`? (default -1): `INT`
- `ConversionPlayer`?: `UUID`
- `Xp`? (default 0): `INT`
- `id`: `STRING`
- `Passengers`?: list of an NBT tag

<a id="save-entity-minecraft-zombified_piglin"></a>
### entity/minecraft:zombified_piglin

`net.minecraft.world.entity.monster.zombie.ZombifiedPiglin` — read from its load, save

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `CanPickUpLoot`? (default false): `BOOL`
- `PersistenceRequired`? (default false): `BOOL`
- `drop_chances`?: map of enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE) to `FLOAT`
- `leash`?: either field or `INT_ARRAY`
- `home_radius`? (default -1): `INT`
- `home_pos`?: `INT_ARRAY`
- `LeftHanded`? (default false): `BOOL`
- `DeathLootTable`?: resource key in minecraft:loot_table
- `DeathLootTableSeed`? (default 0): `LONG`
- `NoAI`? (default false): `BOOL`
- `IsBaby`? (default false): `BOOL`
- `CanBreakDoors`? (default false): `BOOL`
- `anger_end_time`?: `LONG`
- `AngerTime`?: `INT`
- `id`: `STRING`
- `angry_at`?: an NBT tag
- `Passengers`?: list of an NBT tag

<a id="save-level"></a>
### level

`net.minecraft.world.level.storage.LevelStorageSource$LevelStorageAccess` — read from its saveDataTag

- `Data`?: compound `PrimaryLevelData`
  - `ServerBrands`?: list of `STRING`
  - `WasModded`: `BOOL`
  - `removed_features`?: list of `STRING`
  - `Version`?: compound `PrimaryLevelDataVersion`
    - `Name`: `STRING`
    - `Id`: `INT`
    - `Snapshot`: `BOOL`
    - `Series`: `STRING`
  - `version_history`?: list of `INT`
  - `DataVersion`: `INT`
  - `GameType`: `INT`
  - `spawn`: compound `LevelData$RespawnData`
    - `dimension`: resource key in minecraft:dimension
    - `pos`: `INT_ARRAY`
    - `yaw`: `FLOAT`
    - `pitch`: `FLOAT`
  - `Time`: `LONG`
  - `LastPlayed`: `LONG`
  - `LevelName`: `STRING`
  - `version`: `INT`
  - `allowCommands`: `BOOL`
  - `initialized`: `BOOL`
  - `difficulty_settings`: compound `LevelSettings$DifficultySettings`
    - `difficulty`: enum `Difficulty` (var int, ids peaceful/easy/normal/hard: PEACEFUL, EASY, NORMAL, HARD)
    - `hardcore`: `BOOL`
    - `locked`: `BOOL`
  - `singleplayer_uuid`?: `UUID`
  - `DataPacks`? (default DEFAULT): compound `DataPackConfig`
    - `Enabled`: list of `STRING`
    - `Disabled`: list of `STRING`
  - `enabled_features`? (default DEFAULT_FLAGS): list of `IDENTIFIER`

<a id="save-player"></a>
### player

`net.minecraft.server.level.ServerPlayer` — read from its load, saveWithoutId

- `Pos`?: list of `DOUBLE`
- `Motion`?: list of `DOUBLE`
- `Rotation`?: list of `FLOAT`
- `fall_distance`? (default 0.0): `DOUBLE`
- `Fire`? (default 0): `SHORT`
- `Air`?: `SHORT`
- `OnGround`? (default false): `BOOL`
- `Invulnerable`? (default false): `BOOL`
- `invulnerable_time`? (default 0): `INT`
- `PortalCooldown`? (default 0): `INT`
- `UUID`?: `UUID`
- `CustomName`?: a text component
- `CustomNameVisible`? (default false): `BOOL`
- `Silent`? (default false): `BOOL`
- `NoGravity`? (default false): `BOOL`
- `Glowing`? (default false): `BOOL`
- `TicksFrozen`? (default 0): `INT`
- `HasVisualFire`? (default false): `BOOL`
- `data`?: an NBT tag
- `Tags`?: list of `STRING`
- `Team`?: `STRING`
- `AbsorptionAmount`? (default 0.0): `FLOAT`
- `attributes`?: list
  - each: compound `AttributeInstance$Packed`
    - `id`: id in minecraft:attribute
    - `base`? (default 0.0): `DOUBLE`
    - `modifiers`? (default []): list
      - each: compound `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ids add_value/add_multiplied_base/add_multiplied_total: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
- `active_effects`?: an NBT tag
- `Health`?: `FLOAT`
- `HurtTime`? (default 0): `SHORT`
- `DeathTime`? (default 0): `SHORT`
- `FallFlying`? (default false): `BOOL`
- `sleeping_pos`?: `INT_ARRAY`
- `Brain`?: compound `Brain$Packed`
  - `memories`: map of id in minecraft:memory_module_type to an NBT tag
- `last_hurt_by_player_memory_time`? (default 0): `INT`
- `ticks_since_last_hurt_by_mob`? (default 0): `INT`
- `equipment`?: compound of
  - keys: enum `EquipmentSlot` (var int, ids mainhand/offhand/feet/legs/chest/head/body/saddle: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
  - values: recursive `ItemStack`
    - the codec: compound `ItemStack`
      - `id`: id in minecraft:item
      - `count`? (default 1): `INT`
      - `components`? (default EMPTY): an NBT tag
- `locator_bar_icon`?: compound `Waypoint$Icon`
  - `style`: resource key in minecraft:root_id
  - `color`?: `INT`
- `current_impulse_context_reset_grace_time`? (default 0): `INT`
- `current_explosion_impact_pos`?: list of `DOUBLE`
- `Inventory`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `SelectedItemSlot`? (default 0): `INT`
- `SleepTimer`? (default 0): `SHORT`
- `XpP`? (default 0.0): `FLOAT`
- `XpLevel`? (default 0): `INT`
- `XpTotal`? (default 0): `INT`
- `XpSeed`? (default 0): `INT`
- `Score`? (default 0): `INT`
- `foodLevel`? (default 20): `INT`
- `foodTickTimer`? (default 0): `INT`
- `foodSaturationLevel`? (default 5.0): `FLOAT`
- `foodExhaustionLevel`? (default 0.0): `FLOAT`
- `abilities`?: compound `Abilities$Packed`
  - `invulnerable`? (default false): `BOOL`
  - `flying`? (default false): `BOOL`
  - `mayfly`? (default false): `BOOL`
  - `instabuild`? (default false): `BOOL`
  - `mayBuild`? (default true): `BOOL`
  - `flySpeed`? (default 0.05): `FLOAT`
  - `walkSpeed`? (default 0.1): `FLOAT`
- `EnderItems`?: list
  - each: compound `ItemStackWithSlot`
    - `Slot`? (default 0): `BYTE`
    - the codec `ItemStack`, spelled out above
- `LastDeathLocation`?: compound `GlobalPos`
  - `dimension`: resource key in minecraft:dimension
  - `pos`: `INT_ARRAY`
- `warden_spawn_tracker`?: compound `WardenSpawnTracker`
  - `ticks_since_last_warning`? (default 0): `INT`
  - `warning_level`? (default 0): `INT`
  - `cooldown_ticks`? (default 0): `INT`
- `entered_nether_pos`?: list of `DOUBLE`
- `last_explosion_impact_pos`?: list of `DOUBLE`
- `seenCredits`? (default false): `BOOL`
- `recipeBook`?: compound `ServerRecipeBook$Packed`
  - `isGuiOpen`? (default false): `BOOL`
  - `isFilteringCraftable`? (default false): `BOOL`
  - `isFurnaceGuiOpen`? (default false): `BOOL`
  - `isFurnaceFilteringCraftable`? (default false): `BOOL`
  - `isBlastingFurnaceGuiOpen`? (default false): `BOOL`
  - `isBlastingFurnaceFilteringCraftable`? (default false): `BOOL`
  - `isSmokerGuiOpen`? (default false): `BOOL`
  - `isSmokerFilteringCraftable`? (default false): `BOOL`
  - `recipes`: list of resource key in minecraft:recipe
  - `toBeDisplayed`: list of resource key in minecraft:recipe
- `respawn`?: compound `ServerPlayer$RespawnConfig`
  - `dimension`: resource key in minecraft:dimension
  - `pos`: `INT_ARRAY`
  - `yaw`: `FLOAT`
  - `pitch`: `FLOAT`
  - `forced`? (default false): `BOOL`
- `spawn_extra_particles_on_fall`? (default false): `BOOL`
- `raid_omen_position`?: `INT_ARRAY`
- `post_effects`?: an NBT tag
- `ShoulderEntityLeft`?: an NBT tag
- `ShoulderEntityRight`?: an NBT tag
- `DataVersion`: `INT`
- `playerGameType`: `INT`
- `previousPlayerGameType`?: `INT`
- `RootVehicle`?: an NBT tag
- `Dimension`: `STRING`
- `ender_pearls`?: list of an NBT tag
- `Passengers`?: list of an NBT tag


## The records of a world that have a codec

The world options and the dimensions with their generators (`data/minecraft/world_gen_settings.dat`
since 26.1), the respawn point and the data-pack lists of `level.dat` are codec-built records;
they are described on the registries page under shared types and generated into the same Go
package as the formats above, which refer to them.

The saved chunk and the light-only sections: a chunk on disk carries one section below the
world and one above it that hold light and nothing else — no `block_states`, no `biomes` — and
Mojang's reader skips whatever falls outside the level's height. The block-state palette of a
section changed in 26.3: an entry with no properties is written as the bare block id, and only a
state with properties as the compound of `id` and `properties`; until 26.2 every entry was the
compound, under the keys `Name` and `Properties`.
