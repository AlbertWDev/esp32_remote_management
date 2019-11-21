# Esp32 Remote Management API

API REST used to manage an Esp32 board remotely.

___
## RESOURCES

- **SYSTEM**
    - `System.Info`
    - `System.Partitions`
<!--    - `System.OTA` -->
<!--    - `System.Tasks` -->
- **NETWORK**
    - **WIFI**
        - `Network.WiFi.Storage`
        - `Network.WiFi.Status`
- **STORAGE**
<!--
- **GPIO**
- **DEVICES**
    - Devices.Screen
-->
___
<br>

## **`System.Info`**
> System details

| Attribute            | Type    | Description                       |
|----------------------|---------|-----------------------------------|
| **`chip_model`**     | String  | Chip model                        |
| **`chip_revision`**  | Integer | Chip revision number              |
| **`chip_cores`**     | Integer | Number of CPU cores in the chip   |
| **`chip_emb_flash`** | Boolean | Chip has embedded flash memory    |
| **`chip_wifi_bgn`**  | Boolean | Chip has 2.4GHz WiFi              |
| **`chip_bt`**        | Boolean | Chip has Bluetooth Classic        |
| **`chip_ble`**       | Boolean | Chip has Bluetooth LE             |
| **`idf_ver`**        | String  | IDF version as in 'git describe'  |
| **`free_heap`**      | Integer | Size of available heap (in bytes) |

#### Endpoints
| URI                              | HTTP method | Description          |
|----------------------------------|-------------|----------------------|
| `/v1/system/info/`               | **GET**     | Retrieve system info |



## **`System.Partitions`**
> Firmware partitions

| Attribute       | Type    | Description                                               |
|-----------------|---------|-----------------------------------------------------------|
| **`label`**     | String  | Partition label                                           |
| **`type`**      | String  | Partition type<br><small>`APP | DATA | UNKNOWN`</small>   |
| **`subtype`**   | String  | Partition subtype<br><sup>[app]</sup> <small>`FACTORY | OTA_X | UNKNOWN`</small><br><sup>[data]</sup> <small>`OTA | PHY | NVS | COREDUMP | NVS_KEYS |`<br>`EFUSE_EM | ESPHTTPD | FAT | SPIFFS | UNKNOWN`</small> |
| **`address`**   | Integer | Starting address of the partition in flash                |
| **`size`**      | Integer | Size of the partition (in bytes)                          |
| **`encrypted`** | Boolean | Partition is encrypted                                    |

