# Changelog

Every published version, newest first. This file is on the publish
allow-list, so it travels with the package: it is the only thing a
consumer deciding whether to upgrade can read.

## 0.1.1 — 2026-11-10

The package passes the embedded tier in full: `novo pkg build` reports no tier finding, where 0.1.0 reported 165 (the `sibling_check = "warn"` knob is removed from `novo.toml`, so a finding returns as an error).  No public signature changed and no wire byte changed.

- **Builders and framers.**  Every `build_*` of `hci_commands`, `build_command`, `build_h4_command`, `build_h4_acl`, `build_iso_data_packet`, `encode_hci_cmd_msg`, `encode_hci_evt_msg`, `frame_h4` and the UUID expansions write into one list sized from the layout of the packet they build (`list.with_capacity` and `list.try_push`, the bounded-buffer pair of the embedded tier).  A builder asks the arena for one block of the length it returns; before, each one built a literal and grew it with `list.append`.  `build_h4_command` no longer builds the packet and then copies it behind the type byte.
- **`hci_buf`.**  A new module with the writers the builders share: `buf`, `empty`, `put8`, `put16`, `put12` (a Connection_Handle), `put24`, `put32`, `put64`, `put_zeros`, `put_bytes`, `put_range`, `range_len` and `slice`.  `put_range` and `slice` cut to the source as the slice `src[lo:hi]` does, so a short address still gives a short parameter block.  A consumer may use them to build its own packets; they are not HCI vocabulary.
- **Records.**  `DisconnectionComplete`, `CompletedPackets`, `LeConnectionUpdateComplete`, `LeRemoteConnectionParameterRequest`, `EncryptionChange`, `LeBufferSize`, `PeriodicSyncLost` and `PawrSubeventRequest` are `@value` structs: their readers allocate nothing, and a record is copied where it is passed.  A record with a payload list (`AdvertisingReport`, `LeConnectionComplete`, `AclDataPacket`, `LeLongTermKeyRequest`, `CigParametersReturn`, `CreateBigComplete`, `BigSyncEstablished`, `IsoDataPacket`, `PeriodicSyncEstablished`, `PeriodicAdvertisingReport`, `PeriodicSyncTransferReceived`, `PawrResponse`, `PawrResponseReport`, `HciCmdMsg`, `HciEvtMsg` and `BdAddr`) stays a boxed struct, since a `@value` struct cannot hold a list (E2011); its reader builds it in a `system` block, which is the one allocation of the call.  `format_bd_addr` and `hex_byte` build their text in a `system` block as well.
- **Removed.**  The list helpers `hci_codec.le16_bytes` and `hci_commands.le16_octets`, `le32_octets`, `le24` and `iso_handle_octets` were not public and no package of the repository used them; `hci_buf.put16`, `put32`, `put24` and `put12` write the same octets into a buffer.  Migration for a caller outside the tree: `var b = hci_buf.buf(2)` then `hci_buf.put16(var b, v)` in place of `le16_octets(v)`.

`hci_cs`: the Host side of the HCI of Channel Sounding.  The eighteen commands (`cs_read_local_supported_capabilities` to `cs_set_default_security_requirements`), each returning an `HciCsCmd`, a fixed buffer of 256 octets with its opcode; the nine events (`cs_read_remote_capabilities`, `cs_read_remote_fae`, `cs_read_security_enable_complete`, `cs_read_config_complete`, `cs_read_procedure_enable_complete`, `cs_read_test_end_complete`, `cs_read_result` for the Subevent Result and the Continue event) as readers into records; the capability record `HciCsCaps` in its [v1] and [v2] forms; `HciCsStep`, the data of a step of mode 0 to 3 for either role, read from the length it carries; `HciCsProcResult` with `cs_proc_add`, the result of a procedure joined from its events, at most 32 subevents and 256 steps.  The types carry the prefix `HciCs` since the Controller's package has types named `CsCaps` and `CsStep`.

## 0.1.0

First release.  The HCI command and event codecs, the H4 frame codec,
`BdAddr` and `Uuid` of the noble Bluetooth stack, as one package that the
Host and the Controller both depend on.  Stateless, no effects, no limits;
depends on the standard library only.
