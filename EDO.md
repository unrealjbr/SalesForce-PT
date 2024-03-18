# Extract Data from Objects

Note: From this point onwards we will be using Intruder quite a bit. Each `message` payload will contain a MARKER value which is what you should surround the Intruder markers with, to save myself repeating it every time.

Send this repeater request to the intruder. Within the ‘Positions’ tab, modify the `message` parameter value to the following:

{"actions":[{"id":"123;a","descriptor":"serviceComponent://ui.force.components.controllers.lists.selectableListDataProvider.SelectableListDataProviderController/ACTION$getItems","callingDescriptor":"UNKNOWN","params":{"entityNameOrId":"MARKER","layoutType":"FULL","pageSize":100,"currentPage":0,"useTimeout":false,"getCount":false,"enableRowActions":false}}]}
## it can be found at extract.txt file aswell


The `$getItems` method in this specific controller is only one example of a built-in method that can be used to extract total information from an object, there are plenty however this is the one I typically use. In this payload I’m using pretty much the minimum required parameters for it to work. The full definition for this method and others will be provided at the end of the article. Here’s a little overview of the important parameters:

    entityNameOrId - Object name

    getCount - if set to ‘true’, will return the number of records returned

    pageSize - The larger the number, the greater potential number of records returned. Capped at 1000.

    currentPage - If you’ve capped the pageSize but there are more records, incrementing the currentPage value will return the records for the next page

Once you’re happy with these values, ensure that the `MARKER` string is surrounded by the intruder markers. Within the ‘Payloads’ tab, paste the custom objects into the Simple List, along with the following:
```
Case
Account
User
Contact
Document
ContentDocument
ContentVersion
ContentBody
CaseComment
Note
Employee
Attachment
EmailMessage
CaseExternalDocument
Attachment
Lead
Name
EmailTemplate
EmailMessageRelation
```
## this is just an example, maybe theres a diffs in your case.



A full list of default Salesforce objects can be found here, as there is likely some I am missing.

Finally, start the attack! Once the attack is complete, I would re-order it by response length from highest to lowest, as responses of <12,000 typically are either:

    You do not have access to the object

    The only record returned is your own (Guest)

Below is an example `User` object in which the response length indicates a leak:
example_user.PNG

Certain fields in the ‘User’ object will contain null, as they were either not supplied or have additional restrictions as a result of a Salesforce security update as mentioned before. But PII is nearly always available through the ‘Name’, ‘FirstName’, ‘LastName’ fields and occasionally ‘Phone’. In addition to this, some custom fields may be disclosed. Prior to reporting this issue, it’s paramount to ensure this information is not already accessible publicly. If the community has a discussion board where users can post from profiles, this information is likely already accessible. So ensure that throughout the exploitation process, you are not reporting a ‘non issue’. 

Specific objects will return IDs, particularly those related to attachments. Here is how to utilise them (These paths are relative to the base path, not the aura endpoint):

    Document - Prefix 015 - /servlet/servlet.FileDownload?file=ID

    ContentDocument - Prefix 069 - /sfc/servlet.shepherd/document/download/ID

    ContentVersion - Prefix 068 - /sfc/servlet.shepherd/version/download/ID
