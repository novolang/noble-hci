# Commands, events and frames

The wire formats `noble-hci` reads and writes, as the Core Specification
defines them (Vol 4 Part A and Part E).  Every multi-byte field is
little-endian.

## Opcodes

An opcode is 16 bits: the OGF in the top six, the OCF in the low ten.
`pack_opcode(ogf, ocf)` builds one and `opcode_ogf` and `opcode_ocf` split
it.  `hci_commands` has one `opcode_<name>()` per command, so
`opcode_hci_reset()` is `0x0C03` (OGF `0x03`, OCF `0x003`).

## Commands

A command packet is the opcode (2 octets), the parameter length (1 octet)
and the parameters.  `build_<name>(...)` returns the parameter octets of a
command; `hci_codec.build_command(opcode, params)` adds the header, and
`build_h4_command` adds the H4 type byte as well.  `cmd_<name>(...)` in
`hci_codec` does all three for the commands a Host sends most.

## Events

An event packet is the event code (1 octet), the parameter length (1 octet)
and the parameters.  `event_code` and `event_param_len` read the header.
`is_command_complete` and `is_command_status` recognise the two events that
answer a command, and `answered_opcode` and `answered_status` read the
opcode and status out of either.  LE events arrive inside the LE Meta Event
(`0x3E`); `le_meta_subevent` reads the subevent code, and one reader per
subevent returns its fields (`le_connection_complete`,
`le_advertising_reports`, `le_cis_established`, ...).  A reader returns
`None` for a buffer shorter than the event it names.

## Data packets

An ACL packet carries a 12-bit connection handle, a 2-bit packet-boundary
flag, a 2-bit broadcast flag and a 16-bit length.  An ISO packet has the same
header with a 14-bit length; the two bits above it are reserved and are
ignored when a length is read.  `pack_acl_handle_flags` builds the 16-bit
handle-and-flags field and `acl_handle`, `acl_pb_flag` and `acl_bc_flag`
read it.  `acl_data_packet` and `iso_data_packet` read a whole packet;
`build_iso_data_packet` builds one.

## H4

H4 prefixes each packet with one type byte: `0x01` command, `0x02` ACL,
`0x04` event, `0x05` ISO (`0x03`, SCO, is not used by LE).  `frame_h4`
prepends the byte.  `h4_packet_length(buf)` returns the size of the packet
at the start of `buf`, the type byte included: `4 + length` for a command,
`3 + length` for an event, `5 + length` for ACL and ISO.  It returns
`H4_BYTES_NEED_MORE` when the header is not complete and `H4_BYTES_REJECT`
for a type byte that is not one of the four.

## Addresses and UUIDs

A `BdAddr` is a 6-octet array in storage order (octet 0 is the least
significant) and a `BdAddrKind`.  For a random address the two most
significant bits of octet 5 select the kind: `11` static, `01` resolvable
private, `00` non-resolvable private.  `10` is reserved and
`classify_random` answers `NonResolvable` for it.

A 16-bit or 32-bit UUID is an alias for a 128-bit one: the Bluetooth Base
UUID `00000000-0000-1000-8000-00805F9B34FB` with the short value in octets
12 to 15.  `uuid16_to_128` and `uuid32_to_128` expand one, and
`uuid_eq_short_full` compares a short UUID with a 128-bit one without
building the expansion.
