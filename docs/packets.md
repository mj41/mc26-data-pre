# Packets

<!-- This file is a template: gen/docs/packets.mc26tmpl.md, rendered by `mc26 docs`. -->

Every packet of Minecraft 26.3-pre-3 (protocol 1073742159), by
connection state and direction, with its id in that state's table and its fields as
`packet_schema.json` describes them. The node kinds and the primitives are explained in
[protocol.md](protocol.md); the frame around a packet, and what changes the state, are there too.

A field's line is its name and what it is on the wire; a field that is only sometimes present
says when. A `struct` is its fields in order; a `dispatch` is its key, then the payload of the
case the key selects; `list of` is a var int count then that many elements; `optional` a
boolean then the value when the boolean is set. Ids are indexes into a dense table per state
and direction, so a packet added in a later version moves every id after it.

## handshake serverbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:intention`](#pkt-serverbound-minecraft-intention) | `net.minecraft.network.protocol.handshake.ClientIntentionPacket` |

<a id="pkt-serverbound-minecraft-intention"></a>
### minecraft:intention (serverbound, id 0)

- `protocolVersion`: `VAR_INT`
- `hostName`: string (at most 255 characters)
- `port`: `UNSIGNED_SHORT`
- `intention`: enum `ClientIntent` (var int, ids 1/2/3: STATUS, LOGIN, TRANSFER)

