# Bootloader
STM32401CCUx Mbed OS bootloader.

uses 0x10000 bytes of memory leaving 0x30000 bytes free for main program


## Main Program Bootloader Setup ##

**Sample Custom STM32F401CC Target**

`custom_targets.json`
```JSON filename="custom_targets.json"
{
    "ARES": {
        "inherits": ["MCU_STM32F4"],
        "macros_add": [
            "STM32F401xC",
            "HSE_VALUE=16000000U"
        ],
        "overrides": {
            "lse_available": 1
        },
        "device_has_add": [
            "USBDEVICE"
        ],
        "bootloader_supported": true,
        "extra_labels_add": ["ARES"],
        "detect_code": ["1234"],
        "device_name": "STM32F401CCUx"
    }
}
```

`mbed_app.json`
```JSON filename="mbed_app.json"
{
    "target_overrides": {
        "ARES": {
            "target.mbed_app_start": "0x08010000",
            "target.mbed_rom_start": "0x08000000",
            "target.mbed_rom_size" : "0x40000"
        }
    }
}
```

**Watchdog**

The bootloader incorporates a 30s watchdog timer in order to ensure a successful boot. In order to continually function, the Main Program must kick the watchdog every 30s. This can be done in a similar fashion
```
// kicks watchdog every 20s
void _kick_watchdog() {
    Watchdog &watchdog = Watchdog::get_instance();

    // ensure watchdog is started
    if (!watchdog.is_running()){
        watchdog.start(30000);
    }


    while (true) {
        watchdog.kick();
        ThisThread::sleep_for(25s);
    }
}

int main() {
    Thread t;
    t.start(_kick_watchdog());

    while (true) {}
}
```
Care should be taken when using threads to ensure thread priority does not break the watchdog. Priority should be balanced to allow pre-defined faults to be corrected while booting when unknown errors occur
