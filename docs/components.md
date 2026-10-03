# Data components

<!-- This file is a template: gen/docs/components.mc26tmpl.md, rendered by `mc26 docs`. -->

The data components an item stack can carry in Minecraft 26.3-pre-3, with their wire
form as `packet_schema.json` describes it. An item stack on the wire is `ITEM_STACK`
([protocol.md](protocol.md), primitives): a count, then the item id, then a component patch —
counts of added and removed components, then each added component as its type's registry id
followed by the form below, then each removed component as an id. The order of a patch is the
encoder's; a reader must not assume it.

The component's registry id is `data_component_type` in `registries.json`; the table's names are
its entries.

| component | wire form |
|---|---|
| [`minecraft:additional_trade_cost`](#cmp-minecraft-additional_trade_cost) | `VAR_INT` |
| [`minecraft:attack_animation`](#cmp-minecraft-attack_animation) | struct `SwingAnimation` |
| [`minecraft:attack_range`](#cmp-minecraft-attack_range) | struct `AttackRange` |
| [`minecraft:attribute_modifiers`](#cmp-minecraft-attribute_modifiers) | struct `ItemAttributeModifiers` |
| [`minecraft:axolotl/variant`](#cmp-minecraft-axolotl-variant) | enum `Axolotl$Variant` (var int, ordinal: LUCY, WILD, GOLD, CYAN, BLUE) |
| [`minecraft:banner_patterns`](#cmp-minecraft-banner_patterns) | list of struct `BannerPatternLayers$Layer` |
| [`minecraft:base_color`](#cmp-minecraft-base_color) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:bees`](#cmp-minecraft-bees) | list of struct `BeehiveBlockEntity$Occupant` |
| [`minecraft:block_entity_data`](#cmp-minecraft-block_entity_data) | struct `TypedEntityData` |
| [`minecraft:block_state`](#cmp-minecraft-block_state) | map of `STRING` to `STRING` |
| [`minecraft:block_transformer`](#cmp-minecraft-block_transformer) | id in block_transformer |
| [`minecraft:blocks_attacks`](#cmp-minecraft-blocks_attacks) | struct `BlocksAttacks` |
| [`minecraft:break_sound`](#cmp-minecraft-break_sound) | id in sound_event or inline struct `SoundEvent` |
| [`minecraft:brewing_fuel`](#cmp-minecraft-brewing_fuel) | struct `BrewingFuel` |
| [`minecraft:bucket_entity_data`](#cmp-minecraft-bucket_entity_data) | `NBT` |
| [`minecraft:bundle_contents`](#cmp-minecraft-bundle_contents) | list of struct `ItemStackTemplate` |
| [`minecraft:can_break`](#cmp-minecraft-can_break) | struct `AdventureModePredicate` |
| [`minecraft:can_place_on`](#cmp-minecraft-can_place_on) | struct `AdventureModePredicate` |
| [`minecraft:cat/collar`](#cmp-minecraft-cat-collar) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:cat/sound_variant`](#cmp-minecraft-cat-sound_variant) | id in cat_sound_variant |
| [`minecraft:cat/variant`](#cmp-minecraft-cat-variant) | id in cat_variant |
| [`minecraft:charged_projectiles`](#cmp-minecraft-charged_projectiles) | list of struct `ItemStackTemplate` (at most 1024) |
| [`minecraft:chicken/sound_variant`](#cmp-minecraft-chicken-sound_variant) | id in chicken_sound_variant |
| [`minecraft:chicken/variant`](#cmp-minecraft-chicken-variant) | id in chicken_variant |
| [`minecraft:compostable`](#cmp-minecraft-compostable) | struct `Compostable` |
| [`minecraft:consumable`](#cmp-minecraft-consumable) | struct `Consumable` |
| [`minecraft:container`](#cmp-minecraft-container) | list of optional struct `ItemStackTemplate` (at most 256) |
| [`minecraft:container_loot`](#cmp-minecraft-container_loot) | an NBT tag |
| [`minecraft:cooking_fuel`](#cmp-minecraft-cooking_fuel) | struct `CookingFuel` |
| [`minecraft:cow/sound_variant`](#cmp-minecraft-cow-sound_variant) | id in cow_sound_variant |
| [`minecraft:cow/variant`](#cmp-minecraft-cow-variant) | id in cow_variant |
| [`minecraft:creative_slot_lock`](#cmp-minecraft-creative_slot_lock) | nothing |
| [`minecraft:cushion/color`](#cmp-minecraft-cushion-color) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:custom_data`](#cmp-minecraft-custom_data) | an NBT tag |
| [`minecraft:custom_model_data`](#cmp-minecraft-custom_model_data) | struct `CustomModelData` |
| [`minecraft:custom_name`](#cmp-minecraft-custom_name) | `TEXT` |
| [`minecraft:damage`](#cmp-minecraft-damage) | `VAR_INT` |
| [`minecraft:damage_resistant`](#cmp-minecraft-damage_resistant) | struct `DamageResistant` |
| [`minecraft:damage_type`](#cmp-minecraft-damage_type) | id in damage_type |
| [`minecraft:death_protection`](#cmp-minecraft-death_protection) | struct `DeathProtection` |
| [`minecraft:debug_stick_state`](#cmp-minecraft-debug_stick_state) | an NBT tag |
| [`minecraft:dye`](#cmp-minecraft-dye) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:dyed_color`](#cmp-minecraft-dyed_color) | struct `DyedItemColor` |
| [`minecraft:enchantable`](#cmp-minecraft-enchantable) | struct `Enchantable` |
| [`minecraft:enchantment_glint_override`](#cmp-minecraft-enchantment_glint_override) | `BOOL` |
| [`minecraft:enchantments`](#cmp-minecraft-enchantments) | struct `ItemEnchantments` |
| [`minecraft:entity_data`](#cmp-minecraft-entity_data) | struct `TypedEntityData` |
| [`minecraft:equippable`](#cmp-minecraft-equippable) | struct `Equippable` |
| [`minecraft:firework_explosion`](#cmp-minecraft-firework_explosion) | struct `FireworkExplosion` |
| [`minecraft:fireworks`](#cmp-minecraft-fireworks) | struct `Fireworks` |
| [`minecraft:food`](#cmp-minecraft-food) | struct `FoodProperties` |
| [`minecraft:fox/variant`](#cmp-minecraft-fox-variant) | enum `Fox$Variant` (var int, ordinal: RED, SNOW) |
| [`minecraft:frog/variant`](#cmp-minecraft-frog-variant) | id in frog_variant |
| [`minecraft:glider`](#cmp-minecraft-glider) | nothing |
| [`minecraft:horse/variant`](#cmp-minecraft-horse-variant) | enum `Variant` (var int, ordinal: WHITE, CREAMY, CHESTNUT, BROWN, BLACK, GRAY, DARK_BROWN) |
| [`minecraft:instrument`](#cmp-minecraft-instrument) | id in instrument or inline struct `Instrument` |
| [`minecraft:intangible_projectile`](#cmp-minecraft-intangible_projectile) | an NBT tag |
| [`minecraft:interact_animation`](#cmp-minecraft-interact_animation) | struct `SwingAnimation` |
| [`minecraft:item_model`](#cmp-minecraft-item_model) | `IDENTIFIER` |
| [`minecraft:item_name`](#cmp-minecraft-item_name) | `TEXT` |
| [`minecraft:jukebox_playable`](#cmp-minecraft-jukebox_playable) | struct `JukeboxPlayable` |
| [`minecraft:kinetic_weapon`](#cmp-minecraft-kinetic_weapon) | struct `KineticWeapon` |
| [`minecraft:llama/variant`](#cmp-minecraft-llama-variant) | enum `Llama$Variant` (var int, ordinal: CREAMY, WHITE, BROWN, GRAY) |
| [`minecraft:lock`](#cmp-minecraft-lock) | an NBT tag |
| [`minecraft:lodestone_tracker`](#cmp-minecraft-lodestone_tracker) | struct `LodestoneTracker` |
| [`minecraft:lore`](#cmp-minecraft-lore) | list of `TEXT` (at most 256) |
| [`minecraft:map_decorations`](#cmp-minecraft-map_decorations) | an NBT tag |
| [`minecraft:map_id`](#cmp-minecraft-map_id) | `VAR_INT` |
| [`minecraft:map_post_processing`](#cmp-minecraft-map_post_processing) | enum `MapPostProcessing` (var int, ordinal: LOCK, SCALE) |
| [`minecraft:max_damage`](#cmp-minecraft-max_damage) | `VAR_INT` |
| [`minecraft:max_stack_size`](#cmp-minecraft-max_stack_size) | `VAR_INT` |
| [`minecraft:minimum_attack_charge`](#cmp-minecraft-minimum_attack_charge) | `FLOAT` |
| [`minecraft:mob_visibility`](#cmp-minecraft-mob_visibility) | struct `MobVisibility` |
| [`minecraft:mooshroom/variant`](#cmp-minecraft-mooshroom-variant) | enum `MushroomCow$Variant` (var int, ordinal: RED, BROWN) |
| [`minecraft:note_block_sound`](#cmp-minecraft-note_block_sound) | `IDENTIFIER` |
| [`minecraft:ominous_bottle_amplifier`](#cmp-minecraft-ominous_bottle_amplifier) | struct `OminousBottleAmplifier` |
| [`minecraft:painting/variant`](#cmp-minecraft-painting-variant) | id in painting_variant or inline struct `PaintingVariant` |
| [`minecraft:parrot/variant`](#cmp-minecraft-parrot-variant) | enum `Parrot$Variant` (var int, ordinal: RED_BLUE, BLUE, GREEN, YELLOW_BLUE, GRAY) |
| [`minecraft:piercing_weapon`](#cmp-minecraft-piercing_weapon) | struct `PiercingWeapon` |
| [`minecraft:pig/sound_variant`](#cmp-minecraft-pig-sound_variant) | id in pig_sound_variant |
| [`minecraft:pig/variant`](#cmp-minecraft-pig-variant) | id in pig_variant |
| [`minecraft:pot_decorations`](#cmp-minecraft-pot_decorations) | struct `PotDecorations` |
| [`minecraft:potion_contents`](#cmp-minecraft-potion_contents) | struct `PotionContents` |
| [`minecraft:potion_duration_scale`](#cmp-minecraft-potion_duration_scale) | `FLOAT` |
| [`minecraft:profile`](#cmp-minecraft-profile) | struct `ResolvableProfile` |
| [`minecraft:provides_banner_patterns`](#cmp-minecraft-provides_banner_patterns) | set of banner_pattern (a tag or ids) |
| [`minecraft:provides_pottery_pattern`](#cmp-minecraft-provides_pottery_pattern) | id in decorated_pot_pattern |
| [`minecraft:provides_trim_material`](#cmp-minecraft-provides_trim_material) | id in trim_material or inline struct `TrimMaterial` |
| [`minecraft:rabbit/variant`](#cmp-minecraft-rabbit-variant) | enum `Rabbit$Variant` (var int, ids 0/1/2/3/4/5/99: BROWN, WHITE, BLACK, WHITE_SPLOTCHED, GOLD, SALT, EVIL) |
| [`minecraft:rarity`](#cmp-minecraft-rarity) | enum `Rarity` (var int, ordinal: COMMON, UNCOMMON, RARE, EPIC) |
| [`minecraft:recipes`](#cmp-minecraft-recipes) | an NBT tag |
| [`minecraft:repair_cost`](#cmp-minecraft-repair_cost) | `VAR_INT` |
| [`minecraft:repairable`](#cmp-minecraft-repairable) | struct `Repairable` |
| [`minecraft:salmon/size`](#cmp-minecraft-salmon-size) | enum `Salmon$Variant` (var int, ordinal: SMALL, MEDIUM, LARGE) |
| [`minecraft:sheep/color`](#cmp-minecraft-sheep-color) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:shulker/color`](#cmp-minecraft-shulker-color) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:sign_text_back`](#cmp-minecraft-sign_text_back) | struct `SignText` |
| [`minecraft:sign_text_front`](#cmp-minecraft-sign_text_front) | struct `SignText` |
| [`minecraft:stored_enchantments`](#cmp-minecraft-stored_enchantments) | struct `ItemEnchantments` |
| [`minecraft:sulfur_cube_content`](#cmp-minecraft-sulfur_cube_content) | struct `ItemStackTemplate` |
| [`minecraft:suspicious_stew_effects`](#cmp-minecraft-suspicious_stew_effects) | list of struct `SuspiciousStewEffects$Entry` |
| [`minecraft:tool`](#cmp-minecraft-tool) | struct `Tool` |
| [`minecraft:tooltip_display`](#cmp-minecraft-tooltip_display) | struct `TooltipDisplay` |
| [`minecraft:tooltip_style`](#cmp-minecraft-tooltip_style) | `IDENTIFIER` |
| [`minecraft:trim`](#cmp-minecraft-trim) | struct `ArmorTrim` |
| [`minecraft:tropical_fish/base_color`](#cmp-minecraft-tropical_fish-base_color) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:tropical_fish/pattern`](#cmp-minecraft-tropical_fish-pattern) | enum `TropicalFish$Pattern` (var int, ids 0/256/512/768/1024/1280/1/257/513/769/1025/1281: KOB, SUNSTREAK, SNOOPER, DASHER, BRINELY, SPOTTY, FLOPPER, STRIPEY, GLITTER, BLOCKFISH, BETTY, CLAYFISH) |
| [`minecraft:tropical_fish/pattern_color`](#cmp-minecraft-tropical_fish-pattern_color) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:unbreakable`](#cmp-minecraft-unbreakable) | nothing |
| [`minecraft:use_cooldown`](#cmp-minecraft-use_cooldown) | struct `UseCooldown` |
| [`minecraft:use_effects`](#cmp-minecraft-use_effects) | struct `UseEffects` |
| [`minecraft:use_remainder`](#cmp-minecraft-use_remainder) | struct `UseRemainder` |
| [`minecraft:villager/variant`](#cmp-minecraft-villager-variant) | id in villager_type |
| [`minecraft:villager_food`](#cmp-minecraft-villager_food) | struct `VillagerFood` |
| [`minecraft:waxed`](#cmp-minecraft-waxed) | nothing |
| [`minecraft:weapon`](#cmp-minecraft-weapon) | struct `Weapon` |
| [`minecraft:wolf/collar`](#cmp-minecraft-wolf-collar) | enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK) |
| [`minecraft:wolf/sound_variant`](#cmp-minecraft-wolf-sound_variant) | id in wolf_sound_variant |
| [`minecraft:wolf/variant`](#cmp-minecraft-wolf-variant) | id in wolf_variant |
| [`minecraft:writable_book_content`](#cmp-minecraft-writable_book_content) | list of struct `Filterable` (at most 100) |
| [`minecraft:written_book_content`](#cmp-minecraft-written_book_content) | struct `WrittenBookContent` |
| [`minecraft:zombie_nautilus/variant`](#cmp-minecraft-zombie_nautilus-variant) | id in zombie_nautilus_variant |

<a id="cmp-minecraft-additional_trade_cost"></a>
### minecraft:additional_trade_cost

- `VAR_INT`

<a id="cmp-minecraft-attack_animation"></a>
### minecraft:attack_animation

- `type`: enum `SwingAnimationType` (var int, ordinal: NONE, WHACK, STAB)
- `duration`: `VAR_INT`

<a id="cmp-minecraft-attack_range"></a>
### minecraft:attack_range

- `minReach`: `FLOAT`
- `maxReach`: `FLOAT`
- `minCreativeReach`: `FLOAT`
- `maxCreativeReach`: `FLOAT`
- `hitboxMargin`: `FLOAT`
- `mobFactor`: `FLOAT`

<a id="cmp-minecraft-attribute_modifiers"></a>
### minecraft:attribute_modifiers

- `modifiers`: list of
  - struct `ItemAttributeModifiers$Entry`
    - `attribute`: id in attribute
    - `modifier`: struct `AttributeModifier`
      - `id`: `IDENTIFIER`
      - `amount`: `DOUBLE`
      - `operation`: enum `AttributeModifier$Operation` (var int, ordinal: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)
    - `slot`: enum `EquipmentSlotGroup` (var int, ordinal: ANY, MAINHAND, OFFHAND, HAND, FEET, LEGS, CHEST, HEAD, ARMOR, BODY, SADDLE)
    - `display`: dispatch `ItemAttributeModifiers$Display` on enum `ItemAttributeModifiers$Display$Type` (var int, ordinal: DEFAULT, HIDDEN, OVERRIDE)
      - `DEFAULT (0)`: nothing
      - `HIDDEN (1)`: nothing
      - `OVERRIDE (2)`: struct `ItemAttributeModifiers$Display$OverrideText`
        - `component`: `TEXT`

<a id="cmp-minecraft-axolotl-variant"></a>
### minecraft:axolotl/variant

- enum `Axolotl$Variant` (var int, ordinal: LUCY, WILD, GOLD, CYAN, BLUE)

<a id="cmp-minecraft-banner_patterns"></a>
### minecraft:banner_patterns

- list of
  - struct `BannerPatternLayers$Layer`
    - `pattern`: id in banner_pattern, or 0 and the element inline
      - inline: struct `BannerPattern`
        - `assetId`: `IDENTIFIER`
        - `translationKey`: `STRING`
    - `color`: enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-base_color"></a>
### minecraft:base_color

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-bees"></a>
### minecraft:bees

- list of
  - struct `BeehiveBlockEntity$Occupant`
    - `entityData`: struct `TypedEntityData`
      - `type`: id in entity_type
      - `tag`: `NBT`
    - `ticksInHive`: `VAR_INT`
    - `minTicksInHive`: `VAR_INT`

<a id="cmp-minecraft-block_entity_data"></a>
### minecraft:block_entity_data

- `type`: id in block_entity_type
- `tag`: `NBT`

<a id="cmp-minecraft-block_state"></a>
### minecraft:block_state

- map of `STRING` to `STRING`

<a id="cmp-minecraft-block_transformer"></a>
### minecraft:block_transformer

- id in block_transformer

<a id="cmp-minecraft-blocks_attacks"></a>
### minecraft:blocks_attacks

- `blockDelaySeconds`: `FLOAT`
- `disableCooldownScale`: `FLOAT`
- `damageReductions`: list of
  - struct `BlocksAttacks$DamageReduction`
    - `horizontalBlockingAngle`: `FLOAT`
    - `type`: optional set of damage_type (a tag or ids)
    - `base`: `FLOAT`
    - `factor`: `FLOAT`
- `itemDamage`: struct `BlocksAttacks$ItemDamageFunction`
  - `threshold`: `FLOAT`
  - `base`: `FLOAT`
  - `factor`: `FLOAT`
- `bypassedBy`: optional set of damage_type (a tag or ids)
- `blockSound`: optional
  - id in sound_event, or 0 and the element inline
    - inline: struct `SoundEvent`
      - `location`: `IDENTIFIER`
      - `fixedRange`: optional `FLOAT`
- `disableSound`: optional
  - id in sound_event, or 0 and the element inline
    - inline: struct `SoundEvent`
      - `location`: `IDENTIFIER`
      - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-break_sound"></a>
### minecraft:break_sound

- id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-brewing_fuel"></a>
### minecraft:brewing_fuel

- `uses`: either `INT` or resource key in context_int_provider
- `speedMultiplier`: either `FLOAT` or resource key in context_float_provider

<a id="cmp-minecraft-bucket_entity_data"></a>
### minecraft:bucket_entity_data

- `NBT`

<a id="cmp-minecraft-bundle_contents"></a>
### minecraft:bundle_contents

- list of
  - struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`

<a id="cmp-minecraft-can_break"></a>
### minecraft:can_break

- `predicates`: list of
  - struct `BlockPredicate`
    - `blocks`: optional set of block (a tag or ids)
    - `properties`: optional
      - list of
        - struct `StatePropertiesPredicate$PropertyMatcher`
          - `name`: `STRING`
          - `valueMatcher`: either (a boolean, then one side)
            - left: `STRING`
            - right: struct `StatePropertiesPredicate$RangedMatcher`
              - `minValue`: optional `STRING`
              - `maxValue`: optional `STRING`
    - `nbt`: optional `NBT`
    - `components`: struct `DataComponentMatchers`
      - `exact`: list of `TYPED_DATA_COMPONENT`
      - `partial`: list of (at most 64)
        - struct `DataComponentPredicate`
          - `type`: either id in data_component_predicate_type or id in data_component_type
          - `value`: an NBT tag

<a id="cmp-minecraft-can_place_on"></a>
### minecraft:can_place_on

- `predicates`: list of
  - struct `BlockPredicate`
    - `blocks`: optional set of block (a tag or ids)
    - `properties`: optional
      - list of
        - struct `StatePropertiesPredicate$PropertyMatcher`
          - `name`: `STRING`
          - `valueMatcher`: either (a boolean, then one side)
            - left: `STRING`
            - right: struct `StatePropertiesPredicate$RangedMatcher`
              - `minValue`: optional `STRING`
              - `maxValue`: optional `STRING`
    - `nbt`: optional `NBT`
    - `components`: struct `DataComponentMatchers`
      - `exact`: list of `TYPED_DATA_COMPONENT`
      - `partial`: list of (at most 64)
        - struct `DataComponentPredicate`
          - `type`: either id in data_component_predicate_type or id in data_component_type
          - `value`: an NBT tag

<a id="cmp-minecraft-cat-collar"></a>
### minecraft:cat/collar

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-cat-sound_variant"></a>
### minecraft:cat/sound_variant

- id in cat_sound_variant

<a id="cmp-minecraft-cat-variant"></a>
### minecraft:cat/variant

- id in cat_variant

<a id="cmp-minecraft-charged_projectiles"></a>
### minecraft:charged_projectiles

- list of (at most 1024)
  - struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`

<a id="cmp-minecraft-chicken-sound_variant"></a>
### minecraft:chicken/sound_variant

- id in chicken_sound_variant

<a id="cmp-minecraft-chicken-variant"></a>
### minecraft:chicken/variant

- id in chicken_variant

<a id="cmp-minecraft-compostable"></a>
### minecraft:compostable

- `layers`: either `INT` or resource key in context_int_provider

<a id="cmp-minecraft-consumable"></a>
### minecraft:consumable

- `consumeSeconds`: `FLOAT`
- `animation`: enum `ItemUseAnimation` (var int, ordinal: NONE, EAT, DRINK, BLOCK, BOW, TRIDENT, CROSSBOW, SPYGLASS, TOOT_HORN, BRUSH, BUNDLE, SPEAR)
- `sound`: id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`
- `hasConsumeParticles`: `BOOL`
- `onConsumeEffects`: list of
  - dispatch `ConsumeEffect` on id in consume_effect_type
    - `minecraft:apply_effects`: struct `ApplyStatusEffectsConsumeEffect`
      - `effects`: list of
        - struct `MobEffectInstance`
          - `getEffect`: id in mob_effect
          - `asDetails`: struct `MobEffectInstance$Details`
            - `amplifier`: `VAR_INT`
            - `duration`: `VAR_INT`
            - `ambient`: `BOOL`
            - `showParticles`: `BOOL`
            - `showIcon`: `BOOL`
            - `hiddenEffect`: optional a `MobEffectInstance$Details` again
      - `probability`: `FLOAT`
    - `minecraft:remove_effects`: struct `RemoveStatusEffectsConsumeEffect`
      - `effects`: set of mob_effect (a tag or ids)
    - `minecraft:clear_all_effects`: nothing
    - `minecraft:teleport_randomly`: struct `TeleportRandomlyConsumeEffect`
      - `diameter`: `FLOAT`
      - `directionalParticles`: `BOOL`
    - `minecraft:play_sound`: struct `PlaySoundConsumeEffect`
      - `sound`: id in sound_event, or 0 and the element inline
        - inline: struct `SoundEvent`
          - `location`: `IDENTIFIER`
          - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-container"></a>
### minecraft:container

- list of (at most 256)
  - optional
    - struct `ItemStackTemplate`
      - `item`: id in item
      - `count`: `VAR_INT`
      - `components`: `COMPONENT_PATCH`

<a id="cmp-minecraft-container_loot"></a>
### minecraft:container_loot

- an NBT tag

<a id="cmp-minecraft-cooking_fuel"></a>
### minecraft:cooking_fuel

- `burnTime`: either `INT` or resource key in context_int_provider
- `speedMultiplier`: either `FLOAT` or resource key in context_float_provider

<a id="cmp-minecraft-cow-sound_variant"></a>
### minecraft:cow/sound_variant

- id in cow_sound_variant

<a id="cmp-minecraft-cow-variant"></a>
### minecraft:cow/variant

- id in cow_variant

<a id="cmp-minecraft-creative_slot_lock"></a>
### minecraft:creative_slot_lock

- nothing

<a id="cmp-minecraft-cushion-color"></a>
### minecraft:cushion/color

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-custom_data"></a>
### minecraft:custom_data

- an NBT tag

<a id="cmp-minecraft-custom_model_data"></a>
### minecraft:custom_model_data

- `floats`: list of `FLOAT`
- `flags`: list of `BOOL`
- `strings`: list of `STRING`
- `colors`: list of `INT`

<a id="cmp-minecraft-custom_name"></a>
### minecraft:custom_name

- `TEXT`

<a id="cmp-minecraft-damage"></a>
### minecraft:damage

- `VAR_INT`

<a id="cmp-minecraft-damage_resistant"></a>
### minecraft:damage_resistant

- `types`: set of damage_type (a tag or ids)

<a id="cmp-minecraft-damage_type"></a>
### minecraft:damage_type

- id in damage_type

<a id="cmp-minecraft-death_protection"></a>
### minecraft:death_protection

- `deathEffects`: list of
  - dispatch `ConsumeEffect` on id in consume_effect_type
    - `minecraft:apply_effects`: struct `ApplyStatusEffectsConsumeEffect`
      - `effects`: list of
        - struct `MobEffectInstance`
          - `getEffect`: id in mob_effect
          - `asDetails`: struct `MobEffectInstance$Details`
            - `amplifier`: `VAR_INT`
            - `duration`: `VAR_INT`
            - `ambient`: `BOOL`
            - `showParticles`: `BOOL`
            - `showIcon`: `BOOL`
            - `hiddenEffect`: optional a `MobEffectInstance$Details` again
      - `probability`: `FLOAT`
    - `minecraft:remove_effects`: struct `RemoveStatusEffectsConsumeEffect`
      - `effects`: set of mob_effect (a tag or ids)
    - `minecraft:clear_all_effects`: nothing
    - `minecraft:teleport_randomly`: struct `TeleportRandomlyConsumeEffect`
      - `diameter`: `FLOAT`
      - `directionalParticles`: `BOOL`
    - `minecraft:play_sound`: struct `PlaySoundConsumeEffect`
      - `sound`: id in sound_event, or 0 and the element inline
        - inline: struct `SoundEvent`
          - `location`: `IDENTIFIER`
          - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-debug_stick_state"></a>
### minecraft:debug_stick_state

- an NBT tag

<a id="cmp-minecraft-dye"></a>
### minecraft:dye

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-dyed_color"></a>
### minecraft:dyed_color

- `rgb`: `INT`

<a id="cmp-minecraft-enchantable"></a>
### minecraft:enchantable

- `value`: `VAR_INT`

<a id="cmp-minecraft-enchantment_glint_override"></a>
### minecraft:enchantment_glint_override

- `BOOL`

<a id="cmp-minecraft-enchantments"></a>
### minecraft:enchantments

- `enchantments`: map of id in enchantment to `VAR_INT`

<a id="cmp-minecraft-entity_data"></a>
### minecraft:entity_data

- `type`: id in entity_type
- `tag`: `NBT`

<a id="cmp-minecraft-equippable"></a>
### minecraft:equippable

- `slot`: enum `EquipmentSlot` (var int, ids 0/5/1/2/3/4/6/7: MAINHAND, OFFHAND, FEET, LEGS, CHEST, HEAD, BODY, SADDLE)
- `equipSound`: id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`
- `assetId`: optional resource key in root_id
- `cameraOverlay`: optional `IDENTIFIER`
- `allowedEntities`: optional set of entity_type (a tag or ids)
- `dispensable`: `BOOL`
- `swappable`: `BOOL`
- `damageOnHurt`: `BOOL`
- `equipOnInteract`: `BOOL`
- `canBeSheared`: `BOOL`
- `shearingSound`: id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-firework_explosion"></a>
### minecraft:firework_explosion

- `shape`: enum `FireworkExplosion$Shape` (var int, ordinal: SMALL_BALL, LARGE_BALL, STAR, CREEPER, BURST)
- `colors`: list of `INT`
- `fadeColors`: list of `INT`
- `hasTrail`: `BOOL`
- `hasTwinkle`: `BOOL`

<a id="cmp-minecraft-fireworks"></a>
### minecraft:fireworks

- `flightDuration`: `VAR_INT`
- `explosions`: list of (at most 256)
  - struct `FireworkExplosion`
    - `shape`: enum `FireworkExplosion$Shape` (var int, ordinal: SMALL_BALL, LARGE_BALL, STAR, CREEPER, BURST)
    - `colors`: list of `INT`
    - `fadeColors`: list of `INT`
    - `hasTrail`: `BOOL`
    - `hasTwinkle`: `BOOL`

<a id="cmp-minecraft-food"></a>
### minecraft:food

- `nutrition`: `VAR_INT`
- `saturation`: `FLOAT`
- `canAlwaysEat`: `BOOL`

<a id="cmp-minecraft-fox-variant"></a>
### minecraft:fox/variant

- enum `Fox$Variant` (var int, ordinal: RED, SNOW)

<a id="cmp-minecraft-frog-variant"></a>
### minecraft:frog/variant

- id in frog_variant

<a id="cmp-minecraft-glider"></a>
### minecraft:glider

- nothing

<a id="cmp-minecraft-horse-variant"></a>
### minecraft:horse/variant

- enum `Variant` (var int, ordinal: WHITE, CREAMY, CHESTNUT, BROWN, BLACK, GRAY, DARK_BROWN)

<a id="cmp-minecraft-instrument"></a>
### minecraft:instrument

- id in instrument, or 0 and the element inline
  - inline: struct `Instrument`
    - `soundEvent`: id in sound_event, or 0 and the element inline
      - inline: struct `SoundEvent`
        - `location`: `IDENTIFIER`
        - `fixedRange`: optional `FLOAT`
    - `useDuration`: `FLOAT`
    - `range`: `FLOAT`
    - `durabilityDamage`: `VAR_INT`
    - `description`: `TEXT`

<a id="cmp-minecraft-intangible_projectile"></a>
### minecraft:intangible_projectile

- an NBT tag

<a id="cmp-minecraft-interact_animation"></a>
### minecraft:interact_animation

- `type`: enum `SwingAnimationType` (var int, ordinal: NONE, WHACK, STAB)
- `duration`: `VAR_INT`

<a id="cmp-minecraft-item_model"></a>
### minecraft:item_model

- `IDENTIFIER`

<a id="cmp-minecraft-item_name"></a>
### minecraft:item_name

- `TEXT`

<a id="cmp-minecraft-jukebox_playable"></a>
### minecraft:jukebox_playable

- `song`: id in jukebox_song, or 0 and the element inline
  - inline: struct `JukeboxSong`
    - `soundEvent`: id in sound_event, or 0 and the element inline
      - inline: struct `SoundEvent`
        - `location`: `IDENTIFIER`
        - `fixedRange`: optional `FLOAT`
    - `description`: `TEXT`
    - `lengthInSeconds`: `FLOAT`
    - `comparatorOutput`: `VAR_INT`

<a id="cmp-minecraft-kinetic_weapon"></a>
### minecraft:kinetic_weapon

- `contactCooldownTicks`: `VAR_INT`
- `delayTicks`: `VAR_INT`
- `dismountConditions`: optional
  - struct `KineticWeapon$Condition`
    - `maxDurationTicks`: `VAR_INT`
    - `minSpeed`: `FLOAT`
    - `minRelativeSpeed`: `FLOAT`
- `knockbackConditions`: optional
  - struct `KineticWeapon$Condition`
    - `maxDurationTicks`: `VAR_INT`
    - `minSpeed`: `FLOAT`
    - `minRelativeSpeed`: `FLOAT`
- `damageConditions`: optional
  - struct `KineticWeapon$Condition`
    - `maxDurationTicks`: `VAR_INT`
    - `minSpeed`: `FLOAT`
    - `minRelativeSpeed`: `FLOAT`
- `forwardMovement`: `FLOAT`
- `damageMultiplier`: `FLOAT`
- `sound`: optional
  - id in sound_event, or 0 and the element inline
    - inline: struct `SoundEvent`
      - `location`: `IDENTIFIER`
      - `fixedRange`: optional `FLOAT`
- `hitSound`: optional
  - id in sound_event, or 0 and the element inline
    - inline: struct `SoundEvent`
      - `location`: `IDENTIFIER`
      - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-llama-variant"></a>
### minecraft:llama/variant

- enum `Llama$Variant` (var int, ordinal: CREAMY, WHITE, BROWN, GRAY)

<a id="cmp-minecraft-lock"></a>
### minecraft:lock

- an NBT tag

<a id="cmp-minecraft-lodestone_tracker"></a>
### minecraft:lodestone_tracker

- `target`: optional
  - struct `GlobalPos`
    - `dimension`: resource key in dimension
    - `pos`: `BLOCK_POS`
- `tracked`: `BOOL`

<a id="cmp-minecraft-lore"></a>
### minecraft:lore

- list of `TEXT` (at most 256)

<a id="cmp-minecraft-map_decorations"></a>
### minecraft:map_decorations

- an NBT tag

<a id="cmp-minecraft-map_id"></a>
### minecraft:map_id

- `VAR_INT`

<a id="cmp-minecraft-map_post_processing"></a>
### minecraft:map_post_processing

- enum `MapPostProcessing` (var int, ordinal: LOCK, SCALE)

<a id="cmp-minecraft-max_damage"></a>
### minecraft:max_damage

- `VAR_INT`

<a id="cmp-minecraft-max_stack_size"></a>
### minecraft:max_stack_size

- `VAR_INT`

<a id="cmp-minecraft-minimum_attack_charge"></a>
### minecraft:minimum_attack_charge

- `FLOAT`

<a id="cmp-minecraft-mob_visibility"></a>
### minecraft:mob_visibility

- `targetingEntityTypes`: set of entity_type (a tag or ids)
- `visibility`: `FLOAT`

<a id="cmp-minecraft-mooshroom-variant"></a>
### minecraft:mooshroom/variant

- enum `MushroomCow$Variant` (var int, ordinal: RED, BROWN)

<a id="cmp-minecraft-note_block_sound"></a>
### minecraft:note_block_sound

- `IDENTIFIER`

<a id="cmp-minecraft-ominous_bottle_amplifier"></a>
### minecraft:ominous_bottle_amplifier

- `value`: `VAR_INT`

<a id="cmp-minecraft-painting-variant"></a>
### minecraft:painting/variant

- id in painting_variant, or 0 and the element inline
  - inline: struct `PaintingVariant`
    - `width`: `VAR_INT`
    - `height`: `VAR_INT`
    - `assetId`: `IDENTIFIER`
    - `title`: `OPTIONAL_TEXT`
    - `author`: `OPTIONAL_TEXT`

<a id="cmp-minecraft-parrot-variant"></a>
### minecraft:parrot/variant

- enum `Parrot$Variant` (var int, ordinal: RED_BLUE, BLUE, GREEN, YELLOW_BLUE, GRAY)

<a id="cmp-minecraft-piercing_weapon"></a>
### minecraft:piercing_weapon

- `dealsKnockback`: `BOOL`
- `dismounts`: `BOOL`
- `sound`: optional
  - id in sound_event, or 0 and the element inline
    - inline: struct `SoundEvent`
      - `location`: `IDENTIFIER`
      - `fixedRange`: optional `FLOAT`
- `hitSound`: optional
  - id in sound_event, or 0 and the element inline
    - inline: struct `SoundEvent`
      - `location`: `IDENTIFIER`
      - `fixedRange`: optional `FLOAT`

<a id="cmp-minecraft-pig-sound_variant"></a>
### minecraft:pig/sound_variant

- id in pig_sound_variant

<a id="cmp-minecraft-pig-variant"></a>
### minecraft:pig/variant

- id in pig_variant

<a id="cmp-minecraft-pot_decorations"></a>
### minecraft:pot_decorations

- `back`: optional
  - struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`
- `left`: optional
  - struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`
- `right`: optional
  - struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`
- `front`: optional
  - struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`

<a id="cmp-minecraft-potion_contents"></a>
### minecraft:potion_contents

- `potion`: optional id in potion
- `customColor`: optional `INT`
- `customEffects`: list of
  - struct `MobEffectInstance`
    - `getEffect`: id in mob_effect
    - `asDetails`: struct `MobEffectInstance$Details`
      - `amplifier`: `VAR_INT`
      - `duration`: `VAR_INT`
      - `ambient`: `BOOL`
      - `showParticles`: `BOOL`
      - `showIcon`: `BOOL`
      - `hiddenEffect`: optional a `MobEffectInstance$Details` again
- `customName`: optional `STRING`

<a id="cmp-minecraft-potion_duration_scale"></a>
### minecraft:potion_duration_scale

- `FLOAT`

<a id="cmp-minecraft-profile"></a>
### minecraft:profile

- `unpack`: either (a boolean, then one side)
  - left: struct `GameProfile`
    - `id`: `UUID`
    - `name`: `STRING`
    - `properties`: `GAME_PROFILE_PROPERTIES`
  - right: struct `ResolvableProfile$Partial`
    - `name`: optional `STRING`
    - `id`: optional `UUID`
    - `properties`: `GAME_PROFILE_PROPERTIES`
- `skinPatch`: struct `PlayerSkin$Patch`
  - `body`: optional `IDENTIFIER`
  - `cape`: optional `IDENTIFIER`
  - `elytra`: optional `IDENTIFIER`
  - `model`: optional `BOOL`

<a id="cmp-minecraft-provides_banner_patterns"></a>
### minecraft:provides_banner_patterns

- set of banner_pattern (a tag or ids)

<a id="cmp-minecraft-provides_pottery_pattern"></a>
### minecraft:provides_pottery_pattern

- id in decorated_pot_pattern

<a id="cmp-minecraft-provides_trim_material"></a>
### minecraft:provides_trim_material

- id in trim_material, or 0 and the element inline
  - inline: struct `TrimMaterial`
    - `paletteId`: `IDENTIFIER`
    - `description`: `TEXT`

<a id="cmp-minecraft-rabbit-variant"></a>
### minecraft:rabbit/variant

- enum `Rabbit$Variant` (var int, ids 0/1/2/3/4/5/99: BROWN, WHITE, BLACK, WHITE_SPLOTCHED, GOLD, SALT, EVIL)

<a id="cmp-minecraft-rarity"></a>
### minecraft:rarity

- enum `Rarity` (var int, ordinal: COMMON, UNCOMMON, RARE, EPIC)

<a id="cmp-minecraft-recipes"></a>
### minecraft:recipes

- an NBT tag

<a id="cmp-minecraft-repair_cost"></a>
### minecraft:repair_cost

- `VAR_INT`

<a id="cmp-minecraft-repairable"></a>
### minecraft:repairable

- `items`: set of item (a tag or ids)

<a id="cmp-minecraft-salmon-size"></a>
### minecraft:salmon/size

- enum `Salmon$Variant` (var int, ordinal: SMALL, MEDIUM, LARGE)

<a id="cmp-minecraft-sheep-color"></a>
### minecraft:sheep/color

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-shulker-color"></a>
### minecraft:shulker/color

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-sign_text_back"></a>
### minecraft:sign_text_back

- `messages`: `TEXT`
- `filteredMessagesForSerialization`: optional `TEXT`
- `color`: enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)
- `hasGlowingText`: `BOOL`

<a id="cmp-minecraft-sign_text_front"></a>
### minecraft:sign_text_front

- `messages`: `TEXT`
- `filteredMessagesForSerialization`: optional `TEXT`
- `color`: enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)
- `hasGlowingText`: `BOOL`

<a id="cmp-minecraft-stored_enchantments"></a>
### minecraft:stored_enchantments

- `enchantments`: map of id in enchantment to `VAR_INT`

<a id="cmp-minecraft-sulfur_cube_content"></a>
### minecraft:sulfur_cube_content

- `item`: id in item
- `count`: `VAR_INT`
- `components`: `COMPONENT_PATCH`

<a id="cmp-minecraft-suspicious_stew_effects"></a>
### minecraft:suspicious_stew_effects

- list of
  - struct `SuspiciousStewEffects$Entry`
    - `effect`: id in mob_effect
    - `duration`: `VAR_INT`

<a id="cmp-minecraft-tool"></a>
### minecraft:tool

- `rules`: list of
  - struct `Tool$Rule`
    - `blocks`: set of block (a tag or ids)
    - `speed`: optional `FLOAT`
    - `correctForDrops`: optional `BOOL`
- `defaultMiningSpeed`: `FLOAT`
- `damagePerBlock`: `VAR_INT`
- `canDestroyBlocksInCreative`: `BOOL`

<a id="cmp-minecraft-tooltip_display"></a>
### minecraft:tooltip_display

- `hideTooltip`: `BOOL`
- `hiddenComponents`: list of id in data_component_type

<a id="cmp-minecraft-tooltip_style"></a>
### minecraft:tooltip_style

- `IDENTIFIER`

<a id="cmp-minecraft-trim"></a>
### minecraft:trim

- `material`: id in trim_material, or 0 and the element inline
  - inline: struct `TrimMaterial`
    - `paletteId`: `IDENTIFIER`
    - `description`: `TEXT`
- `pattern`: id in trim_pattern, or 0 and the element inline
  - inline: struct `TrimPattern`
    - `assetId`: `IDENTIFIER`
    - `description`: `TEXT`
    - `decal`: `BOOL`

<a id="cmp-minecraft-tropical_fish-base_color"></a>
### minecraft:tropical_fish/base_color

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-tropical_fish-pattern"></a>
### minecraft:tropical_fish/pattern

- enum `TropicalFish$Pattern` (var int, ids 0/256/512/768/1024/1280/1/257/513/769/1025/1281: KOB, SUNSTREAK, SNOOPER, DASHER, BRINELY, SPOTTY, FLOPPER, STRIPEY, GLITTER, BLOCKFISH, BETTY, CLAYFISH)

<a id="cmp-minecraft-tropical_fish-pattern_color"></a>
### minecraft:tropical_fish/pattern_color

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-unbreakable"></a>
### minecraft:unbreakable

- nothing

<a id="cmp-minecraft-use_cooldown"></a>
### minecraft:use_cooldown

- `seconds`: `FLOAT`
- `cooldownGroup`: optional `IDENTIFIER`

<a id="cmp-minecraft-use_effects"></a>
### minecraft:use_effects

- `canSprint`: `BOOL`
- `interactVibrations`: `BOOL`
- `speedMultiplier`: `FLOAT`

<a id="cmp-minecraft-use_remainder"></a>
### minecraft:use_remainder

- `convertInto`: struct `ItemStackTemplate`
  - `item`: id in item
  - `count`: `VAR_INT`
  - `components`: `COMPONENT_PATCH`

<a id="cmp-minecraft-villager-variant"></a>
### minecraft:villager/variant

- id in villager_type

<a id="cmp-minecraft-villager_food"></a>
### minecraft:villager_food

- `nutrition`: `VAR_INT`

<a id="cmp-minecraft-waxed"></a>
### minecraft:waxed

- nothing

<a id="cmp-minecraft-weapon"></a>
### minecraft:weapon

- `itemDamagePerAttack`: `VAR_INT`
- `disableBlockingForSeconds`: `FLOAT`

<a id="cmp-minecraft-wolf-collar"></a>
### minecraft:wolf/collar

- enum `DyeColor` (var int, ordinal: WHITE, ORANGE, MAGENTA, LIGHT_BLUE, YELLOW, LIME, PINK, GRAY, LIGHT_GRAY, CYAN, PURPLE, BLUE, BROWN, GREEN, RED, BLACK)

<a id="cmp-minecraft-wolf-sound_variant"></a>
### minecraft:wolf/sound_variant

- id in wolf_sound_variant

<a id="cmp-minecraft-wolf-variant"></a>
### minecraft:wolf/variant

- id in wolf_variant

<a id="cmp-minecraft-writable_book_content"></a>
### minecraft:writable_book_content

- list of (at most 100)
  - struct `Filterable`
    - `raw`: string (at most 1024 characters)
    - `filtered`: optional string (at most 1024 characters)

<a id="cmp-minecraft-written_book_content"></a>
### minecraft:written_book_content

- `title`: struct `Filterable`
  - `raw`: string (at most 32 characters)
  - `filtered`: optional string (at most 32 characters)
- `author`: `STRING`
- `generation`: `VAR_INT`
- `pages`: list of
  - struct `Filterable`
    - `raw`: `TEXT`
    - `filtered`: optional `TEXT`
- `resolved`: `BOOL`

<a id="cmp-minecraft-zombie_nautilus-variant"></a>
### minecraft:zombie_nautilus/variant

- id in zombie_nautilus_variant

