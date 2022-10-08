# HydroBlockchain Ice Protocol Contract Review

### The Ice protocol creates a secure and decentralized way to create, sign, store, and verify financial documents.


#### This Contract forms File Storage / Stamping / Encryption part of Hydro Protocols


Line: #1

```
pragma solidity ^0.5.1;
```

The first line tells you that the source code is written for Solidity version 0.5.1, or a newer version of the language up to the latest version. This is to ensure that the contract is compilable with a new compiler version. Pragmas are common instructions for compilers about how to treat the source code (e.g. pragma once).

Lines: #3 - 13

```
import "./SnowflakeInterface.sol";
import "./IdentityRegistryInterface.sol";

import "./SafeMath.sol";

import "./IceGlobal.sol";
import "./IceSort.sol";

import "./IceFMSAdv.sol";

import "./IceFMS.sol";

```

These lines import these contracts and interfaces to be used in this contract.

```
import "./SnowflakeInterface.sol";
```

This line imports the Snowflake protocol Interface which is an uinveral identity standard for creating and managing digital identity.

```
import "./IdentityRegistryInterface.sol";
```

This line imports IdentityRegistryInterface which is used to contains identity view functions, identity management functions, and Recovery management functions.

```
import "./SafeMath.sol";
```

This line imports SafeMath library functions which has functions to check and prevent overflow and underflow.

```
import "./IceGlobal.sol";
```

This line imports Ice Protocol Global Items Libray which is a part of many contracts that Ice protocol uses to form a robust File Management System.

IceGlobal.sol has some structs and mapping used in this contract.

To define Global Record for a given Item

```
    struct GlobalRecord {
        uint i1; // store associated global index 1 for access
        uint i2; // store associated global index 2 for access
    }
```

To define ownership info of a given Item.

```
    struct ItemOwner {
        uint EIN; // the EIN of the owner
        uint index; // the key at which the item is stored
    }
```

To define global file association with EIN

- Combining EIN and itemIndex and properties will give access to item data.

```
    struct Association {
        ItemOwner ownerInfo; // To Store Iteminfo
        ItemOwner stampingRecipient;  // to store stamping recipient

        bool isFile; // whether the Item is File or Group
        bool isHidden; // Whether the item is hidden or not
        bool deleted; // whether the association is deleted

        uint32 stampingInitiated; // Whether the item stamping is initiated, contains 0 or the timestamp
        uint32 stampingCompleted; // Whether the stamping is completed, contains 0 or the timestamp
        bool stampingRejected; // Whether the staping is rejected by recipient

        uint8 sharedToCount; // the count of sharing

        mapping (uint8 => ItemOwner) sharedTo; // to contain share to
        mapping (uint => bool) sharedToEINMapping; // contains EIN Mapping of shared to
    }
```

To define state and flags for Individual things, used in cases where state change should be atomic

```
   struct UserMeta {
      bool hasAvatar;
   }
```

```
import "./IceSort.sol";
```

This line imports Ice Protocol Sort Libray which creates sorting order for maximizing space utilization.

The SortOrder struct is used from this library.

```
 /* To define the order required to have double linked list */
    struct SortOrder {
        uint next; // the next ID of the order
        uint prev; // the prev ID of the order

        uint pointerID; // what it should point to in the mapping

        bool active; // whether the node is active or not
    }
```

```
import "./IceFMSAdv.sol";
```

This line imports Ice Protocol Files / Groups / Users Meta Management System Libray which defines file sharing and stamping functions.

This library also imports the GlobalRecord struct from IceGlobal library

```
import "./IceFMS.sol";
```

Ice Protocol Files / Groups / Users Meta Management System Libray which defines file structure of all stored files.

Two structs are used from this library here

- To define File structure of all stored files

```
    struct File {
        // File Meta Data
        IceGlobal.GlobalRecord rec; // store the association in global record

        // File Properties
        bytes protocolMeta; // store metadata of the protocol
        FileMeta fileMeta; // store metadata associated with file

        // File Properties - Encryption Properties
        mapping (address => bytes32) encryptedHash; // Maps Individual address to the stored hash

        // File Other Properties
        uint associatedGroupIndex; // to store the group index of the group that holds the file
        uint associatedGroupFileIndex; // to store the mapping of file in the specific group order
        uint transferEIN; // To record EIN of the user to whom trasnfer is inititated
        uint transferIndex; // To record the transfer specific index of the transferee

        // File Transfer Properties
        mapping (uint => uint) transferHistory; // To maintain histroy of transfer of all EIN
    }
```

- To connect Files in linear grouping, sort of like a folder, 0 or default grooupID is root