## status clientbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:status_response`](#pkt-clientbound-minecraft-status_response) | `net.minecraft.network.protocol.status.ClientboundStatusResponsePacket` |
| 1 | [`minecraft:pong_response`](#pkt-clientbound-minecraft-pong_response) | `net.minecraft.network.protocol.ping.ClientboundPongResponsePacket` |

<a id="pkt-clientbound-minecraft-status_response"></a>
### minecraft:status_response (clientbound, id 0)

- `status`: `JSON_TEXT` (max=32767)

<a id="pkt-clientbound-minecraft-pong_response"></a>
### minecraft:pong_response (clientbound, id 1)

- `time`: `LONG`

## status serverbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:status_request`](#pkt-serverbound-minecraft-status_request) | `net.minecraft.network.protocol.status.ServerboundStatusRequestPacket` |
| 1 | [`minecraft:ping_request`](#pkt-serverbound-minecraft-ping_request) | `net.minecraft.network.protocol.ping.ServerboundPingRequestPacket` |

<a id="pkt-serverbound-minecraft-status_request"></a>
### minecraft:status_request (serverbound, id 0)

- nothing

<a id="pkt-serverbound-minecraft-ping_request"></a>
### minecraft:ping_request (serverbound, id 1)

- `time`: `LONG`

## login clientbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:login_disconnect`](#pkt-clientbound-minecraft-login_disconnect) | `net.minecraft.network.protocol.login.ClientboundLoginDisconnectPacket` |
| 1 | [`minecraft:hello`](#pkt-clientbound-minecraft-hello) | `net.minecraft.network.protocol.login.ClientboundHelloPacket` |
| 2 | [`minecraft:login_finished`](#pkt-clientbound-minecraft-login_finished) | `net.minecraft.network.protocol.login.ClientboundLoginFinishedPacket` |
| 3 | [`minecraft:login_compression`](#pkt-clientbound-minecraft-login_compression) | `net.minecraft.network.protocol.login.ClientboundLoginCompressionPacket` |
| 4 | [`minecraft:custom_query`](#pkt-clientbound-minecraft-custom_query) | `net.minecraft.network.protocol.login.ClientboundCustomQueryPacket` |
| 5 | [`minecraft:cookie_request`](#pkt-clientbound-minecraft-cookie_request) | `net.minecraft.network.protocol.cookie.ClientboundCookieRequestPacket` |

<a id="pkt-clientbound-minecraft-login_disconnect"></a>
### minecraft:login_disconnect (clientbound, id 0)

- `reason`: `JSON_TEXT` (max=262144)

<a id="pkt-clientbound-minecraft-hello"></a>
### minecraft:hello (clientbound, id 1)

- `serverId`: string (at most 20 characters)
- `publicKey`: `BYTE_ARRAY`
- `challenge`: `BYTE_ARRAY`
- `shouldAuthenticate`: `BOOL`

<a id="pkt-clientbound-minecraft-login_finished"></a>
### minecraft:login_finished (clientbound, id 2)

- `gameProfile`: struct `GameProfile`
  - `id`: `UUID`
  - `name`: `STRING`
  - `properties`: `GAME_PROFILE_PROPERTIES`
- `sessionId`: `UUID`

<a id="pkt-clientbound-minecraft-login_compression"></a>
### minecraft:login_compression (clientbound, id 3)

- `compressionThreshold`: `VAR_INT`

<a id="pkt-clientbound-minecraft-custom_query"></a>
### minecraft:custom_query (clientbound, id 4)

- `transactionId`: `VAR_INT`
- `identifier`: `IDENTIFIER`
- `payload`: `REST_BYTES`

<a id="pkt-clientbound-minecraft-cookie_request"></a>
### minecraft:cookie_request (clientbound, id 5)

- `key`: `IDENTIFIER`

## login serverbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:hello`](#pkt-serverbound-minecraft-hello) | `net.minecraft.network.protocol.login.ServerboundHelloPacket` |
| 1 | [`minecraft:key`](#pkt-serverbound-minecraft-key) | `net.minecraft.network.protocol.login.ServerboundKeyPacket` |
| 2 | [`minecraft:custom_query_answer`](#pkt-serverbound-minecraft-custom_query_answer) | `net.minecraft.network.protocol.login.ServerboundCustomQueryAnswerPacket` |
| 3 | [`minecraft:login_acknowledged`](#pkt-serverbound-minecraft-login_acknowledged) | `net.minecraft.network.protocol.login.ServerboundLoginAcknowledgedPacket` |
| 4 | [`minecraft:cookie_response`](#pkt-serverbound-minecraft-cookie_response) | `net.minecraft.network.protocol.cookie.ServerboundCookieResponsePacket` |

<a id="pkt-serverbound-minecraft-hello"></a>
### minecraft:hello (serverbound, id 0)

- `name`: string (at most 16 characters)
- `profileId`: `UUID`

<a id="pkt-serverbound-minecraft-key"></a>
### minecraft:key (serverbound, id 1)

- `keybytes`: `BYTE_ARRAY`
- `encryptedChallenge`: `BYTE_ARRAY`

<a id="pkt-serverbound-minecraft-custom_query_answer"></a>
### minecraft:custom_query_answer (serverbound, id 2)

- `transactionId`: `VAR_INT`
- `payload`: optional `REST_BYTES`

<a id="pkt-serverbound-minecraft-login_acknowledged"></a>
### minecraft:login_acknowledged (serverbound, id 3)

- nothing

<a id="pkt-serverbound-minecraft-cookie_response"></a>
### minecraft:cookie_response (serverbound, id 4)

- `key`: `IDENTIFIER`
- `payload`: optional `BYTE_ARRAY` (max=5120)

## configuration clientbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:cookie_request`](#pkt-clientbound-minecraft-cookie_request) | `net.minecraft.network.protocol.cookie.ClientboundCookieRequestPacket` |
| 1 | [`minecraft:custom_payload`](#pkt-clientbound-minecraft-custom_payload) | `net.minecraft.network.protocol.common.ClientboundCustomPayloadPacket` |
| 2 | [`minecraft:disconnect`](#pkt-clientbound-minecraft-disconnect) | `net.minecraft.network.protocol.common.ClientboundDisconnectPacket` |
| 3 | [`minecraft:finish_configuration`](#pkt-clientbound-minecraft-finish_configuration) | `net.minecraft.network.protocol.configuration.ClientboundFinishConfigurationPacket` |
| 4 | [`minecraft:keep_alive`](#pkt-clientbound-minecraft-keep_alive) | `net.minecraft.network.protocol.common.ClientboundKeepAlivePacket` |
| 5 | [`minecraft:ping`](#pkt-clientbound-minecraft-ping) | `net.minecraft.network.protocol.common.ClientboundPingPacket` |
| 6 | [`minecraft:reset_chat`](#pkt-clientbound-minecraft-reset_chat) | `net.minecraft.network.protocol.configuration.ClientboundResetChatPacket` |
| 7 | [`minecraft:registry_data`](#pkt-clientbound-minecraft-registry_data) | `net.minecraft.network.protocol.configuration.ClientboundRegistryDataPacket` |
| 8 | [`minecraft:resource_pack_pop`](#pkt-clientbound-minecraft-resource_pack_pop) | `net.minecraft.network.protocol.common.ClientboundResourcePackPopPacket` |
| 9 | [`minecraft:resource_pack_push`](#pkt-clientbound-minecraft-resource_pack_push) | `net.minecraft.network.protocol.common.ClientboundResourcePackPushPacket` |
| 10 | [`minecraft:post_effects`](#pkt-clientbound-minecraft-post_effects) | `net.minecraft.network.protocol.common.ClientboundPostEffectsPacket` |
| 11 | [`minecraft:store_cookie`](#pkt-clientbound-minecraft-store_cookie) | `net.minecraft.network.protocol.common.ClientboundStoreCookiePacket` |
| 12 | [`minecraft:transfer`](#pkt-clientbound-minecraft-transfer) | `net.minecraft.network.protocol.common.ClientboundTransferPacket` |
| 13 | [`minecraft:update_enabled_features`](#pkt-clientbound-minecraft-update_enabled_features) | `net.minecraft.network.protocol.configuration.ClientboundUpdateEnabledFeaturesPacket` |
| 14 | [`minecraft:update_tags`](#pkt-clientbound-minecraft-update_tags) | `net.minecraft.network.protocol.common.ClientboundUpdateTagsPacket` |
| 15 | [`minecraft:select_known_packs`](#pkt-clientbound-minecraft-select_known_packs) | `net.minecraft.network.protocol.configuration.ClientboundSelectKnownPacks` |
| 16 | [`minecraft:custom_report_details`](#pkt-clientbound-minecraft-custom_report_details) | `net.minecraft.network.protocol.common.ClientboundCustomReportDetailsPacket` |
| 17 | [`minecraft:server_links`](#pkt-clientbound-minecraft-server_links) | `net.minecraft.network.protocol.common.ClientboundServerLinksPacket` |
| 18 | [`minecraft:clear_dialog`](#pkt-clientbound-minecraft-clear_dialog) | `net.minecraft.network.protocol.common.ClientboundClearDialogPacket` |
| 19 | [`minecraft:show_dialog`](#pkt-clientbound-minecraft-show_dialog) | `net.minecraft.network.protocol.common.ClientboundShowDialogPacket` |
| 20 | [`minecraft:code_of_conduct`](#pkt-clientbound-minecraft-code_of_conduct) | `net.minecraft.network.protocol.configuration.ClientboundCodeOfConductPacket` |

<a id="pkt-clientbound-minecraft-cookie_request"></a>
### minecraft:cookie_request (clientbound, id 0)

- `key`: `IDENTIFIER`

<a id="pkt-clientbound-minecraft-custom_payload"></a>
### minecraft:custom_payload (clientbound, id 1)

- `channel`: `IDENTIFIER`
- `data`: `REST_BYTES`

<a id="pkt-clientbound-minecraft-disconnect"></a>
### minecraft:disconnect (clientbound, id 2)

- `TEXT`

<a id="pkt-clientbound-minecraft-finish_configuration"></a>
### minecraft:finish_configuration (clientbound, id 3)

- nothing

<a id="pkt-clientbound-minecraft-keep_alive"></a>
### minecraft:keep_alive (clientbound, id 4)

- `id`: `LONG`

<a id="pkt-clientbound-minecraft-ping"></a>
### minecraft:ping (clientbound, id 5)

- `id`: `INT`

<a id="pkt-clientbound-minecraft-reset_chat"></a>
### minecraft:reset_chat (clientbound, id 6)

- nothing

<a id="pkt-clientbound-minecraft-registry_data"></a>
### minecraft:registry_data (clientbound, id 7)

- `registry`: `IDENTIFIER`
- `entries`: list of
  - struct `RegistrySynchronization$PackedRegistryEntry`
    - `id`: `IDENTIFIER`
    - `data`: optional `NBT`

<a id="pkt-clientbound-minecraft-resource_pack_pop"></a>
### minecraft:resource_pack_pop (clientbound, id 8)

- `id`: optional `UUID`

<a id="pkt-clientbound-minecraft-resource_pack_push"></a>
### minecraft:resource_pack_push (clientbound, id 9)

- `id`: `UUID`
- `url`: `STRING`
- `hash`: string (at most 40 characters)
- `required`: `BOOL`
- `prompt`: optional `TEXT`

<a id="pkt-clientbound-minecraft-post_effects"></a>
### minecraft:post_effects (clientbound, id 10)

- `postEffects`: list of `IDENTIFIER`

<a id="pkt-clientbound-minecraft-store_cookie"></a>
### minecraft:store_cookie (clientbound, id 11)

- `key`: `IDENTIFIER`
- `payload`: `BYTE_ARRAY` (max=5120)

<a id="pkt-clientbound-minecraft-transfer"></a>
### minecraft:transfer (clientbound, id 12)

- `host`: string
- `port`: `VAR_INT`

<a id="pkt-clientbound-minecraft-update_enabled_features"></a>
### minecraft:update_enabled_features (clientbound, id 13)

- `features`: list of `IDENTIFIER`

<a id="pkt-clientbound-minecraft-update_tags"></a>
### minecraft:update_tags (clientbound, id 14)

- `tags`: map
  - key: `IDENTIFIER`
  - value: struct `TagNetworkSerialization$NetworkPayload`
    - `tags`: map of `IDENTIFIER` to list of `VAR_INT`

<a id="pkt-clientbound-minecraft-select_known_packs"></a>
### minecraft:select_known_packs (clientbound, id 15)

- `knownPacks`: list of
  - struct `KnownPack`
    - `namespace`: `STRING`
    - `id`: `STRING`
    - `version`: `STRING`

<a id="pkt-clientbound-minecraft-custom_report_details"></a>
### minecraft:custom_report_details (clientbound, id 16)

- `details`: map of string (at most 128 characters) to string (at most 4096 characters) (at most 32)

<a id="pkt-clientbound-minecraft-server_links"></a>
### minecraft:server_links (clientbound, id 17)

- `links`: list of
  - struct `ServerLinks$UntrustedEntry`
    - `type`: either enum `ServerLinks$KnownLinkType` (var int, ordinal: BUG_REPORT, COMMUNITY_GUIDELINES, SUPPORT, STATUS, FEEDBACK, COMMUNITY, WEBSITE, FORUMS, NEWS, ANNOUNCEMENTS) or `TEXT`
    - `link`: `STRING`

<a id="pkt-clientbound-minecraft-clear_dialog"></a>
### minecraft:clear_dialog (clientbound, id 18)

- nothing

<a id="pkt-clientbound-minecraft-show_dialog"></a>
### minecraft:show_dialog (clientbound, id 19)

- `dialog`: id in dialog or inline an NBT tag

<a id="pkt-clientbound-minecraft-code_of_conduct"></a>
### minecraft:code_of_conduct (clientbound, id 20)

- `codeOfConduct`: `STRING`

## configuration serverbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:client_information`](#pkt-serverbound-minecraft-client_information) | `net.minecraft.network.protocol.common.ServerboundClientInformationPacket` |
| 1 | [`minecraft:cookie_response`](#pkt-serverbound-minecraft-cookie_response) | `net.minecraft.network.protocol.cookie.ServerboundCookieResponsePacket` |
| 2 | [`minecraft:custom_payload`](#pkt-serverbound-minecraft-custom_payload) | `net.minecraft.network.protocol.common.ServerboundCustomPayloadPacket` |
| 3 | [`minecraft:finish_configuration`](#pkt-serverbound-minecraft-finish_configuration) | `net.minecraft.network.protocol.configuration.ServerboundFinishConfigurationPacket` |
| 4 | [`minecraft:keep_alive`](#pkt-serverbound-minecraft-keep_alive) | `net.minecraft.network.protocol.common.ServerboundKeepAlivePacket` |
| 5 | [`minecraft:pong`](#pkt-serverbound-minecraft-pong) | `net.minecraft.network.protocol.common.ServerboundPongPacket` |
| 6 | [`minecraft:resource_pack`](#pkt-serverbound-minecraft-resource_pack) | `net.minecraft.network.protocol.common.ServerboundResourcePackPacket` |
| 7 | [`minecraft:select_known_packs`](#pkt-serverbound-minecraft-select_known_packs) | `net.minecraft.network.protocol.configuration.ServerboundSelectKnownPacks` |
| 8 | [`minecraft:custom_click_action`](#pkt-serverbound-minecraft-custom_click_action) | `net.minecraft.network.protocol.common.ServerboundCustomClickActionPacket` |
| 9 | [`minecraft:accept_code_of_conduct`](#pkt-serverbound-minecraft-accept_code_of_conduct) | `net.minecraft.network.protocol.configuration.ServerboundAcceptCodeOfConductPacket` |

<a id="pkt-serverbound-minecraft-client_information"></a>
### minecraft:client_information (serverbound, id 0)

- `information`: struct `ClientInformation`
  - `language`: string (at most 16 characters)
  - `viewDistance`: `BYTE`
  - `chatVisibility`: enum `ChatVisiblity` (var int, ordinal: FULL, SYSTEM, HIDDEN)
  - `chatColors`: `BOOL`
  - `modelCustomisation`: `UNSIGNED_BYTE`
  - `mainHand`: enum `HumanoidArm` (var int, ordinal: LEFT, RIGHT)
  - `textFilteringEnabled`: `BOOL`
  - `allowsListing`: `BOOL`
  - `particleStatus`: enum `ParticleStatus` (var int, ordinal: ALL, DECREASED, MINIMAL)

<a id="pkt-serverbound-minecraft-cookie_response"></a>
### minecraft:cookie_response (serverbound, id 1)

- `key`: `IDENTIFIER`
- `payload`: optional `BYTE_ARRAY` (max=5120)

<a id="pkt-serverbound-minecraft-custom_payload"></a>
### minecraft:custom_payload (serverbound, id 2)

- `channel`: `IDENTIFIER`
- `data`: `REST_BYTES`

<a id="pkt-serverbound-minecraft-finish_configuration"></a>
### minecraft:finish_configuration (serverbound, id 3)

- nothing

<a id="pkt-serverbound-minecraft-keep_alive"></a>
### minecraft:keep_alive (serverbound, id 4)

- `id`: `LONG`

<a id="pkt-serverbound-minecraft-pong"></a>
### minecraft:pong (serverbound, id 5)

- `id`: `INT`

<a id="pkt-serverbound-minecraft-resource_pack"></a>
### minecraft:resource_pack (serverbound, id 6)

- `id`: `UUID`
- `action`: enum `ServerboundResourcePackPacket$Action` (var int, ordinal: SUCCESSFULLY_LOADED, DECLINED, FAILED_DOWNLOAD, ACCEPTED, DOWNLOADED, INVALID_URL, FAILED_RELOAD, DISCARDED)

<a id="pkt-serverbound-minecraft-select_known_packs"></a>
### minecraft:select_known_packs (serverbound, id 7)

- `knownPacks`: list of (at most 64)
  - struct `KnownPack`
    - `namespace`: `STRING`
    - `id`: `STRING`
    - `version`: `STRING`

<a id="pkt-serverbound-minecraft-custom_click_action"></a>
### minecraft:custom_click_action (serverbound, id 8)

- `id`: `IDENTIFIER`
- `payload`: length-prefixed an NBT tag

<a id="pkt-serverbound-minecraft-accept_code_of_conduct"></a>
### minecraft:accept_code_of_conduct (serverbound, id 9)

- nothing

## play clientbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:bundle_delimiter`](#pkt-clientbound-minecraft-bundle_delimiter) | `net.minecraft.network.protocol.game.ClientboundBundleDelimiterPacket` |
| 1 | [`minecraft:add_entity`](#pkt-clientbound-minecraft-add_entity) | `net.minecraft.network.protocol.game.ClientboundAddEntityPacket` |
| 2 | [`minecraft:animate`](#pkt-clientbound-minecraft-animate) | `net.minecraft.network.protocol.game.ClientboundAnimatePacket` |
| 3 | [`minecraft:award_stats`](#pkt-clientbound-minecraft-award_stats) | `net.minecraft.network.protocol.game.ClientboundAwardStatsPacket` |
| 4 | [`minecraft:block_changed_ack`](#pkt-clientbound-minecraft-block_changed_ack) | `net.minecraft.network.protocol.game.ClientboundBlockChangedAckPacket` |
| 5 | [`minecraft:block_destruction`](#pkt-clientbound-minecraft-block_destruction) | `net.minecraft.network.protocol.game.ClientboundBlockDestructionPacket` |
| 6 | [`minecraft:block_entity_data`](#pkt-clientbound-minecraft-block_entity_data) | `net.minecraft.network.protocol.game.ClientboundBlockEntityDataPacket` |
| 7 | [`minecraft:block_event`](#pkt-clientbound-minecraft-block_event) | `net.minecraft.network.protocol.game.ClientboundBlockEventPacket` |
| 8 | [`minecraft:block_update`](#pkt-clientbound-minecraft-block_update) | `net.minecraft.network.protocol.game.ClientboundBlockUpdatePacket` |
| 9 | [`minecraft:boss_event`](#pkt-clientbound-minecraft-boss_event) | `net.minecraft.network.protocol.game.ClientboundBossEventPacket` |
| 10 | [`minecraft:change_difficulty`](#pkt-clientbound-minecraft-change_difficulty) | `net.minecraft.network.protocol.game.ClientboundChangeDifficultyPacket` |
| 11 | [`minecraft:chunk_batch_finished`](#pkt-clientbound-minecraft-chunk_batch_finished) | `net.minecraft.network.protocol.game.ClientboundChunkBatchFinishedPacket` |
| 12 | [`minecraft:chunk_batch_start`](#pkt-clientbound-minecraft-chunk_batch_start) | `net.minecraft.network.protocol.game.ClientboundChunkBatchStartPacket` |
| 13 | [`minecraft:chunks_biomes`](#pkt-clientbound-minecraft-chunks_biomes) | `net.minecraft.network.protocol.game.ClientboundChunksBiomesPacket` |
| 14 | [`minecraft:clear_titles`](#pkt-clientbound-minecraft-clear_titles) | `net.minecraft.network.protocol.game.ClientboundClearTitlesPacket` |
| 15 | [`minecraft:command_suggestions`](#pkt-clientbound-minecraft-command_suggestions) | `net.minecraft.network.protocol.game.ClientboundCommandSuggestionsPacket` |
| 16 | [`minecraft:commands`](#pkt-clientbound-minecraft-commands) | `net.minecraft.network.protocol.game.ClientboundCommandsPacket` |
| 17 | [`minecraft:container_close`](#pkt-clientbound-minecraft-container_close) | `net.minecraft.network.protocol.game.ClientboundContainerClosePacket` |
| 18 | [`minecraft:container_set_content`](#pkt-clientbound-minecraft-container_set_content) | `net.minecraft.network.protocol.game.ClientboundContainerSetContentPacket` |
| 19 | [`minecraft:container_set_data`](#pkt-clientbound-minecraft-container_set_data) | `net.minecraft.network.protocol.game.ClientboundContainerSetDataPacket` |
| 20 | [`minecraft:container_set_slot`](#pkt-clientbound-minecraft-container_set_slot) | `net.minecraft.network.protocol.game.ClientboundContainerSetSlotPacket` |
| 21 | [`minecraft:cookie_request`](#pkt-clientbound-minecraft-cookie_request) | `net.minecraft.network.protocol.cookie.ClientboundCookieRequestPacket` |
| 22 | [`minecraft:cooldown`](#pkt-clientbound-minecraft-cooldown) | `net.minecraft.network.protocol.game.ClientboundCooldownPacket` |
| 23 | [`minecraft:custom_chat_completions`](#pkt-clientbound-minecraft-custom_chat_completions) | `net.minecraft.network.protocol.game.ClientboundCustomChatCompletionsPacket` |
| 24 | [`minecraft:custom_payload`](#pkt-clientbound-minecraft-custom_payload) | `net.minecraft.network.protocol.common.ClientboundCustomPayloadPacket` |
| 25 | [`minecraft:damage_event`](#pkt-clientbound-minecraft-damage_event) | `net.minecraft.network.protocol.game.ClientboundDamageEventPacket` |
| 26 | [`minecraft:debug/block_value`](#pkt-clientbound-minecraft-debug-block_value) | `net.minecraft.network.protocol.game.ClientboundDebugBlockValuePacket` |
| 27 | [`minecraft:debug/chunk_value`](#pkt-clientbound-minecraft-debug-chunk_value) | `net.minecraft.network.protocol.game.ClientboundDebugChunkValuePacket` |
| 28 | [`minecraft:debug/entity_value`](#pkt-clientbound-minecraft-debug-entity_value) | `net.minecraft.network.protocol.game.ClientboundDebugEntityValuePacket` |
| 29 | [`minecraft:debug/event`](#pkt-clientbound-minecraft-debug-event) | `net.minecraft.network.protocol.game.ClientboundDebugEventPacket` |
| 30 | [`minecraft:debug_sample`](#pkt-clientbound-minecraft-debug_sample) | `net.minecraft.network.protocol.game.ClientboundDebugSamplePacket` |
| 31 | [`minecraft:delete_chat`](#pkt-clientbound-minecraft-delete_chat) | `net.minecraft.network.protocol.game.ClientboundDeleteChatPacket` |
| 32 | [`minecraft:disconnect`](#pkt-clientbound-minecraft-disconnect) | `net.minecraft.network.protocol.common.ClientboundDisconnectPacket` |
| 33 | [`minecraft:disguised_chat`](#pkt-clientbound-minecraft-disguised_chat) | `net.minecraft.network.protocol.game.ClientboundDisguisedChatPacket` |
| 34 | [`minecraft:entity_event`](#pkt-clientbound-minecraft-entity_event) | `net.minecraft.network.protocol.game.ClientboundEntityEventPacket` |
| 35 | [`minecraft:entity_position_sync`](#pkt-clientbound-minecraft-entity_position_sync) | `net.minecraft.network.protocol.game.ClientboundEntityPositionSyncPacket` |
| 36 | [`minecraft:explode`](#pkt-clientbound-minecraft-explode) | `net.minecraft.network.protocol.game.ClientboundExplodePacket` |
| 37 | [`minecraft:add_transient_block`](#pkt-clientbound-minecraft-add_transient_block) | `net.minecraft.network.protocol.game.ClientboundAddTransientBlockPacket` |
| 38 | [`minecraft:forget_level_chunk`](#pkt-clientbound-minecraft-forget_level_chunk) | `net.minecraft.network.protocol.game.ClientboundForgetLevelChunkPacket` |
| 39 | [`minecraft:game_event`](#pkt-clientbound-minecraft-game_event) | `net.minecraft.network.protocol.game.ClientboundGameEventPacket` |
| 40 | [`minecraft:game_rule_values`](#pkt-clientbound-minecraft-game_rule_values) | `net.minecraft.network.protocol.game.ClientboundGameRuleValuesPacket` |
| 41 | [`minecraft:game_test_highlight_pos`](#pkt-clientbound-minecraft-game_test_highlight_pos) | `net.minecraft.network.protocol.game.ClientboundGameTestHighlightPosPacket` |
| 42 | [`minecraft:mount_screen_open`](#pkt-clientbound-minecraft-mount_screen_open) | `net.minecraft.network.protocol.game.ClientboundMountScreenOpenPacket` |
| 43 | [`minecraft:hurt_animation`](#pkt-clientbound-minecraft-hurt_animation) | `net.minecraft.network.protocol.game.ClientboundHurtAnimationPacket` |
| 44 | [`minecraft:initialize_border`](#pkt-clientbound-minecraft-initialize_border) | `net.minecraft.network.protocol.game.ClientboundInitializeBorderPacket` |
| 45 | [`minecraft:keep_alive`](#pkt-clientbound-minecraft-keep_alive) | `net.minecraft.network.protocol.common.ClientboundKeepAlivePacket` |
| 46 | [`minecraft:level_chunk_with_light`](#pkt-clientbound-minecraft-level_chunk_with_light) | `net.minecraft.network.protocol.game.ClientboundLevelChunkWithLightPacket` |
| 47 | [`minecraft:level_event`](#pkt-clientbound-minecraft-level_event) | `net.minecraft.network.protocol.game.ClientboundLevelEventPacket` |
| 48 | [`minecraft:level_particles`](#pkt-clientbound-minecraft-level_particles) | `net.minecraft.network.protocol.game.ClientboundLevelParticlesPacket` |
| 49 | [`minecraft:light_update`](#pkt-clientbound-minecraft-light_update) | `net.minecraft.network.protocol.game.ClientboundLightUpdatePacket` |
| 50 | [`minecraft:login`](#pkt-clientbound-minecraft-login) | `net.minecraft.network.protocol.game.ClientboundLoginPacket` |
| 51 | [`minecraft:low_disk_space_warning`](#pkt-clientbound-minecraft-low_disk_space_warning) | `net.minecraft.network.protocol.game.ClientboundLowDiskSpaceWarningPacket` |
| 52 | [`minecraft:map_item_data`](#pkt-clientbound-minecraft-map_item_data) | `net.minecraft.network.protocol.game.ClientboundMapItemDataPacket` |
| 53 | [`minecraft:merchant_offers`](#pkt-clientbound-minecraft-merchant_offers) | `net.minecraft.network.protocol.game.ClientboundMerchantOffersPacket` |
| 54 | [`minecraft:move_entity_pos`](#pkt-clientbound-minecraft-move_entity_pos) | `net.minecraft.network.protocol.game.ClientboundMoveEntityPacket$Pos` |
| 55 | [`minecraft:move_entity_pos_rot`](#pkt-clientbound-minecraft-move_entity_pos_rot) | `net.minecraft.network.protocol.game.ClientboundMoveEntityPacket$PosRot` |
| 56 | [`minecraft:move_minecart_along_track`](#pkt-clientbound-minecraft-move_minecart_along_track) | `net.minecraft.network.protocol.game.ClientboundMoveMinecartPacket` |
| 57 | [`minecraft:move_entity_rot`](#pkt-clientbound-minecraft-move_entity_rot) | `net.minecraft.network.protocol.game.ClientboundMoveEntityPacket$Rot` |
| 58 | [`minecraft:move_vehicle`](#pkt-clientbound-minecraft-move_vehicle) | `net.minecraft.network.protocol.game.ClientboundMoveVehiclePacket` |
| 59 | [`minecraft:open_book`](#pkt-clientbound-minecraft-open_book) | `net.minecraft.network.protocol.game.ClientboundOpenBookPacket` |
| 60 | [`minecraft:open_screen`](#pkt-clientbound-minecraft-open_screen) | `net.minecraft.network.protocol.game.ClientboundOpenScreenPacket` |
| 61 | [`minecraft:open_sign_editor`](#pkt-clientbound-minecraft-open_sign_editor) | `net.minecraft.network.protocol.game.ClientboundOpenSignEditorPacket` |
| 62 | [`minecraft:ping`](#pkt-clientbound-minecraft-ping) | `net.minecraft.network.protocol.common.ClientboundPingPacket` |
| 63 | [`minecraft:pong_response`](#pkt-clientbound-minecraft-pong_response) | `net.minecraft.network.protocol.ping.ClientboundPongResponsePacket` |
| 64 | [`minecraft:place_ghost_recipe`](#pkt-clientbound-minecraft-place_ghost_recipe) | `net.minecraft.network.protocol.game.ClientboundPlaceGhostRecipePacket` |
| 65 | [`minecraft:player_abilities`](#pkt-clientbound-minecraft-player_abilities) | `net.minecraft.network.protocol.game.ClientboundPlayerAbilitiesPacket` |
| 66 | [`minecraft:player_chat`](#pkt-clientbound-minecraft-player_chat) | `net.minecraft.network.protocol.game.ClientboundPlayerChatPacket` |
| 67 | [`minecraft:player_combat_end`](#pkt-clientbound-minecraft-player_combat_end) | `net.minecraft.network.protocol.game.ClientboundPlayerCombatEndPacket` |
| 68 | [`minecraft:player_combat_enter`](#pkt-clientbound-minecraft-player_combat_enter) | `net.minecraft.network.protocol.game.ClientboundPlayerCombatEnterPacket` |
| 69 | [`minecraft:player_combat_kill`](#pkt-clientbound-minecraft-player_combat_kill) | `net.minecraft.network.protocol.game.ClientboundPlayerCombatKillPacket` |
| 70 | [`minecraft:player_info_remove`](#pkt-clientbound-minecraft-player_info_remove) | `net.minecraft.network.protocol.game.ClientboundPlayerInfoRemovePacket` |
| 71 | [`minecraft:player_info_update`](#pkt-clientbound-minecraft-player_info_update) | `net.minecraft.network.protocol.game.ClientboundPlayerInfoUpdatePacket` |
| 72 | [`minecraft:player_look_at`](#pkt-clientbound-minecraft-player_look_at) | `net.minecraft.network.protocol.game.ClientboundPlayerLookAtPacket` |
| 73 | [`minecraft:player_position`](#pkt-clientbound-minecraft-player_position) | `net.minecraft.network.protocol.game.ClientboundPlayerPositionPacket` |
| 74 | [`minecraft:player_rotation`](#pkt-clientbound-minecraft-player_rotation) | `net.minecraft.network.protocol.game.ClientboundPlayerRotationPacket` |
| 75 | [`minecraft:recipe_book_add`](#pkt-clientbound-minecraft-recipe_book_add) | `net.minecraft.network.protocol.game.ClientboundRecipeBookAddPacket` |
| 76 | [`minecraft:recipe_book_remove`](#pkt-clientbound-minecraft-recipe_book_remove) | `net.minecraft.network.protocol.game.ClientboundRecipeBookRemovePacket` |
| 77 | [`minecraft:recipe_book_settings`](#pkt-clientbound-minecraft-recipe_book_settings) | `net.minecraft.network.protocol.game.ClientboundRecipeBookSettingsPacket` |
| 78 | [`minecraft:remove_entities`](#pkt-clientbound-minecraft-remove_entities) | `net.minecraft.network.protocol.game.ClientboundRemoveEntitiesPacket` |
| 79 | [`minecraft:remove_mob_effect`](#pkt-clientbound-minecraft-remove_mob_effect) | `net.minecraft.network.protocol.game.ClientboundRemoveMobEffectPacket` |
| 80 | [`minecraft:reset_score`](#pkt-clientbound-minecraft-reset_score) | `net.minecraft.network.protocol.game.ClientboundResetScorePacket` |
| 81 | [`minecraft:resource_pack_pop`](#pkt-clientbound-minecraft-resource_pack_pop) | `net.minecraft.network.protocol.common.ClientboundResourcePackPopPacket` |
| 82 | [`minecraft:resource_pack_push`](#pkt-clientbound-minecraft-resource_pack_push) | `net.minecraft.network.protocol.common.ClientboundResourcePackPushPacket` |
| 83 | [`minecraft:post_effects`](#pkt-clientbound-minecraft-post_effects) | `net.minecraft.network.protocol.common.ClientboundPostEffectsPacket` |
| 84 | [`minecraft:respawn`](#pkt-clientbound-minecraft-respawn) | `net.minecraft.network.protocol.game.ClientboundRespawnPacket` |
| 85 | [`minecraft:rotate_head`](#pkt-clientbound-minecraft-rotate_head) | `net.minecraft.network.protocol.game.ClientboundRotateHeadPacket` |
| 86 | [`minecraft:section_blocks_update`](#pkt-clientbound-minecraft-section_blocks_update) | `net.minecraft.network.protocol.game.ClientboundSectionBlocksUpdatePacket` |
| 87 | [`minecraft:select_advancements_tab`](#pkt-clientbound-minecraft-select_advancements_tab) | `net.minecraft.network.protocol.game.ClientboundSelectAdvancementsTabPacket` |
| 88 | [`minecraft:server_data`](#pkt-clientbound-minecraft-server_data) | `net.minecraft.network.protocol.game.ClientboundServerDataPacket` |
| 89 | [`minecraft:set_action_bar_text`](#pkt-clientbound-minecraft-set_action_bar_text) | `net.minecraft.network.protocol.game.ClientboundSetActionBarTextPacket` |
| 90 | [`minecraft:set_border_center`](#pkt-clientbound-minecraft-set_border_center) | `net.minecraft.network.protocol.game.ClientboundSetBorderCenterPacket` |
| 91 | [`minecraft:set_border_lerp_size`](#pkt-clientbound-minecraft-set_border_lerp_size) | `net.minecraft.network.protocol.game.ClientboundSetBorderLerpSizePacket` |
| 92 | [`minecraft:set_border_size`](#pkt-clientbound-minecraft-set_border_size) | `net.minecraft.network.protocol.game.ClientboundSetBorderSizePacket` |
| 93 | [`minecraft:set_border_warning_delay`](#pkt-clientbound-minecraft-set_border_warning_delay) | `net.minecraft.network.protocol.game.ClientboundSetBorderWarningDelayPacket` |
| 94 | [`minecraft:set_border_warning_distance`](#pkt-clientbound-minecraft-set_border_warning_distance) | `net.minecraft.network.protocol.game.ClientboundSetBorderWarningDistancePacket` |
| 95 | [`minecraft:set_camera`](#pkt-clientbound-minecraft-set_camera) | `net.minecraft.network.protocol.game.ClientboundSetCameraPacket` |
| 96 | [`minecraft:set_chunk_cache_center`](#pkt-clientbound-minecraft-set_chunk_cache_center) | `net.minecraft.network.protocol.game.ClientboundSetChunkCacheCenterPacket` |
| 97 | [`minecraft:set_chunk_cache_radius`](#pkt-clientbound-minecraft-set_chunk_cache_radius) | `net.minecraft.network.protocol.game.ClientboundSetChunkCacheRadiusPacket` |
| 98 | [`minecraft:set_cursor_item`](#pkt-clientbound-minecraft-set_cursor_item) | `net.minecraft.network.protocol.game.ClientboundSetCursorItemPacket` |
| 99 | [`minecraft:set_default_spawn_position`](#pkt-clientbound-minecraft-set_default_spawn_position) | `net.minecraft.network.protocol.game.ClientboundSetDefaultSpawnPositionPacket` |
| 100 | [`minecraft:set_display_objective`](#pkt-clientbound-minecraft-set_display_objective) | `net.minecraft.network.protocol.game.ClientboundSetDisplayObjectivePacket` |
| 101 | [`minecraft:set_entity_data`](#pkt-clientbound-minecraft-set_entity_data) | `net.minecraft.network.protocol.game.ClientboundSetEntityDataPacket` |
| 102 | [`minecraft:set_entity_link`](#pkt-clientbound-minecraft-set_entity_link) | `net.minecraft.network.protocol.game.ClientboundSetEntityLinkPacket` |
| 103 | [`minecraft:set_entity_motion`](#pkt-clientbound-minecraft-set_entity_motion) | `net.minecraft.network.protocol.game.ClientboundSetEntityMotionPacket` |
| 104 | [`minecraft:set_equipment`](#pkt-clientbound-minecraft-set_equipment) | `net.minecraft.network.protocol.game.ClientboundSetEquipmentPacket` |
| 105 | [`minecraft:set_experience`](#pkt-clientbound-minecraft-set_experience) | `net.minecraft.network.protocol.game.ClientboundSetExperiencePacket` |
| 106 | [`minecraft:set_health`](#pkt-clientbound-minecraft-set_health) | `net.minecraft.network.protocol.game.ClientboundSetHealthPacket` |
| 107 | [`minecraft:set_held_slot`](#pkt-clientbound-minecraft-set_held_slot) | `net.minecraft.network.protocol.game.ClientboundSetHeldSlotPacket` |
| 108 | [`minecraft:set_objective`](#pkt-clientbound-minecraft-set_objective) | `net.minecraft.network.protocol.game.ClientboundSetObjectivePacket` |
| 109 | [`minecraft:set_passengers`](#pkt-clientbound-minecraft-set_passengers) | `net.minecraft.network.protocol.game.ClientboundSetPassengersPacket` |
| 110 | [`minecraft:set_player_inventory`](#pkt-clientbound-minecraft-set_player_inventory) | `net.minecraft.network.protocol.game.ClientboundSetPlayerInventoryPacket` |
| 111 | [`minecraft:set_player_team`](#pkt-clientbound-minecraft-set_player_team) | `net.minecraft.network.protocol.game.ClientboundSetPlayerTeamPacket` |
| 112 | [`minecraft:set_score`](#pkt-clientbound-minecraft-set_score) | `net.minecraft.network.protocol.game.ClientboundSetScorePacket` |
| 113 | [`minecraft:set_simulation_distance`](#pkt-clientbound-minecraft-set_simulation_distance) | `net.minecraft.network.protocol.game.ClientboundSetSimulationDistancePacket` |
| 114 | [`minecraft:set_subtitle_text`](#pkt-clientbound-minecraft-set_subtitle_text) | `net.minecraft.network.protocol.game.ClientboundSetSubtitleTextPacket` |
| 115 | [`minecraft:set_time`](#pkt-clientbound-minecraft-set_time) | `net.minecraft.network.protocol.game.ClientboundSetTimePacket` |
| 116 | [`minecraft:set_title_text`](#pkt-clientbound-minecraft-set_title_text) | `net.minecraft.network.protocol.game.ClientboundSetTitleTextPacket` |
| 117 | [`minecraft:set_titles_animation`](#pkt-clientbound-minecraft-set_titles_animation) | `net.minecraft.network.protocol.game.ClientboundSetTitlesAnimationPacket` |
| 118 | [`minecraft:sound_entity`](#pkt-clientbound-minecraft-sound_entity) | `net.minecraft.network.protocol.game.ClientboundSoundEntityPacket` |
| 119 | [`minecraft:sound`](#pkt-clientbound-minecraft-sound) | `net.minecraft.network.protocol.game.ClientboundSoundPacket` |
| 120 | [`minecraft:start_configuration`](#pkt-clientbound-minecraft-start_configuration) | `net.minecraft.network.protocol.game.ClientboundStartConfigurationPacket` |
| 121 | [`minecraft:stop_sound`](#pkt-clientbound-minecraft-stop_sound) | `net.minecraft.network.protocol.game.ClientboundStopSoundPacket` |
| 122 | [`minecraft:store_cookie`](#pkt-clientbound-minecraft-store_cookie) | `net.minecraft.network.protocol.common.ClientboundStoreCookiePacket` |
| 123 | [`minecraft:swing_animation`](#pkt-clientbound-minecraft-swing_animation) | `net.minecraft.network.protocol.game.ClientboundSwingAnimationPacket` |
| 124 | [`minecraft:system_chat`](#pkt-clientbound-minecraft-system_chat) | `net.minecraft.network.protocol.game.ClientboundSystemChatPacket` |
| 125 | [`minecraft:tab_list`](#pkt-clientbound-minecraft-tab_list) | `net.minecraft.network.protocol.game.ClientboundTabListPacket` |
| 126 | [`minecraft:tag_query`](#pkt-clientbound-minecraft-tag_query) | `net.minecraft.network.protocol.game.ClientboundTagQueryPacket` |
| 127 | [`minecraft:take_item_entity`](#pkt-clientbound-minecraft-take_item_entity) | `net.minecraft.network.protocol.game.ClientboundTakeItemEntityPacket` |
| 128 | [`minecraft:teleport_entity`](#pkt-clientbound-minecraft-teleport_entity) | `net.minecraft.network.protocol.game.ClientboundTeleportEntityPacket` |
| 129 | [`minecraft:test_instance_block_status`](#pkt-clientbound-minecraft-test_instance_block_status) | `net.minecraft.network.protocol.game.ClientboundTestInstanceBlockStatus` |
| 130 | [`minecraft:ticking_state`](#pkt-clientbound-minecraft-ticking_state) | `net.minecraft.network.protocol.game.ClientboundTickingStatePacket` |
| 131 | [`minecraft:ticking_step`](#pkt-clientbound-minecraft-ticking_step) | `net.minecraft.network.protocol.game.ClientboundTickingStepPacket` |
| 132 | [`minecraft:transfer`](#pkt-clientbound-minecraft-transfer) | `net.minecraft.network.protocol.common.ClientboundTransferPacket` |
| 133 | [`minecraft:update_advancements`](#pkt-clientbound-minecraft-update_advancements) | `net.minecraft.network.protocol.game.ClientboundUpdateAdvancementsPacket` |
| 134 | [`minecraft:update_attributes`](#pkt-clientbound-minecraft-update_attributes) | `net.minecraft.network.protocol.game.ClientboundUpdateAttributesPacket` |
| 135 | [`minecraft:update_mob_effect`](#pkt-clientbound-minecraft-update_mob_effect) | `net.minecraft.network.protocol.game.ClientboundUpdateMobEffectPacket` |
| 136 | [`minecraft:update_recipes`](#pkt-clientbound-minecraft-update_recipes) | `net.minecraft.network.protocol.game.ClientboundUpdateRecipesPacket` |
| 137 | [`minecraft:update_tags`](#pkt-clientbound-minecraft-update_tags) | `net.minecraft.network.protocol.common.ClientboundUpdateTagsPacket` |
| 138 | [`minecraft:projectile_power`](#pkt-clientbound-minecraft-projectile_power) | `net.minecraft.network.protocol.game.ClientboundProjectilePowerPacket` |
| 139 | [`minecraft:custom_report_details`](#pkt-clientbound-minecraft-custom_report_details) | `net.minecraft.network.protocol.common.ClientboundCustomReportDetailsPacket` |
| 140 | [`minecraft:server_links`](#pkt-clientbound-minecraft-server_links) | `net.minecraft.network.protocol.common.ClientboundServerLinksPacket` |
| 141 | [`minecraft:waypoint`](#pkt-clientbound-minecraft-waypoint) | `net.minecraft.network.protocol.game.ClientboundTrackedWaypointPacket` |
| 142 | [`minecraft:clear_dialog`](#pkt-clientbound-minecraft-clear_dialog) | `net.minecraft.network.protocol.common.ClientboundClearDialogPacket` |
| 143 | [`minecraft:show_dialog`](#pkt-clientbound-minecraft-show_dialog) | `net.minecraft.network.protocol.common.ClientboundShowDialogPacket` |

<a id="pkt-clientbound-minecraft-bundle_delimiter"></a>
### minecraft:bundle_delimiter (clientbound, id 0)

- nothing

<a id="pkt-clientbound-minecraft-add_entity"></a>
### minecraft:add_entity (clientbound, id 1)

- `id`: `VAR_INT`
- `uuid`: `UUID`
- `type`: id in entity_type
- `x`: `DOUBLE`
- `y`: `DOUBLE`
- `z`: `DOUBLE`
- `movement`: `LP_VEC3`
- `xRot`: `BYTE`
- `yRot`: `BYTE`
- `yHeadRot`: `BYTE`
- `data`: `VAR_INT`

<a id="pkt-clientbound-minecraft-animate"></a>
### minecraft:animate (clientbound, id 2)

- `id`: `VAR_INT`
- `action`: `UNSIGNED_BYTE`

<a id="pkt-clientbound-minecraft-award_stats"></a>
### minecraft:award_stats (clientbound, id 3)

- map
  - key: dispatch `Stat` on id in stat_type
    - `minecraft:mined`: id in block
    - `minecraft:crafted`: id in item
    - `minecraft:used`: id in item
    - `minecraft:broken`: id in item
    - `minecraft:picked_up`: id in item
    - `minecraft:dropped`: id in item
    - `minecraft:killed`: id in entity_type
    - `minecraft:killed_by`: id in entity_type
    - `minecraft:custom`: id in custom_stat
  - value: `VAR_INT`

<a id="pkt-clientbound-minecraft-block_changed_ack"></a>
### minecraft:block_changed_ack (clientbound, id 4)

- `sequence`: `VAR_INT`

<a id="pkt-clientbound-minecraft-block_destruction"></a>
### minecraft:block_destruction (clientbound, id 5)

- `id`: `VAR_INT`
- `pos`: `BLOCK_POS`
- `progress`: `UNSIGNED_BYTE`

<a id="pkt-clientbound-minecraft-block_entity_data"></a>
### minecraft:block_entity_data (clientbound, id 6)

- `getPos`: `BLOCK_POS`
- `getType`: id in block_entity_type
- `getTag`: `NBT`

<a id="pkt-clientbound-minecraft-block_event"></a>
### minecraft:block_event (clientbound, id 7)

- `pos`: `BLOCK_POS`
- `b0`: `UNSIGNED_BYTE`
- `b1`: `UNSIGNED_BYTE`
- `block`: id in block

<a id="pkt-clientbound-minecraft-block_update"></a>
### minecraft:block_update (clientbound, id 8)

- `getPos`: `BLOCK_POS`
- `getBlockState`: id in Block.BLOCK_STATE_REGISTRY

<a id="pkt-clientbound-minecraft-boss_event"></a>
### minecraft:boss_event (clientbound, id 9)

- `id`: `UUID`
- `operation`: dispatch `ClientboundBossEventPacketPayload` on enum `ClientboundBossEventPacket$OperationType` (var int, ordinal: ADD, REMOVE, UPDATE_PROGRESS, UPDATE_NAME, UPDATE_STYLE, UPDATE_PROPERTIES)
  - `ADD (0)`: struct `ClientboundBossEventPacket$AddOperation`
    - `name`: `TEXT`
    - `progress`: `FLOAT`
    - `color`: enum `BossEvent$BossBarColor` (var int, ordinal: PINK, BLUE, RED, GREEN, YELLOW, PURPLE, WHITE)
    - `overlay`: enum `BossEvent$BossBarOverlay` (var int, ordinal: PROGRESS, NOTCHED_6, NOTCHED_10, NOTCHED_12, NOTCHED_20)
    - `flags`: `UNSIGNED_BYTE`
  - `REMOVE (1)`: nothing
  - `UPDATE_PROGRESS (2)`: struct `ClientboundBossEventPacket$UpdateProgressOperation`
    - `progress`: `FLOAT`
  - `UPDATE_NAME (3)`: struct `ClientboundBossEventPacket$UpdateNameOperation`
    - `name`: `TEXT`
  - `UPDATE_STYLE (4)`: struct `ClientboundBossEventPacket$UpdateStyleOperation`
    - `color`: enum `BossEvent$BossBarColor` (var int, ordinal: PINK, BLUE, RED, GREEN, YELLOW, PURPLE, WHITE)
    - `overlay`: enum `BossEvent$BossBarOverlay` (var int, ordinal: PROGRESS, NOTCHED_6, NOTCHED_10, NOTCHED_12, NOTCHED_20)
  - `UPDATE_PROPERTIES (5)`: struct `ClientboundBossEventPacket$UpdatePropertiesOperation`
    - `input`: `UNSIGNED_BYTE`

<a id="pkt-clientbound-minecraft-change_difficulty"></a>
### minecraft:change_difficulty (clientbound, id 10)

- `difficulty`: enum `Difficulty` (var int, ordinal: PEACEFUL, EASY, NORMAL, HARD)
- `locked`: `BOOL`

<a id="pkt-clientbound-minecraft-chunk_batch_finished"></a>
### minecraft:chunk_batch_finished (clientbound, id 11)

- `batchSize`: `VAR_INT`

<a id="pkt-clientbound-minecraft-chunk_batch_start"></a>
### minecraft:chunk_batch_start (clientbound, id 12)

- nothing

<a id="pkt-clientbound-minecraft-chunks_biomes"></a>
### minecraft:chunks_biomes (clientbound, id 13)

- `chunkBiomeData`: list of
  - struct `ClientboundChunksBiomesPacket$ChunkBiomeData`
    - `pos`: `CHUNK_POS`
    - `buffer`: `BYTE_ARRAY` (max=2097152)

<a id="pkt-clientbound-minecraft-clear_titles"></a>
### minecraft:clear_titles (clientbound, id 14)

- `resetTimes`: `BOOL`

<a id="pkt-clientbound-minecraft-command_suggestions"></a>
### minecraft:command_suggestions (clientbound, id 15)

- `id`: `VAR_INT`
- `start`: `VAR_INT`
- `length`: `VAR_INT`
- `suggestions`: list of
  - struct `ClientboundCommandSuggestionsPacket$Entry`
    - `text`: `STRING`
    - `tooltip`: `OPTIONAL_TEXT`

<a id="pkt-clientbound-minecraft-commands"></a>
### minecraft:commands (clientbound, id 16)

- `entries`: list of
  - struct `ClientboundCommandsPacket$Entry`
    - `flags`: `BYTE`
    - `children`: `VAR_INT_ARRAY`
    - `redirect`: `VAR_INT` (when `flags` & 8 != 0)
    - `id`: string (when `flags` & 3 == 2)
    - `argumentType`: dispatch `ArgumentTypeInfo` on id in command_argument_type (when `flags` & 3 == 2)
      - `brigadier:bool`: nothing
      - `brigadier:float`: struct `FloatArgumentInfo$Template`
        - `flags`: `BYTE` as bits (numberHasMin@0:1, numberHasMax@1:1)
        - `min`: `FLOAT` (when `flags.numberHasMin` != 0)
        - `max`: `FLOAT` (when `flags.numberHasMax` != 0)
      - `brigadier:double`: struct `DoubleArgumentInfo$Template`
        - `flags`: `BYTE` as bits (numberHasMin@0:1, numberHasMax@1:1)
        - `min`: `DOUBLE` (when `flags.numberHasMin` != 0)
        - `max`: `DOUBLE` (when `flags.numberHasMax` != 0)
      - `brigadier:integer`: struct `IntegerArgumentInfo$Template`
        - `flags`: `BYTE` as bits (numberHasMin@0:1, numberHasMax@1:1)
        - `min`: `INT` (when `flags.numberHasMin` != 0)
        - `max`: `INT` (when `flags.numberHasMax` != 0)
      - `brigadier:long`: struct `LongArgumentInfo$Template`
        - `flags`: `BYTE` as bits (numberHasMin@0:1, numberHasMax@1:1)
        - `min`: `LONG` (when `flags.numberHasMin` != 0)
        - `max`: `LONG` (when `flags.numberHasMax` != 0)
      - `brigadier:string`: enum `StringArgumentType$StringType` (var int, ordinal: SINGLE_WORD, QUOTABLE_PHRASE, GREEDY_PHRASE)
      - `minecraft:entity`: `BYTE`
      - `minecraft:game_profile`: nothing
      - `minecraft:block_pos`: nothing
      - `minecraft:column_pos`: nothing
      - `minecraft:vec3`: nothing
      - `minecraft:vec2`: nothing
      - `minecraft:block_state`: nothing
      - `minecraft:block_predicate`: nothing
      - `minecraft:item_stack`: nothing
      - `minecraft:item_predicate`: nothing
      - `minecraft:team_color`: nothing
      - `minecraft:hex_color`: nothing
      - `minecraft:component`: nothing
      - `minecraft:style`: nothing
      - `minecraft:message`: nothing
      - `minecraft:nbt_compound_tag`: nothing
      - `minecraft:nbt_tag`: nothing
      - `minecraft:nbt_path`: nothing
      - `minecraft:objective`: nothing
      - `minecraft:objective_criteria`: nothing
      - `minecraft:operation`: nothing
      - `minecraft:particle`: nothing
      - `minecraft:angle`: nothing
      - `minecraft:rotation`: nothing
      - `minecraft:scoreboard_slot`: nothing
      - `minecraft:score_holder`: `BYTE`
      - `minecraft:swizzle`: nothing
      - `minecraft:team`: nothing
      - `minecraft:item_slot`: nothing
      - `minecraft:item_slots`: nothing
      - `minecraft:resource_location`: nothing
      - `minecraft:function`: nothing
      - `minecraft:entity_anchor`: nothing
      - `minecraft:int_range`: nothing
      - `minecraft:float_range`: nothing
      - `minecraft:dimension`: nothing
      - `minecraft:gamemode`: nothing
      - `minecraft:time`: `INT`
      - `minecraft:resource_or_tag`: `REGISTRY_KEY`
      - `minecraft:resource_or_tag_key`: `REGISTRY_KEY`
      - `minecraft:resource`: `REGISTRY_KEY`
      - `minecraft:resource_key`: `REGISTRY_KEY`
      - `minecraft:resource_selector`: `REGISTRY_KEY`
      - `minecraft:template_mirror`: nothing
      - `minecraft:template_rotation`: nothing
      - `minecraft:heightmap`: nothing
      - `minecraft:loot_table`: nothing
      - `minecraft:loot_predicate`: nothing
      - `minecraft:loot_modifier`: nothing
      - `minecraft:context_float_provider`: nothing
      - `minecraft:context_int_provider`: nothing
      - `minecraft:slot_source`: nothing
      - `minecraft:dialog`: nothing
      - `minecraft:feature`: nothing
      - `minecraft:swing_animation`: nothing
      - `minecraft:uuid`: nothing
    - `suggestionId`: `IDENTIFIER` (when `flags` & 3 == 2, and `flags` & 16 != 0)
    - `clientboundCommandsPacketLiteralNodeStub`: struct `ClientboundCommandsPacket$LiteralNodeStub` (when `flags` & 3 == 1)
      - `id`: string
- `rootIndex`: `VAR_INT`

<a id="pkt-clientbound-minecraft-container_close"></a>
### minecraft:container_close (clientbound, id 17)

- `containerId`: `CONTAINER_ID`

<a id="pkt-clientbound-minecraft-container_set_content"></a>
### minecraft:container_set_content (clientbound, id 18)

- `containerId`: `CONTAINER_ID`
- `stateId`: `VAR_INT`
- `items`: `OPTIONAL_ITEM_STACK_LIST`
- `carriedItem`: `OPTIONAL_ITEM_STACK`

<a id="pkt-clientbound-minecraft-container_set_data"></a>
### minecraft:container_set_data (clientbound, id 19)

- `containerId`: `CONTAINER_ID`
- `id`: `SHORT`
- `value`: `SHORT`

<a id="pkt-clientbound-minecraft-container_set_slot"></a>
### minecraft:container_set_slot (clientbound, id 20)

- `containerId`: `CONTAINER_ID`
- `stateId`: `VAR_INT`
- `slot`: `SHORT`
- `itemStack`: `OPTIONAL_ITEM_STACK`

<a id="pkt-clientbound-minecraft-cookie_request"></a>
### minecraft:cookie_request (clientbound, id 21)

- `key`: `IDENTIFIER`

<a id="pkt-clientbound-minecraft-cooldown"></a>
### minecraft:cooldown (clientbound, id 22)

- `cooldownGroup`: `IDENTIFIER`
- `duration`: `VAR_INT`

<a id="pkt-clientbound-minecraft-custom_chat_completions"></a>
### minecraft:custom_chat_completions (clientbound, id 23)

- `action`: enum `ClientboundCustomChatCompletionsPacket$Action` (var int, ordinal: ADD, REMOVE, SET)
- `entries`: list of `STRING`

<a id="pkt-clientbound-minecraft-custom_payload"></a>
### minecraft:custom_payload (clientbound, id 24)

- `channel`: `IDENTIFIER`
- `data`: `REST_BYTES`

<a id="pkt-clientbound-minecraft-damage_event"></a>
### minecraft:damage_event (clientbound, id 25)

- `entityId`: `VAR_INT`
- `sourceType`: id in damage_type
- `sourceCauseId`: `VAR_INT`
- `sourceDirectId`: `VAR_INT`
- `sourcePosition`: optional
  - struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`

<a id="pkt-clientbound-minecraft-debug-block_value"></a>
### minecraft:debug/block_value (clientbound, id 26)

- `blockPos`: `BLOCK_POS`
- `update`: dispatch `DebugSubscription$Update` on id in debug_subscription
  - `minecraft:dedicated_server_tick_time`: optional nothing
  - `minecraft:bees`: optional
    - struct `DebugBeeInfo`
      - `hivePos`: optional `BLOCK_POS`
      - `flowerPos`: optional `BLOCK_POS`
      - `travelTicks`: `VAR_INT`
      - `blacklistedHives`: list of `BLOCK_POS`
  - `minecraft:brains`: optional
    - struct `DebugBrainDump`
      - `name`: `STRING`
      - `profession`: `STRING`
      - `xp`: `INT`
      - `health`: `FLOAT`
      - `maxHealth`: `FLOAT`
      - `inventory`: `STRING`
      - `wantsGolem`: `BOOL`
      - `angerLevel`: `INT`
      - `activities`: list of `STRING`
      - `behaviors`: list of `STRING`
      - `memories`: list of `STRING`
      - `gossips`: list of `STRING`
      - `pois`: list of `BLOCK_POS`
      - `potentialPois`: list of `BLOCK_POS`
  - `minecraft:breezes`: optional
    - struct `DebugBreezeInfo`
      - `attackTarget`: optional `VAR_INT`
      - `jumpTarget`: optional `BLOCK_POS`
  - `minecraft:goal_selectors`: optional
    - struct `DebugGoalInfo`
      - `goals`: list of
        - struct `DebugGoalInfo$DebugGoal`
          - `priority`: `VAR_INT`
          - `isRunning`: `BOOL`
          - `name`: string (at most 255 characters)
  - `minecraft:entity_paths`: optional
    - struct `DebugPathInfo`
      - `path`: struct `Path`
        - `reached`: `BOOL`
        - `nextNodeIndex`: `INT`
        - `target`: `BLOCK_POS`
        - `nodes`: list of
          - struct `Node`
            - `x`: `INT`
            - `y`: `INT`
            - `z`: `INT`
            - `walkedDistance`: `FLOAT`
            - `costMalus`: `FLOAT`
            - `closed`: `BOOL`
            - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
            - `f`: `FLOAT`
        - `debugData`: struct `Path$DebugData`
          - `openSet`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
          - `closedSet`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
          - `targetNodes`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
      - `maxNodeDistance`: `FLOAT`
  - `minecraft:entity_block_intersections`: optional enum `DebugEntityBlockIntersection` (var int, ordinal: IN_BLOCK, IN_FLUID, IN_AIR)
  - `minecraft:bee_hives`: optional
    - struct `DebugHiveInfo`
      - `type`: id in block
      - `occupantCount`: `VAR_INT`
      - `honeyLevel`: `VAR_INT`
      - `sedated`: `BOOL`
  - `minecraft:pois`: optional
    - struct `DebugPoiInfo`
      - `pos`: `BLOCK_POS`
      - `poiType`: id in point_of_interest_type
      - `freeTicketCount`: `VAR_INT`
  - `minecraft:redstone_wire_orientations`: optional id in Orientation
  - `minecraft:village_sections`: optional nothing
  - `minecraft:raids`: optional list of `BLOCK_POS`
  - `minecraft:structures`: optional
    - list of
      - struct `DebugStructureInfo`
        - `boundingBox`: struct `BoundingBox`
          - `minX`: `BLOCK_POS`
          - `maxX`: `BLOCK_POS`
        - `pieces`: list of
          - struct `DebugStructureInfo$Piece`
            - `boundingBox`: struct `BoundingBox`
              - `minX`: `BLOCK_POS`
              - `maxX`: `BLOCK_POS`
            - `isStart`: `BOOL`
  - `minecraft:game_event_listeners`: optional
    - struct `DebugGameEventListenerInfo`
      - `listenerRadius`: `VAR_INT`
  - `minecraft:neighbor_updates`: optional `BLOCK_POS`
  - `minecraft:game_events`: optional
    - struct `DebugGameEventInfo`
      - `event`: id in game_event
      - `pos`: struct `Vec3`
        - `x`: `DOUBLE`
        - `y`: `DOUBLE`
        - `z`: `DOUBLE`

<a id="pkt-clientbound-minecraft-debug-chunk_value"></a>
### minecraft:debug/chunk_value (clientbound, id 27)

- `chunkPos`: `CHUNK_POS`
- `update`: dispatch `DebugSubscription$Update` on id in debug_subscription
  - `minecraft:dedicated_server_tick_time`: optional nothing
  - `minecraft:bees`: optional
    - struct `DebugBeeInfo`
      - `hivePos`: optional `BLOCK_POS`
      - `flowerPos`: optional `BLOCK_POS`
      - `travelTicks`: `VAR_INT`
      - `blacklistedHives`: list of `BLOCK_POS`
  - `minecraft:brains`: optional
    - struct `DebugBrainDump`
      - `name`: `STRING`
      - `profession`: `STRING`
      - `xp`: `INT`
      - `health`: `FLOAT`
      - `maxHealth`: `FLOAT`
      - `inventory`: `STRING`
      - `wantsGolem`: `BOOL`
      - `angerLevel`: `INT`
      - `activities`: list of `STRING`
      - `behaviors`: list of `STRING`
      - `memories`: list of `STRING`
      - `gossips`: list of `STRING`
      - `pois`: list of `BLOCK_POS`
      - `potentialPois`: list of `BLOCK_POS`
  - `minecraft:breezes`: optional
    - struct `DebugBreezeInfo`
      - `attackTarget`: optional `VAR_INT`
      - `jumpTarget`: optional `BLOCK_POS`
  - `minecraft:goal_selectors`: optional
    - struct `DebugGoalInfo`
      - `goals`: list of
        - struct `DebugGoalInfo$DebugGoal`
          - `priority`: `VAR_INT`
          - `isRunning`: `BOOL`
          - `name`: string (at most 255 characters)
  - `minecraft:entity_paths`: optional
    - struct `DebugPathInfo`
      - `path`: struct `Path`
        - `reached`: `BOOL`
        - `nextNodeIndex`: `INT`
        - `target`: `BLOCK_POS`
        - `nodes`: list of
          - struct `Node`
            - `x`: `INT`
            - `y`: `INT`
            - `z`: `INT`
            - `walkedDistance`: `FLOAT`
            - `costMalus`: `FLOAT`
            - `closed`: `BOOL`
            - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
            - `f`: `FLOAT`
        - `debugData`: struct `Path$DebugData`
          - `openSet`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
          - `closedSet`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
          - `targetNodes`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
      - `maxNodeDistance`: `FLOAT`
  - `minecraft:entity_block_intersections`: optional enum `DebugEntityBlockIntersection` (var int, ordinal: IN_BLOCK, IN_FLUID, IN_AIR)
  - `minecraft:bee_hives`: optional
    - struct `DebugHiveInfo`
      - `type`: id in block
      - `occupantCount`: `VAR_INT`
      - `honeyLevel`: `VAR_INT`
      - `sedated`: `BOOL`
  - `minecraft:pois`: optional
    - struct `DebugPoiInfo`
      - `pos`: `BLOCK_POS`
      - `poiType`: id in point_of_interest_type
      - `freeTicketCount`: `VAR_INT`
  - `minecraft:redstone_wire_orientations`: optional id in Orientation
  - `minecraft:village_sections`: optional nothing
  - `minecraft:raids`: optional list of `BLOCK_POS`
  - `minecraft:structures`: optional
    - list of
      - struct `DebugStructureInfo`
        - `boundingBox`: struct `BoundingBox`
          - `minX`: `BLOCK_POS`
          - `maxX`: `BLOCK_POS`
        - `pieces`: list of
          - struct `DebugStructureInfo$Piece`
            - `boundingBox`: struct `BoundingBox`
              - `minX`: `BLOCK_POS`
              - `maxX`: `BLOCK_POS`
            - `isStart`: `BOOL`
  - `minecraft:game_event_listeners`: optional
    - struct `DebugGameEventListenerInfo`
      - `listenerRadius`: `VAR_INT`
  - `minecraft:neighbor_updates`: optional `BLOCK_POS`
  - `minecraft:game_events`: optional
    - struct `DebugGameEventInfo`
      - `event`: id in game_event
      - `pos`: struct `Vec3`
        - `x`: `DOUBLE`
        - `y`: `DOUBLE`
        - `z`: `DOUBLE`

<a id="pkt-clientbound-minecraft-debug-entity_value"></a>
### minecraft:debug/entity_value (clientbound, id 28)

- `entityId`: `VAR_INT`
- `update`: dispatch `DebugSubscription$Update` on id in debug_subscription
  - `minecraft:dedicated_server_tick_time`: optional nothing
  - `minecraft:bees`: optional
    - struct `DebugBeeInfo`
      - `hivePos`: optional `BLOCK_POS`
      - `flowerPos`: optional `BLOCK_POS`
      - `travelTicks`: `VAR_INT`
      - `blacklistedHives`: list of `BLOCK_POS`
  - `minecraft:brains`: optional
    - struct `DebugBrainDump`
      - `name`: `STRING`
      - `profession`: `STRING`
      - `xp`: `INT`
      - `health`: `FLOAT`
      - `maxHealth`: `FLOAT`
      - `inventory`: `STRING`
      - `wantsGolem`: `BOOL`
      - `angerLevel`: `INT`
      - `activities`: list of `STRING`
      - `behaviors`: list of `STRING`
      - `memories`: list of `STRING`
      - `gossips`: list of `STRING`
      - `pois`: list of `BLOCK_POS`
      - `potentialPois`: list of `BLOCK_POS`
  - `minecraft:breezes`: optional
    - struct `DebugBreezeInfo`
      - `attackTarget`: optional `VAR_INT`
      - `jumpTarget`: optional `BLOCK_POS`
  - `minecraft:goal_selectors`: optional
    - struct `DebugGoalInfo`
      - `goals`: list of
        - struct `DebugGoalInfo$DebugGoal`
          - `priority`: `VAR_INT`
          - `isRunning`: `BOOL`
          - `name`: string (at most 255 characters)
  - `minecraft:entity_paths`: optional
    - struct `DebugPathInfo`
      - `path`: struct `Path`
        - `reached`: `BOOL`
        - `nextNodeIndex`: `INT`
        - `target`: `BLOCK_POS`
        - `nodes`: list of
          - struct `Node`
            - `x`: `INT`
            - `y`: `INT`
            - `z`: `INT`
            - `walkedDistance`: `FLOAT`
            - `costMalus`: `FLOAT`
            - `closed`: `BOOL`
            - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
            - `f`: `FLOAT`
        - `debugData`: struct `Path$DebugData`
          - `openSet`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
          - `closedSet`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
          - `targetNodes`: list of
            - struct `Node`
              - `x`: `INT`
              - `y`: `INT`
              - `z`: `INT`
              - `walkedDistance`: `FLOAT`
              - `costMalus`: `FLOAT`
              - `closed`: `BOOL`
              - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
              - `f`: `FLOAT`
      - `maxNodeDistance`: `FLOAT`
  - `minecraft:entity_block_intersections`: optional enum `DebugEntityBlockIntersection` (var int, ordinal: IN_BLOCK, IN_FLUID, IN_AIR)
  - `minecraft:bee_hives`: optional
    - struct `DebugHiveInfo`
      - `type`: id in block
      - `occupantCount`: `VAR_INT`
      - `honeyLevel`: `VAR_INT`
      - `sedated`: `BOOL`
  - `minecraft:pois`: optional
    - struct `DebugPoiInfo`
      - `pos`: `BLOCK_POS`
      - `poiType`: id in point_of_interest_type
      - `freeTicketCount`: `VAR_INT`
  - `minecraft:redstone_wire_orientations`: optional id in Orientation
  - `minecraft:village_sections`: optional nothing
  - `minecraft:raids`: optional list of `BLOCK_POS`
  - `minecraft:structures`: optional
    - list of
      - struct `DebugStructureInfo`
        - `boundingBox`: struct `BoundingBox`
          - `minX`: `BLOCK_POS`
          - `maxX`: `BLOCK_POS`
        - `pieces`: list of
          - struct `DebugStructureInfo$Piece`
            - `boundingBox`: struct `BoundingBox`
              - `minX`: `BLOCK_POS`
              - `maxX`: `BLOCK_POS`
            - `isStart`: `BOOL`
  - `minecraft:game_event_listeners`: optional
    - struct `DebugGameEventListenerInfo`
      - `listenerRadius`: `VAR_INT`
  - `minecraft:neighbor_updates`: optional `BLOCK_POS`
  - `minecraft:game_events`: optional
    - struct `DebugGameEventInfo`
      - `event`: id in game_event
      - `pos`: struct `Vec3`
        - `x`: `DOUBLE`
        - `y`: `DOUBLE`
        - `z`: `DOUBLE`

<a id="pkt-clientbound-minecraft-debug-event"></a>
### minecraft:debug/event (clientbound, id 29)

- `event`: dispatch `DebugSubscription$Event` on id in debug_subscription
  - `minecraft:dedicated_server_tick_time`: nothing
  - `minecraft:bees`: struct `DebugBeeInfo`
    - `hivePos`: optional `BLOCK_POS`
    - `flowerPos`: optional `BLOCK_POS`
    - `travelTicks`: `VAR_INT`
    - `blacklistedHives`: list of `BLOCK_POS`
  - `minecraft:brains`: struct `DebugBrainDump`
    - `name`: `STRING`
    - `profession`: `STRING`
    - `xp`: `INT`
    - `health`: `FLOAT`
    - `maxHealth`: `FLOAT`
    - `inventory`: `STRING`
    - `wantsGolem`: `BOOL`
    - `angerLevel`: `INT`
    - `activities`: list of `STRING`
    - `behaviors`: list of `STRING`
    - `memories`: list of `STRING`
    - `gossips`: list of `STRING`
    - `pois`: list of `BLOCK_POS`
    - `potentialPois`: list of `BLOCK_POS`
  - `minecraft:breezes`: struct `DebugBreezeInfo`
    - `attackTarget`: optional `VAR_INT`
    - `jumpTarget`: optional `BLOCK_POS`
  - `minecraft:goal_selectors`: struct `DebugGoalInfo`
    - `goals`: list of
      - struct `DebugGoalInfo$DebugGoal`
        - `priority`: `VAR_INT`
        - `isRunning`: `BOOL`
        - `name`: string (at most 255 characters)
  - `minecraft:entity_paths`: struct `DebugPathInfo`
    - `path`: struct `Path`
      - `reached`: `BOOL`
      - `nextNodeIndex`: `INT`
      - `target`: `BLOCK_POS`
      - `nodes`: list of
        - struct `Node`
          - `x`: `INT`
          - `y`: `INT`
          - `z`: `INT`
          - `walkedDistance`: `FLOAT`
          - `costMalus`: `FLOAT`
          - `closed`: `BOOL`
          - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
          - `f`: `FLOAT`
      - `debugData`: struct `Path$DebugData`
        - `openSet`: list of
          - struct `Node`
            - `x`: `INT`
            - `y`: `INT`
            - `z`: `INT`
            - `walkedDistance`: `FLOAT`
            - `costMalus`: `FLOAT`
            - `closed`: `BOOL`
            - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
            - `f`: `FLOAT`
        - `closedSet`: list of
          - struct `Node`
            - `x`: `INT`
            - `y`: `INT`
            - `z`: `INT`
            - `walkedDistance`: `FLOAT`
            - `costMalus`: `FLOAT`
            - `closed`: `BOOL`
            - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
            - `f`: `FLOAT`
        - `targetNodes`: list of
          - struct `Node`
            - `x`: `INT`
            - `y`: `INT`
            - `z`: `INT`
            - `walkedDistance`: `FLOAT`
            - `costMalus`: `FLOAT`
            - `closed`: `BOOL`
            - `type`: enum `PathType` (var int, ordinal: BLOCKED, OPEN, WALKABLE, WALKABLE_DOOR, TRAPDOOR, POWDER_SNOW, ON_TOP_OF_POWDER_SNOW, FENCE, LAVA, WATER, WATER_BORDER, RAIL, UNPASSABLE_RAIL, FIRE_IN_NEIGHBOR, FIRE, DAMAGING_IN_NEIGHBOR, DAMAGING, DOOR_OPEN, DOOR_WOOD_CLOSED, DOOR_IRON_CLOSED, BREACH, LEAVES, STICKY_HONEY, COCOA, DAMAGE_CAUTIOUS, ON_TOP_OF_TRAPDOOR, BIG_MOBS_CLOSE_TO_DANGER)
            - `f`: `FLOAT`
    - `maxNodeDistance`: `FLOAT`
  - `minecraft:entity_block_intersections`: enum `DebugEntityBlockIntersection` (var int, ordinal: IN_BLOCK, IN_FLUID, IN_AIR)
  - `minecraft:bee_hives`: struct `DebugHiveInfo`
    - `type`: id in block
    - `occupantCount`: `VAR_INT`
    - `honeyLevel`: `VAR_INT`
    - `sedated`: `BOOL`
  - `minecraft:pois`: struct `DebugPoiInfo`
    - `pos`: `BLOCK_POS`
    - `poiType`: id in point_of_interest_type
    - `freeTicketCount`: `VAR_INT`
  - `minecraft:redstone_wire_orientations`: id in Orientation
  - `minecraft:village_sections`: nothing
  - `minecraft:raids`: list of `BLOCK_POS`
  - `minecraft:structures`: list of
    - struct `DebugStructureInfo`
      - `boundingBox`: struct `BoundingBox`
        - `minX`: `BLOCK_POS`
        - `maxX`: `BLOCK_POS`
      - `pieces`: list of
        - struct `DebugStructureInfo$Piece`
          - `boundingBox`: struct `BoundingBox`
            - `minX`: `BLOCK_POS`
            - `maxX`: `BLOCK_POS`
          - `isStart`: `BOOL`
  - `minecraft:game_event_listeners`: struct `DebugGameEventListenerInfo`
    - `listenerRadius`: `VAR_INT`
  - `minecraft:neighbor_updates`: `BLOCK_POS`
  - `minecraft:game_events`: struct `DebugGameEventInfo`
    - `event`: id in game_event
    - `pos`: struct `Vec3`
      - `x`: `DOUBLE`
      - `y`: `DOUBLE`
      - `z`: `DOUBLE`

<a id="pkt-clientbound-minecraft-debug_sample"></a>
### minecraft:debug_sample (clientbound, id 30)

- `sample`: `LONG_ARRAY`
- `debugSampleType`: enum `RemoteDebugSampleType` (var int, ordinal: TICK_TIME)

<a id="pkt-clientbound-minecraft-delete_chat"></a>
### minecraft:delete_chat (clientbound, id 31)

- `messageSignature`: struct `MessageSignature$Packed`
  - `id`: `VAR_INT`
  - `fullSignature`: `FIXED_BYTES` (len=256) (when `id` == 0)

<a id="pkt-clientbound-minecraft-disconnect"></a>
### minecraft:disconnect (clientbound, id 32)

- `TEXT`

<a id="pkt-clientbound-minecraft-disguised_chat"></a>
### minecraft:disguised_chat (clientbound, id 33)

- `message`: `TEXT`
- `chatType`: struct `ChatType$Bound`
  - `chatType`: id in chat_type, or 0 and the element inline
    - inline: struct `ChatType`
      - `chat`: struct `ChatTypeDecoration`
        - `translationKey`: `STRING`
        - `parameters`: list of enum `ChatTypeDecoration$Parameter` (var int, ordinal: SENDER, TARGET, CONTENT)
        - `style`: an NBT tag
      - `narration`: struct `ChatTypeDecoration`
        - `translationKey`: `STRING`
        - `parameters`: list of enum `ChatTypeDecoration$Parameter` (var int, ordinal: SENDER, TARGET, CONTENT)
        - `style`: an NBT tag
  - `name`: `TEXT`
  - `targetName`: `OPTIONAL_TEXT`

<a id="pkt-clientbound-minecraft-entity_event"></a>
### minecraft:entity_event (clientbound, id 34)

- `entityId`: `INT`
- `eventId`: `BYTE`

<a id="pkt-clientbound-minecraft-entity_position_sync"></a>
### minecraft:entity_position_sync (clientbound, id 35)

- `id`: `VAR_INT`
- `position`: dispatch `PositionPath` on enum `PositionPath$Type` (var int, ordinal: LINEAR, STEPPED)
  - `LINEAR (0)`: struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
  - `STEPPED (1)`: list of
    - struct `PositionStep`
      - `position`: struct `Vec3`
        - `x`: `DOUBLE`
        - `y`: `DOUBLE`
        - `z`: `DOUBLE`
      - `tickOffset`: `VAR_INT`
- `yRot`: `FLOAT`
- `xRot`: `FLOAT`
- `onGround`: `BOOL`

<a id="pkt-clientbound-minecraft-explode"></a>
### minecraft:explode (clientbound, id 36)

- `center`: struct `Vec3`
  - `x`: `DOUBLE`
  - `y`: `DOUBLE`
  - `z`: `DOUBLE`
- `radius`: `FLOAT`
- `blockCount`: `INT`
- `playerKnockback`: optional
  - struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
- `explosionParticle`: dispatch `ParticleOptions` on id in particle_type
  - `minecraft:angry_villager`: nothing
  - `minecraft:block`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:block_marker`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:bubble`: nothing
  - `minecraft:sulfur_bubbles`: nothing
  - `minecraft:noxious_gas`: nothing
  - `minecraft:noxious_gas_cloud`: nothing
  - `minecraft:geyser`: struct `GeyserParticleOptions`
    - `waterBlocks`: `INT`
  - `minecraft:geyser_base`: struct `GeyserBaseParticleOptions`
    - `waterBlocks`: `INT`
    - `burstImpulseBase`: `FLOAT`
  - `minecraft:geyser_poof`: struct `GeyserBaseParticleOptions`
    - `waterBlocks`: `INT`
    - `burstImpulseBase`: `FLOAT`
  - `minecraft:geyser_plume`: struct `GeyserParticleOptions`
    - `waterBlocks`: `INT`
  - `minecraft:cloud`: nothing
  - `minecraft:copper_fire_flame`: nothing
  - `minecraft:crit`: nothing
  - `minecraft:damage_indicator`: nothing
  - `minecraft:dragon_breath`: `FLOAT`
  - `minecraft:dripping_lava`: nothing
  - `minecraft:falling_lava`: nothing
  - `minecraft:landing_lava`: nothing
  - `minecraft:dripping_water`: nothing
  - `minecraft:falling_water`: nothing
  - `minecraft:dust`: struct `DustParticleOptions`
    - `color`: `INT`
    - `getScale`: `FLOAT`
  - `minecraft:dust_color_transition`: struct `DustColorTransitionOptions`
    - `fromColor`: `INT`
    - `toColor`: `INT`
    - `getScale`: `FLOAT`
  - `minecraft:effect`: struct `SpellParticleOption`
    - `color`: `INT`
    - `power`: `FLOAT`
  - `minecraft:elder_guardian`: nothing
  - `minecraft:enchanted_hit`: nothing
  - `minecraft:enchant`: nothing
  - `minecraft:end_rod`: nothing
  - `minecraft:entity_effect`: `INT`
  - `minecraft:explosion_emitter`: nothing
  - `minecraft:explosion`: nothing
  - `minecraft:gust`: nothing
  - `minecraft:small_gust`: nothing
  - `minecraft:gust_emitter_large`: nothing
  - `minecraft:gust_emitter_small`: nothing
  - `minecraft:sonic_boom`: nothing
  - `minecraft:falling_dust`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:firework`: nothing
  - `minecraft:fishing`: nothing
  - `minecraft:flame`: nothing
  - `minecraft:infested`: nothing
  - `minecraft:cherry_leaves`: nothing
  - `minecraft:pale_oak_leaves`: nothing
  - `minecraft:red_poplar_leaves`: nothing
  - `minecraft:orange_poplar_leaves`: nothing
  - `minecraft:yellow_poplar_leaves`: nothing
  - `minecraft:tinted_leaves`: `INT`
  - `minecraft:sculk_soul`: nothing
  - `minecraft:sculk_charge`: struct `SculkChargeParticleOptions`
    - `roll`: `FLOAT`
  - `minecraft:sculk_charge_pop`: nothing
  - `minecraft:soul_fire_flame`: nothing
  - `minecraft:soul`: nothing
  - `minecraft:flash`: `INT`
  - `minecraft:happy_villager`: nothing
  - `minecraft:composter`: nothing
  - `minecraft:heart`: nothing
  - `minecraft:instant_effect`: struct `SpellParticleOption`
    - `color`: `INT`
    - `power`: `FLOAT`
  - `minecraft:item`: struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`
  - `minecraft:vibration`: struct `VibrationParticleOption`
    - `getDestination`: dispatch `PositionSource` on id in position_source_type
      - `minecraft:block`: struct `BlockPositionSource`
        - `pos`: `BLOCK_POS`
      - `minecraft:entity`: struct `EntityPositionSource`
        - `getId`: `VAR_INT`
        - `yOffset`: `FLOAT`
    - `getArrivalInTicks`: `VAR_INT`
  - `minecraft:trail`: struct `TrailParticleOption`
    - `target`: struct `Vec3`
      - `x`: `DOUBLE`
      - `y`: `DOUBLE`
      - `z`: `DOUBLE`
    - `color`: `INT`
    - `duration`: `VAR_INT`
  - `minecraft:pause_mob_growth`: nothing
  - `minecraft:reset_mob_growth`: nothing
  - `minecraft:item_slime`: nothing
  - `minecraft:item_cobweb`: nothing
  - `minecraft:item_snowball`: nothing
  - `minecraft:large_smoke`: nothing
  - `minecraft:lava`: nothing
  - `minecraft:mycelium`: nothing
  - `minecraft:note`: nothing
  - `minecraft:poof`: nothing
  - `minecraft:portal`: nothing
  - `minecraft:rain`: nothing
  - `minecraft:smoke`: nothing
  - `minecraft:white_smoke`: nothing
  - `minecraft:sneeze`: nothing
  - `minecraft:spit`: nothing
  - `minecraft:squid_ink`: nothing
  - `minecraft:sweep_attack`: nothing
  - `minecraft:totem_of_undying`: nothing
  - `minecraft:underwater`: nothing
  - `minecraft:splash`: nothing
  - `minecraft:witch`: nothing
  - `minecraft:bubble_pop`: nothing
  - `minecraft:current_down`: nothing
  - `minecraft:bubble_column_up`: nothing
  - `minecraft:nautilus`: nothing
  - `minecraft:dolphin`: nothing
  - `minecraft:campfire_cosy_smoke`: nothing
  - `minecraft:campfire_signal_smoke`: nothing
  - `minecraft:dripping_honey`: nothing
  - `minecraft:falling_honey`: nothing
  - `minecraft:landing_honey`: nothing
  - `minecraft:falling_nectar`: nothing
  - `minecraft:falling_spore_blossom`: nothing
  - `minecraft:ash`: nothing
  - `minecraft:crimson_spore`: nothing
  - `minecraft:warped_spore`: nothing
  - `minecraft:spore_blossom_air`: nothing
  - `minecraft:dripping_obsidian_tear`: nothing
  - `minecraft:falling_obsidian_tear`: nothing
  - `minecraft:landing_obsidian_tear`: nothing
  - `minecraft:reverse_portal`: nothing
  - `minecraft:white_ash`: nothing
  - `minecraft:small_flame`: nothing
  - `minecraft:snowflake`: nothing
  - `minecraft:dripping_dripstone_lava`: nothing
  - `minecraft:falling_dripstone_lava`: nothing
  - `minecraft:dripping_dripstone_water`: nothing
  - `minecraft:falling_dripstone_water`: nothing
  - `minecraft:glow_squid_ink`: nothing
  - `minecraft:glow`: nothing
  - `minecraft:wax_on`: nothing
  - `minecraft:wax_off`: nothing
  - `minecraft:electric_spark`: nothing
  - `minecraft:scrape`: nothing
  - `minecraft:shriek`: struct `ShriekParticleOption`
    - `delay`: `VAR_INT`
  - `minecraft:egg_crack`: nothing
  - `minecraft:dust_plume`: nothing
  - `minecraft:trial_spawner_detection`: nothing
  - `minecraft:trial_spawner_detection_ominous`: nothing
  - `minecraft:vault_connection`: nothing
  - `minecraft:dust_pillar`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:ominous_spawning`: nothing
  - `minecraft:raid_omen`: nothing
  - `minecraft:trial_omen`: nothing
  - `minecraft:block_crumble`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:firefly`: nothing
  - `minecraft:sulfur_cube_goo`: nothing
- `explosionSound`: id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`
- `blockParticles`: list of
  - struct `Weighted`
    - `value`: struct `ExplosionParticleInfo`
      - `particle`: dispatch `ParticleOptions` on id in particle_type
        - `minecraft:angry_villager`: nothing
        - `minecraft:block`: id in Block.BLOCK_STATE_REGISTRY
        - `minecraft:block_marker`: id in Block.BLOCK_STATE_REGISTRY
        - `minecraft:bubble`: nothing
        - `minecraft:sulfur_bubbles`: nothing
        - `minecraft:noxious_gas`: nothing
        - `minecraft:noxious_gas_cloud`: nothing
        - `minecraft:geyser`: struct `GeyserParticleOptions`
          - `waterBlocks`: `INT`
        - `minecraft:geyser_base`: struct `GeyserBaseParticleOptions`
          - `waterBlocks`: `INT`
          - `burstImpulseBase`: `FLOAT`
        - `minecraft:geyser_poof`: struct `GeyserBaseParticleOptions`
          - `waterBlocks`: `INT`
          - `burstImpulseBase`: `FLOAT`
        - `minecraft:geyser_plume`: struct `GeyserParticleOptions`
          - `waterBlocks`: `INT`
        - `minecraft:cloud`: nothing
        - `minecraft:copper_fire_flame`: nothing
        - `minecraft:crit`: nothing
        - `minecraft:damage_indicator`: nothing
        - `minecraft:dragon_breath`: `FLOAT`
        - `minecraft:dripping_lava`: nothing
        - `minecraft:falling_lava`: nothing
        - `minecraft:landing_lava`: nothing
        - `minecraft:dripping_water`: nothing
        - `minecraft:falling_water`: nothing
        - `minecraft:dust`: struct `DustParticleOptions`
          - `color`: `INT`
          - `getScale`: `FLOAT`
        - `minecraft:dust_color_transition`: struct `DustColorTransitionOptions`
          - `fromColor`: `INT`
          - `toColor`: `INT`
          - `getScale`: `FLOAT`
        - `minecraft:effect`: struct `SpellParticleOption`
          - `color`: `INT`
          - `power`: `FLOAT`
        - `minecraft:elder_guardian`: nothing
        - `minecraft:enchanted_hit`: nothing
        - `minecraft:enchant`: nothing
        - `minecraft:end_rod`: nothing
        - `minecraft:entity_effect`: `INT`
        - `minecraft:explosion_emitter`: nothing
        - `minecraft:explosion`: nothing
        - `minecraft:gust`: nothing
        - `minecraft:small_gust`: nothing
        - `minecraft:gust_emitter_large`: nothing
        - `minecraft:gust_emitter_small`: nothing
        - `minecraft:sonic_boom`: nothing
        - `minecraft:falling_dust`: id in Block.BLOCK_STATE_REGISTRY
        - `minecraft:firework`: nothing
        - `minecraft:fishing`: nothing
        - `minecraft:flame`: nothing
        - `minecraft:infested`: nothing
        - `minecraft:cherry_leaves`: nothing
        - `minecraft:pale_oak_leaves`: nothing
        - `minecraft:red_poplar_leaves`: nothing
        - `minecraft:orange_poplar_leaves`: nothing
        - `minecraft:yellow_poplar_leaves`: nothing
        - `minecraft:tinted_leaves`: `INT`
        - `minecraft:sculk_soul`: nothing
        - `minecraft:sculk_charge`: struct `SculkChargeParticleOptions`
          - `roll`: `FLOAT`
        - `minecraft:sculk_charge_pop`: nothing
        - `minecraft:soul_fire_flame`: nothing
        - `minecraft:soul`: nothing
        - `minecraft:flash`: `INT`
        - `minecraft:happy_villager`: nothing
        - `minecraft:composter`: nothing
        - `minecraft:heart`: nothing
        - `minecraft:instant_effect`: struct `SpellParticleOption`
          - `color`: `INT`
          - `power`: `FLOAT`
        - `minecraft:item`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
        - `minecraft:vibration`: struct `VibrationParticleOption`
          - `getDestination`: dispatch `PositionSource` on id in position_source_type
            - `minecraft:block`: struct `BlockPositionSource`
              - `pos`: `BLOCK_POS`
            - `minecraft:entity`: struct `EntityPositionSource`
              - `getId`: `VAR_INT`
              - `yOffset`: `FLOAT`
          - `getArrivalInTicks`: `VAR_INT`
        - `minecraft:trail`: struct `TrailParticleOption`
          - `target`: struct `Vec3`
            - `x`: `DOUBLE`
            - `y`: `DOUBLE`
            - `z`: `DOUBLE`
          - `color`: `INT`
          - `duration`: `VAR_INT`
        - `minecraft:pause_mob_growth`: nothing
        - `minecraft:reset_mob_growth`: nothing
        - `minecraft:item_slime`: nothing
        - `minecraft:item_cobweb`: nothing
        - `minecraft:item_snowball`: nothing
        - `minecraft:large_smoke`: nothing
        - `minecraft:lava`: nothing
        - `minecraft:mycelium`: nothing
        - `minecraft:note`: nothing
        - `minecraft:poof`: nothing
        - `minecraft:portal`: nothing
        - `minecraft:rain`: nothing
        - `minecraft:smoke`: nothing
        - `minecraft:white_smoke`: nothing
        - `minecraft:sneeze`: nothing
        - `minecraft:spit`: nothing
        - `minecraft:squid_ink`: nothing
        - `minecraft:sweep_attack`: nothing
        - `minecraft:totem_of_undying`: nothing
        - `minecraft:underwater`: nothing
        - `minecraft:splash`: nothing
        - `minecraft:witch`: nothing
        - `minecraft:bubble_pop`: nothing
        - `minecraft:current_down`: nothing
        - `minecraft:bubble_column_up`: nothing
        - `minecraft:nautilus`: nothing
        - `minecraft:dolphin`: nothing
        - `minecraft:campfire_cosy_smoke`: nothing
        - `minecraft:campfire_signal_smoke`: nothing
        - `minecraft:dripping_honey`: nothing
        - `minecraft:falling_honey`: nothing
        - `minecraft:landing_honey`: nothing
        - `minecraft:falling_nectar`: nothing
        - `minecraft:falling_spore_blossom`: nothing
        - `minecraft:ash`: nothing
        - `minecraft:crimson_spore`: nothing
        - `minecraft:warped_spore`: nothing
        - `minecraft:spore_blossom_air`: nothing
        - `minecraft:dripping_obsidian_tear`: nothing
        - `minecraft:falling_obsidian_tear`: nothing
        - `minecraft:landing_obsidian_tear`: nothing
        - `minecraft:reverse_portal`: nothing
        - `minecraft:white_ash`: nothing
        - `minecraft:small_flame`: nothing
        - `minecraft:snowflake`: nothing
        - `minecraft:dripping_dripstone_lava`: nothing
        - `minecraft:falling_dripstone_lava`: nothing
        - `minecraft:dripping_dripstone_water`: nothing
        - `minecraft:falling_dripstone_water`: nothing
        - `minecraft:glow_squid_ink`: nothing
        - `minecraft:glow`: nothing
        - `minecraft:wax_on`: nothing
        - `minecraft:wax_off`: nothing
        - `minecraft:electric_spark`: nothing
        - `minecraft:scrape`: nothing
        - `minecraft:shriek`: struct `ShriekParticleOption`
          - `delay`: `VAR_INT`
        - `minecraft:egg_crack`: nothing
        - `minecraft:dust_plume`: nothing
        - `minecraft:trial_spawner_detection`: nothing
        - `minecraft:trial_spawner_detection_ominous`: nothing
        - `minecraft:vault_connection`: nothing
        - `minecraft:dust_pillar`: id in Block.BLOCK_STATE_REGISTRY
        - `minecraft:ominous_spawning`: nothing
        - `minecraft:raid_omen`: nothing
        - `minecraft:trial_omen`: nothing
        - `minecraft:block_crumble`: id in Block.BLOCK_STATE_REGISTRY
        - `minecraft:firefly`: nothing
        - `minecraft:sulfur_cube_goo`: nothing
      - `scaling`: `FLOAT`
      - `speed`: `FLOAT`
    - `weight`: `VAR_INT`
- `playSound`: `BOOL`

<a id="pkt-clientbound-minecraft-add_transient_block"></a>
### minecraft:add_transient_block (clientbound, id 37)

- `pos`: `BLOCK_POS`
- `blockState`: id in Block.BLOCK_STATE_REGISTRY

<a id="pkt-clientbound-minecraft-forget_level_chunk"></a>
### minecraft:forget_level_chunk (clientbound, id 38)

- `pos`: `CHUNK_POS`

<a id="pkt-clientbound-minecraft-game_event"></a>
### minecraft:game_event (clientbound, id 39)

- `event`: `UNSIGNED_BYTE`
- `param`: `FLOAT`

<a id="pkt-clientbound-minecraft-game_rule_values"></a>
### minecraft:game_rule_values (clientbound, id 40)

- map of resource key in game_rule to `STRING`

<a id="pkt-clientbound-minecraft-game_test_highlight_pos"></a>
### minecraft:game_test_highlight_pos (clientbound, id 41)

- `absolutePos`: `BLOCK_POS`
- `relativePos`: `BLOCK_POS`

<a id="pkt-clientbound-minecraft-mount_screen_open"></a>
### minecraft:mount_screen_open (clientbound, id 42)

- `containerId`: `CONTAINER_ID`
- `inventoryColumns`: `VAR_INT`
- `entityId`: `INT`

<a id="pkt-clientbound-minecraft-hurt_animation"></a>
### minecraft:hurt_animation (clientbound, id 43)

- `id`: `VAR_INT`
- `yaw`: `FLOAT`

<a id="pkt-clientbound-minecraft-initialize_border"></a>
### minecraft:initialize_border (clientbound, id 44)

- `newCenterX`: `DOUBLE`
- `newCenterZ`: `DOUBLE`
- `oldSize`: `DOUBLE`
- `newSize`: `DOUBLE`
- `lerpTime`: `VAR_LONG`
- `newAbsoluteMaxSize`: `VAR_INT`
- `warningBlocks`: `VAR_INT`
- `warningTime`: `VAR_INT`

<a id="pkt-clientbound-minecraft-keep_alive"></a>
### minecraft:keep_alive (clientbound, id 45)

- `id`: `LONG`

<a id="pkt-clientbound-minecraft-level_chunk_with_light"></a>
### minecraft:level_chunk_with_light (clientbound, id 46)

- `x`: `INT`
- `z`: `INT`
- `chunkData`: struct `ClientboundLevelChunkPacketData`
  - `heightmaps`: map of enum `Heightmap$Types` (var int, ordinal: WORLD_SURFACE_WG, WORLD_SURFACE, OCEAN_FLOOR_WG, OCEAN_FLOOR, MOTION_BLOCKING, MOTION_BLOCKING_NO_LEAVES) to `LONG_ARRAY`
  - `buffer`: `CHUNK_SECTIONS`
  - `blockEntitiesData`: list of
    - struct `ClientboundLevelChunkPacketData$BlockEntityInfo`
      - `packedXZ`: `BYTE`
      - `y`: `SHORT`
      - `type`: id in block_entity_type
      - `tag`: `OPTIONAL_NBT`
- `lightData`: struct `ClientboundLightUpdatePacketData`
  - `skyYMask`: `BYTE_BIT_SET`
  - `blockYMask`: `BYTE_BIT_SET`
  - `emptySkyYMask`: `BYTE_BIT_SET`
  - `emptyBlockYMask`: `BYTE_BIT_SET`
  - `skyUpdates`: list of `BYTE_ARRAY` (max=2048)
  - `blockUpdates`: list of `BYTE_ARRAY` (max=2048)

<a id="pkt-clientbound-minecraft-level_event"></a>
### minecraft:level_event (clientbound, id 47)

- `type`: `INT`
- `pos`: `BLOCK_POS`
- `data`: `INT`
- `globalEvent`: `BOOL`

<a id="pkt-clientbound-minecraft-level_particles"></a>
### minecraft:level_particles (clientbound, id 48)

- `particle`: dispatch `ParticleOptions` on id in particle_type
  - `minecraft:angry_villager`: nothing
  - `minecraft:block`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:block_marker`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:bubble`: nothing
  - `minecraft:sulfur_bubbles`: nothing
  - `minecraft:noxious_gas`: nothing
  - `minecraft:noxious_gas_cloud`: nothing
  - `minecraft:geyser`: struct `GeyserParticleOptions`
    - `waterBlocks`: `INT`
  - `minecraft:geyser_base`: struct `GeyserBaseParticleOptions`
    - `waterBlocks`: `INT`
    - `burstImpulseBase`: `FLOAT`
  - `minecraft:geyser_poof`: struct `GeyserBaseParticleOptions`
    - `waterBlocks`: `INT`
    - `burstImpulseBase`: `FLOAT`
  - `minecraft:geyser_plume`: struct `GeyserParticleOptions`
    - `waterBlocks`: `INT`
  - `minecraft:cloud`: nothing
  - `minecraft:copper_fire_flame`: nothing
  - `minecraft:crit`: nothing
  - `minecraft:damage_indicator`: nothing
  - `minecraft:dragon_breath`: `FLOAT`
  - `minecraft:dripping_lava`: nothing
  - `minecraft:falling_lava`: nothing
  - `minecraft:landing_lava`: nothing
  - `minecraft:dripping_water`: nothing
  - `minecraft:falling_water`: nothing
  - `minecraft:dust`: struct `DustParticleOptions`
    - `color`: `INT`
    - `getScale`: `FLOAT`
  - `minecraft:dust_color_transition`: struct `DustColorTransitionOptions`
    - `fromColor`: `INT`
    - `toColor`: `INT`
    - `getScale`: `FLOAT`
  - `minecraft:effect`: struct `SpellParticleOption`
    - `color`: `INT`
    - `power`: `FLOAT`
  - `minecraft:elder_guardian`: nothing
  - `minecraft:enchanted_hit`: nothing
  - `minecraft:enchant`: nothing
  - `minecraft:end_rod`: nothing
  - `minecraft:entity_effect`: `INT`
  - `minecraft:explosion_emitter`: nothing
  - `minecraft:explosion`: nothing
  - `minecraft:gust`: nothing
  - `minecraft:small_gust`: nothing
  - `minecraft:gust_emitter_large`: nothing
  - `minecraft:gust_emitter_small`: nothing
  - `minecraft:sonic_boom`: nothing
  - `minecraft:falling_dust`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:firework`: nothing
  - `minecraft:fishing`: nothing
  - `minecraft:flame`: nothing
  - `minecraft:infested`: nothing
  - `minecraft:cherry_leaves`: nothing
  - `minecraft:pale_oak_leaves`: nothing
  - `minecraft:red_poplar_leaves`: nothing
  - `minecraft:orange_poplar_leaves`: nothing
  - `minecraft:yellow_poplar_leaves`: nothing
  - `minecraft:tinted_leaves`: `INT`
  - `minecraft:sculk_soul`: nothing
  - `minecraft:sculk_charge`: struct `SculkChargeParticleOptions`
    - `roll`: `FLOAT`
  - `minecraft:sculk_charge_pop`: nothing
  - `minecraft:soul_fire_flame`: nothing
  - `minecraft:soul`: nothing
  - `minecraft:flash`: `INT`
  - `minecraft:happy_villager`: nothing
  - `minecraft:composter`: nothing
  - `minecraft:heart`: nothing
  - `minecraft:instant_effect`: struct `SpellParticleOption`
    - `color`: `INT`
    - `power`: `FLOAT`
  - `minecraft:item`: struct `ItemStackTemplate`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: `COMPONENT_PATCH`
  - `minecraft:vibration`: struct `VibrationParticleOption`
    - `getDestination`: dispatch `PositionSource` on id in position_source_type
      - `minecraft:block`: struct `BlockPositionSource`
        - `pos`: `BLOCK_POS`
      - `minecraft:entity`: struct `EntityPositionSource`
        - `getId`: `VAR_INT`
        - `yOffset`: `FLOAT`
    - `getArrivalInTicks`: `VAR_INT`
  - `minecraft:trail`: struct `TrailParticleOption`
    - `target`: struct `Vec3`
      - `x`: `DOUBLE`
      - `y`: `DOUBLE`
      - `z`: `DOUBLE`
    - `color`: `INT`
    - `duration`: `VAR_INT`
  - `minecraft:pause_mob_growth`: nothing
  - `minecraft:reset_mob_growth`: nothing
  - `minecraft:item_slime`: nothing
  - `minecraft:item_cobweb`: nothing
  - `minecraft:item_snowball`: nothing
  - `minecraft:large_smoke`: nothing
  - `minecraft:lava`: nothing
  - `minecraft:mycelium`: nothing
  - `minecraft:note`: nothing
  - `minecraft:poof`: nothing
  - `minecraft:portal`: nothing
  - `minecraft:rain`: nothing
  - `minecraft:smoke`: nothing
  - `minecraft:white_smoke`: nothing
  - `minecraft:sneeze`: nothing
  - `minecraft:spit`: nothing
  - `minecraft:squid_ink`: nothing
  - `minecraft:sweep_attack`: nothing
  - `minecraft:totem_of_undying`: nothing
  - `minecraft:underwater`: nothing
  - `minecraft:splash`: nothing
  - `minecraft:witch`: nothing
  - `minecraft:bubble_pop`: nothing
  - `minecraft:current_down`: nothing
  - `minecraft:bubble_column_up`: nothing
  - `minecraft:nautilus`: nothing
  - `minecraft:dolphin`: nothing
  - `minecraft:campfire_cosy_smoke`: nothing
  - `minecraft:campfire_signal_smoke`: nothing
  - `minecraft:dripping_honey`: nothing
  - `minecraft:falling_honey`: nothing
  - `minecraft:landing_honey`: nothing
  - `minecraft:falling_nectar`: nothing
  - `minecraft:falling_spore_blossom`: nothing
  - `minecraft:ash`: nothing
  - `minecraft:crimson_spore`: nothing
  - `minecraft:warped_spore`: nothing
  - `minecraft:spore_blossom_air`: nothing
  - `minecraft:dripping_obsidian_tear`: nothing
  - `minecraft:falling_obsidian_tear`: nothing
  - `minecraft:landing_obsidian_tear`: nothing
  - `minecraft:reverse_portal`: nothing
  - `minecraft:white_ash`: nothing
  - `minecraft:small_flame`: nothing
  - `minecraft:snowflake`: nothing
  - `minecraft:dripping_dripstone_lava`: nothing
  - `minecraft:falling_dripstone_lava`: nothing
  - `minecraft:dripping_dripstone_water`: nothing
  - `minecraft:falling_dripstone_water`: nothing
  - `minecraft:glow_squid_ink`: nothing
  - `minecraft:glow`: nothing
  - `minecraft:wax_on`: nothing
  - `minecraft:wax_off`: nothing
  - `minecraft:electric_spark`: nothing
  - `minecraft:scrape`: nothing
  - `minecraft:shriek`: struct `ShriekParticleOption`
    - `delay`: `VAR_INT`
  - `minecraft:egg_crack`: nothing
  - `minecraft:dust_plume`: nothing
  - `minecraft:trial_spawner_detection`: nothing
  - `minecraft:trial_spawner_detection_ominous`: nothing
  - `minecraft:vault_connection`: nothing
  - `minecraft:dust_pillar`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:ominous_spawning`: nothing
  - `minecraft:raid_omen`: nothing
  - `minecraft:trial_omen`: nothing
  - `minecraft:block_crumble`: id in Block.BLOCK_STATE_REGISTRY
  - `minecraft:firefly`: nothing
  - `minecraft:sulfur_cube_goo`: nothing
- `overrideLimiter`: `BOOL`
- `alwaysShow`: `BOOL`
- `x`: `DOUBLE`
- `y`: `DOUBLE`
- `z`: `DOUBLE`
- `xDist`: `FLOAT`
- `yDist`: `FLOAT`
- `zDist`: `FLOAT`
- `xMaxSpeed`: `FLOAT`
- `yMaxSpeed`: `FLOAT`
- `zMaxSpeed`: `FLOAT`
- `count`: `VAR_INT`
- `randomizationType`: enum `ClientboundLevelParticlesPacket$RandomizationType` (var int, ordinal: DEFAULT, ALTERNATIVE, ALTERNATIVE_WITH_SPEED)

<a id="pkt-clientbound-minecraft-light_update"></a>
### minecraft:light_update (clientbound, id 49)

- `x`: `VAR_INT`
- `z`: `VAR_INT`
- `lightData`: struct `ClientboundLightUpdatePacketData`
  - `skyYMask`: `BYTE_BIT_SET`
  - `blockYMask`: `BYTE_BIT_SET`
  - `emptySkyYMask`: `BYTE_BIT_SET`
  - `emptyBlockYMask`: `BYTE_BIT_SET`
  - `skyUpdates`: list of `BYTE_ARRAY` (max=2048)
  - `blockUpdates`: list of `BYTE_ARRAY` (max=2048)

<a id="pkt-clientbound-minecraft-login"></a>
### minecraft:login (clientbound, id 50)

- `playerId`: `INT`
- `hardcore`: `BOOL`
- `levels`: list of resource key in dimension
- `maxPlayers`: `VAR_INT`
- `chunkRadius`: `VAR_INT`
- `simulationDistance`: `VAR_INT`
- `reducedDebugInfo`: `BOOL`
- `showDeathScreen`: `BOOL`
- `doLimitedCrafting`: `BOOL`
- `commonPlayerSpawnInfo`: struct `CommonPlayerSpawnInfo`
  - `dimensionType`: id in dimension_type
  - `dimension`: resource key in dimension
  - `seed`: `LONG`
  - `gameType`: enum `GameType` (var int, ordinal: SURVIVAL, CREATIVE, ADVENTURE, SPECTATOR)
  - `previousGameType`: `OPTIONAL_VAR_INT`
  - `isDebug`: `BOOL`
  - `isFlat`: `BOOL`
  - `lastDeathLocation`: optional
    - struct `GlobalPos`
      - `dimension`: resource key in dimension
      - `pos`: `BLOCK_POS`
  - `portalCooldown`: `VAR_INT`
  - `seaLevel`: `VAR_INT`
- `onlineMode`: `BOOL`
- `enforcesSecureChat`: `BOOL`

<a id="pkt-clientbound-minecraft-low_disk_space_warning"></a>
### minecraft:low_disk_space_warning (clientbound, id 51)

- nothing

<a id="pkt-clientbound-minecraft-map_item_data"></a>
### minecraft:map_item_data (clientbound, id 52)

- `mapId`: `VAR_INT`
- `scale`: `BYTE`
- `locked`: `BOOL`
- `decorations`: optional
  - list of
    - struct `MapDecoration`
      - `type`: id in map_decoration_type
      - `x`: `BYTE`
      - `y`: `BYTE`
      - `rot`: `BYTE`
      - `name`: `OPTIONAL_TEXT`
- `colorPatch`: struct `MapItemSavedData$MapPatch`
  - `width`: `UNSIGNED_BYTE`
  - `height`: `UNSIGNED_BYTE` (when `width` > 0)
  - `startX`: `UNSIGNED_BYTE` (when `width` > 0)
  - `startY`: `UNSIGNED_BYTE` (when `width` > 0)
  - `mapColors`: `BYTE_ARRAY` (when `width` > 0)

<a id="pkt-clientbound-minecraft-merchant_offers"></a>
### minecraft:merchant_offers (clientbound, id 53)

- `containerId`: `CONTAINER_ID`
- `offers`: list of
  - struct `MerchantOffer`
    - `baseCostA`: struct `ItemCost`
      - `item`: id in item
      - `count`: `VAR_INT`
      - `components`: list of `TYPED_DATA_COMPONENT`
    - `result`: `ITEM_STACK`
    - `costB`: optional
      - struct `ItemCost`
        - `item`: id in item
        - `count`: `VAR_INT`
        - `components`: list of `TYPED_DATA_COMPONENT`
    - `isExhausted`: `BOOL`
    - `uses`: `INT`
    - `maxUses`: `INT`
    - `xp`: `INT`
    - `specialPriceDiff`: `INT`
    - `priceMultiplier`: `FLOAT`
    - `demand`: `INT`
- `villagerLevel`: `VAR_INT`
- `villagerXp`: `VAR_INT`
- `showProgress`: `BOOL`
- `canRestock`: `BOOL`

<a id="pkt-clientbound-minecraft-move_entity_pos"></a>
### minecraft:move_entity_pos (clientbound, id 54)

- `id`: `VAR_INT`
- `flags`: `VAR_INT` as bits (onGround@0:1, stepCount@1:31)
- `vecDeltaSteppedDeltaStep`: `flags.stepCount` times (when `flags.stepCount` > 0)
  - struct `VecDelta$Stepped$DeltaStep`
    - `ticks`: `VAR_INT`
    - `xa`: `SHORT`
    - `ya`: `SHORT`
    - `za`: `SHORT`
- `vecDeltaLinear`: struct `VecDelta$Linear` (when `flags.stepCount` <= 0)
  - `xa`: `SHORT`
  - `ya`: `SHORT`
  - `za`: `SHORT`

<a id="pkt-clientbound-minecraft-move_entity_pos_rot"></a>
### minecraft:move_entity_pos_rot (clientbound, id 55)

- `id`: `VAR_INT`
- `flags`: `VAR_INT` as bits (onGround@0:1, stepCount@1:31)
- `vecDeltaSteppedDeltaStep`: `flags.stepCount` times (when `flags.stepCount` > 0)
  - struct `VecDelta$Stepped$DeltaStep`
    - `ticks`: `VAR_INT`
    - `xa`: `SHORT`
    - `ya`: `SHORT`
    - `za`: `SHORT`
- `vecDeltaLinear`: struct `VecDelta$Linear` (when `flags.stepCount` <= 0)
  - `xa`: `SHORT`
  - `ya`: `SHORT`
  - `za`: `SHORT`
- `yRot`: `BYTE`
- `xRot`: `BYTE`

<a id="pkt-clientbound-minecraft-move_minecart_along_track"></a>
### minecraft:move_minecart_along_track (clientbound, id 56)

- `entityId`: `VAR_INT`
- `lerpSteps`: list of
  - struct `NewMinecartBehavior$MinecartStep`
    - `position`: struct `Vec3`
      - `x`: `DOUBLE`
      - `y`: `DOUBLE`
      - `z`: `DOUBLE`
    - `movement`: struct `Vec3`
      - `x`: `DOUBLE`
      - `y`: `DOUBLE`
      - `z`: `DOUBLE`
    - `yRot`: `ROTATION_BYTE`
    - `xRot`: `ROTATION_BYTE`
    - `weight`: `FLOAT`

<a id="pkt-clientbound-minecraft-move_entity_rot"></a>
### minecraft:move_entity_rot (clientbound, id 57)

- `id`: `VAR_INT`
- `onGround`: `BOOL`
- `yRot`: `BYTE`
- `xRot`: `BYTE`

<a id="pkt-clientbound-minecraft-move_vehicle"></a>
### minecraft:move_vehicle (clientbound, id 58)

- `position`: struct `Vec3`
  - `x`: `DOUBLE`
  - `y`: `DOUBLE`
  - `z`: `DOUBLE`
- `yRot`: `FLOAT`
- `xRot`: `FLOAT`

<a id="pkt-clientbound-minecraft-open_book"></a>
### minecraft:open_book (clientbound, id 59)

- `hand`: enum `InteractionHand` (var int, ordinal: MAIN_HAND, OFF_HAND)

<a id="pkt-clientbound-minecraft-open_screen"></a>
### minecraft:open_screen (clientbound, id 60)

- `getContainerId`: `CONTAINER_ID`
- `getType`: id in menu
- `getTitle`: `TEXT`

<a id="pkt-clientbound-minecraft-open_sign_editor"></a>
### minecraft:open_sign_editor (clientbound, id 61)

- `pos`: `BLOCK_POS`
- `slot`: enum `SignTextSlot` (var int, ordinal: BACK, FRONT)

<a id="pkt-clientbound-minecraft-ping"></a>
### minecraft:ping (clientbound, id 62)

- `id`: `INT`

<a id="pkt-clientbound-minecraft-pong_response"></a>
### minecraft:pong_response (clientbound, id 63)

- `time`: `LONG`

<a id="pkt-clientbound-minecraft-place_ghost_recipe"></a>
### minecraft:place_ghost_recipe (clientbound, id 64)

- `containerId`: `CONTAINER_ID`
- `recipeDisplay`: dispatch `RecipeDisplay` on id in recipe_display
  - `minecraft:crafting_shapeless`: struct `ShapelessCraftingRecipeDisplay`
    - `ingredients`: list of
      - dispatch `SlotDisplay` on id in slot_display
        - `minecraft:empty`: nothing
        - `minecraft:any_fuel`: nothing
        - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
          - `display`: a `SlotDisplay` again
        - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
          - `source`: a `SlotDisplay` again
          - `component`: id in data_component_type
        - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
          - `item`: id in item
        - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
          - `stack`: struct `ItemStackTemplate`
            - `item`: id in item
            - `count`: `VAR_INT`
            - `components`: `COMPONENT_PATCH`
        - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
          - `tag`: set of item (a tag or ids)
        - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
          - `dye`: a `SlotDisplay` again
          - `target`: a `SlotDisplay` again
        - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
          - `base`: a `SlotDisplay` again
          - `material`: a `SlotDisplay` again
          - `pattern`: id in trim_pattern, or 0 and the element inline
            - inline: struct `TrimPattern`
              - `assetId`: `IDENTIFIER`
              - `description`: `TEXT`
              - `decal`: `BOOL`
        - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
          - `input`: a `SlotDisplay` again
          - `remainder`: a `SlotDisplay` again
        - `minecraft:composite`: struct `SlotDisplay$Composite`
          - `contents`: list of a `SlotDisplay` again
    - `result`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
  - `minecraft:crafting_shaped`: struct `ShapedCraftingRecipeDisplay`
    - `width`: `VAR_INT`
    - `height`: `VAR_INT`
    - `ingredients`: list of
      - dispatch `SlotDisplay` on id in slot_display
        - `minecraft:empty`: nothing
        - `minecraft:any_fuel`: nothing
        - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
          - `display`: a `SlotDisplay` again
        - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
          - `source`: a `SlotDisplay` again
          - `component`: id in data_component_type
        - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
          - `item`: id in item
        - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
          - `stack`: struct `ItemStackTemplate`
            - `item`: id in item
            - `count`: `VAR_INT`
            - `components`: `COMPONENT_PATCH`
        - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
          - `tag`: set of item (a tag or ids)
        - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
          - `dye`: a `SlotDisplay` again
          - `target`: a `SlotDisplay` again
        - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
          - `base`: a `SlotDisplay` again
          - `material`: a `SlotDisplay` again
          - `pattern`: id in trim_pattern, or 0 and the element inline
            - inline: struct `TrimPattern`
              - `assetId`: `IDENTIFIER`
              - `description`: `TEXT`
              - `decal`: `BOOL`
        - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
          - `input`: a `SlotDisplay` again
          - `remainder`: a `SlotDisplay` again
        - `minecraft:composite`: struct `SlotDisplay$Composite`
          - `contents`: list of a `SlotDisplay` again
    - `result`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
  - `minecraft:furnace`: struct `FurnaceRecipeDisplay`
    - `ingredient`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `fuel`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `result`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `duration`: `VAR_INT`
    - `experience`: `FLOAT`
  - `minecraft:stonecutter`: struct `StonecutterRecipeDisplay`
    - `input`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `result`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
  - `minecraft:smithing`: struct `SmithingRecipeDisplay`
    - `template`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `base`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `addition`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `result`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again
    - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
      - `minecraft:empty`: nothing
      - `minecraft:any_fuel`: nothing
      - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
        - `display`: a `SlotDisplay` again
      - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
        - `source`: a `SlotDisplay` again
        - `component`: id in data_component_type
      - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
        - `item`: id in item
      - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
        - `stack`: struct `ItemStackTemplate`
          - `item`: id in item
          - `count`: `VAR_INT`
          - `components`: `COMPONENT_PATCH`
      - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
        - `tag`: set of item (a tag or ids)
      - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
        - `dye`: a `SlotDisplay` again
        - `target`: a `SlotDisplay` again
      - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
        - `base`: a `SlotDisplay` again
        - `material`: a `SlotDisplay` again
        - `pattern`: id in trim_pattern, or 0 and the element inline
          - inline: struct `TrimPattern`
            - `assetId`: `IDENTIFIER`
            - `description`: `TEXT`
            - `decal`: `BOOL`
      - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
        - `input`: a `SlotDisplay` again
        - `remainder`: a `SlotDisplay` again
      - `minecraft:composite`: struct `SlotDisplay$Composite`
        - `contents`: list of a `SlotDisplay` again

<a id="pkt-clientbound-minecraft-player_abilities"></a>
### minecraft:player_abilities (clientbound, id 65)

- `bitfield`: `BYTE`
- `flyingSpeed`: `FLOAT`
- `walkingSpeed`: `FLOAT`

<a id="pkt-clientbound-minecraft-player_chat"></a>
### minecraft:player_chat (clientbound, id 66)

- `globalIndex`: `VAR_INT`
- `sender`: `UUID`
- `index`: `VAR_INT`
- `signature`: optional `MESSAGE_SIGNATURE`
- `body`: struct `SignedMessageBody$Packed`
  - `content`: string (at most 256 characters)
  - `timeStamp`: `INSTANT`
  - `salt`: `LONG`
  - `lastSeen`: struct `LastSeenMessages$Packed`
    - `entries`: list of (at most 20)
      - struct `MessageSignature$Packed`
        - `id`: `VAR_INT`
        - `fullSignature`: `FIXED_BYTES` (len=256) (when `id` == 0)
- `unsignedContent`: `OPTIONAL_TEXT`
- `filterMask`: dispatch `FilterMask` on enum `FilterMask$Type` (var int, ordinal: PASS_THROUGH, FULLY_FILTERED, PARTIALLY_FILTERED)
  - `PASS_THROUGH (0)`: nothing
  - `FULLY_FILTERED (1)`: nothing
  - `PARTIALLY_FILTERED (2)`: `BYTE_BIT_SET`
- `chatType`: struct `ChatType$Bound`
  - `chatType`: id in chat_type, or 0 and the element inline
    - inline: struct `ChatType`
      - `chat`: struct `ChatTypeDecoration`
        - `translationKey`: `STRING`
        - `parameters`: list of enum `ChatTypeDecoration$Parameter` (var int, ordinal: SENDER, TARGET, CONTENT)
        - `style`: an NBT tag
      - `narration`: struct `ChatTypeDecoration`
        - `translationKey`: `STRING`
        - `parameters`: list of enum `ChatTypeDecoration$Parameter` (var int, ordinal: SENDER, TARGET, CONTENT)
        - `style`: an NBT tag
  - `name`: `TEXT`
  - `targetName`: `OPTIONAL_TEXT`

<a id="pkt-clientbound-minecraft-player_combat_end"></a>
### minecraft:player_combat_end (clientbound, id 67)

- `duration`: `VAR_INT`

<a id="pkt-clientbound-minecraft-player_combat_enter"></a>
### minecraft:player_combat_enter (clientbound, id 68)

- nothing

<a id="pkt-clientbound-minecraft-player_combat_kill"></a>
### minecraft:player_combat_kill (clientbound, id 69)

- `playerId`: `VAR_INT`
- `message`: `TEXT`

<a id="pkt-clientbound-minecraft-player_info_remove"></a>
### minecraft:player_info_remove (clientbound, id 70)

- `profileIds`: list of `UUID`

<a id="pkt-clientbound-minecraft-player_info_update"></a>
### minecraft:player_info_update (clientbound, id 71)

- `actions`: enum set of `ClientboundPlayerInfoUpdatePacket$Action` (8 bits)
- `entries`: list of
  - struct `ClientboundPlayerInfoUpdatePacket$EntryBuilder` (guarded by an EnumSet of ClientboundPlayerInfoUpdatePacket$Action)
    - `profileId`: `UUID`
    - `name`: `STRING` (when the set has `ADD_PLAYER`)
    - `properties`: `GAME_PROFILE_PROPERTIES` (when the set has `ADD_PLAYER`)
    - `chatSession`: optional (when the set has `INITIALIZE_CHAT`)
      - struct `RemoteChatSession$Data`
        - `sessionId`: `UUID`
        - `profilePublicKey`: struct `ProfilePublicKey$Data`
          - `expiresAt`: `INSTANT`
          - `key`: `PUBLIC_KEY`
          - `keySignature`: `BYTE_ARRAY` (max=4096)
    - `gameMode`: enum `GameType` (var int, ordinal: SURVIVAL, CREATIVE, ADVENTURE, SPECTATOR) (when the set has `UPDATE_GAME_MODE`)
    - `listed`: `BOOL` (when the set has `UPDATE_LISTED`)
    - `latency`: `VAR_INT` (when the set has `UPDATE_LATENCY`)
    - `displayName`: optional `TEXT` (when the set has `UPDATE_DISPLAY_NAME`)
    - `listOrder`: `VAR_INT` (when the set has `UPDATE_LIST_ORDER`)
    - `showHat`: `BOOL` (when the set has `UPDATE_HAT`)

<a id="pkt-clientbound-minecraft-player_look_at"></a>
### minecraft:player_look_at (clientbound, id 72)

- `fromAnchor`: enum `EntityAnchorArgument$Anchor` (var int, ordinal: FEET, EYES)
- `x`: `DOUBLE`
- `y`: `DOUBLE`
- `z`: `DOUBLE`
- `atEntity`: `BOOL`
- `entity`: `VAR_INT` (when `atEntity` is true)
- `toAnchor`: enum `EntityAnchorArgument$Anchor` (var int, ordinal: FEET, EYES) (when `atEntity` is true)

<a id="pkt-clientbound-minecraft-player_position"></a>
### minecraft:player_position (clientbound, id 73)

- `id`: `VAR_INT`
- `change`: struct `PositionMoveRotation`
  - `position`: struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
  - `deltaMovement`: struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
  - `yRot`: `FLOAT`
  - `xRot`: `FLOAT`
- `relatives`: `INT`

<a id="pkt-clientbound-minecraft-player_rotation"></a>
### minecraft:player_rotation (clientbound, id 74)

- `yRot`: `FLOAT`
- `relativeY`: `BOOL`
- `xRot`: `FLOAT`
- `relativeX`: `BOOL`

<a id="pkt-clientbound-minecraft-recipe_book_add"></a>
### minecraft:recipe_book_add (clientbound, id 75)

- `entries`: list of
  - struct `ClientboundRecipeBookAddPacket$Entry`
    - `contents`: struct `RecipeDisplayEntry`
      - `id`: struct `RecipeDisplayId`
        - `index`: `VAR_INT`
      - `display`: dispatch `RecipeDisplay` on id in recipe_display
        - `minecraft:crafting_shapeless`: struct `ShapelessCraftingRecipeDisplay`
          - `ingredients`: list of
            - dispatch `SlotDisplay` on id in slot_display
              - `minecraft:empty`: nothing
              - `minecraft:any_fuel`: nothing
              - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
                - `display`: a `SlotDisplay` again
              - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
                - `source`: a `SlotDisplay` again
                - `component`: id in data_component_type
              - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
                - `item`: id in item
              - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
                - `stack`: struct `ItemStackTemplate`
                  - `item`: id in item
                  - `count`: `VAR_INT`
                  - `components`: `COMPONENT_PATCH`
              - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
                - `tag`: set of item (a tag or ids)
              - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
                - `dye`: a `SlotDisplay` again
                - `target`: a `SlotDisplay` again
              - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
                - `base`: a `SlotDisplay` again
                - `material`: a `SlotDisplay` again
                - `pattern`: id in trim_pattern, or 0 and the element inline
                  - inline: struct `TrimPattern`
                    - `assetId`: `IDENTIFIER`
                    - `description`: `TEXT`
                    - `decal`: `BOOL`
              - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
                - `input`: a `SlotDisplay` again
                - `remainder`: a `SlotDisplay` again
              - `minecraft:composite`: struct `SlotDisplay$Composite`
                - `contents`: list of a `SlotDisplay` again
          - `result`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
        - `minecraft:crafting_shaped`: struct `ShapedCraftingRecipeDisplay`
          - `width`: `VAR_INT`
          - `height`: `VAR_INT`
          - `ingredients`: list of
            - dispatch `SlotDisplay` on id in slot_display
              - `minecraft:empty`: nothing
              - `minecraft:any_fuel`: nothing
              - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
                - `display`: a `SlotDisplay` again
              - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
                - `source`: a `SlotDisplay` again
                - `component`: id in data_component_type
              - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
                - `item`: id in item
              - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
                - `stack`: struct `ItemStackTemplate`
                  - `item`: id in item
                  - `count`: `VAR_INT`
                  - `components`: `COMPONENT_PATCH`
              - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
                - `tag`: set of item (a tag or ids)
              - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
                - `dye`: a `SlotDisplay` again
                - `target`: a `SlotDisplay` again
              - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
                - `base`: a `SlotDisplay` again
                - `material`: a `SlotDisplay` again
                - `pattern`: id in trim_pattern, or 0 and the element inline
                  - inline: struct `TrimPattern`
                    - `assetId`: `IDENTIFIER`
                    - `description`: `TEXT`
                    - `decal`: `BOOL`
              - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
                - `input`: a `SlotDisplay` again
                - `remainder`: a `SlotDisplay` again
              - `minecraft:composite`: struct `SlotDisplay$Composite`
                - `contents`: list of a `SlotDisplay` again
          - `result`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
        - `minecraft:furnace`: struct `FurnaceRecipeDisplay`
          - `ingredient`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `fuel`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `result`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `duration`: `VAR_INT`
          - `experience`: `FLOAT`
        - `minecraft:stonecutter`: struct `StonecutterRecipeDisplay`
          - `input`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `result`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
        - `minecraft:smithing`: struct `SmithingRecipeDisplay`
          - `template`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `base`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `addition`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `result`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
          - `craftingStation`: dispatch `SlotDisplay` on id in slot_display
            - `minecraft:empty`: nothing
            - `minecraft:any_fuel`: nothing
            - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
              - `display`: a `SlotDisplay` again
            - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
              - `source`: a `SlotDisplay` again
              - `component`: id in data_component_type
            - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
              - `item`: id in item
            - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
              - `stack`: struct `ItemStackTemplate`
                - `item`: id in item
                - `count`: `VAR_INT`
                - `components`: `COMPONENT_PATCH`
            - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
              - `tag`: set of item (a tag or ids)
            - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
              - `dye`: a `SlotDisplay` again
              - `target`: a `SlotDisplay` again
            - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
              - `base`: a `SlotDisplay` again
              - `material`: a `SlotDisplay` again
              - `pattern`: id in trim_pattern, or 0 and the element inline
                - inline: struct `TrimPattern`
                  - `assetId`: `IDENTIFIER`
                  - `description`: `TEXT`
                  - `decal`: `BOOL`
            - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
              - `input`: a `SlotDisplay` again
              - `remainder`: a `SlotDisplay` again
            - `minecraft:composite`: struct `SlotDisplay$Composite`
              - `contents`: list of a `SlotDisplay` again
      - `group`: `OPTIONAL_VAR_INT`
      - `category`: id in recipe_book_category
      - `craftingRequirements`: optional list of set of item (a tag or ids)
    - `flags`: `BYTE`
- `replace`: `BOOL`

<a id="pkt-clientbound-minecraft-recipe_book_remove"></a>
### minecraft:recipe_book_remove (clientbound, id 76)

- `recipes`: list of
  - struct `RecipeDisplayId`
    - `index`: `VAR_INT`

<a id="pkt-clientbound-minecraft-recipe_book_settings"></a>
### minecraft:recipe_book_settings (clientbound, id 77)

- `bookSettings`: struct `RecipeBookSettings`
  - `crafting`: struct `RecipeBookSettings$TypeSettings`
    - `open`: `BOOL`
    - `filtering`: `BOOL`
  - `furnace`: struct `RecipeBookSettings$TypeSettings`
    - `open`: `BOOL`
    - `filtering`: `BOOL`
  - `blastFurnace`: struct `RecipeBookSettings$TypeSettings`
    - `open`: `BOOL`
    - `filtering`: `BOOL`
  - `smoker`: struct `RecipeBookSettings$TypeSettings`
    - `open`: `BOOL`
    - `filtering`: `BOOL`

<a id="pkt-clientbound-minecraft-remove_entities"></a>
### minecraft:remove_entities (clientbound, id 78)

- `entityIds`: list of `VAR_INT`

<a id="pkt-clientbound-minecraft-remove_mob_effect"></a>
### minecraft:remove_mob_effect (clientbound, id 79)

- `entityId`: `VAR_INT`
- `effect`: id in mob_effect

<a id="pkt-clientbound-minecraft-reset_score"></a>
### minecraft:reset_score (clientbound, id 80)

- `owner`: string
- `objectiveName`: optional `STRING`

<a id="pkt-clientbound-minecraft-resource_pack_pop"></a>
### minecraft:resource_pack_pop (clientbound, id 81)

- `id`: optional `UUID`

<a id="pkt-clientbound-minecraft-resource_pack_push"></a>
### minecraft:resource_pack_push (clientbound, id 82)

- `id`: `UUID`
- `url`: `STRING`
- `hash`: string (at most 40 characters)
- `required`: `BOOL`
- `prompt`: optional `TEXT`

<a id="pkt-clientbound-minecraft-post_effects"></a>
### minecraft:post_effects (clientbound, id 83)

- `postEffects`: list of `IDENTIFIER`

<a id="pkt-clientbound-minecraft-respawn"></a>
### minecraft:respawn (clientbound, id 84)

- `commonPlayerSpawnInfo`: struct `CommonPlayerSpawnInfo`
  - `dimensionType`: id in dimension_type
  - `dimension`: resource key in dimension
  - `seed`: `LONG`
  - `gameType`: enum `GameType` (var int, ordinal: SURVIVAL, CREATIVE, ADVENTURE, SPECTATOR)
  - `previousGameType`: `OPTIONAL_VAR_INT`
  - `isDebug`: `BOOL`
  - `isFlat`: `BOOL`
  - `lastDeathLocation`: optional
    - struct `GlobalPos`
      - `dimension`: resource key in dimension
      - `pos`: `BLOCK_POS`
  - `portalCooldown`: `VAR_INT`
  - `seaLevel`: `VAR_INT`
- `dataToKeep`: `BYTE`

<a id="pkt-clientbound-minecraft-rotate_head"></a>
### minecraft:rotate_head (clientbound, id 85)

- `entityId`: `VAR_INT`
- `yHeadRot`: `BYTE`

<a id="pkt-clientbound-minecraft-section_blocks_update"></a>
### minecraft:section_blocks_update (clientbound, id 86)

- `sectionPos`: `LONG`
- `states`: list of `VAR_LONG`

<a id="pkt-clientbound-minecraft-select_advancements_tab"></a>
### minecraft:select_advancements_tab (clientbound, id 87)

- `tab`: optional `IDENTIFIER`

<a id="pkt-clientbound-minecraft-server_data"></a>
### minecraft:server_data (clientbound, id 88)

- `motd`: `TEXT`
- `iconBytes`: optional `BYTE_ARRAY`

<a id="pkt-clientbound-minecraft-set_action_bar_text"></a>
### minecraft:set_action_bar_text (clientbound, id 89)

- `text`: `TEXT`

<a id="pkt-clientbound-minecraft-set_border_center"></a>
### minecraft:set_border_center (clientbound, id 90)

- `newCenterX`: `DOUBLE`
- `newCenterZ`: `DOUBLE`

<a id="pkt-clientbound-minecraft-set_border_lerp_size"></a>
### minecraft:set_border_lerp_size (clientbound, id 91)

- `oldSize`: `DOUBLE`
- `newSize`: `DOUBLE`
- `lerpTime`: `VAR_LONG`

<a id="pkt-clientbound-minecraft-set_border_size"></a>
### minecraft:set_border_size (clientbound, id 92)

- `size`: `DOUBLE`

<a id="pkt-clientbound-minecraft-set_border_warning_delay"></a>
### minecraft:set_border_warning_delay (clientbound, id 93)

- `warningDelay`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_border_warning_distance"></a>
### minecraft:set_border_warning_distance (clientbound, id 94)

- `warningBlocks`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_camera"></a>
### minecraft:set_camera (clientbound, id 95)

- `cameraId`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_chunk_cache_center"></a>
### minecraft:set_chunk_cache_center (clientbound, id 96)

- `x`: `VAR_INT`
- `z`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_chunk_cache_radius"></a>
### minecraft:set_chunk_cache_radius (clientbound, id 97)

- `radius`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_cursor_item"></a>
### minecraft:set_cursor_item (clientbound, id 98)

- `contents`: `OPTIONAL_ITEM_STACK`

<a id="pkt-clientbound-minecraft-set_default_spawn_position"></a>
### minecraft:set_default_spawn_position (clientbound, id 99)

- `respawnData`: struct `LevelData$RespawnData`
  - `globalPos`: struct `GlobalPos`
    - `dimension`: resource key in dimension
    - `pos`: `BLOCK_POS`
  - `yaw`: `FLOAT`
  - `pitch`: `FLOAT`

<a id="pkt-clientbound-minecraft-set_display_objective"></a>
### minecraft:set_display_objective (clientbound, id 100)

- `slot`: enum `DisplaySlot` (var int, ordinal: LIST, SIDEBAR, BELOW_NAME, TEAM_BLACK, TEAM_DARK_BLUE, TEAM_DARK_GREEN, TEAM_DARK_AQUA, TEAM_DARK_RED, TEAM_DARK_PURPLE, TEAM_GOLD, TEAM_GRAY, TEAM_DARK_GRAY, TEAM_BLUE, TEAM_GREEN, TEAM_AQUA, TEAM_RED, TEAM_LIGHT_PURPLE, TEAM_YELLOW, TEAM_WHITE)
- `objectiveName`: string

<a id="pkt-clientbound-minecraft-set_entity_data"></a>
### minecraft:set_entity_data (clientbound, id 101)

- `id`: `VAR_INT`
- `packedItems`: `ENTITY_DATA`

<a id="pkt-clientbound-minecraft-set_entity_link"></a>
### minecraft:set_entity_link (clientbound, id 102)

- `sourceId`: `INT`
- `destId`: `INT`

<a id="pkt-clientbound-minecraft-set_entity_motion"></a>
### minecraft:set_entity_motion (clientbound, id 103)

- `id`: `VAR_INT`
- `movement`: `LP_VEC3`

<a id="pkt-clientbound-minecraft-set_equipment"></a>
### minecraft:set_equipment (clientbound, id 104)

- `entity`: `VAR_INT`
- `slots`: entries until `slotId` & -128 == 0
  - struct `ClientboundSetEquipmentEntry`
    - `slotId`: `BYTE`
    - `itemStack`: `OPTIONAL_ITEM_STACK`

<a id="pkt-clientbound-minecraft-set_experience"></a>
### minecraft:set_experience (clientbound, id 105)

- `experienceProgress`: `FLOAT`
- `experienceLevel`: `VAR_INT`
- `totalExperience`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_health"></a>
### minecraft:set_health (clientbound, id 106)

- `health`: `FLOAT`
- `food`: `VAR_INT`
- `saturation`: `FLOAT`

<a id="pkt-clientbound-minecraft-set_held_slot"></a>
### minecraft:set_held_slot (clientbound, id 107)

- `slot`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_objective"></a>
### minecraft:set_objective (clientbound, id 108)

- `objectiveName`: string
- `method`: `BYTE`
- `displayName`: `TEXT` (when `method` == 2 or `method` == 0)
- `renderType`: enum `ObjectiveCriteria$RenderType` (var int, ordinal: INTEGER, HEARTS) (when `method` == 2 or `method` == 0)
- `numberFormat`: optional (when `method` == 2 or `method` == 0)
  - dispatch `NumberFormat` on id in number_format_type
    - `minecraft:blank`: nothing
    - `minecraft:styled`: struct `StyledFormat`
      - `style`: an NBT tag
    - `minecraft:fixed`: struct `FixedFormat`
      - `value`: `TEXT`

<a id="pkt-clientbound-minecraft-set_passengers"></a>
### minecraft:set_passengers (clientbound, id 109)

- `vehicle`: `VAR_INT`
- `passengers`: `VAR_INT_ARRAY`

<a id="pkt-clientbound-minecraft-set_player_inventory"></a>
### minecraft:set_player_inventory (clientbound, id 110)

- `slot`: `VAR_INT`
- `contents`: `OPTIONAL_ITEM_STACK`

<a id="pkt-clientbound-minecraft-set_player_team"></a>
### minecraft:set_player_team (clientbound, id 111)

- `name`: string
- `method`: `BYTE`
- `parameters`: struct `ClientboundSetPlayerTeamPacket$Parameters` (when `method` in [0 2])
  - `displayName`: `TEXT`
  - `playerPrefix`: `TEXT`
  - `playerSuffix`: `TEXT`
  - `nameTagVisibility`: enum `Team$Visibility` (var int, ordinal: ALWAYS, NEVER, HIDE_FOR_OTHER_TEAMS, HIDE_FOR_OWN_TEAM)
  - `collisionRule`: enum `Team$CollisionRule` (var int, ordinal: ALWAYS, NEVER, PUSH_OTHER_TEAMS, PUSH_OWN_TEAM)
  - `color`: optional enum `TeamColor` (var int, ordinal: BLACK, DARK_BLUE, DARK_GREEN, DARK_AQUA, DARK_RED, DARK_PURPLE, GOLD, GRAY, DARK_GRAY, BLUE, GREEN, AQUA, RED, LIGHT_PURPLE, YELLOW, WHITE)
  - `options`: `BYTE`
- `players`: list of `STRING` (when `method` in [0 3 4])

<a id="pkt-clientbound-minecraft-set_score"></a>
### minecraft:set_score (clientbound, id 112)

- `owner`: `STRING`
- `objectiveName`: `STRING`
- `score`: `VAR_INT`
- `display`: `OPTIONAL_TEXT`
- `numberFormat`: optional
  - dispatch `NumberFormat` on id in number_format_type
    - `minecraft:blank`: nothing
    - `minecraft:styled`: struct `StyledFormat`
      - `style`: an NBT tag
    - `minecraft:fixed`: struct `FixedFormat`
      - `value`: `TEXT`

<a id="pkt-clientbound-minecraft-set_simulation_distance"></a>
### minecraft:set_simulation_distance (clientbound, id 113)

- `simulationDistance`: `VAR_INT`

<a id="pkt-clientbound-minecraft-set_subtitle_text"></a>
### minecraft:set_subtitle_text (clientbound, id 114)

- `text`: `TEXT`

<a id="pkt-clientbound-minecraft-set_time"></a>
### minecraft:set_time (clientbound, id 115)

- `gameTime`: `LONG`
- `clockUpdates`: map
  - key: id in world_clock
  - value: struct `ClockNetworkState`
    - `totalTicks`: `VAR_LONG`
    - `partialTick`: `FLOAT`
    - `rate`: `FLOAT`

<a id="pkt-clientbound-minecraft-set_title_text"></a>
### minecraft:set_title_text (clientbound, id 116)

- `text`: `TEXT`

<a id="pkt-clientbound-minecraft-set_titles_animation"></a>
### minecraft:set_titles_animation (clientbound, id 117)

- `fadeIn`: `INT`
- `stay`: `INT`
- `fadeOut`: `INT`

<a id="pkt-clientbound-minecraft-sound_entity"></a>
### minecraft:sound_entity (clientbound, id 118)

- `sound`: id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`
- `source`: enum `SoundSource` (var int, ordinal: MASTER, MUSIC, RECORDS, WEATHER, BLOCKS, HOSTILE, NEUTRAL, PLAYERS, AMBIENT, VOICE, UI)
- `id`: `VAR_INT`
- `volume`: `FLOAT`
- `pitch`: `FLOAT`
- `seed`: `LONG`

<a id="pkt-clientbound-minecraft-sound"></a>
### minecraft:sound (clientbound, id 119)

- `sound`: id in sound_event, or 0 and the element inline
  - inline: struct `SoundEvent`
    - `location`: `IDENTIFIER`
    - `fixedRange`: optional `FLOAT`
- `source`: enum `SoundSource` (var int, ordinal: MASTER, MUSIC, RECORDS, WEATHER, BLOCKS, HOSTILE, NEUTRAL, PLAYERS, AMBIENT, VOICE, UI)
- `x`: `INT`
- `y`: `INT`
- `z`: `INT`
- `volume`: `FLOAT`
- `pitch`: `FLOAT`
- `seed`: `LONG`

<a id="pkt-clientbound-minecraft-start_configuration"></a>
### minecraft:start_configuration (clientbound, id 120)

- nothing

<a id="pkt-clientbound-minecraft-stop_sound"></a>
### minecraft:stop_sound (clientbound, id 121)

- `flags`: `BYTE`
- `source`: enum `SoundSource` (var int, ordinal: MASTER, MUSIC, RECORDS, WEATHER, BLOCKS, HOSTILE, NEUTRAL, PLAYERS, AMBIENT, VOICE, UI) (when `flags` & 1 != 0)
- `name`: `IDENTIFIER` (when `flags` & 2 != 0)

<a id="pkt-clientbound-minecraft-store_cookie"></a>
### minecraft:store_cookie (clientbound, id 122)

- `key`: `IDENTIFIER`
- `payload`: `BYTE_ARRAY` (max=5120)

<a id="pkt-clientbound-minecraft-swing_animation"></a>
### minecraft:swing_animation (clientbound, id 123)

- `entityId`: `VAR_INT`
- `hand`: enum `InteractionHand` (var int, ordinal: MAIN_HAND, OFF_HAND)
- `animation`: struct `SwingAnimation`
  - `type`: enum `SwingAnimationType` (var int, ordinal: NONE, WHACK, STAB)
  - `duration`: `VAR_INT`

<a id="pkt-clientbound-minecraft-system_chat"></a>
### minecraft:system_chat (clientbound, id 124)

- `content`: `TEXT`
- `overlay`: `BOOL`

<a id="pkt-clientbound-minecraft-tab_list"></a>
### minecraft:tab_list (clientbound, id 125)

- `header`: `TEXT`
- `footer`: `TEXT`

<a id="pkt-clientbound-minecraft-tag_query"></a>
### minecraft:tag_query (clientbound, id 126)

- `transactionId`: `VAR_INT`
- `tag`: `NBT`

<a id="pkt-clientbound-minecraft-take_item_entity"></a>
### minecraft:take_item_entity (clientbound, id 127)

- `itemId`: `VAR_INT`
- `playerId`: `VAR_INT`
- `amount`: `VAR_INT`

<a id="pkt-clientbound-minecraft-teleport_entity"></a>
### minecraft:teleport_entity (clientbound, id 128)

- `id`: `VAR_INT`
- `change`: struct `PositionMoveRotation`
  - `position`: struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
  - `deltaMovement`: struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
  - `yRot`: `FLOAT`
  - `xRot`: `FLOAT`
- `relatives`: `INT`
- `onGround`: `BOOL`

<a id="pkt-clientbound-minecraft-test_instance_block_status"></a>
### minecraft:test_instance_block_status (clientbound, id 129)

- `status`: `TEXT`
- `size`: optional
  - struct `Vec3i`
    - `getX`: `VAR_INT`
    - `getY`: `VAR_INT`
    - `getZ`: `VAR_INT`

<a id="pkt-clientbound-minecraft-ticking_state"></a>
### minecraft:ticking_state (clientbound, id 130)

- `tickRate`: `FLOAT`
- `isFrozen`: `BOOL`

<a id="pkt-clientbound-minecraft-ticking_step"></a>
### minecraft:ticking_step (clientbound, id 131)

- `tickSteps`: `VAR_INT`

<a id="pkt-clientbound-minecraft-transfer"></a>
### minecraft:transfer (clientbound, id 132)

- `host`: string
- `port`: `VAR_INT`

<a id="pkt-clientbound-minecraft-update_advancements"></a>
### minecraft:update_advancements (clientbound, id 133)

- `shouldReset`: `BOOL`
- `added`: list of
  - struct `ClientboundUpdateAdvancementsPacket$PositionedAdvancement`
    - `advancement`: struct `AdvancementHolder`
      - `id`: `IDENTIFIER`
      - `value`: struct `Advancement`
        - `parent`: optional `IDENTIFIER`
        - `display`: optional
          - struct `DisplayInfo`
            - `title`: `TEXT`
            - `description`: `TEXT`
            - `icon`: struct `ItemStackTemplate`
              - `item`: id in item
              - `count`: `VAR_INT`
              - `components`: `COMPONENT_PATCH`
            - `type`: enum `AdvancementType` (var int, ordinal: TASK, CHALLENGE, GOAL)
            - `flags`: `INT`
            - `background`: struct `ClientAsset$ResourceTexture` (when `flags` & 1 != 0)
              - `texture`: `IDENTIFIER`
        - `requirements`: list of list of `STRING`
        - `sendsTelemetryEvent`: `BOOL`
    - `x`: `FLOAT`
    - `y`: `FLOAT`
- `removed`: list of `IDENTIFIER`
- `progress`: map
  - key: `IDENTIFIER`
  - value: struct `AdvancementProgress`
    - `criteria`: map of `STRING` to optional `INSTANT`
- `showAdvancements`: `BOOL`

<a id="pkt-clientbound-minecraft-update_attributes"></a>
### minecraft:update_attributes (clientbound, id 134)

- `getEntityId`: `VAR_INT`
- `getValues`: list of (at most 128)
  - struct `ClientboundUpdateAttributesPacket$AttributeSnapshot`
    - `attribute`: id in attribute
    - `base`: `DOUBLE`
    - `modifiers`: list of
      - struct `AttributeModifier`
        - `id`: `IDENTIFIER`
        - `amount`: `DOUBLE`
        - `operation`: enum `AttributeModifier$Operation` (var int, ordinal: ADD_VALUE, ADD_MULTIPLIED_BASE, ADD_MULTIPLIED_TOTAL)

<a id="pkt-clientbound-minecraft-update_mob_effect"></a>
### minecraft:update_mob_effect (clientbound, id 135)

- `entityId`: `VAR_INT`
- `effect`: id in mob_effect
- `effectAmplifier`: `VAR_INT`
- `effectDurationTicks`: `VAR_INT`
- `flags`: `BYTE`

<a id="pkt-clientbound-minecraft-update_recipes"></a>
### minecraft:update_recipes (clientbound, id 136)

- `itemSets`: map of resource key in type_key to list of id in item
- `stonecutterRecipes`: struct `SelectableRecipe$SingleInputSet`
  - `entries`: list of
    - struct `SelectableRecipe$SingleInputEntry`
      - `input`: set of item (a tag or ids)
      - `recipe`: struct `SelectableRecipe`
        - `optionDisplay`: dispatch `SlotDisplay` on id in slot_display
          - `minecraft:empty`: nothing
          - `minecraft:any_fuel`: nothing
          - `minecraft:with_any_potion`: struct `SlotDisplay$WithAnyPotion`
            - `display`: a `SlotDisplay` again
          - `minecraft:only_with_component`: struct `SlotDisplay$OnlyWithComponent`
            - `source`: a `SlotDisplay` again
            - `component`: id in data_component_type
          - `minecraft:item`: struct `SlotDisplay$ItemSlotDisplay`
            - `item`: id in item
          - `minecraft:item_stack`: struct `SlotDisplay$ItemStackSlotDisplay`
            - `stack`: struct `ItemStackTemplate`
              - `item`: id in item
              - `count`: `VAR_INT`
              - `components`: `COMPONENT_PATCH`
          - `minecraft:tag`: struct `SlotDisplay$TagSlotDisplay`
            - `tag`: set of item (a tag or ids)
          - `minecraft:dyed`: struct `SlotDisplay$DyedSlotDemo`
            - `dye`: a `SlotDisplay` again
            - `target`: a `SlotDisplay` again
          - `minecraft:smithing_trim`: struct `SlotDisplay$SmithingTrimDemoSlotDisplay`
            - `base`: a `SlotDisplay` again
            - `material`: a `SlotDisplay` again
            - `pattern`: id in trim_pattern, or 0 and the element inline
              - inline: struct `TrimPattern`
                - `assetId`: `IDENTIFIER`
                - `description`: `TEXT`
                - `decal`: `BOOL`
          - `minecraft:with_remainder`: struct `SlotDisplay$WithRemainder`
            - `input`: a `SlotDisplay` again
            - `remainder`: a `SlotDisplay` again
          - `minecraft:composite`: struct `SlotDisplay$Composite`
            - `contents`: list of a `SlotDisplay` again

<a id="pkt-clientbound-minecraft-update_tags"></a>
### minecraft:update_tags (clientbound, id 137)

- `tags`: map
  - key: `IDENTIFIER`
  - value: struct `TagNetworkSerialization$NetworkPayload`
    - `tags`: map of `IDENTIFIER` to list of `VAR_INT`

<a id="pkt-clientbound-minecraft-projectile_power"></a>
### minecraft:projectile_power (clientbound, id 138)

- `id`: `VAR_INT`
- `accelerationPower`: `DOUBLE`

<a id="pkt-clientbound-minecraft-custom_report_details"></a>
### minecraft:custom_report_details (clientbound, id 139)

- `details`: map of string (at most 128 characters) to string (at most 4096 characters) (at most 32)

<a id="pkt-clientbound-minecraft-server_links"></a>
### minecraft:server_links (clientbound, id 140)

- `links`: list of
  - struct `ServerLinks$UntrustedEntry`
    - `type`: either enum `ServerLinks$KnownLinkType` (var int, ordinal: BUG_REPORT, COMMUNITY_GUIDELINES, SUPPORT, STATUS, FEEDBACK, COMMUNITY, WEBSITE, FORUMS, NEWS, ANNOUNCEMENTS) or `TEXT`
    - `link`: `STRING`

<a id="pkt-clientbound-minecraft-waypoint"></a>
### minecraft:waypoint (clientbound, id 141)

- `operation`: enum `ClientboundTrackedWaypointPacket$Operation` (var int, ordinal: TRACK, UNTRACK, UPDATE)
- `waypoint`: struct `TrackedWaypoint`
  - `identifier`: either `UUID` or `STRING`
  - `icon`: struct `Waypoint$Icon`
    - `style`: resource key in root_id
    - `color`: optional
      - struct `RGBColor`
        - `red`: `UNSIGNED_BYTE`
        - `green`: `UNSIGNED_BYTE`
        - `blue`: `UNSIGNED_BYTE`
  - `type`: dispatch `TrackedWaypointPayload` on enum `TrackedWaypoint$Type` (var int, ordinal: EMPTY, VEC3I, CHUNK, AZIMUTH)
    - `EMPTY (0)`: nothing
    - `VEC3I (1)`: struct `TrackedWaypoint$Vec3iWaypoint`
      - `vector`: struct `Vec3i`
        - `x`: `VAR_INT`
        - `y`: `VAR_INT`
        - `z`: `VAR_INT`
    - `CHUNK (2)`: struct `TrackedWaypoint$ChunkWaypoint`
      - `chunkPos`: struct `ChunkPos`
        - `x`: `VAR_INT`
        - `z`: `VAR_INT`
    - `AZIMUTH (3)`: struct `TrackedWaypoint$AzimuthWaypoint`
      - `angle`: `FLOAT`

<a id="pkt-clientbound-minecraft-clear_dialog"></a>
### minecraft:clear_dialog (clientbound, id 142)

- nothing

<a id="pkt-clientbound-minecraft-show_dialog"></a>
### minecraft:show_dialog (clientbound, id 143)

- `dialog`: id in dialog or inline an NBT tag

## play serverbound

| id | packet | Java class |
|---:|---|---|
| 0 | [`minecraft:accept_teleportation`](#pkt-serverbound-minecraft-accept_teleportation) | `net.minecraft.network.protocol.game.ServerboundAcceptTeleportationPacket` |
| 1 | [`minecraft:attack`](#pkt-serverbound-minecraft-attack) | `net.minecraft.network.protocol.game.ServerboundAttackPacket` |
| 2 | [`minecraft:block_entity_tag_query`](#pkt-serverbound-minecraft-block_entity_tag_query) | `net.minecraft.network.protocol.game.ServerboundBlockEntityTagQueryPacket` |
| 3 | [`minecraft:bundle_item_selected`](#pkt-serverbound-minecraft-bundle_item_selected) | `net.minecraft.network.protocol.game.ServerboundSelectBundleItemPacket` |
| 4 | [`minecraft:change_difficulty`](#pkt-serverbound-minecraft-change_difficulty) | `net.minecraft.network.protocol.game.ServerboundChangeDifficultyPacket` |
| 5 | [`minecraft:change_game_mode`](#pkt-serverbound-minecraft-change_game_mode) | `net.minecraft.network.protocol.game.ServerboundChangeGameModePacket` |
| 6 | [`minecraft:chat_ack`](#pkt-serverbound-minecraft-chat_ack) | `net.minecraft.network.protocol.game.ServerboundChatAckPacket` |
| 7 | [`minecraft:chat_command`](#pkt-serverbound-minecraft-chat_command) | `net.minecraft.network.protocol.game.ServerboundChatCommandPacket` |
| 8 | [`minecraft:chat_command_signed`](#pkt-serverbound-minecraft-chat_command_signed) | `net.minecraft.network.protocol.game.ServerboundChatCommandSignedPacket` |
| 9 | [`minecraft:chat`](#pkt-serverbound-minecraft-chat) | `net.minecraft.network.protocol.game.ServerboundChatPacket` |
| 10 | [`minecraft:chat_session_update`](#pkt-serverbound-minecraft-chat_session_update) | `net.minecraft.network.protocol.game.ServerboundChatSessionUpdatePacket` |
| 11 | [`minecraft:chunk_batch_received`](#pkt-serverbound-minecraft-chunk_batch_received) | `net.minecraft.network.protocol.game.ServerboundChunkBatchReceivedPacket` |
| 12 | [`minecraft:client_command`](#pkt-serverbound-minecraft-client_command) | `net.minecraft.network.protocol.game.ServerboundClientCommandPacket` |
| 13 | [`minecraft:client_tick_end`](#pkt-serverbound-minecraft-client_tick_end) | `net.minecraft.network.protocol.game.ServerboundClientTickEndPacket` |
| 14 | [`minecraft:client_information`](#pkt-serverbound-minecraft-client_information) | `net.minecraft.network.protocol.common.ServerboundClientInformationPacket` |
| 15 | [`minecraft:command_suggestion`](#pkt-serverbound-minecraft-command_suggestion) | `net.minecraft.network.protocol.game.ServerboundCommandSuggestionPacket` |
| 16 | [`minecraft:configuration_acknowledged`](#pkt-serverbound-minecraft-configuration_acknowledged) | `net.minecraft.network.protocol.game.ServerboundConfigurationAcknowledgedPacket` |
| 17 | [`minecraft:container_button_click`](#pkt-serverbound-minecraft-container_button_click) | `net.minecraft.network.protocol.game.ServerboundContainerButtonClickPacket` |
| 18 | [`minecraft:container_click`](#pkt-serverbound-minecraft-container_click) | `net.minecraft.network.protocol.game.ServerboundContainerClickPacket` |
| 19 | [`minecraft:container_close`](#pkt-serverbound-minecraft-container_close) | `net.minecraft.network.protocol.game.ServerboundContainerClosePacket` |
| 20 | [`minecraft:container_slot_state_changed`](#pkt-serverbound-minecraft-container_slot_state_changed) | `net.minecraft.network.protocol.game.ServerboundContainerSlotStateChangedPacket` |
| 21 | [`minecraft:cookie_response`](#pkt-serverbound-minecraft-cookie_response) | `net.minecraft.network.protocol.cookie.ServerboundCookieResponsePacket` |
| 22 | [`minecraft:custom_payload`](#pkt-serverbound-minecraft-custom_payload) | `net.minecraft.network.protocol.common.ServerboundCustomPayloadPacket` |
| 23 | [`minecraft:debug_subscription_request`](#pkt-serverbound-minecraft-debug_subscription_request) | `net.minecraft.network.protocol.game.ServerboundDebugSubscriptionRequestPacket` |
| 24 | [`minecraft:edit_book`](#pkt-serverbound-minecraft-edit_book) | `net.minecraft.network.protocol.game.ServerboundEditBookPacket` |
| 25 | [`minecraft:entity_tag_query`](#pkt-serverbound-minecraft-entity_tag_query) | `net.minecraft.network.protocol.game.ServerboundEntityTagQueryPacket` |
| 26 | [`minecraft:interact`](#pkt-serverbound-minecraft-interact) | `net.minecraft.network.protocol.game.ServerboundInteractPacket` |
| 27 | [`minecraft:jigsaw_generate`](#pkt-serverbound-minecraft-jigsaw_generate) | `net.minecraft.network.protocol.game.ServerboundJigsawGeneratePacket` |
| 28 | [`minecraft:keep_alive`](#pkt-serverbound-minecraft-keep_alive) | `net.minecraft.network.protocol.common.ServerboundKeepAlivePacket` |
| 29 | [`minecraft:lock_difficulty`](#pkt-serverbound-minecraft-lock_difficulty) | `net.minecraft.network.protocol.game.ServerboundLockDifficultyPacket` |
| 30 | [`minecraft:move_player_pos`](#pkt-serverbound-minecraft-move_player_pos) | `net.minecraft.network.protocol.game.ServerboundMovePlayerPacket$Pos` |
| 31 | [`minecraft:move_player_pos_rot`](#pkt-serverbound-minecraft-move_player_pos_rot) | `net.minecraft.network.protocol.game.ServerboundMovePlayerPacket$PosRot` |
| 32 | [`minecraft:move_player_rot`](#pkt-serverbound-minecraft-move_player_rot) | `net.minecraft.network.protocol.game.ServerboundMovePlayerPacket$Rot` |
| 33 | [`minecraft:move_player_status_only`](#pkt-serverbound-minecraft-move_player_status_only) | `net.minecraft.network.protocol.game.ServerboundMovePlayerPacket$StatusOnly` |
| 34 | [`minecraft:move_vehicle`](#pkt-serverbound-minecraft-move_vehicle) | `net.minecraft.network.protocol.game.ServerboundMoveVehiclePacket` |
| 35 | [`minecraft:paddle_boat`](#pkt-serverbound-minecraft-paddle_boat) | `net.minecraft.network.protocol.game.ServerboundPaddleBoatPacket` |
| 36 | [`minecraft:pick_item_from_block`](#pkt-serverbound-minecraft-pick_item_from_block) | `net.minecraft.network.protocol.game.ServerboundPickItemFromBlockPacket` |
| 37 | [`minecraft:pick_item_from_entity`](#pkt-serverbound-minecraft-pick_item_from_entity) | `net.minecraft.network.protocol.game.ServerboundPickItemFromEntityPacket` |
| 38 | [`minecraft:ping_request`](#pkt-serverbound-minecraft-ping_request) | `net.minecraft.network.protocol.ping.ServerboundPingRequestPacket` |
| 39 | [`minecraft:place_recipe`](#pkt-serverbound-minecraft-place_recipe) | `net.minecraft.network.protocol.game.ServerboundPlaceRecipePacket` |
| 40 | [`minecraft:player_abilities`](#pkt-serverbound-minecraft-player_abilities) | `net.minecraft.network.protocol.game.ServerboundPlayerAbilitiesPacket` |
| 41 | [`minecraft:player_action`](#pkt-serverbound-minecraft-player_action) | `net.minecraft.network.protocol.game.ServerboundPlayerActionPacket` |
| 42 | [`minecraft:player_command`](#pkt-serverbound-minecraft-player_command) | `net.minecraft.network.protocol.game.ServerboundPlayerCommandPacket` |
| 43 | [`minecraft:player_input`](#pkt-serverbound-minecraft-player_input) | `net.minecraft.network.protocol.game.ServerboundPlayerInputPacket` |
| 44 | [`minecraft:player_loaded`](#pkt-serverbound-minecraft-player_loaded) | `net.minecraft.network.protocol.game.ServerboundPlayerLoadedPacket` |
| 45 | [`minecraft:pong`](#pkt-serverbound-minecraft-pong) | `net.minecraft.network.protocol.common.ServerboundPongPacket` |
| 46 | [`minecraft:punch`](#pkt-serverbound-minecraft-punch) | `net.minecraft.network.protocol.game.ServerboundPunchPacket` |
| 47 | [`minecraft:recipe_book_change_settings`](#pkt-serverbound-minecraft-recipe_book_change_settings) | `net.minecraft.network.protocol.game.ServerboundRecipeBookChangeSettingsPacket` |
| 48 | [`minecraft:recipe_book_seen_recipe`](#pkt-serverbound-minecraft-recipe_book_seen_recipe) | `net.minecraft.network.protocol.game.ServerboundRecipeBookSeenRecipePacket` |
| 49 | [`minecraft:rename_item`](#pkt-serverbound-minecraft-rename_item) | `net.minecraft.network.protocol.game.ServerboundRenameItemPacket` |
| 50 | [`minecraft:resource_pack`](#pkt-serverbound-minecraft-resource_pack) | `net.minecraft.network.protocol.common.ServerboundResourcePackPacket` |
| 51 | [`minecraft:seen_advancements`](#pkt-serverbound-minecraft-seen_advancements) | `net.minecraft.network.protocol.game.ServerboundSeenAdvancementsPacket` |
| 52 | [`minecraft:select_trade`](#pkt-serverbound-minecraft-select_trade) | `net.minecraft.network.protocol.game.ServerboundSelectTradePacket` |
| 53 | [`minecraft:set_beacon`](#pkt-serverbound-minecraft-set_beacon) | `net.minecraft.network.protocol.game.ServerboundSetBeaconPacket` |
| 54 | [`minecraft:set_carried_item`](#pkt-serverbound-minecraft-set_carried_item) | `net.minecraft.network.protocol.game.ServerboundSetCarriedItemPacket` |
| 55 | [`minecraft:set_command_block`](#pkt-serverbound-minecraft-set_command_block) | `net.minecraft.network.protocol.game.ServerboundSetCommandBlockPacket` |
| 56 | [`minecraft:set_command_minecart`](#pkt-serverbound-minecraft-set_command_minecart) | `net.minecraft.network.protocol.game.ServerboundSetCommandMinecartPacket` |
| 57 | [`minecraft:set_creative_mode_slot`](#pkt-serverbound-minecraft-set_creative_mode_slot) | `net.minecraft.network.protocol.game.ServerboundSetCreativeModeSlotPacket` |
| 58 | [`minecraft:set_game_rule`](#pkt-serverbound-minecraft-set_game_rule) | `net.minecraft.network.protocol.game.ServerboundSetGameRulePacket` |
| 59 | [`minecraft:set_jigsaw_block`](#pkt-serverbound-minecraft-set_jigsaw_block) | `net.minecraft.network.protocol.game.ServerboundSetJigsawBlockPacket` |
| 60 | [`minecraft:set_structure_block`](#pkt-serverbound-minecraft-set_structure_block) | `net.minecraft.network.protocol.game.ServerboundSetStructureBlockPacket` |
| 61 | [`minecraft:set_test_block`](#pkt-serverbound-minecraft-set_test_block) | `net.minecraft.network.protocol.game.ServerboundSetTestBlockPacket` |
| 62 | [`minecraft:sign_update`](#pkt-serverbound-minecraft-sign_update) | `net.minecraft.network.protocol.game.ServerboundSignUpdatePacket` |
| 63 | [`minecraft:spectator_action`](#pkt-serverbound-minecraft-spectator_action) | `net.minecraft.network.protocol.game.ServerboundSpectatorActionPacket` |
| 64 | [`minecraft:teleport_to_entity`](#pkt-serverbound-minecraft-teleport_to_entity) | `net.minecraft.network.protocol.game.ServerboundTeleportToEntityPacket` |
| 65 | [`minecraft:test_instance_block_action`](#pkt-serverbound-minecraft-test_instance_block_action) | `net.minecraft.network.protocol.game.ServerboundTestInstanceBlockActionPacket` |
| 66 | [`minecraft:use_item_on`](#pkt-serverbound-minecraft-use_item_on) | `net.minecraft.network.protocol.game.ServerboundUseItemOnPacket` |
| 67 | [`minecraft:use_item`](#pkt-serverbound-minecraft-use_item) | `net.minecraft.network.protocol.game.ServerboundUseItemPacket` |
| 68 | [`minecraft:custom_click_action`](#pkt-serverbound-minecraft-custom_click_action) | `net.minecraft.network.protocol.common.ServerboundCustomClickActionPacket` |

<a id="pkt-serverbound-minecraft-accept_teleportation"></a>
### minecraft:accept_teleportation (serverbound, id 0)

- `id`: `VAR_INT`
- `x`: `DOUBLE`
- `y`: `DOUBLE`
- `z`: `DOUBLE`
- `yRot`: `FLOAT`
- `xRot`: `FLOAT`

<a id="pkt-serverbound-minecraft-attack"></a>
### minecraft:attack (serverbound, id 1)

- `entityId`: `VAR_INT`

<a id="pkt-serverbound-minecraft-block_entity_tag_query"></a>
### minecraft:block_entity_tag_query (serverbound, id 2)

- `transactionId`: `VAR_INT`
- `pos`: `BLOCK_POS`

<a id="pkt-serverbound-minecraft-bundle_item_selected"></a>
### minecraft:bundle_item_selected (serverbound, id 3)

- `slotId`: `VAR_INT`
- `selectedItemIndex`: `VAR_INT`

<a id="pkt-serverbound-minecraft-change_difficulty"></a>
### minecraft:change_difficulty (serverbound, id 4)

- `difficulty`: enum `Difficulty` (var int, ordinal: PEACEFUL, EASY, NORMAL, HARD)

<a id="pkt-serverbound-minecraft-change_game_mode"></a>
### minecraft:change_game_mode (serverbound, id 5)

- `mode`: enum `GameType` (var int, ordinal: SURVIVAL, CREATIVE, ADVENTURE, SPECTATOR)

<a id="pkt-serverbound-minecraft-chat_ack"></a>
### minecraft:chat_ack (serverbound, id 6)

- `offset`: `VAR_INT`

<a id="pkt-serverbound-minecraft-chat_command"></a>
### minecraft:chat_command (serverbound, id 7)

- `command`: string

<a id="pkt-serverbound-minecraft-chat_command_signed"></a>
### minecraft:chat_command_signed (serverbound, id 8)

- `command`: `STRING`
- `timeStamp`: `INSTANT`
- `salt`: `LONG`
- `argumentSignatures`: struct `ArgumentSignatures`
  - `entries`: list of (at most 8)
    - struct `ArgumentSignatures$Entry`
      - `name`: string (at most 16 characters)
      - `signature`: `MESSAGE_SIGNATURE`
- `lastSeenMessages`: struct `LastSeenMessages$Update`
  - `offset`: `VAR_INT`
  - `acknowledged`: `FIXED_BIT_SET` (bits=20)
  - `checksum`: `BYTE`

<a id="pkt-serverbound-minecraft-chat"></a>
### minecraft:chat (serverbound, id 9)

- `message`: string (at most 256 characters)
- `timeStamp`: `INSTANT`
- `salt`: `LONG`
- `signature`: optional `MESSAGE_SIGNATURE`
- `lastSeenMessages`: struct `LastSeenMessages$Update`
  - `offset`: `VAR_INT`
  - `acknowledged`: `FIXED_BIT_SET` (bits=20)
  - `checksum`: `BYTE`

<a id="pkt-serverbound-minecraft-chat_session_update"></a>
### minecraft:chat_session_update (serverbound, id 10)

- `sessionId`: `UUID`
- `profilePublicKey`: struct `ProfilePublicKey$Data`
  - `expiresAt`: `INSTANT`
  - `key`: `PUBLIC_KEY`
  - `keySignature`: `BYTE_ARRAY` (max=4096)

<a id="pkt-serverbound-minecraft-chunk_batch_received"></a>
### minecraft:chunk_batch_received (serverbound, id 11)

- `desiredChunksPerTick`: `FLOAT`

<a id="pkt-serverbound-minecraft-client_command"></a>
### minecraft:client_command (serverbound, id 12)

- `action`: enum `ServerboundClientCommandPacket$Action` (var int, ordinal: PERFORM_RESPAWN, REQUEST_STATS, REQUEST_GAMERULE_VALUES)

<a id="pkt-serverbound-minecraft-client_tick_end"></a>
### minecraft:client_tick_end (serverbound, id 13)

- nothing

<a id="pkt-serverbound-minecraft-client_information"></a>
### minecraft:client_information (serverbound, id 14)

- `information`: struct `ClientInformation`
  - `language`: string (at most 16 characters)
  - `viewDistance`: `BYTE`
  - `chatVisibility`: enum `ChatVisiblity` (var int, ordinal: FULL, SYSTEM, HIDDEN)
  - `chatColors`: `BOOL`
  - `modelCustomisation`: `UNSIGNED_BYTE`
  - `mainHand`: enum `HumanoidArm` (var int, ordinal: LEFT, RIGHT)
  - `textFilteringEnabled`: `BOOL`
  - `allowsListing`: `BOOL`
  - `particleStatus`: enum `ParticleStatus` (var int, ordinal: ALL, DECREASED, MINIMAL)

<a id="pkt-serverbound-minecraft-command_suggestion"></a>
### minecraft:command_suggestion (serverbound, id 15)

- `id`: `VAR_INT`
- `command`: string (at most 32500 characters)

<a id="pkt-serverbound-minecraft-configuration_acknowledged"></a>
### minecraft:configuration_acknowledged (serverbound, id 16)

- nothing

<a id="pkt-serverbound-minecraft-container_button_click"></a>
### minecraft:container_button_click (serverbound, id 17)

- `containerId`: `CONTAINER_ID`
- `buttonId`: `VAR_INT`

<a id="pkt-serverbound-minecraft-container_click"></a>
### minecraft:container_click (serverbound, id 18)

- `containerId`: `CONTAINER_ID`
- `stateId`: `VAR_INT`
- `slotNum`: `SHORT`
- `buttonNum`: `BYTE`
- `containerInput`: enum `ContainerInput` (var int, ordinal: PICKUP, QUICK_MOVE, SWAP, CLONE, THROW, QUICK_CRAFT, PICKUP_ALL)
- `changedSlots`: map (at most 128)
  - key: `SHORT`
  - value: optional
    - struct `HashedStack$ActualItem`
      - `item`: id in item
      - `count`: `VAR_INT`
      - `components`: struct `HashedPatchMap`
        - `addedComponents`: map of id in data_component_type to `INT` (at most 256)
        - `removedComponents`: list of id in data_component_type (at most 256)
- `carriedItem`: optional
  - struct `HashedStack$ActualItem`
    - `item`: id in item
    - `count`: `VAR_INT`
    - `components`: struct `HashedPatchMap`
      - `addedComponents`: map of id in data_component_type to `INT` (at most 256)
      - `removedComponents`: list of id in data_component_type (at most 256)

<a id="pkt-serverbound-minecraft-container_close"></a>
### minecraft:container_close (serverbound, id 19)

- `containerId`: `CONTAINER_ID`

<a id="pkt-serverbound-minecraft-container_slot_state_changed"></a>
### minecraft:container_slot_state_changed (serverbound, id 20)

- `slotId`: `VAR_INT`
- `containerId`: `CONTAINER_ID`
- `newState`: `BOOL`

<a id="pkt-serverbound-minecraft-cookie_response"></a>
### minecraft:cookie_response (serverbound, id 21)

- `key`: `IDENTIFIER`
- `payload`: optional `BYTE_ARRAY` (max=5120)

<a id="pkt-serverbound-minecraft-custom_payload"></a>
### minecraft:custom_payload (serverbound, id 22)

- `channel`: `IDENTIFIER`
- `data`: `REST_BYTES`

<a id="pkt-serverbound-minecraft-debug_subscription_request"></a>
### minecraft:debug_subscription_request (serverbound, id 23)

- id in debug_subscription

<a id="pkt-serverbound-minecraft-edit_book"></a>
### minecraft:edit_book (serverbound, id 24)

- `slot`: `VAR_INT`
- `pages`: list of string (at most 1024 characters) (at most 100)
- `title`: optional string (at most 32 characters)

<a id="pkt-serverbound-minecraft-entity_tag_query"></a>
### minecraft:entity_tag_query (serverbound, id 25)

- `transactionId`: `VAR_INT`
- `entityId`: `VAR_INT`

<a id="pkt-serverbound-minecraft-interact"></a>
### minecraft:interact (serverbound, id 26)

- `entityId`: `VAR_INT`
- `hand`: enum `InteractionHand` (var int, ordinal: MAIN_HAND, OFF_HAND)
- `location`: `LP_VEC3`
- `usingSecondaryAction`: `BOOL`

<a id="pkt-serverbound-minecraft-jigsaw_generate"></a>
### minecraft:jigsaw_generate (serverbound, id 27)

- `pos`: `BLOCK_POS`
- `levels`: `VAR_INT`
- `keepJigsaws`: `BOOL`

<a id="pkt-serverbound-minecraft-keep_alive"></a>
### minecraft:keep_alive (serverbound, id 28)

- `id`: `LONG`

<a id="pkt-serverbound-minecraft-lock_difficulty"></a>
### minecraft:lock_difficulty (serverbound, id 29)

- `locked`: `BOOL`

<a id="pkt-serverbound-minecraft-move_player_pos"></a>
### minecraft:move_player_pos (serverbound, id 30)

- `x`: `DOUBLE`
- `y`: `DOUBLE`
- `z`: `DOUBLE`
- `flags`: `UNSIGNED_BYTE` as bits (onGround@0:1, horizontalCollision@1:1)

<a id="pkt-serverbound-minecraft-move_player_pos_rot"></a>
### minecraft:move_player_pos_rot (serverbound, id 31)

- `x`: `DOUBLE`
- `y`: `DOUBLE`
- `z`: `DOUBLE`
- `yRot`: `FLOAT`
- `xRot`: `FLOAT`
- `flags`: `UNSIGNED_BYTE` as bits (onGround@0:1, horizontalCollision@1:1)

<a id="pkt-serverbound-minecraft-move_player_rot"></a>
### minecraft:move_player_rot (serverbound, id 32)

- `yRot`: `FLOAT`
- `xRot`: `FLOAT`
- `flags`: `UNSIGNED_BYTE` as bits (onGround@0:1, horizontalCollision@1:1)

<a id="pkt-serverbound-minecraft-move_player_status_only"></a>
### minecraft:move_player_status_only (serverbound, id 33)

- `UNSIGNED_BYTE` as bits (onGround@0:1, horizontalCollision@1:1)

<a id="pkt-serverbound-minecraft-move_vehicle"></a>
### minecraft:move_vehicle (serverbound, id 34)

- `movingTo`: struct `PositionAndRotation`
  - `position`: struct `Vec3`
    - `x`: `DOUBLE`
    - `y`: `DOUBLE`
    - `z`: `DOUBLE`
  - `yRot`: `FLOAT`
  - `xRot`: `FLOAT`
- `onGround`: `BOOL`

<a id="pkt-serverbound-minecraft-paddle_boat"></a>
### minecraft:paddle_boat (serverbound, id 35)

- `left`: `BOOL`
- `right`: `BOOL`

<a id="pkt-serverbound-minecraft-pick_item_from_block"></a>
### minecraft:pick_item_from_block (serverbound, id 36)

- `pos`: `BLOCK_POS`
- `includeData`: `BOOL`

<a id="pkt-serverbound-minecraft-pick_item_from_entity"></a>
### minecraft:pick_item_from_entity (serverbound, id 37)

- `id`: `VAR_INT`
- `includeData`: `BOOL`

<a id="pkt-serverbound-minecraft-ping_request"></a>
### minecraft:ping_request (serverbound, id 38)

- `time`: `LONG`

<a id="pkt-serverbound-minecraft-place_recipe"></a>
### minecraft:place_recipe (serverbound, id 39)

- `containerId`: `CONTAINER_ID`
- `recipe`: struct `RecipeDisplayId`
  - `index`: `VAR_INT`
- `useMaxItems`: `BOOL`

<a id="pkt-serverbound-minecraft-player_abilities"></a>
### minecraft:player_abilities (serverbound, id 40)

- `input`: `BYTE`

<a id="pkt-serverbound-minecraft-player_action"></a>
### minecraft:player_action (serverbound, id 41)

- `action`: enum `ServerboundPlayerActionPacket$Action` (var int, ordinal: START_DESTROY_BLOCK, CHANGE_DESTROY_DIRECTION, ABORT_DESTROY_BLOCK, STOP_DESTROY_BLOCK, DROP_ALL_ITEMS, DROP_ITEM, RELEASE_USE_ITEM, SWAP_ITEM_WITH_OFFHAND, STAB)
- `pos`: `BLOCK_POS`
- `direction`: `UNSIGNED_BYTE`
- `sequence`: `VAR_INT`

<a id="pkt-serverbound-minecraft-player_command"></a>
### minecraft:player_command (serverbound, id 42)

- `id`: `VAR_INT`
- `action`: enum `ServerboundPlayerCommandPacket$Action` (var int, ordinal: STOP_SLEEPING, START_SPRINTING, STOP_SPRINTING, START_RIDING_JUMP, STOP_RIDING_JUMP, OPEN_INVENTORY, START_FALL_FLYING)
- `data`: `VAR_INT`

<a id="pkt-serverbound-minecraft-player_input"></a>
### minecraft:player_input (serverbound, id 43)

- `input`: `BYTE`

<a id="pkt-serverbound-minecraft-player_loaded"></a>
### minecraft:player_loaded (serverbound, id 44)

- nothing

<a id="pkt-serverbound-minecraft-pong"></a>
### minecraft:pong (serverbound, id 45)

- `id`: `INT`

<a id="pkt-serverbound-minecraft-punch"></a>
### minecraft:punch (serverbound, id 46)

- nothing

<a id="pkt-serverbound-minecraft-recipe_book_change_settings"></a>
### minecraft:recipe_book_change_settings (serverbound, id 47)

- `bookType`: enum `RecipeBookType` (var int, ordinal: CRAFTING, FURNACE, BLAST_FURNACE, SMOKER)
- `isOpen`: `BOOL`
- `isFiltering`: `BOOL`

<a id="pkt-serverbound-minecraft-recipe_book_seen_recipe"></a>
### minecraft:recipe_book_seen_recipe (serverbound, id 48)

- `recipe`: struct `RecipeDisplayId`
  - `index`: `VAR_INT`

<a id="pkt-serverbound-minecraft-rename_item"></a>
### minecraft:rename_item (serverbound, id 49)

- `name`: string

<a id="pkt-serverbound-minecraft-resource_pack"></a>
### minecraft:resource_pack (serverbound, id 50)

- `id`: `UUID`
- `action`: enum `ServerboundResourcePackPacket$Action` (var int, ordinal: SUCCESSFULLY_LOADED, DECLINED, FAILED_DOWNLOAD, ACCEPTED, DOWNLOADED, INVALID_URL, FAILED_RELOAD, DISCARDED)

<a id="pkt-serverbound-minecraft-seen_advancements"></a>
### minecraft:seen_advancements (serverbound, id 51)

- `action`: enum `ServerboundSeenAdvancementsPacket$Action` (var int, ordinal: OPENED_TAB, CLOSED_SCREEN)
- `tab`: `IDENTIFIER` (when `action` == OPENED_TAB)

<a id="pkt-serverbound-minecraft-select_trade"></a>
### minecraft:select_trade (serverbound, id 52)

- `item`: `VAR_INT`

<a id="pkt-serverbound-minecraft-set_beacon"></a>
### minecraft:set_beacon (serverbound, id 53)

- `primary`: optional id in mob_effect
- `secondary`: optional id in mob_effect

<a id="pkt-serverbound-minecraft-set_carried_item"></a>
### minecraft:set_carried_item (serverbound, id 54)

- `slot`: `SHORT`

<a id="pkt-serverbound-minecraft-set_command_block"></a>
### minecraft:set_command_block (serverbound, id 55)

- `pos`: `BLOCK_POS`
- `command`: string
- `mode`: enum `CommandBlockEntity$Mode` (var int, ordinal: SEQUENCE, AUTO, REDSTONE)
- `flags`: `BYTE`

<a id="pkt-serverbound-minecraft-set_command_minecart"></a>
### minecraft:set_command_minecart (serverbound, id 56)

- `entity`: `VAR_INT`
- `command`: string
- `trackOutput`: `BOOL`

<a id="pkt-serverbound-minecraft-set_creative_mode_slot"></a>
### minecraft:set_creative_mode_slot (serverbound, id 57)

- `slotNum`: `SHORT`
- `itemStack`: `UNTRUSTED_ITEM_STACK`

<a id="pkt-serverbound-minecraft-set_game_rule"></a>
### minecraft:set_game_rule (serverbound, id 58)

- `entries`: list of
  - struct `ServerboundSetGameRulePacket$Entry`
    - `gameRuleKey`: resource key in game_rule
    - `value`: `STRING`

<a id="pkt-serverbound-minecraft-set_jigsaw_block"></a>
### minecraft:set_jigsaw_block (serverbound, id 59)

- `pos`: `BLOCK_POS`
- `name`: `IDENTIFIER`
- `target`: `IDENTIFIER`
- `pool`: `IDENTIFIER`
- `finalState`: string
- `joint`: string enum `JigsawBlockEntity$JointType` (rollable, aligned)
- `selectionPriority`: `VAR_INT`
- `placementPriority`: `VAR_INT`

<a id="pkt-serverbound-minecraft-set_structure_block"></a>
### minecraft:set_structure_block (serverbound, id 60)

- `pos`: `BLOCK_POS`
- `updateType`: enum `StructureBlockEntity$UpdateType` (var int, ordinal: UPDATE_DATA, SAVE_AREA, LOAD_AREA, SCAN_AREA)
- `mode`: enum `StructureMode` (var int, ordinal: SAVE, LOAD, CORNER, DATA)
- `name`: string
- `offset`: struct `BlockPos`
  - `x`: `BYTE`
  - `y`: `BYTE`
  - `z`: `BYTE`
- `size`: struct `Vec3i`
  - `x`: `BYTE`
  - `y`: `BYTE`
  - `z`: `BYTE`
- `mirror`: enum `Mirror` (var int, ordinal: NONE, LEFT_RIGHT, FRONT_BACK)
- `rotation`: enum `Rotation` (var int, ordinal: NONE, CLOCKWISE_90, CLOCKWISE_180, COUNTERCLOCKWISE_90)
- `data`: string (at most 128 characters)
- `integrity`: `FLOAT`
- `seed`: `VAR_LONG`
- `flags`: `BYTE`

<a id="pkt-serverbound-minecraft-set_test_block"></a>
### minecraft:set_test_block (serverbound, id 61)

- `position`: `BLOCK_POS`
- `mode`: enum `TestBlockMode` (var int, ordinal: START, LOG, FAIL, ACCEPT)
- `message`: `STRING`

<a id="pkt-serverbound-minecraft-sign_update"></a>
### minecraft:sign_update (serverbound, id 62)

- `pos`: `BLOCK_POS`
- `lines`: string (at most 384 characters)
- `slot`: enum `SignTextSlot` (var int, ordinal: BACK, FRONT)

<a id="pkt-serverbound-minecraft-spectator_action"></a>
### minecraft:spectator_action (serverbound, id 63)

- `spectateEntityId`: `OPTIONAL_VAR_INT`

<a id="pkt-serverbound-minecraft-teleport_to_entity"></a>
### minecraft:teleport_to_entity (serverbound, id 64)

- `uuid`: `UUID`

<a id="pkt-serverbound-minecraft-test_instance_block_action"></a>
### minecraft:test_instance_block_action (serverbound, id 65)

- `pos`: `BLOCK_POS`
- `action`: enum `ServerboundTestInstanceBlockActionPacket$Action` (var int, ordinal: INIT, QUERY, SET, RESET, SAVE, EXPORT, RUN)
- `data`: struct `TestInstanceBlockEntity$Data`
  - `test`: optional resource key in test_instance
  - `size`: struct `Vec3i`
    - `getX`: `VAR_INT`
    - `getY`: `VAR_INT`
    - `getZ`: `VAR_INT`
  - `rotation`: enum `Rotation` (var int, ordinal: NONE, CLOCKWISE_90, CLOCKWISE_180, COUNTERCLOCKWISE_90)
  - `ignoreEntities`: `BOOL`
  - `status`: enum `TestInstanceBlockEntity$Status` (var int, ordinal: CLEARED, RUNNING, FINISHED)
  - `errorMessage`: optional `TEXT`

<a id="pkt-serverbound-minecraft-use_item_on"></a>
### minecraft:use_item_on (serverbound, id 66)

- `hand`: enum `InteractionHand` (var int, ordinal: MAIN_HAND, OFF_HAND)
- `hitResult`: struct `BlockHitResult`
  - `pos`: `BLOCK_POS`
  - `direction`: enum `Direction` (var int, ordinal: DOWN, UP, NORTH, SOUTH, WEST, EAST)
  - `location`: `FLOAT`
  - `x`: `FLOAT`
  - `z`: `FLOAT`
  - `inside`: `BOOL`
  - `worldBorderHit`: `BOOL`
- `sequence`: `VAR_INT`

<a id="pkt-serverbound-minecraft-use_item"></a>
### minecraft:use_item (serverbound, id 67)

- `hand`: enum `InteractionHand` (var int, ordinal: MAIN_HAND, OFF_HAND)
- `sequence`: `VAR_INT`
- `yRot`: `FLOAT`
- `xRot`: `FLOAT`

<a id="pkt-serverbound-minecraft-custom_click_action"></a>
### minecraft:custom_click_action (serverbound, id 68)

- `id`: `IDENTIFIER`
- `payload`: length-prefixed an NBT tag

