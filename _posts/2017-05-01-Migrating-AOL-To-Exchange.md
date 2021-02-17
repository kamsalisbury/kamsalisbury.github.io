Small businesses and solo-entrepreneurs still use AOL. They began using email long ago when AOL was marketed with a CDROM disc and the Internet was accessed using a FAX modem. Migrating AOL email accounts to Exchange email is possible with several manual steps.

## Import AOL email to Exchange
Connect both the AOL and new Exchange email accounts to the same Outlook client profile, then drag-n-drop the emails from AOL email folders to Exchange.

## Importing AOL Contacts
AOL email contacts and contact groups will not migrate automatically, here are the steps to migrate them.

1. Exporting AOL Mail Contacts
```
From AOL Mail, view Contacts
Click More, then Export
Save the contacts.csv file
```
*NOTE:* This export procedure does not export the Contact Lists from the AOL Mail Contacts! Contacts.csv does not have the Contact List groups saved in it!

2. Resolve issue with contacts.csv
Exporting AOL Mail Address Book will not import to Outlook on Windows. Outlook responds with an error. [https://support.microsoft.com/en-us/help/2799195/outlook-shows-a-translation-error-during-the-import-of-a-csv-file](https://support.microsoft.com/en-us/help/2799195/outlook-shows-a-translation-error-during-the-import-of-a-csv-file)

“Translation Error. A file error has occurred in the Comma Separated Values translator while initializing a translator to build a field map. Outlook was unable to retrieve the data from the file *filename*. Verify that you have the correct file, that you have permission to open it, and that it is not open in another program."

Why? Because AOL Mail exports a Comma Separated Value (CVS) file that does not use the CR+LF (Carriage Return + Line Feed) control characters to represent line breaks. You cannot see them visually in Notepad or Excel, but the end of each line of that exported file has a special character that means “this is the end of this line in this file”, but the special character is wrong for use on a Windows computer. Fix the contacts.csv file by opening the file with Microsoft Excel, then Save As a new file, contacts-corrected.csv

3. Import contacts-corrected.csv to Outlook
Follow the official contacts import procedure [https://support.office.com/en-us/article/import-contacts-to-Outlook-bb796340-b58a-46c1-90c7-b549b8f3c5f8](https://support.office.com/en-us/article/import-contacts-to-Outlook-bb796340-b58a-46c1-90c7-b549b8f3c5f8)

## Copy AOL Mail Contact Lists (Groups)
1. From AOL Mail Contacts, scroll down to the first Contact List
2. Click the Contact List, then click Email List. AOL Mail displays a new email to that specific Contact List
3. Click inside the To: box of the new email, then CTRL+A to select all the emails in the To: box. For MacOS, CMD+A is equivalent to CTRL+A. CTRL+C to copy the selected emails to the Windows clipboard. For MacOS, CMD+C is equivalent to CTRL+C.
4. Open a new Notepad file, CTRL+V to paste the copied emails into the Notepad application. For MacOS, CMD+V is equivalent to CTRL+V.
5. In Notepad, CTRL+H to replace all commas “,” with semicolons “;”. CTRL+A to select all emails again, CTRL+C to copy the revised list of emails to the Windows clipboard.
6. Open Outlook, from the Contacts view, click New Contact Group. Enter the same Contact List name for the new Contact Group name.
7. Click Add Members, then from Outlook Contacts, Click inside the Members box, CTRL+V to paste the emails copied form Notepad Click Save & Close.
8. Return to step 1 of this section and repeat this section's procedure for each AOL Mail Contact List.

*TIP* I recommend saving the Notepad file for future reference, name the file the same as the AOL Mail Contact List name. After the AOL Mail account is deactivated, the contacts-corrected.csv and each saved contact list file will be the only reference to the past AOL Mail Contacts content.