```

    struct Group {
        IceGlobal.GlobalRecord rec; // store the association in global record

        string name; // the name of the Group

        mapping (uint => IceSort.SortOrder) groupFilesOrder; // the order of files in the current group
        uint groupFilesCount; // To keep the count of group files
    }
```

/\* ******\*\*\*******

- DEFINE VARIABLES
  ******\*\*\******* \*/

Lines: #57 - #59

```
    mapping (uint => mapping(uint => IceGlobal.Association)) globalItems;
    uint public globalIndex1; // store the first index of association to retrieve files
    uint public globalIndex2; // store the second index of association to retrieve files
```

This is a 2 dimentional mapping named globalItems that maps the mapping of the Association struct in IceGlobal library to a uint. The uint is the key to retrieve mapping of each item stored.

Lines: #64

```
  mapping (uint => IceGlobal.UserMeta) public usermeta;
```

This line is a mapping that fetches and stores to UserMeta struct in IceGlobal library and maps it to a uint its mapping name as usermeta.

Line: #69 

```
    mapping (uint => mapping(uint => IceFMS.File)) files;
```

This line is a mapping that points to File Struct in IceFMS Libray and maps it to a uint its mapping name as files.

Line: #70
```
    mapping (uint => mapping(uint => IceSort.SortOrder)) public fileOrder; // Store round robin order of files
```
This line points to the SortOrder struct and maps it to a uint with its mapping name as fileOrder.

Line: #71
```
    mapping (uint => uint) public fileCount; // store the maximum file count reached to provide looping functionality
```
This line is a mapping of uint to uint that acts as a counter for the files. It is named fileCount. 


Line: #77
```
    mapping (uint => mapping(uint => IceFMS.Group)) groups;
```
This is a 2 dimentional mapping named groups that maps the mapping of the Group struct in IceFMS library to a uint. The uint is the key to retrieve mapping of the group of each user(EIN).

Line: #78
```
    mapping (uint => mapping(uint => IceSort.SortOrder)) public groupOrder; 
```
This is a 2 dimentional mapping named groupOrder that maps the mapping of the SortOrder struct in IceSort library to a uint. The uint is the key to store the round robin order of group.

Line: #79
```
    mapping (uint => uint) public groupCount; 
```
This line is a mapping of uint to uint that acts as a counter for the groups. It is named groupCount. It stores the maximum group count reached to provide looping functionality


Line: #84
```
mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) public transfers;
```
This is a 2 dimentional mapping named transfers that maps the mapping of the GlobalRecord struct in IceGlobal library to a uint. The uint is the key to retrieve mapping of transfer of each user(EIN). It look up the incoming transfer request stored on a given index.

Line: #85
```
    mapping (uint => mapping(uint => IceSort.SortOrder)) public transferOrder; 
```
This is a 2 dimentional mapping named transferOrder that maps the mapping of the SortOrder struct in IceSort library to a uint. The uint is the key to store the round robin order of transfers.

Line: #86
```
    mapping (uint => uint) public transferCount; 
```
This line is a mapping of uint to uint that acts as a counter for the transfers. It is named transferCount. It store the maximum transfer request count reached to provide looping functionality.

Line: #91
```
mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) public shares;
```
This is a 2 dimentional mapping named shares that maps the mapping of the GlobalRecord struct in IceGlobal library to a uint. The uint is the key to retrieve mapping of sharing files of each user(EIN). It look up the incoming sharing files stored on a given index.

Line: #92
```
mapping (uint => mapping(uint => IceSort.SortOrder)) public shareOrder;
```
This is a 2 dimentional mapping named shareOrder that maps the mapping of the SortOrder struct in IceSort library to a uint. The uint is the key to store the round robin order of sharing.

Line: #93
```
    mapping (uint => uint) public shareCount; 
```
This line is a mapping of uint to uint that acts as a counter for the sharing files. It is named transferCount. It store the maximum sahred items count reached to provide looping functionality.

Line: #98
```
mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) public stampings;
```
This is a 2 dimentional mapping named stampings that maps the mapping of the GlobalRecord struct in IceGlobal library to a uint. The uint is the key to retrieve mapping of sharing files with stamps of each user(EIN). It look up the incoming stamping files stored on a given index.

Line: #99
```
mapping (uint => mapping(uint => IceSort.SortOrder)) public stampingOrder;
```
This is a 2 dimentional mapping named stampingOrder that maps the mapping of the SortOrder struct in IceSort library to a uint. The uint is the key to storeStore round robin order of stamping.

Line: #100
```
   mapping (uint => uint) public stampingCount; 
```
This line is a mapping of uint to uint that acts as a counter for the sharing files. It is named stampingCount. It stores the maximum file index reached to provide looping functionality

