# StorageHelperRK
Library for storing data in EEPROM, file system, SD card, FRAM, etc. on Particle devices

## Persistent Data

This library makes makes it easy to store simple persistent data on a variety of storage mechanisms including:

- Retained memory
- FRAM (MB85RCxx connected by I2C)
- Emulated EEPROM in Device OS
- POSIX flash filesystem (Particle Gen 3 devices and P2)
- SdFat (Micro SD card)
- Spiffs (SPI flash chips)

It's possible to add additional storage mechanisms by creating a subclass. It's not necessary to modify the library to add new storage mechanisms.

### Data storage

- Data includes C/C++ primitive types (int, float, double, bool, and other types like uint32_t) and C-strings of up to a maximum length that you configure.

- Data is always cached in RAM, so it can be read quickly and efficiently, with no latency and low overhead.

- You typically write simple get and set accessor functions for your data fields, so the correct data type is passed and returned.

- Data is stored in binary format on your underlying storage method for efficiency. 

- Data is extensible in future versions of your code. While you cannot modify the existing fields in any way, you can add additional data at the end of the structure. If the data on disk is smaller than the current structure, it is zeroed out.

- Data integrity detection includes magic bytes, length and version information, and a 32-bit hash of the data. These must match for be considered valid.

- Deferred updates can be enabled which allow you to save values frequently, but write them to storage less often to avoid flash wear and to not slow down the thread that is updating.

- The code is thread safe and can be used from loop, worker threads, or software timers.

### Example

This is from the example 04-persistent.cpp. This is the code you're write for your data.

```cpp
class MyPersistentData : public StorageHelperRK::PersistentDataFile {
public:
	class MyData {
	public:
		StorageHelperRK::PersistentDataBase::SavedDataHeader header;
		int test1;
		bool test2;
		double test3;
		char test4[10];
		// OK to add more fields here 
	};

	static const uint32_t DATA_MAGIC = 0x20a99e73;
	static const uint16_t DATA_VERSION = 1;

	MyPersistentData() : PersistentDataFile(&myData.header, sizeof(MyData), DATA_MAGIC, DATA_VERSION) {};

	int getValue_test1() const {
		return getValue<int>(offsetof(MyData, test1));
	}

	void setValue_test1(int value) {
		setValue<int>(offsetof(MyData, test1), value);
	}

	bool getValue_test2() const {
		return getValue<bool>(offsetof(MyData, test2));
	}

	void setValue_test2(bool value) {
		setValue<bool>(offsetof(MyData, test2), value);
	}

	double getValue_test3() const {
		return getValue<double>(offsetof(MyData, test3));
	}

	void setValue_test3(double value) {
		setValue<double>(offsetof(MyData, test3), value);
	}

	String getValue_test4() const {
		String result;
		getValueString(offsetof(MyData, test4), sizeof(MyData::test4), result);
		return result;
	}
	bool setValue_test4(const char *str) {
		return setValueString(offsetof(MyData, test4), sizeof(MyData::test4), str);
	}

    void logData(const char *msg) {
        Log.info("%s: %d, %d, %lf, %s", msg, myData.test1, (int)myData.test2, myData.test3, myData.test4);
    }


	MyData myData;
};

```

#### Code walk-through

You implement a subclass for your data. In this case, it's called `MyPersistentData` but you probably will want a more application-specific name.

The second part of the statement is what you are subclassing, and will be one of:

- StorageHelperRK::PersistentDataFile data in a file (POSIX, SdFat, SPIFFS)
- StorageHelperRK::PersistentDataRetained for retained memory
- StorageHelperRK::PersistentDataEEPROM for the emulated EEPROM in Particle Device OS
- StorageHelperRK::PersistentDataFRAM for MB85RCxx I2C FRAM

```cpp
class MyPersistentData : public StorageHelperRK::PersistentDataFile {
```

The next part is the description of your actual data you want to store.

- The structure must always begin with `StorageHelperRK::PersistentDataBase::SavedDataHeader header;`. This structure adds 16 bytes of overhead and contains the magic bytes, version information, hash. etc.
- You can add additional fields are desired for your application using C/C++ primitive types.
- You can also add c-string variables. In this example test4 can be up to 9 characters. If you pass a value longer than that for set, the value will be truncated.
- Once you release software with a data structure, you must never modify the existing fields, including resizing string fields or reordering fields.
- You can, however, add additional fields at the end at any time.

```cpp
class MyData {
public:
    StorageHelperRK::PersistentDataBase::SavedDataHeader header;
    int test1;
    bool test2;
    double test3;
    char test4[10];
    // OK to add more fields here 
};

```

- The magic bytes are 4 random bytes that you pick that identify your data structure. Since retained memory, EEPROM, etc. could contain contents left over from a previous application, this helps prevent invalid data from being used. If you completely change your data structure, you may want to pick new magic bytes.
- There is also a version field. If either the magic bytes or version do not match, the existing data will be erased.
- Do not update the version when simply adding fields at the end of the structure!
- The constructor declaration is boilerplate that will typically look something like this, though declarations for EEPROM and FRAM have extra parameters.
- For file systems, the filename to use is specified later.

```cpp
static const uint32_t DATA_MAGIC = 0x20a99e73;
static const uint16_t DATA_VERSION = 1;

MyPersistentData() : PersistentDataFile(&myData.header, sizeof(MyData), DATA_MAGIC, DATA_VERSION) {};
```



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

