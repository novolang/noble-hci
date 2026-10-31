# noble-hci

The HCI seam of the noble Bluetooth Low Energy stack: what a Host and a
Controller exchange over H4, UART, USB or an in-RAM link, written and read
as bytes.  The package holds opcode and event constants, one builder per
command, one reader per event field, the H4 frame codec, and the `BdAddr`
and `Uuid` types.  It has no state, no effects and no limits, and it
depends on the standard library only.

| Module | Holds |
|---|---|
| `hci_commands` | one `opcode_<name>()` per command and one `build_<name>(...)` for every command that takes parameters (Core Vol 4 Part E, section 7) |
| `hci_frame` | OGF and event-code constants, `pack_opcode`, the ACL handle and flag fields, the `HciCmd`, `HciEvt` and `HciAcl` frames, `HciCmdMsg` and `HciEvtMsg` with their encoders and decoders |
| `hci_codec` | the H4-framed command builders (`cmd_hci_reset`, `cmd_le_set_advertising_data`, ...), the readers of events and ISO and ACL data packets, and `build_iso_data_packet` |
| `hci_h4` | the H4 packet-type byte, `frame_h4`, and `h4_packet_length`, which tells a deframer how many bytes the next packet occupies |
| `bd_addr` | `BdAddr`, `BdAddrKind`, `classify_random` and `format_bd_addr` |
| `uuid` | the Bluetooth Base UUID, 16-bit and 32-bit expansion to 128 bits, comparison without allocating, and the little-endian wire form |

A reader of an event returns `?T` where the event can be malformed: a
buffer that is too short answers `None`, never a partial value.  The
scalar readers (`event_code`, `event_param_len`, `uuid16_from_le_bytes`)
answer `-1` for a buffer too short to hold the field.  A deframer calls
`h4_packet_length` on the bytes it has: `H4_BYTES_NEED_MORE` (-1) means
wait for more, `H4_BYTES_REJECT` (-2) means the type byte is not one the
LE system uses.

```novo
use hci_codec
use hci_h4

// The H4 frame of HCI_Reset is 01 03 0C 00, and its length is the length the deframer reads.
fn reset_frame_is_whole() -> Bool
    let w = hci_codec.cmd_hci_reset()
    hci_h4.h4_packet_length(w) == list.len(w)
```

The Host (`orbit/ble`) and the Controller both depend on this package, so
the two sides share one definition of the wire.  The commands and events
are the subset the stack uses; a command with no entry is not built here.
`docs/hci.md` describes the byte layouts.

`novo test` in this directory runs the unit tests beside each module and
the edge tests in `tests/edge_tests.nv`.