Line: #105
```
mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) public stampingsReq;
```
This is a 2 dimentional mapping named stampingsReq that maps the mapping of the GlobalRecord struct in IceGlobal library to a uint. The uint is the key to retrieve mapping of stanping requests of each user(EIN). It look up the incoming stamping requests stored on a given index.

Line: #99
```
mapping (uint => mapping(uint => IceSort.SortOrder)) public stampingReqOrder;
```
This is a 2 dimentional mapping named stampingOrder that maps the mapping of the SortOrder struct in IceSort library to a uint. The uint is the key to store round robin order of stamping requests

Line: #100
```
mapping (uint => uint) public stampingReqCount;
```
This line is a mapping of uint to uint that acts as a counter for the sharing files. It is named stampingReqOrder. It stores the maximum file index reached to provide looping functionality


Line: #112
```
mapping (uint => mapping(uint => bool)) public whitelist;
```
This line has a mapping named whitelist that  maps to a bool for each user to know if has been whitelisted or not

Line: #113
```
mapping (uint => mapping(uint => bool)) public blacklist;
```
This line has a mapping named blacklist that  maps to a bool for each user to know if has been whitelisted or not

Lines: #117 - #118
```
SnowflakeInterface public snowflake;
IdentityRegistryInterface public identityRegistry;
```
These lines references SnowflakeInterface and stores it as snowflake, it does same for IdentityRegistryInterface and stores it as identityRegistry

/* ***************
    * DEFINE EVENTS
    *************** */

Line: #124
```
event ItemHidden(uint EIN, uint fileIndex, bool status);
```
This event is trigged to give the status of an item, if it is hidden or not

Line: #127
```
event ItemShareChange(uint EIN, uint fileIndex);
```
This event is trigged to give the status of an item, if it is shared or not

Line: #129
```
event FileCreated(uint EIN, uint fileIndex, string fileName);
```
This event is trigged to give the status of a file, if it is created or not

Line: #132
```
event FileRenamed(uint EIN, uint fileIndex, string fileName);
```
This event is trigged when a file is renamed

Line: #136
```
 event FileMoved(uint EIN, uint fileIndex, uint groupIndex, uint groupFileIndex);
```
This event is trigged when a file is moved to another group

Line: #139
```
 event FileDeleted(uint EIN, uint fileIndex);
```
This event is trigged when a file is deleted

Line: #142
```
 event GroupCreated(uint EIN, uint groupIndex, string groupName);
```
This event is trigged when a group is created

Line: #145
```
event GroupRenamed(uint EIN, uint groupIndex, string groupName);
```
This event is trigged when an existing group is renamed


Line: #148
```
 event GroupDeleted(uint EIN, uint groupIndex, uint groupReplacedIndex);
```
This event is trigged when an existing group is deleted

Line: #151
```
event SharingCompleted(uint EIN, uint index1, uint index2, uint recipientEIN);
```
This event is trigged when sharing is completed

Line: #154
```
event SharingRejected(uint EIN, uint index1, uint index2, uint recipientEIN);
```
This event is trigged when sharing is rejected

Line: #157
```
event SharingRejected(uint EIN, uint index1, uint index2, uint recipientEIN);
```
This event is trigged when sharing is removed

Line: #160
```
event StampingInitiated(uint EIN, uint index1, uint index2, uint recipientEIN);
```
This event is trigged when stamping is initiated

Line: #163
```
event StampingAccepted(uint EIN, uint index1, uint index2, uint recipientEIN);
```
This event is trigged when stamping is accepted by the recipient

Line: #166
```
event StampingRejected(uint EIN, uint index1, uint index2, uint recipientEIN);
```
This event is trigged when stamping is rejected by the recipient

Line: #169
```
event StampingRevoked(uint EIN, uint index1, uint index2, uint recipientEIN);

```
This event is trigged when stamping is revoked by the recipient

Line: #172
```
 event FileTransferInitiated(uint EIN, uint index1, uint index2, uint recipientEIN);

```
This event is trigged when file transfer to another user is initiated

Line: #175
```
 event FileTransferAccepted(uint EIN, uint index1, uint index2, uint recipientEIN);

```
This event is trigged when file transfer to another user is accepted

Line: #178
```
 event FileTransferRejected(uint EIN, uint index1, uint index2, uint recipientEIN);

```
This event is trigged when file transfer to another user is rejected

Line: #181
```
 event FileTransferRevoked(uint EIN, uint index1, uint index2, uint recipientEIN);

```
This event is trigged when file transfer to another user is revoked

Line: #184
```
 event AddedToWhitelist(uint EIN, uint recipientEIN);

```
This event is trigged when whitelist is updated. When a non-owner user is added to the whitelist

