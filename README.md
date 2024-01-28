# NimBLE Rust wrapper for ESP32
[![crates.io](https://img.shields.io/crates/v/esp32-nimble)](https://crates.io/crates/esp32-nimble)
[![build](https://github.com/taks/esp32-nimble/actions/workflows/ci.yml/badge.svg)](https://github.com/taks/esp32-nimble/actions/workflows/ci.yml)
[![License](https://img.shields.io/crates/l/esp32-nimble)](https://github.com/taks/esp32-nimble/blob/develop/LICENSE)
[![Documentation](https://img.shields.io/badge/docs-esp32--nimble-brightgreen)](https://taks.github.io/esp32-nimble/esp32_nimble/index.html)

This is a Rust wrapper for the NimBLE Bluetooth stack for ESP32.
Inspired by [NimBLE-Arduino](https://github.com/h2zero/NimBLE-Arduino).

## Usage
Add below settings to your project's `sdkconfig.defaults`.
```
CONFIG_BT_ENABLED=y
CONFIG_BT_BLE_ENABLED=y
CONFIG_BT_BLUEDROID_ENABLED=n
CONFIG_BT_NIMBLE_ENABLED=y
```

- To enable Extended advertising, additionally append `CONFIG_BT_NIMBLE_EXT_ADV=y`.<br>
  (For use with ESP32C3, ESP32S3, ESP32H2 ONLY)

### Configuring for iOS Auto-Reconnection

To enable seamless auto-reconnection of iOS devices with your ESP32 BLE server, you need to adjust settings in both the `sdkconfig` file and your Rust code.

#### Update `sdkconfig`

Include this line in your `sdkconfig`:

```
CONFIG_BT_NIMBLE_NVS_PERSIST=y
```

Setting `CONFIG_BT_NIMBLE_NVS_PERSIST` to `y` ensures that bonding information is saved in the device's Non-Volatile Storage (NVS). This step is crucial for allowing iOS devices to automatically reconnect without the need for rebonding after the ESP32 has been reset or powered off and on again.

#### Configure Security Options in Rust Code

Properly setting the security options in your Rust implementation is key:

```rust
device
  .security()
  .set_auth(AuthReq::Bond) // Bonding enables key storage for reconnection
  .set_io_cap(SecurityIOCap::NoInputNoOutput) // You can choose any IO capability
  .resolve_rpa(); // Crucial for managing iOS's dynamic Bluetooth addresses
```

- `.set_auth(AuthReq::Bond)` sets up bonding, crucial for storing security keys that enable future automatic reconnections.
- `.resolve_rpa()`: This function is essential for adapting to the changing Bluetooth addresses used by iOS devices, a feature known as Resolvable Private Address (RPA). It's vital for maintaining reliable and seamless connections with iOS devices, ensuring that your ESP32 device can recognize and reconnect to an iOS device even when its Bluetooth address changes.