#### Endpoints
| URI                             | HTTP method | Description                      |
|---------------------------------|-------------|----------------------------------|
| `/v1/system/partitions/`        | **GET**     | Retrieve partitions list         |
| `/v1/system/partitions/<label>` | **GET**     | Retrieve partition details       |
| `/v1/system/partitions/<label>` | **PUT**     | Update partition content via OTA ![](https://img.shields.io/badge/-WIP-blue) |
| `/v1/system/partitions/<label>` | **DELETE**  | Erase partition content ![](https://img.shields.io/badge/-WIP-blue) |



## **`Network.WiFi.Storage`**
> Stored WiFi network credentials

| Attribute       | Type    | Description                    |
|-----------------|---------|--------------------------------|
| **`ssid`**      | String  | SSID of the stored network     |
| **`password`**  | String  | Password of the stored network |
| **`score`**     | Integer | Network score (times used)     |

#### Endpoints
| URI                               | HTTP method | Description                        |
|-----------------------------------|-------------|------------------------------------|
| `/v1/network/wifi/storage/`       | **GET**     | Retrieve all stored WiFi networks  |
| `/v1/network/wifi/storage/<SSID>` | **GET**     | Retrieve stored WiFi network       |
| `/v1/network/wifi/storage/`       | **POST**    | Store new WiFi network credentials |
| `/v1/network/wifi/storage/<SSID>` | **PUT**     | Update WiFi network credentials ![](https://img.shields.io/badge/-WIP-blue) |
| `/v1/network/wifi/storage/<SSID>` | **DELETE**  | Remove a stored WiFi network       |



## **`Network.WiFi.Status`**
> WiFi status:
>   - Whether AP & STA are active
>   - AP details (MAC, IP, netmask, gateway, SSID)
>   - STA details (MAC, IP, netmask, gateway, SSID)

| Attribute         | Type   | Description                                              |
|-------------------|--------|----------------------------------------------------------|
| **`mode`**        | String | Active interfaces (AP, STA)<br><small>`NULL | STA | AP | APSTA`</small> |
| **`sta`**         | Object | STA details                                              |
| **`sta.mac`**     | String | (STA interface) MAC address                              |
| **`sta.ip`**      | String | (STA interface) IP address                               |
| **`sta.netmask`** | String | (STA interface) Network mask                             |
| **`sta.gw`**      | String | (STA interface) Gateway IP address                       |
| **`sta.ssid`**    | String | SSID of the network the board is connected to            |
| **`ap`**          | Object | AP details                                               |
| **`ap.mac`**      | String | (AP interface) MAC address                               |
| **`ap.ip`**       | String | (AP interface) IP address                                |
| **`ap.netmask`**  | String | (AP interface) Network mask                              |
| **`ap.gw`**       | String | (AP interface) Gateway IP address                        |
| **`ap.ssid`**     | String | SSID of the board Access Point                           |

#### Endpoints
| URI                        | HTTP method | Description                   |
|----------------------------|-------------|-------------------------------|
| `/v1/network/wifi/status/` | **GET**     | Get board WiFi network status |
| `/v1/network/wifi/status/` | **PUT**     | Update WiFi network status ![](https://img.shields.io/badge/-WIP-blue)<br><ul><li>Start/Stop AP</li><li>Change STA connected network</li><li>Change AP settings</li></ul> |



## **`Storage`**
> Storage (directories & files)

#### Directory entry
| Attribute      | Type    | Description                                   |
|----------------|---------|-----------------------------------------------|
| **`name`**     | String  | Directory entry name                          |
| **`type`**     | String  | Entry type<br><small>`DIR | FILE`</small>     |
| **`mode`**     | Integer | UNIX-like file mode                           |
| **`modified`** | Long    | Last modified time (UNIX timestamp)           |
| **`size`**     | Integer | File size in bytes. 0 if entry is a directory |

#### Endpoints
| URI                                  | HTTP method | Description                                       |
|--------------------------------------|-------------|---------------------------------------------------|
| `/v1/storage/<mount_point>/<node>*/` | **GET**     | List directory                                    |
| `/v1/storage/<mount_point>/<node>*`  | **GET**     | Get file                                          |
| `/v1/storage/<mount_point>/<node>*/` | **PUT**     | Create directory                                  |
| `/v1/storage/<mount_point>/<node>*`  | **PUT**     | Create or update file (content overwritten)       |
| `/v1/storage/<mount_point>/<node>*/` | **POST**    | Change directory attributes<br>_(See note below)_ ![](https://img.shields.io/badge/-WIP-blue) |
| `/v1/storage/<mount_point>/<node>*`  | **POST**    | Change file attributes<br>_(See note below)_      ![](https://img.shields.io/badge/-WIP-blue) |
| `/v1/storage/<mount_point>/<node>*/` | **DELETE**  | Delete directory                                  |
| `/v1/storage/<mount_point>/<node>*`  | **DELETE**  | Delete file                                       |

<!--
| `/v1/storage/`                       | **GET**     | List of mount points ![](https://img.shields.io/badge/-WIP-blue) |
-->

> **NOTE**
>
> In the POST request, required changes must be provided in JSON format.
>
> **Only `rename` is allowed by now.**
> ``` jsonc
> {
>   "rename": "/<mount_point>/<new_name>"
> }
> ```