Line: #185
```

event RemovedFromWhitelist(uint EIN, uint recipientEIN);

```
This event is trigged when whitelist is updated. When a non-owner user is removed to the whitelist

Line: #188
```
event AddedToBlacklist(uint EIN, uint recipientEIN);

```
This event is trigged when blacklist is updated. When a non-owner user is added to the blacklist

Line: #189
```
event RemovedFromBlacklist(uint EIN, uint recipientEIN);
```
This event is trigged when whitelist is updated. When a non-owner user is removed to the blacklist


/* ***************
* DEFINE CONSTRUCTORS AND RELATED FUNCTIONS
*************** */

Lines: #195 - #198
```
constructor (address snowflakeAddress) public {
    snowflake = SnowflakeInterface(snowflakeAddress);
    identityRegistry = IdentityRegistryInterface(snowflake.identityRegistryAddress());
}
```
The constructor is initialized with the deployed snowflake address and sets snowflake and identityRegistry.



/* ***************
* DEFINE CONTRACT FUNCTIONS
*************** */

Lines: #215 - #229
```
 function getGlobalItems(
        uint _index1, 
        uint _index2
    )
    external view
    returns (
        uint ownerEIN, 
        uint itemRecord, 
        bool isFile, 
        bool isHidden, 
        bool deleted, 
        uint sharedToCount
    ) {
        // Logic
        (ownerEIN, itemRecord, isFile, isHidden, deleted, sharedToCount) = globalItems[_index1][_index2].getGlobalItems();
    }
```
This function gets the global items info from the entire File Management System of Ice. It accepts the first(_index1) and second(_index2) index of the item as parameters and returns 
- the EIN of the user who owns the item(ownerEIN), 
- the record of that item in relation to the individual user(itemRecord), 
- if the item is file or group(isFile), 
- if the item is hidder or visible(isHidden), 
- if the item is already deleted from the individual user perspective(deleted), 
- the count of sharing the item has(sharedToCount)


Lines: #242 - #262
```
function getGlobalItemsStampingInfo(
    uint _index1, 
    uint _index2
)
external view
returns (
    uint stampingRecipient,
    uint stampingRecipientIndex,
    uint32 stampingInitiated,
    uint32 stampingCompleted,
    bool stampingRejected
) {
    // Logic
    (
        stampingRecipient, 
        stampingRecipientIndex, 
        stampingInitiated, 
        stampingCompleted, 
        stampingRejected
    ) = globalItems[_index1][_index2].getGlobalItemsStampingInfo();
}
```
This function gets the global items stamping info from the entire File Management System of Ice. It accepts the first(_index1) and second(_index2) index of the item as parameters and returns 
- the EIN of the recipient for whom stamping is requested / denied / completed(stampingRecipient), 
- the item index mapped in the mapping of stampingsReq of that recipient(stampingRecipientIndex), 
- either returns 0 (false) or timestamp when the stamping was initiated(stampingInitiated), 
- either returns 0 (false) or timestamp when the stamping was completed(stampingCompleted), 
- if the stamping was rejected by the recipient(stampingRejected), 


Lines: #270 - #287
```
 function hideGlobalItem(
        uint _index1, 
        uint _index2, 
        bool _isHidden
    ) 
    external {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);
        
        // Check Restrictions
        globalItems[_index1][_index2].condItemOwner(ein);
        
        // Logic
        globalItems[_index1][_index2].isHidden = _isHidden;
        
        // Trigger Event
        emit ItemHidden(ein, globalItems[_index1][_index2].ownerInfo.EIN, _isHidden); 
    }
```

This function gets the global items. It accepts the first(_index1) and second(_index2) index of the item  and a bool (_isHidden) as parameters.
It gets the user EIN, checks Restrictions and set the values inputed. It then emits the ItemHidden event.



