## Not sure why I implemented this, should've used protobuf

# StructBuf and StreamUtils

**StructBuf** is a simple framework that performs C code generation for serialization of structures defined in a special language.
The idea is somewhat similar to **protobuf**, but the implementation is minimal, tailored for C code generation.

## Types

The following primitive types are supported:
- **byte**: 1 byte (rendered as uint8_t)
- **word**: 2 bytes (rendered as uint16_t)
- **dword**: 4 bytes (rendered as uint32_t)

Special **struct** type is supported, allowing to group fields of primitive types and other structures.
**TODO**: enum types?

A field can be defined as a single field or an array, and the maximum number of elements in an array can be limited to a specific number.
The maximum size of an array can affect overall structure size, i.e.
- if (size <= 256), 8-bit will be used to store array length (uint8_t);
- if (size > 256) and (size <= 65536), 16-bit will be used to store array length (uint16_t);
- otherwise (size > 65536), 32-bit will be used to store array length (uint32_t).

Example:
```
struct StoreBlock {
  dword blockId;
  dword version;
  byte type;
}

struct Folder {
  byte folderId;
  byte parentFolderId;
  byte[256] folderName;
}

struct FoldersBlock {
  StoreBlock block;
  Folder[] folders;
}
```

Multidimensional arrays are not supported directly, but can be emulated with the following construct:
```
struct ArrayElement {
  byte[256] element;
}

struct 2dArray {
 ArrayElement[256] elements;
}
```
Thinking about native multidimensional arrays, I'm not sure if a native implementation would be worth the effort. Mainly, it's pretty hard to come up with elegant design. I guess that should be close to the reasoning why only one dimension is directly supported in protobuf (**repeated** keyword).

Comments:
At this time only C-style single-line comments are supported.
e.g.:
`// This is a comment`
`struct Important{ //this structure contains important data`

**TODO**: multi-line comments in struct file (`/* ... */`)
**TODO**: check that structure can't be empty (`Struct {}`)
**TODO**: Test code coverage?
**TODO**: Endianness concerns? Java ByteBuffer allows to specify it like this: `byteBuffer.order(ByteOrder.LITTLE_ENDIAN);`

## Structures and Functions generated

A corresponding set of C structures will be generated for the aforementioned StructBuf structures:
```
typedef struct {
    uint32_t blockId;
    uint32_t version;
    uint8_t type;
} StoreBlock;

typedef struct {
    uint8_t folderId;
    uint8_t parentFolderId;
    uint8_t folderNameLength;
    uint8_t* folderName;
} Folder;

typedef struct {
    StoreBlock block;
    uint8_t foldersLength;
    Folder* folders;
} FoldersBlock;
```

Also, for each StructBuf structure the following functions will be generated:
- `Struct sb_createStruct(uint32_t field,`
  `uint8_t arrayFieldLength, uint8_t* arrayField,`
  `SubStructure subStructure,`
  `uint8_t subStructureArrayLength, SubStructure* subStructureArray);`
- `void sb_saveStruct(void** cursor, Struct* _Struct);`
- `Struct* sb_readStruct(void** cursor);`
- `uint8_t sb_initStruct(void** cursor, Struct* _Struct);`
- `void sb_cleanStruct(Struct* _Struct);`
- `void sb_deleteStruct(Struct* _Struct);`
- `uint32_t sb_getStructSizeStruct(Struct* _Struct);`
- `uint8_t sb_isValidBufferSizeStruct(void** cursor, uint32_t bufferSize);`

