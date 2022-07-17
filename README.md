# StorageHelperRK
Library for storing data in EEPROM, file system, SD card, FRAM, etc. on Particle devices

## Persistent Data

This library makes makes it easy to store simple persistent data on a variety of storage mechanisms including:

- Retained memory
- FRAM
- Emulated EEPROM in Device OS
- POSIX flash filesystem (Particle Gen 3 devices)
- SdFat (Micro SD card)
- Spiffs (SPI flash chips)

It's possible to add additional storage mechanisms by creating a subclass. It's not necessary to modify the library to add new storage mechanisms.



## File System Abstraction

There is a very limited file system abstraction as part of this library. It includes the bare minimum of functionality:

- open (with read, write, append, truncate, and/or create)
- close
- seek
- read
- write
- truncate

There are currently adapters for:

- POSIX (Particle Gen 3 devices)
- SdFat (Micro SD card)
- Spiffs (SPI flash)

