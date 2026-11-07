# Changelog

Every published version, newest first. This file is on the publish
allow-list, so it travels with the package: it is the only thing a
consumer deciding whether to upgrade can read.

## Unreleased

`hci_cs`: the Host side of the HCI of Channel Sounding.  The eighteen commands (`cs_read_local_supported_capabilities` to `cs_set_default_security_requirements`), each returning an `HciCsCmd`, a fixed buffer of 256 octets with its opcode; the nine events (`cs_read_remote_capabilities`, `cs_read_remote_fae`, `cs_read_security_enable_complete`, `cs_read_config_complete`, `cs_read_procedure_enable_complete`, `cs_read_test_end_complete`, `cs_read_result` for the Subevent Result and the Continue event) as readers into records; the capability record `HciCsCaps` in its [v1] and [v2] forms; `HciCsStep`, the data of a step of mode 0 to 3 for either role, read from the length it carries; `HciCsProcResult` with `cs_proc_add`, the result of a procedure joined from its events, at most 32 subevents and 256 steps.  The types carry the prefix `HciCs` since the Controller's package has types named `CsCaps` and `CsStep`.

## 0.1.0

First release.  The HCI command and event codecs, the H4 frame codec,
`BdAddr` and `Uuid` of the noble Bluetooth stack, as one package that the
Host and the Controller both depend on.  Stateless, no effects, no limits;
depends on the standard library only.
