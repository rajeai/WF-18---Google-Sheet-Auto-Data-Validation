Workflow Documentation: Google Sheet Auto Data Validation
Workflow Name: Google Sheet Auto Data Validation
Objective: Automatically validate Google Sheets data, flag invalid records, update the sheet, and send Gmail alerts.
1. Architecture
Google Sheets Trigger
   ↓
IF Name Empty?
 ├─True → Set(Error='Name is empty') ┐
 └─False
      ↓
IF Email Empty?
 ├─True → Set(Error='Email is empty') ┐
 └─False
      ↓
IF Email Format?
 ├─False → Set(Error='Invalid Email Format') ┐
 └─True
      ↓
IF Phone Valid?
 ├─False → Set(Error='Invalid Phone Number') ┐
 └─True
      ↓
IF Age Empty?
 ├─True → Set(Error='Age is empty') ┐
 └─False
      ↓
IF Age Range?
 ├─False → Set(Error='Age must be between 18 and 100') ┐
 └─True → Set(Validation_Status='Valid')
All invalid Set nodes → Merge(Append) → Gmail → Google Sheets Update (Invalid)
Valid Set → Google Sheets Update (Valid)

2. Google Sheet Structure
Name	Email	Phone	Age	City	Validation_Status	Error
3. Nodes
Google Sheets Trigger
Trigger when a row is added/updated. Uses row_number from trigger.
IF - Name Empty
Condition: {{$json.Name}} is empty
Set - Name Error
Validation_Status=Invalid, Error='Name is empty', Include Other Input Fields=ON
IF - Email Empty
{{$json.Email}} is empty
Set - Email Error
Validation_Status=Invalid, Error='Email is empty'
IF - Email Regex
Regex: ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$
Set - Invalid Email
Validation_Status=Invalid, Error='Invalid Email Format'
IF - Phone Regex
Regex: ^[0-9]{10}$ ; Convert types where required=ON
Set - Invalid Phone
Validation_Status=Invalid, Error='Invalid Phone Number'
IF - Age Empty
{{$json.Age}} is empty
Set - Age Empty
Validation_Status=Invalid, Error='Age is empty'
IF - Age Range
Age>=18 AND Age<=100
Set - Age Range Error
Validation_Status=Invalid, Error='Age must be between 18 and 100'
Set - Valid
Validation_Status=Valid; Include Other Input Fields=ON
Merge
Mode: Append. Merge all invalid branches.
Gmail
Subject: Google Sheet Validation Alert. Body includes Name, Email, Phone, Age, City, Error.
Google Sheets Update (Valid)
Match on row_number. Update Validation_Status=Valid and Error=''
Google Sheets Update (Invalid)
Match on row_number. Update Validation_Status=Invalid and Error field.
4. Important Expressions
Name: {{$json.Name}}
Email: {{$json.Email}}
Phone: {{$json.Phone}}
Age: {{ Number($json.Age) }}
Row Number: {{$json.row_number}}
Validation_Status: {{$json.Validation_Status}}
Error: {{$json.Error}}

5. Gmail Template
Invalid data detected.

Validation Status: {{ $json.Validation_Status }}
Error: {{ $json.Error }}

Name: {{ $json.Name }}
Email: {{ $json.Email }}
Phone: {{ $json.Phone }}
Age: {{ $json.Age }}
City: {{ $json.City }}

6. Test Cases
•	Empty Name -> Invalid + Email alert
•	Empty Email -> Invalid + Email alert
•	Invalid Email Format -> Invalid + Email alert
•	Invalid Phone -> Invalid + Email alert
•	Empty Age -> Invalid + Email alert
•	Age >100 -> Invalid + Email alert
•	Correct invalid data -> Validation_Status becomes Valid and Error is cleared
7. Bugs Fixed During Development
•	Changed matching from Name to row_number.
•	Cleared Error field when record becomes Valid.
•	Enabled Convert types where required for phone regex.
•	Removed change_type because trigger returned empty values.
•	Used Merge(Append) instead of multiple Gmail nodes.
8. Production Improvements
•	Store validation logs in a Validation_Log sheet.
•	Prevent duplicate Gmail alerts.
•	Add timestamp and user columns.
•	Replace long IF chain with Code/Switch node for scalability.