Those functions will use lower-level StreamUtils library under the hood to read and write data.  
For more information on StreamUtils and cursors see [Appendix 1: StreamUtils](#appendix-1-streamutils).

### create function

A constructor-like function with a parameter for each field of a struct, to instantiate structures manually.
**TODO**: Current implementation returns created structure by value and takes non-array parameters by value as well. One "pro" of this is that it's easy to distinguish between arrays and single fields because pointers mean arrays, and all allocations are on stack, so I'm not 100% sure if an optimization might be desired here.

### save function

Writes serialized structure to the memory at position defined by cursor.

### init function

Initializes structure by deserializing data on position defined by cursor.
Return value:
- 0 (false) - initialization succeeded;
- Error code.
  **TODO**: error codes need to be defined and checks implemented, e.g. array length exceeds limit, also in .
  **TODO**: there are a lot of functions that I expected to return error codes, but didn't implement yet. What needs to be done, is figure out if those functions really need to return error codes, and consolidate error code handling, including nested calls. Also, some functions use 1 as success, while others use 0. This needs to be consolidated as well. In other words, error code handling should be a solid and uniform concept across the framework.
  **TODO**: consider init similar to strncpy? As of now it feels to me that it's cleaner to separate the concerns in multiple functions.

### read function

A build-up on top of init function, that also creates the return structure by allocating memory with malloc.

### clean function

Release memory for all substructures of a structure by **free**-ing all **malloc**-ed substructures in a structure recursively, but without **free**-ing the main structure memory.
This function should be used (and is used internally) in cases when an array of structures is **malloc**-ed, so we can't **free** each structure individually, and only **free**-ing the entire array is possible.

### delete function

Release memory for substructures of a structure using **clean** function and then **free** the main structure memory.
This should be used for individually **malloc**-ed structures only, not for arrays of structures.

### getStructSize function

Returns serialized size in bytes for a given structure.
Should be checked before serializing into a buffer to prevent writing data outside of buffer boundaries.
Return value:
- structure serialized size.

### isValidBufferSize function

The idea of this function is to examine the layout of a structure in a buffer before deserialization and make sure it doesn't exceed a certain size.
This check would ensure that the internals of a buffer are not corrupted and the resulting deserialization wouldn't exceed some designated size when deserialized (in the simplest case, the size of the holding buffer itself).
Also, if the buffer is received from an unreliable source, this check will prevent maliciously structured buffers to cause operations on regions of memory outside of designated space.
Return value:
- 0 (false) - validation failed;
- 1 (true) - validation succeeded.

## Accessor Structures and Functions generated

Accessors are a special lightweight type of structures that allows to perform read/update operations on buffers in-place. Accessors are fully independent from fully de-serializable struct implementation and will likely stay that way, since there is no obvious need to intersect the two worlds.

Pros and cons:
- minimal overhead on creation, no data copying from buffer;
- very small constant size / footprint, only a few pointers to positions within an existing buffer are stored;
- reduces memory operations on init, allows to reuse and move around the originally acquired buffer without complicated transformations / restructuring;
- However, the downside is that it comes with access overhead, especially for nested arrays of structures.
- Such lightweight access can be useful in situations when the overhead of deserializing buffers and creating a full set of C structures is an overkill, like "read 0 or 1 times" access pattern.
- Another use case would be a situation when it's most optimal to keep the original buffer unchanged, like reading a buffer from network and immediately writing to drive or sending it to the next host with minimal changes.

Going further with the same model that we used in examples above, to implement Accessors, the following set of C structures will be generated:
Example: Accessor structures in C
```
typedef struct {
    void* blockId;
    void* version;
    void* type;
} StoreBlockAccessor;

typedef struct {
    void* folderId;
    void* parentFolderId;
    void* folderName;
} FolderAccessor;

typedef struct {
    void* block;
    void* folders;
} FoldersBlockAccessor;
```
For Accessors, a structure with special fields and corresponding accessor functions is created, where each field is a pointer to the position of the corresponding field in the buffer. Provided this information, accessor functions can be generated that allow to operate on structures stored in buffer directly, without need to deserialize.
Such operations, however, are limited to reads and in-place updates; for instance, adding or removing an element to an array would be impossible.

For each StructBuf structure the following functions will be generated:
- main init function:
  - `uint8_t sb_initStruct_Accessor(void** cursor, StructAccessor* accessor);`
    This function initializes an accessor from buffer.

- for primitive type fields:
  - `uint_type* sb_getStruct_field_Pointer(StructAccessor* accessor);`
    This function returns a pointer to field data in buffer. The return type used depends on the type of a given field.

- for primitive type arrays:
  - `uint_type sb_getStruct_field_ArrayLength(StructAccessor* accessor);`
    *ArrayLength()* function returns the length of a corresponding array *by value*. We intentionally don't return pointer to the corresponding length position in the buffer, because changing that value would have the meaning of resizing of the corresponding array and is not an operation that's supported in-place. The return type depends on maximum size of the corresponding array.

  - `uint_type* sb_getStruct_field_ArrayPointer(StructAccessor* accessor);`
    *ArrayPointer()* function returns a pointer to array data in the buffer.The return type used depends on the type of a given field.

- for structure fields:
  - `uint8_t sb_initStruct_field_Accessor(StructAccessor* parentAccessor, SubStructAccessor* accessorToInit);`
    This function takes a pointer to *SubStruct* Accessor to fill its pointer structure from the corresponding substructure in the buffer described by parent Accessor.

- for structure arrays:
  - `uint_type sb_getStruct_field_ArrayLength(PhraseBlockAccessor* accessor);`
    *ArrayLength()* function returns the length of a corresponding array *by value*, similarly to primitive type arrays. The return type depends on maximum size of the corresponding array.

  - `uint8_t sb_initStruct_field_AccessorArray(PhraseBlockAccessor* parentAccessor, StorePhraseHistoryAccessor* accessorArrayToInit, uint8_t length);`
    *init..AccessorArray()* function, similarly to single structure field initializer function, fills the fields of *SubStruct* Accessor structures, but it's doing it for an array of such Accessor objects.

## Command Line
**TODO**: Instrumentalize StructBuf (cmd line tool)
PicoCli:
- input file name
- file type (C-file, H-file, StreamUtils.c, StremaUtils.h)
- output file name
- add accessor bindings

## Appendix 1: StreamUtils

StreamUtils library defines functions to load data from and save data to buffers that work with cursors.
The code can be found in **resources** folder (StreamUtils.cpp, StreamUtils.h).

The idea of cursor is as follows.
1. Let's say, a buffer is created, e.g. `void* buffer = malloc(4096);`
   At this point `buffer` variable contains the address of the start of the block.
2. The goal is to read a series of values from or write a series of values to the buffer sequentially, across different functions, while keeping track of current position to read next value from (or write to).
   For that purpose we transfer a pointer to a pointer, so when it's transferred across functions it factually updates the same value of the original pointer to our buffer. When such cursor is passed around, all functions have access to the same base pointer and can register the advancements related to read/write operations in a way visible to all other functions working on the same buffer.
   E.g.:
```
void** cursor = &buffer;

uint32_t id = streamLoadUint32_t(cursor);

uint8_t valuesLength = streamLoadUint8_t(cursor);

uint32_t* values = (uint32_t*)malloc(valuesLength*sizeof(uint32_t));
streamLoadData(cursor, values, valuesLength*sizeof(uint32_t));
```
Above, **streamLoadUint32_t** function loads 4 bytes and increments the cursor by 4 bytes, the following **streamLoadUint8_t** reads 1 byte from updated position and increments the pointer by 1, etc.
Since the actual pointer that's being incremented is located in **buffer** variable, we need to pass it to those functions by pointer, so that the position updates are not lost and updated position can be used by the following function calls.

### StreamUtils Functions:

- `void moveCursor(void** cursor, size_t moveBy);`
  Advance cursor by **moveBy** bytes.

- `void streamLoadData(void** cursor, void* to, size_t size);`
  Load **size** bytes from cursor position to buffer, advance cursor.

- `uint8_t streamLoadUint8_t(void** cursor);`
  Load byte from cursor position, advance cursor.
- `uint16_t streamLoadUint16_t(void** cursor);`
  Load word (2 bytes) from cursor position, advance cursor.
- `uint32_t streamLoadUint32_t(void** cursor);`
  Load double word (4 bytes) from cursor position, advance cursor.

- `void streamSaveData(void** cursor, void* from, size_t size);`
  Save **size** bytes from buffer to cursor position, advance cursor.

- `void streamSaveUint8_t(void** cursor, uint8_t value);`
  Save byte to cursor position, advance cursor.
- `void streamSaveUint16_t(void** cursor, uint16_t value);`
  Save word (2 bytes) to cursor position, advance cursor.
- `void streamSaveUint32_t(void** cursor, uint32_t value);`
  Save double word (4 bytes) to cursor position, advance cursor.