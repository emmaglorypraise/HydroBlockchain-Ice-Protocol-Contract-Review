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














### References
- https://github.com/HydroBlockchain/protocol-whitepapers/blob/master/Ice/Ice_DRAFT.md

- https://github.com/HydroBlockchain/protocol-whitepapers/tree/master/Ice


````
