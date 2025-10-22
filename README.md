# Build Status

| Source       | `main` Status | `main` Timestamp |
|--------------|--------------|------------|
| **CISAGOV** | ![Build Status](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2Fcisagov%2Ficsnpp-s7comm%2Fcommits%2Fmain%2Fstatus&query=state&label=build) | ![Last Build](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2Fcisagov%2Ficsnpp-s7comm%2Fcommits%2Fmain%2Fstatus&query=statuses[0].updated_at&label=last%20build&color=lightgrey) |
| **Development** | ![Build Status](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2Fparsitects%2Ficsnpp-s7comm%2Fcommits%2Fmain%2Fstatus&query=state&label=build) | ![Last Build](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fapi.github.com%2Frepos%2Fparsitects%2Ficsnpp-s7comm%2Fcommits%2Fmain%2Fstatus&query=statuses[0].updated_at&label=last%20build&color=lightgrey) |

# ICSNPP-S7COMM

Industrial Control Systems Network Protocol Parsers (ICSNPP) - s7comm, s7comm-plus, and COTP.

## Overview

ICSNPP-S7COMM is a Zeek plugin for parsing and logging fields within the s7comm, s7comm-plus, and COTP protocols.

This plugin was developed to be fully customizable. To drill down into specific packets and log certain variables, users can add the logging functionality to [scripts/icsnpp/s7comm/main.zeek](scripts/icsnpp/s7comm/main.zeek). The functions within [scripts/icsnpp/s7comm/main.zeek](scripts/icsnpp/s7comm/main.zeek) and [src/events.bif](src/events.bif) are good guides for adding new logging functionality.

This parser currently produces five log files. These log files are defined in [scripts/icsnpp/s7comm/main.zeek](scripts/icsnpp/s7comm/main.zeek).
* cotp.log
* s7comm.log
* s7comm_read_szl.log
* s7comm_upload_download.log
* s7comm_plus.log

For additional information on these log files, see the *Logging Capabilities* section below.

## Installation

### Package Manager