Lines: #299 - #319
```
function getGlobalItemsMapping(
        uint _index1, 
        uint _index2, 
        uint8 _ofType, 
        uint8 _mappedIndex
    )
    external view
    returns (
        uint mappedToEIN, 
        uint atIndex
    ) {
        // Allocalte based on type.
        if (_ofType == uint8(IceGlobal.AsscProp.sharedTo)) {
            mappedToEIN = globalItems[_index1][_index2].sharedTo[_mappedIndex].EIN;
            atIndex = globalItems[_index1][_index2].sharedTo[_mappedIndex].index;
        }
        else if (_ofType == uint8(IceGlobal.AsscProp.stampedTo)) {
            mappedToEIN = globalItems[_index1][_index2].stampingRecipient.EIN;
            atIndex = globalItems[_index1][_index2].stampingRecipient.index; 
        }
    }
```
This function gets the info of mapping to user for a specific global item. It accepts the first(_index1) and second(_index2) index of the item, the type(where 0 is shares and 1 is stampings) and mappedIndex as the index as parameters and returns 
- the user (EIN) at mappedToEIN 
- the specific index in question(atIndex), only returns on shares types
The returned values are allocated based on type

 // 2. FILE FUNCTIONS
    /**

Lines: #330 - #339
```
function getFileIndexes(
        uint _ein, 
        uint _seedPointer,
        uint16 _limit, 
        bool _asc
    )
    external view
    returns (uint[20] memory fileIndexes) {
        fileIndexes = fileOrder[_ein].getIndexes(_seedPointer, _limit, _asc);
    }

```
This function getsthe desired amount of files indexes (max 20 at times) of an EIN. It accepts the EIN of the user(_ein), the seed of the order from which it should begin(_seedPointer), the limit of file indexes requested and (_limit), the order by which the files will be presented(_asc) and returns the array of file indexes for the specified users(fileIndexes)



Lines: #351 - #368
```
function getFileInfo(
        uint _ein, 
        uint _fileIndex
    )
    external view 
    returns (
        uint8 protocol, 
        bytes memory protocolMeta, 
        string memory fileName, 
        bytes32 fileHash, 
        bytes22 hashExtraInfo,
        uint8 hashFunction,
        uint8 hashSize,
        bool encryptedStatus
    ) {
        // Logic
        (protocol, protocolMeta, fileName, fileHash, hashExtraInfo, hashFunction, hashSize, encryptedStatus) = files[_ein][_fileIndex].getFileInfo();
}

```
This function gets the file info of an EIN. It accepts the EIN of the user(_ein), the _fileIndex is the index of the file and returns protocolMeta which contains essesntial information about the protocol if any, the name of the file along with the extension and the hash information including hashFunction, hashSize, encryptedStatus of an EIN.



Lines: #375 - #380
```
 function getFileOtherInfo(uint _ein, uint _fileIndex)
    external view
    returns (uint32 timestamp, uint associatedGroupIndex, uint associatedGroupFileIndex) {
        // Logic
        (timestamp, associatedGroupIndex, associatedGroupFileIndex) = files[_ein][_fileIndex].getFileOtherInfo();
}

```
This function gets the file info of an EIN. It accepts the EIN of the user(_ein)and the _fileIndex is the index of the file, it sets it for the user.


Lines: #387 - #392
```
function getFileTransferInfo(uint _ein, uint _fileIndex)
    external view
    returns (uint transCount, uint transEIN, uint transIndex, bool forTrans) {
        // Logic
        (transCount, transEIN, transIndex, forTrans) = files[_ein][_fileIndex].getFileTransferInfo();
}

```
This function gets the file transfer info of an EIN. It accepts the EIN of the user(_ein)and the _fileIndex is the index of the file, it sets the tranasfer info for the user.


Lines: #400 - #404
```
 function getFileTransferOwners(uint _ein, uint _fileIndex, uint _transferCount)
    external view
    returns (uint recipientEIN) {
        recipientEIN = files[_ein][_fileIndex].getFileTransferOwners(_transferCount);
}

```
This function gets the file transfer owner info of an EIN. It accepts the EIN of the user(_ein)and the _fileIndex is the index of the file and transferCount, it sets the transfer info for the user.


Lines: #417 - #498
```
function addFile(
        uint8 _op, 
        uint8 _protocol, 
        bytes memory _protocolMeta, 
        bytes32 _name, 
        bytes32 _hash,
        bytes22 _hashExtraInfo,
        uint8 _hashFunction,
        uint8 _hashSize,
        bool _encrypted, 
        bytes32 _encryptedHash, 
        uint _groupIndex
    )
    public {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);
        
        // Check constraints
        _groupIndex.condValidItem(groupCount[ein]);

        // To fill with global index if need be
        IceGlobal.GlobalRecord memory rec;
        
        // Create File
        uint nextIndex;
            
        // OP 0 - Normal | 1 - Avatar
        if (_op == 0) {
            // Reserve Global Index
            (globalIndex1, globalIndex2) = IceGlobal.reserveGlobalItemSlot(globalIndex1, globalIndex2);
        
            // Create the record
            rec = IceGlobal.GlobalRecord(globalIndex1, globalIndex2);
        
            // Create File Next Index
            nextIndex = fileCount[ein] + 1;
            
            // Add to globalItems
            globalItems.addItemToGlobalItems(rec.i1, rec.i2, ein, nextIndex, true, false, 0);
        }
        
        // Finally create the file object (EIN)
        files[ein][nextIndex].createFileObject(
            _protocolMeta,
            
            _groupIndex, 
            groups[ein][_groupIndex].groupFilesCount
        );
        
        // Assign global item record 
        files[ein][nextIndex].rec = rec;
        
        // Also create meta object
        files[ein][nextIndex].createFileMetaObject(
            _protocol,
            _name,
            _hash,
            _hashExtraInfo,
            _hashFunction,
            _hashSize,
            _encrypted
        );
        
        // OP 0 - Normal | 1 - Avatar
        if (_op == 0) {
            fileCount[ein] = files[ein][nextIndex].writeFile(
                groups[ein][_groupIndex], 
                _groupIndex, 
                fileOrder[ein], 
                fileCount[ein], 
                nextIndex, 
                ein, 
                _encryptedHash
            );
        }
        else if (_op == 1) {
            usermeta[ein].hasAvatar = true;
        }
        
        // Trigger Event
        emit FileCreated(ein, nextIndex, IceFMS.bytes32ToString(_name));
}

```
This function is used to add a file. It accepts parameters like protocol used(_protocol), the metadata used by the protocol if any(_protocolMeta), the name of the file(_name), the first split hash of the stored file(_hash1), the second split hash of the stored file(_hash2), defines if the file is encrypted or not(_encrypted), defines the encrypted public key password for the sender address(_encryptedHash), defines the index of the group of file(_groupIndex)




Lines: #417 - #498
```
function changeFileName(
        uint _fileIndex, 
        bytes32 _name
    )
    external {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);

        // Logic
        files[ein][_fileIndex].fileMeta.name = _name;

        // Trigger Event
        emit FileRenamed(ein, _fileIndex, IceFMS.bytes32ToString(_name));
}


```
This function is used to change a file name. It accepts parameters of the index where file is stored(_fileIndex) and the name of stored file(_name) and changes the name of the file. It emits the FileRenamed event afterwards.




Lines: #525 - #538
```
function moveFileToGroup(
        uint _fileIndex, 
        uint _newGroupIndex
    )
    external {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);

        // Logic
        uint groupFileIndex = files[ein][_fileIndex].moveFileToGroup(_fileIndex, groups[ein], groupOrder[ein], _newGroupIndex, globalItems);

        // Trigger Event
        emit FileMoved(ein, _fileIndex, _newGroupIndex, groupFileIndex);
}


```
This function is used to move a file to another group. It accepts parameters of the index where file is stored(_fileIndex) and the the index of the new group where file has to be moved. It emits the FileMoved event afterwards.



Lines: #544 - #554
```
function deleteFile(uint _fileIndex)
    external {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);

        // Delegate the call
        _deleteFileAnyOwner(ein, _fileIndex);
        
        // Trigger Event
        emit FileDeleted(ein, _fileIndex);
}

```
This function is used to delete a file of the owner. It accepts parameters of the index where file is stored(_fileIndex). It emits the FileDeleted event afterwards.


Lines: #561 - #581
```
function _deleteFileAnyOwner(
        uint _ein, 
        uint _fileIndex
    )
    internal {
        // Logic
        files[_ein].deleteFile(
            _ein,
            _fileIndex,
            files[_ein][_fileIndex].rec.getGlobalItemViaRecord(globalItems),
            
            fileOrder[_ein],
            fileCount,
            groups[_ein][files[_ein][_fileIndex].associatedGroupIndex],
            groupOrder[_ein][files[_ein][_fileIndex].associatedGroupIndex],
            
            shares,
            shareOrder,
            shareCount
        );
}

```
This function is used to delete file of any EIN. It accepts parameters of the owner EIN(_ein) and the index where file is stored(_fileIndex). It emits the FileDeleted event afterwards.


// 3. GROUP FILES FUNCTIONS
    /**



Lines: #592 - #602
```
  function getGroupFileIndexes(
        uint _ein, 
        uint _groupIndex, 
        uint _seedPointer, 
        uint16 _limit, 
        bool _asc
    )
    external view
    returns (uint[20] memory groupFileIndexes) {
        return groups[_ein][_groupIndex].groupFilesOrder.getIndexes(_seedPointer, _limit, _asc);
    }

```
This function is used to get all the files of an EIN associated with a group. It accepts parameters of the owner EIN(_ein) and the group index where group is stored(_groupIndex_), the seed of order from which it should begin(_seedPointer), the limit of file indexes requested(_limit) and the order by which the files will be presented(_asc). 


// 4. GROUP FUNCTIONS
    /**


Lines: #612 - #623
```
    function getGroup(
        uint _ein, 
        uint _groupIndex
    )
    external view
    returns (
        uint index, 
        string memory name
    ) {
        // Logic
        (index, name) = groups[_ein][_groupIndex].getGroup(_groupIndex, groupCount[_ein]);
    }

```
This function is used to return group info for an EIN. It accepts parameters of the owner EIN(_ein) and the group index where group is stored(_groupIndex_), it returns the index of the group(index), the name associated with the group(name).


Lines: #633 - #637
```
 function getGroupIndexes(uint _ein, uint _seedPointer, uint16 _limit, bool _asc)
    external view
    returns (uint[20] memory groupIndexes) {
        groupIndexes = groupOrder[_ein].getIndexes(_seedPointer, _limit, _asc);
}

```
This function is used to return group indexes used to retrieve info about group. It accepts parameters of the owner EIN(_ein), the seed of order from which it should begin(_seedPointer), it returns the index of the group(index), the indexes of the groups associated with the ein in the preferred order(groupIndexes).


Lines: #643 - #662
```
 function createGroup(string memory _groupName)
    public {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);
        
        // Logic
        uint nextGroupIndex;
        (globalIndex1, globalIndex2, nextGroupIndex) = groups[ein].createGroup(
            ein, 
            _groupName, 
            groupOrder[ein], 
            groupCount, 
            globalItems,
            globalIndex1,
            globalIndex2
        );
        
        // Trigger Event
        emit GroupCreated(ein, nextGroupIndex, _groupName);
    }


```
This function is used to create a new group for the user. It accepts parameters of the name of the group(_groupName).
It emits the GroupCreated event.


Lines: #688 - #710
```
    function renameGroup(
        uint _groupIndex, 
        string calldata _groupName
    )
    external  {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);

        // Logic
        groups[ein][_groupIndex].renameGroup(_groupIndex, groupCount[ein], _groupName);
        
        // Trigger Event
        emit GroupRenamed(ein, _groupIndex, _groupName);
    }

```
This function is used to rename an existing Group for the user / ein. It accepts parameters of the new name of the group(_groupName) and describes the associated index of the group for the user / ein(_groupIndex). It emits the GroupRenamed event.


Lines: #688 - #710
```
   function deleteGroup(uint _groupIndex)
    external {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);

        // Logic
        uint currentGroupIndex = groups[ein].deleteGroup(
            ein,
            
            _groupIndex,
            groupOrder[ein], 
            groupCount, 
            
            shares,
            shareOrder,
            shareCount,
            
            globalItems
        );
        
        // Trigger Event
        emit GroupDeleted(ein, _groupIndex, currentGroupIndex);
    }
```
This function is used to delete an existing group for the user / ein. It accepts parameters of the associated index of the group for the user / ein(_groupIndex). It emits the GroupDeleted event.

 // 4. SHARING FUNCTIONS
    /**

Lines: #719 - #733
```
   function shareItemToEINs(uint[] calldata _toEINs, uint _itemIndex, bool _isFile)
    external {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);
        
        // Check if item is file or group and accordingly check if Item is valid & Logic
        if (_isFile == true) { 
            _itemIndex.condValidItem(fileCount[ein]);
            shares.shareItemToEINs(globalItems, shareOrder, shareCount, blacklist, files[ein][_itemIndex].rec, ein, _toEINs);
        }
        else {
            _itemIndex.condValidItem(groupCount[ein]);
            shares.shareItemToEINs(globalItems, shareOrder, shareCount, blacklist, groups[ein][_itemIndex].rec, ein, _toEINs);
        }
    }
```
This function is used to share an item to other users, always called by owner of the Item. It accepts parameters of the array of EINs which the item should be shared to(_toEINs), the index of the item to be shared to(_itemIndex), indicates if the item is file or group(_isFile)



Lines: #741 - #768
```
 function removeShareFromEINs(
        uint[] memory _fromEINs, 
        uint _itemIndex, 
        bool _isFile
    )
    public {
        // Get user EIN
        uint ein = identityRegistry.getEIN(msg.sender);

        // Check if item is file or group and accordingly check if Item is valid & Logic
        IceGlobal.GlobalRecord memory rec;
        if (_isFile == true) { 
            _itemIndex.condValidItem(fileCount[ein]);
            rec = files[ein][_itemIndex].rec;
        }
        else {
            _itemIndex.condValidItem(groupCount[ein]);
            rec = groups[ein][_itemIndex].rec;
        }
        
        shares.removeShareFromEINs(
            ein, 
            _fromEINs,
            globalItems[rec.i1][rec.i2], 
            shareOrder, 
            shareCount
        );
    }
```
This function is used to remove a shared item from the multiple user's mapping, always called by owner of the Item. It accepts parameters of the array of EINs which the item should be removed from sharing(_fromEINs), the index of the item on the owner's mapping(_itemIndex), indicates if the item is file or group(_isFile)



Lines: #774 - #786
```
 function removeSharingItemBySharee(uint _itemIndex) 
    external {
        // Logic
        uint shareeEIN = identityRegistry.getEIN(msg.sender);
        IceGlobal.Association storage globalItem = shares[shareeEIN][_itemIndex].getGlobalItemViaRecord(globalItems);
        
        shares[shareeEIN].removeSharingItemBySharee( 
            shareeEIN,
            globalItem, 
            shareOrder[shareeEIN], 
            shareCount
        );
    }
```
This function is used to remove shared item by the user to whom the item is shared. It accepts parameter the index of the item on the  in shares(_itemIndex)

 // 5. STAMPING FUNCTIONS
    /**

Lines: #795 - #852
```
function initiateStampingOfItem(
        uint _itemIndex,
        bool _isFile,
        uint _recipientEIN
    )
    external {
        // Logic
        if (_isFile) {
            stampingsReq[_recipientEIN].initiateStampingOfItem(
                stampingReqOrder[_recipientEIN],
                stampingReqCount,
                
                identityRegistry.getEIN(msg.sender),
                _recipientEIN,
                _itemIndex,
                fileCount[identityRegistry.getEIN(msg.sender)],
                
                files[identityRegistry.getEIN(msg.sender)][_itemIndex].rec.getGlobalItemViaRecord(globalItems),
                files[identityRegistry.getEIN(msg.sender)][_itemIndex].rec,
                
                blacklist,
                identityRegistry
            );
            
            // Trigger Event
            emit StampingInitiated(
                identityRegistry.getEIN(msg.sender), 
                files[identityRegistry.getEIN(msg.sender)][_itemIndex].rec.i1, 
                files[identityRegistry.getEIN(msg.sender)][_itemIndex].rec.i2, 
                _recipientEIN
            );
        }
        else {
            stampingsReq[_recipientEIN].initiateStampingOfItem(
                stampingReqOrder[_recipientEIN],
                stampingReqCount,
                
                identityRegistry.getEIN(msg.sender),
                _recipientEIN,
                _itemIndex,
                groupCount[identityRegistry.getEIN(msg.sender)],
                
                groups[identityRegistry.getEIN(msg.sender)][_itemIndex].rec.getGlobalItemViaRecord(globalItems),
                groups[identityRegistry.getEIN(msg.sender)][_itemIndex].rec,
                
                blacklist,
                identityRegistry
            );
        
            // Trigger Event
            emit StampingInitiated(
                identityRegistry.getEIN(msg.sender), 
                groups[identityRegistry.getEIN(msg.sender)][_itemIndex].rec.i1, 
                groups[identityRegistry.getEIN(msg.sender)][_itemIndex].rec.i2, 
                _recipientEIN
            );
        }
    }
    
```
This function is used to initiate stamping of an item by the owner of that item. It accepts parameters of the index of the item on the owner's mapping(_itemIndex), indicates if the item is file or group(_isFile), the recipient EIN of the user who has to stamp the item. IT emits the stampingInitiated event afterwards.


Lines: #858 - #887
```
      function acceptStamping(
        uint _stampingReqIndex
    )
    external {
        // Get user EIN
        uint recipientEIN = identityRegistry.getEIN(msg.sender);
        
        // Logic 
        stampings[recipientEIN].acceptStamping(
            stampingOrder[recipientEIN],
            stampingCount,
            
            stampingsReq[recipientEIN],
            stampingReqOrder[recipientEIN],
            stampingReqCount,
            
            stampingsReq[recipientEIN][_stampingReqIndex].getGlobalItemViaRecord(globalItems),
            
            recipientEIN,
            _stampingReqIndex
        );
        
        // Trigger Event
        emit StampingAccepted(
            identityRegistry.getEIN(msg.sender), 
            stampingsReq[recipientEIN][_stampingReqIndex].i1, 
            stampingsReq[recipientEIN][_stampingReqIndex].i2, 
            recipientEIN
        );
    } 
```
This function is used to initiate stamping of an item by the owner of that item. It accepts parameters of the the index of the item present in the Stamping Requests mapping of the recipient(_stampingReqIndex), indicates if the item is file or group(_isFile), the recipient EIN of the user who has to stamp the item. I emits the stampingccepted;

### References
- https://github.com/HydroBlockchain/protocol-whitepapers/blob/master/Ice/Ice_DRAFT.md

- https://github.com/HydroBlockchain/protocol-whitepapers/tree/master/Ice


````
