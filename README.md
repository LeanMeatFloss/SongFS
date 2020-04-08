# SongFS

A simple file system for on-chip devices, it can drive external or internal flash or RAM.


## Directory Definitions

### API

Including the head files containing functions that the user of SongFS want to use.

### DRIVER

Including the head files containing variables or functions that need user to define to drive flash drive. Including:
- LOGIC_BLOCK_LENGTH
- ERASURE_DATA
- WRITE_ALIGN_LENGTH
- LOGIC_BLOCK_READ
- LOGIC_BLOCK_WRITE

## Key Definitions

### Logic Block:

The logic block is an abstract concept, which can be treated as a superset of the flash block. i.e. should have the same properties as the flash block:

- A known length "LOGIC_BLOCK_LENGTH(LOGIC_BLOCK_NUM)"
- The data to be programmed should aligned to a minium length size "WRITE_ALIGN_LENGTH(LOGIC_BLOCK_NUM)", including the address and the data length.
- Can be erasure standalone
- The data should be 0xFF or 0x00 ("ERASURE_DATA(LOGIC_BLOCK_NUM)") in all address of block after erasure
- If a range of data which aligned to the "WRITE_ALIGN_LENGTH(LOGIC_BLOCK_NUM)" and fulfilled with "ERASURE_DATA(LOGIC_BLOCK_NUM)", the data can be programmed. On the other hand, if the range of data are not fulfilled with  "ERASURE_DATA(LOGIC_BLOCK_NUM)", it cannot be programmed.

As a superset of the flash block, the logic block has those extend features and properties:

- The start address of a logic block is fixed to 0x00, which means the range of logic blocks is [ 0x00,LOGIC_BLOCK_LENGTH(LOGIC_BLOCK_NUM)-1 ]
- The file system (SongFS) can read or write the logic block's data by function LOGIC_BLOCK_READ and LOGIC_BLOCK_WRITE

With those definitions, flowing parts can be abstracted as a logic block:

- A range LOGIC_BLOCK_LENGTH of RAM, and the WRITE_ALIGN_LENGTH=1,ERASURE_DATA=0x00
- Two flash blocks with same WRITE_ALIGN_LENGTH=1,ERASURE_DATA=0xFF, and the LOGIC_BLOCK_LENGTH is the sum value of the two flash blocks.

### LOGIC_BLOCK_OPERATION_RETURNVALUE

The return value of the LOGIC_BLOCK_READ and LOGIC_BLOCK_WRITE function, which have those sub definitions:

- LOGIC_BLOCK_OPERATION_RETRY
  Notify the SongFS that the operation is accepted but the SongFS need to retry the same function with the same input.
- LOGIC_BLOCK_OPERATION_END
  Notify the SongFS that the operation is accepted and finished successfully.
- LOGIC_BLOCK_OPERATION_ERROR
  Notify the SongFS that the operation is declined with any error. And the SongFS can get error information by ERRORINFO.

### ERRORINFO

A buffer of ERRORINFO_LENGTH address size, which can filled with error info.



### LOGIC_BLOCK_NUM

Can be treated as the id of the logic block, each logic block should have a unique number in the system.

### LOGIC_BLOCK_LENGTH(LOGIC_BLOCK_NUM)

The length of the logic block.

### ERASURE_DATA(LOGIC_BLOCK_NUM)

The data after erasure in the logic block, mostly is 0xFF.

### WRITE_ALIGN_LENGTH(LOGIC_BLOCK_NUM)

The data to be programmed should aligned to a minium length size "WRITE_ALIGN_LENGTH(LOGIC_BLOCK_NUM)", including the address and the data length.

>Tips:
The value is length not size, because in some system like TI, one address stored two bytes.

### LOGIC_BLOCK_READ(LOGIC_BLOCK_NUM,STARTADDRESS,LENGTH,TARGETADDRESS,ERRORINFO)

A basic function which need the programmer to realize. SongFS using this function to read a range of data in the logic block.
>Tips:
The target address will be located at RAM with sufficient length.

### LOGIC_BLOCK_WRITE(LOGIC_BLOCK_NUM,STARTADDRESS,LENGTH,TARGETADDRESS,ERRORINFO)

A basic function which need the programmer to realize. SongFS using this function to write a range of data in the logic block.
>Tips:
The file system will align the data to the WRITE_ALIGN_LENGTH(LOGIC_BLOCK_NUM), which means it is no need to double-check.

