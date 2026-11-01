- Five type of Windows file systems: **FAT12**, **FAT16**,**FAT32**,**NTFS**, and **exFAT**.
- FAT16 and FAT12 are no longer used in modern Windows. Main focus is the NTFS.
- FAT32 (**File Alloction Table**) is widely used across many types of storage devices such as USB memory stick. **32 in the name FAT32 says that it use 32 bits of data for identifying data clusters on a storage device.**
	- Pros of FAT32:
		- Device Compatibility - it can be use on computers, digital cameras, gaming consoles, smartphones, tablets, and more.
		- Operating system cross compatibility - It works on all Windows OS starting from Windows 95 and is also supported by MacOS and Linux.
	- Cons of FAT32:
		- Can only be used with files that are less than 4GB.
		- No built-in data protection or file compression features.
		- Must use third-party tools for file encryption.
- **NTFS (New Technology File System)**: 
	- Default Windows File system since Windows NT 3.1.
	- Making up for the shortcomings of FAT32, and also has better support for metadata and better performance due to improved data structuring.
	- Pros of NTFS:
		- NTFS is reliable and can restore the consistency of the file system in the event of system failure or power loss.
		- Provides security by allowing us to set granular permissions on both files and folders.
		- Support very large-sized partitions.
		- Has journaling built-in, meaning that file modifications are logged.
	- Cons of NTFS:
		- Most mobile devices do not support NTFS.
		- Older media devices such TVs and digital cameras do not offer support for NTFS storage devices.

### Permissions

| Permission Type      | Description                                                                                                                                                                                                                                                                                                                                                                                |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Full Control         | Allows reading, writing, changing, deleting of files/folders.                                                                                                                                                                                                                                                                                                                              |
| Modify               | Allows reading, writing, and deleting of files/folders.                                                                                                                                                                                                                                                                                                                                    |
| List Folder Contents | Allows for viewing and listing folders and subfolders as well as executing files. Folders only inherit this permission.                                                                                                                                                                                                                                                                    |
| Read and Execute     | Allows for viewing and listing files and subfolders as well as executing files. Files and folders inherit this permission.                                                                                                                                                                                                                                                                 |
| Write                | Allows for adding files to folders and subfolders and writing to a file.                                                                                                                                                                                                                                                                                                                   |
| Read                 | Allows for viewing and listing of folders and subfolders and viewing a file's contents.                                                                                                                                                                                                                                                                                                    |
| Traverse Folder      | This allows or denies the ability to move through folders to reach other files or folders. For example, a user may not have permission to list the directory contents or view files in the documents or web apps directory in this example c:\users\bsmith\documents\webapps\backups\backup_02042020.zip but with Traverse Folder permissions applied, they can access the backup archive. |
- Files and folders inherit the NTFS permissions of their parent folder for ease of administration.

### Integrity Control Access Control List (icacls)
- Files and folders in Windows can be managed using the File Explorer GUI.
- **icacls utility** is a command line utility for NTFS permissions over files and folders.
- `icacls C:\Windows` to list NTFS permissions.
```
	C:\htb> icacls c:\windows
	c:\windows NT SERVICE\TrustedInstaller:(F)
	           NT SERVICE\TrustedInstaller:(CI)(IO)(F)
	           NT AUTHORITY\SYSTEM:(M)
	           NT AUTHORITY\SYSTEM:(OI)(CI)(IO)(F)
	           BUILTIN\Administrators:(M)
	           BUILTIN\Administrators:(OI)(CI)(IO)(F)
	           BUILTIN\Users:(RX)
	           BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
	           CREATOR OWNER:(OI)(CI)(IO)(F)
	           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(RX)
	           APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
	           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(RX)
	           APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APPLICATION PACKAGES:(OI)(CI)(IO)(GR,GE)
	
	Successfully processed 1 files; Failed processing 0 files
```

- The resource access level is listed after each user in the output.
	- `(CI)`: **container** **inherit**
	- `(OI)`: **object** **inherit**
	- `(IO)`: **inherit** **only**
	- `(NP)`: **do not propagate inherit**
	- `(I)`: **permission inherited from parent container**
- In the above example, `NT Authority\System` has object inherit, container inherit, inherit only and full access permissions. This means that this account has full control over all the file system object in this directory and sub-directories.
-  Basic access permissions:
	- **F** : Full access.
	- **D** : Delete access.
	- **N** : no access.
	- **M** : modify access.
	- **RX** : read and execute access.
	- **R** : read-only access.
	- **W** : write-only access.
- `icacls c:\users /grant joe:f` This gives joe full access to \users folder
- `icacls c:\users /remove joe` Revokes the permissions.
Syntax
   Add or remove permissions:
```
      ICACLS Name
        _ [**/grant**[:r] _User_:_Permission_[...]]
            [**/deny** _User_:_Permission_[...]]
               [**/remove**[:g|:d]] _User_[...]]
                  [**/inheritance**:e|d|r ]
                     [**/setintegritylevel** _Level_[...]]
                        [/T] [/C] [/L] [/Q]
```