This script is available as a package for [Zeek Package Manger](https://docs.zeek.org/projects/package-manager/en/stable/index.html)

```bash
zkg refresh
zkg install icsnpp-s7comm
```

If this package is installed from ZKG, it will be added to the available plugins. This can be tested by running `zeek -N`. If installed correctly, you will see `ICSNPP::S7COMM`.

If ZKG is configured to load packages (see @load packages in quickstart guide), this plugin and these scripts will automatically be loaded and ready to go.
[ZKG Quickstart Guide](https://docs.zeek.org/projects/package-manager/en/stable/quickstart.html)

If users are not using site/local.zeek or another site installation of Zeek and want to run this package on a packet capture, they can add `icsnpp/s7comm` to the command to run this plugin's scripts on the packet capture:

```bash
git clone https://github.com/cisagov/icsnpp-s7comm.git
zeek -Cr icsnpp-s7comm/tests/traces/s7comm_plus_example.pcap icsnpp/s7comm
```

### Manual Install

To install this package manually, clone this repository and run the configure and make commands as shown below.

```bash
git clone https://github.com/cisagov/icsnpp-s7comm.git
cd icsnpp-s7comm/
./configure
make
```

If these commands succeed, you will end up with a newly create build directory. This contains all the files needed to run/test this plugin. The easiest way to test the parser is to point the ZEEK_PLUGIN_PATH environment variable to this build directory.

```bash
export ZEEK_PLUGIN_PATH=$PWD/build/
zeek -N # Ensure everything compiled correctly and you are able to see ICSNPP::S7COMM
```

Once users have tested the functionality locally and it appears to have compiled correctly, they can install it system-wide:
```bash
sudo make install
unset ZEEK_PLUGIN_PATH
zeek -N # Ensure everything installed correctly and you are able to see ICSNPP::S7COMM
```

To run this plugin in a site deployment, users will need to add the line `@load icsnpp/s7comm` to the `site/local.zeek` file to load this plugin's scripts.

If users are not using site/local.zeek or another site installation of Zeek and want to run this package on a packet capture, they can add `icsnpp/s7comm` to the command to run this plugin's scripts on the packet capture:

```bash
zeek -Cr icsnpp-s7comm/tests/traces/s7comm_plus_example.pcap icsnpp/s7comm
```

If users want to deploy this on an already existing Zeek implementation and don't want to build the plugin on the machine, they can extract the ICSNPP_S7comm.tgz file to the directory of the established ZEEK_PLUGIN_PATH (default is `${ZEEK_INSTALLATION_DIR}/lib/zeek/plugins/`).

```bash
tar xvzf build/ICSNPP_S7comm.tgz -C $ZEEK_PLUGIN_PATH 
```

## Logging Capabilities

### COTP Log (cotp.log)

#### Overview

This log captures COTP information for every COTP packet and logs it to **cotp.log**.

#### Fields Captured

| Field             | Type      | Description                                                       |
| ----------------- |-----------|-------------------------------------------------------------------| 
| ts                | time      | Timestamp                                                         |
| uid               | string    | Unique ID for this connection                                     |
| id                | conn_id   | Default Zeek connection info (IP addresses, ports)                |
| is_orig           | bool      | True if the packet is sent from the originator                    |
| source_h          | address   | Source IP address (see *Source and Destination Fields*)           |
| source_p          | port      | Source port (see *Source and Destination Fields*)                 |
| destination_h     | address   | Destination IP address (see *Source and Destination Fields*)      |
| destination_p     | port      | Destination port (see *Source and Destination Fields*)            |
| pdu_code          | string    | COTP PDU type code (in hex)                                       |
| pdu_name          | string    | COTP PDU name                                                     |

### S7COMM Header Log (s7comm.log)

#### Overview

This log captures s7comm header information for every s7comm packet and logs it to **s7comm.log**.

#### Fields Captured

| Field                 | Type      | Description                                                   |
| --------------------- |-----------|---------------------------------------------------------------|
| ts                    | time      | Timestamp                                                     |
| uid                   | string    | Unique ID for this connection                                 |
| id                    | conn_id   | Default Zeek connection info (IP addresses, ports)            |
| is_orig               | bool      | True if the packet is sent from the originator                |
| source_h              | address   | Source IP address (see *Source and Destination Fields*)       |
| source_p              | port      | Source port (see *Source and Destination Fields*)             |
| destination_h         | address   | Destination IP address (see *Source and Destination Fields*)  |
| destination_p         | port      | Destination port (see *Source and Destination Fields*)        |
| rosctr_code           | count     | Remote operating service control code (in hex)                |
| rosctr_name           | string    | Remote operating service control name                         |
| pdu_reference         | count     | Reference ID used to link requests to responses               |
| function_code         | string    | Parameter function code (in hex)                              |
| function_name         | string    | Parameter function name                                       |
| subfunction_code      | string    | User-Data subfunction code (in hex)                           |
| subfunction_name      | string    | User-Data subfunction name                                    |
| error_class           | string    | Error class name                                              |
| error_code            | string    | Error code within error class                                 |

### S7COMM Read-SZL Log (s7comm_read_szl.log)

#### Overview

This log captures information for the common S7Comm Read-SZL function. This data is logged to **s7comm_read_szl.log**.

#### Fields Captured

| Field                 | Type      | Description                                                   |
| --------------------- |-----------|---------------------------------------------------------------|
| ts                    | time      | Timestamp                                                     |
| uid                   | string    | Unique ID for this connection                                 |
| id                    | conn_id   | Default Zeek connection info (IP addresses, ports)            |
| is_orig               | bool      | True if the packet is sent from the originator                |
| source_h              | address   | Source IP address (see *Source and Destination Fields*)       |
| source_p              | port      | Source port (see *Source and Destination Fields*)             |
| destination_h         | address   | Destination IP address (see *Source and Destination Fields*)  |
| destination_p         | port      | Destination port (see *Source and Destination Fields*)        |
| pdu_reference         | count     | Reference ID used to link requests to responses               |
| method                | string    | Request or response                                           |
| szl_id                | string    | SZL ID (in hex)                                               |
| szl_id_name           | string    | Meaning of SZL ID                                             |
| szl_index             | string    | SZL index (in hex)                                            |
| return_code           | string    | Return code (in hex)                                          |
| return_code_name      | string    | Meaning of return code                                        |

### S7COMM Upload-Download Log (s7comm_upload_download.log)

#### Overview

This log captures information for the S7Comm Upload and Download functions (see list below). This data is logged to **s7comm_upload_download.log**:
* Start Upload
* Upload
* End Upload
* Request Download
* Download Block
* Download Ended

#### Fields Captured

| Field                  | Type      | Description                                                  |
| ---------------------- |-----------|--------------------------------------------------------------|
| ts                     | time      | Timestamp                                                    |
| uid                    | string    | Unique ID for this connection                                |
| id                     | conn_id   | Default Zeek connection info (IP addresses, ports)           |
| is_orig                | bool      | True if the packet is sent from the originator               |
| source_h               | address   | Source IP address (see *Source and Destination Fields*)      |
| source_p               | port      | Source port (see *Source and Destination Fields*)            |
| destination_h          | address   | Destination IP address (see *Source and Destination Fields*) |
| destination_p          | port      | Destination port (see *Source and Destination Fields*)       |
| rosctr                 | count     | Remote operating service control                             |
| pdu_reference          | count     | Reference ID used to link requests to responses              |
| function_code          | count     | Parameter function code                                      |
| function_status        | count     | Function status                                              |
| session_id             | count     | Session ID                                                   |
| blocklength            | count     | Length of block to upload/download                           |
| filename               | string    | Filename of block to upload/download                         |
| block_type             | string    | Block type to upload/download                                |
| block_number           | string    | Block number to upload/download                              |
| destination_filesystem | string    | Destination filesystem of upload/download                    |

### S7COMM-PLUS Log (s7comm_plus.log)

#### Overview

This log captures s7comm-plus header information for every s7comm-plus packet and logs it to **s7comm_plus.log**.

#### Fields Captured

| Field                 | Type      | Description                                                   |
| --------------------- |-----------|---------------------------------------------------------------|
| ts                    | time      | Timestamp                                                     |
| uid                   | string    | Unique ID for this connection                                 |
| id                    | conn_id   | Default Zeek connection info (IP addresses, ports)            |
| is_orig               | bool      | True if the packet is sent from the originator                |
| source_h              | address   | Source IP address (see *Source and Destination Fields*)       |
| source_p              | port      | Source port (see *Source and Destination Fields*)             |
| destination_h         | address   | Destination IP address (see *Source and Destination Fields*)  |
| destination_p         | port      | Destination port (see *Source and Destination Fields*)        |
| version               | count     | S7comm-plus version                                           |
| opcode                | string    | Opcode code (in hex)                                          |
| opcode_name           | string    | Opcode name                                                   |
| function_code         | string    | Opcode function code (in hex)                                 |
| function_name         | string    | Opcode function name                                          |

### Source and Destination Fields

#### Overview

Zeek's typical behavior is to focus on and log packets from the originator and not log packets from the responder. However, most ICS protocols contain useful information in the responses, so the ICSNPP parsers log both originator and responses packets. Zeek's default behavior, defined in its `id` struct, is to never switch these originator/responder roles which leads to inconsistencies and inaccuracies when looking at ICS traffic that logs responses.

The default Zeek `id` struct contains the following logged fields:
* id.orig_h (Original Originator/Source Host)
* id.orig_p (Original Originator/Source Port)
* id.resp_h (Original Responder/Destination Host)
* id.resp_p (Original Responder/Destination Port)

Additionally, the `is_orig` field is a boolean field that is set to T (True) when the id_orig fields are the true originators/source and F (False) when the id_resp fields are the true originators/source.

To not break existing platforms that utilize the default `id` struct and `is_orig` field functionality, the ICSNPP team has added four new fields to each log file instead of changing Zeek's default behavior. These four new fields provide the accurate information regarding source and destination IP addresses and ports:
* source_h (True Originator/Source Host)
* source_p (True Originator/Source Port)
* destination_h (True Responder/Destination Host)
* destination_p (True Responder/Destination Port)

The pseudocode below shows the relationship between the `id` struct, `is_orig` field, and the new `source` and `destination` fields.

```
if is_orig == True
    source_h == id.orig_h
    source_p == id.orig_p
    destination_h == id.resp_h
    destination_p == id.resp_p
if is_orig == False
    source_h == id.resp_h
    source_p == id.resp_p
    destination_h == id.orig_h
    destination_p == id.orig_p
```

#### Example

The table below shows an example of these fields in the log files. The first log in the table represents a Modbus request from 192.168.1.10 -> 192.168.1.200 and the second log represents a Modbus reply from 192.168.1.200 -> 192.168.1.10. As shown in the table below, the `id` structure lists both packets as having the same originator and responder, but the `source` and `destination` fields reflect the true source and destination of these packets.

| id.orig_h    | id.orig_p | id.resp_h     | id.resp_p | is_orig | source_h      | source_p | destination_h | destination_p |
| ------------ | --------- |---------------|-----------|---------|---------------|----------|---------------|-------------- |
| 192.168.1.10 | 47785     | 192.168.1.200 | 502       | T       | 192.168.1.10  | 47785    | 192.168.1.200 | 502           |
| 192.168.1.10 | 47785     | 192.168.1.200 | 502       | F       | 192.168.1.200 | 502      | 192.168.1.10  | 47785         |

## S7COMM File Extraction

S7COMM contains two functions for sending and receiving files: Upload and Download-Block. This plugin will extract files sent via these two functions and pass the extracted files to Zeek's file analysis framework.

## Coverage

See [Logging Capabilities](#logging-capabilities) for detailed information of the parser coverage.

S7Comm is a proprietary protocol which makes coverage information and statistics difficult to compose. All coverage details in this section include information and statistics based on the S7Comm and S7Comm plus traffic observed in test networks and publicly available packet captures.

### General/Header Logging

The general log files for COTP (cotp.log), S7Comm (s7comm.log), and S7Comm Plus (s7comm_plus.log) contain basic header information for all known (~100%) COTP, S7Comm and S7Comm Plus messages.

### Detailed Logging

Detailed logging for 1 S7comm function is logged in the read szl log file (s7comm_read_szl.log), 6 S7Comm functions are logged in the upload download log file (s7comm_upload_download.log), and 5 S7comm functions are logged as part of the general S7comm log file (s7comm.log). ~30% of known S7comm functions contain detailed logging.

There is currently no (0%) detailed logging for S7Comm Plus functions outside of the S7Comm Plus general/header log file (s7comm_plus.log).

### License

Copyright 2023 Battelle Energy Alliance, LLC. Released under the terms of the 3-Clause BSD License (see [`LICENSE.txt`](./LICENSE.txt)).
