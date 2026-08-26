Cruze Auto


Cruze Auto is an Android automation application designed to automate USSD-based transactions triggered by incoming M-Pesa transaction messages.


The application combines SMS transaction detection, product/USSD mapping, AccessibilityService automation, transaction tracking, airtime management, training, and configurable execution controls into a single Android application.



Features


Transaction Automation




Detect incoming M-Pesa transaction messages.


Extract transaction information such as:



Sender/client phone number


Transaction amount


Sender name where available






Match captured transaction information against configured products.


Automatically execute the corresponding USSD transaction.


Replace configured placeholders such as CR with the required phone number.


Track the complete transaction lifecycle.




USSD Modes


Simple USSD Mode


Executes the complete USSD code in a single operation.


Example:


*544*5*2*0712345678*2*1#



Advanced USSD Mode


Executes USSD transactions step by step using Android AccessibilityService.


Example:


*180*5*2*CR*2*1#



The application can:




Start the USSD session.


Wait for the carrier menu.


Detect the displayed menu.


Identify the trained option.


Enter the required response.


Press SEND.


Wait for the next carrier response.


Continue until the transaction reaches its final step.


Detect success or failure.


Automatically mark the transaction as completed when the final required action succeeds.





Advanced USSD Training


The training system allows the application to learn carrier menu structures.


Training should store the semantic option associated with a step, rather than storing the entire carrier menu as the keyword.


For example:


Carrier menu:

1. Buy for self
2. Buy for other number
3. Check balance
...



If the required option is:


2. Buy for other number



the training data should associate the step with:


Keyword: Buy for other number
Option: 2



This prevents entire menus from being incorrectly duplicated across multiple training steps.


Training Requirements




Each product can have its own training configuration.


Each USSD step should maintain its own learned mapping.


Carrier menu text should be used for matching, not blindly stored as the keyword.


Training must not execute the final purchasing step.


Trained and untrained products should be clearly distinguishable.


Training should tolerate reasonable carrier menu changes.


The application should wait for the carrier interface before entering data.


Random or unrelated accessibility text must never be interpreted as the next USSD step.





AccessibilityService


Advanced USSD automation uses Android's AccessibilityService.


The service is responsible for:




Detecting USSD dialogs.


Reading visible accessibility nodes.


Finding input fields.


Finding SEND/confirmation controls.


Entering required values.


Clicking the correct controls.


Waiting for the next USSD state.


Detecting success messages.


Detecting error messages.


Closing invalid or failed USSD sessions.


Preventing interaction with unrelated applications.




The automation engine must only perform USSD actions when the expected carrier interface is detected.



Transaction State Management


Transactions can be categorized into:




Pending


Executing


Completed


Failed


Unmatched




The transaction state should only advance when the corresponding operation has actually succeeded.


For example:


Pending
   ↓
Executing
   ↓
Carrier response detected
   ↓
Final action successful
   ↓
Completed



A transaction must not be marked as completed merely because the USSD dialog was opened.



Automatic Completion


After the final required USSD action:




Detect the carrier's final response.


Verify that the response represents a successful transaction.


Confirm that no additional required USSD step remains.


Mark the transaction as:




COMPLETED



If the carrier reports an error, the transaction must instead become:


FAILED




Airtime Management


The application can support automatic airtime checking.


Features include:




Manual airtime checking.


Automatic airtime refresh.


Configurable refresh interval.


Dashboard airtime display.


Airtime USSD configuration.




Example airtime USSD:


*144#



The automatic refresh interval must be greater than 5 minutes.



Dual SIM Support


The application is designed to support devices with multiple SIM cards.


The selected SIM should be respected when initiating USSD operations.


The application should avoid unnecessarily displaying the Android SIM-selection dialog when a valid SIM has already been selected in the application.



Dashboard


The dashboard provides a central view of application activity.


It can display:




Completed transactions


Failed transactions


Executing transactions


Pending transactions


Unmatched transactions


Captured clients


Airtime information


Product/offer information




Transaction counters can be organized by date and reset for the new day without deleting historical transaction records.



Settings


The application provides configurable settings for automation and appearance.


Possible settings include:


Automation




Simple/Advanced USSD mode


Execution timeout


Delay between USSD actions


Selected SIM


Automatic airtime refresh


Airtime refresh interval


Product automation controls




Appearance




Dark mode


Application font size


Dashboard background customization


Color selection




Font-size settings should apply consistently throughout the application rather than only inside the Settings screen.



Performance


Performance is a major design requirement.


The application should prioritize:




Minimal execution latency.


Fast USSD state detection.


Efficient accessibility-node processing.


Minimal unnecessary UI updates.


Smooth scrolling.


Immediate user interactions.


Lightweight background processing.


Efficient database queries.


Avoidance of unnecessary animations.




Timing-sensitive automation should use reliable synchronization with the carrier UI rather than arbitrary long delays.



Architecture


The project should follow a modular architecture.


Each new feature should preferably exist as an isolated component/module/entity and communicate with existing functionality through controlled interfaces.


Example:


Cruze Auto
│
├── UI
│   ├── Dashboard
│   ├── Transactions
│   ├── Products
│   ├── Settings
│   └── Training
│
├── Automation
│   ├── USSD Engine
│   ├── Accessibility Service
│   ├── USSD State Detector
│   ├── Menu Matcher
│   └── Execution Controller
│
├── Transactions
│   ├── Transaction Manager
│   ├── Transaction Repository
│   └── Transaction State
│
├── Messaging
│   └── M-Pesa SMS Receiver
│
├── Training
│   ├── Training Manager
│   ├── Keyword Mapper
│   └── Menu-State Mapping
│
├── Airtime
│   └── Airtime Manager
│
├── Products
│   └── Product/Offer Manager
│
└── Database
    ├── Transactions
    ├── Products
    ├── Training Data
    └── Settings



Existing working components should not be unnecessarily modified when adding new functionality.



Technology Stack


Recommended primary technologies:




Kotlin


Android SDK


Jetpack Compose for modern UI


Android AccessibilityService


Room / SQLite for local data


Coroutines for asynchronous operations


Android SMS APIs where permitted


Android Telephony APIs


Material Design




Kotlin is preferred because the application depends heavily on Android-native capabilities such as AccessibilityService, telephony, SIM management, permissions, and background execution.



Permissions


Depending on enabled functionality, the application may require permissions or special access for:




SMS access


Phone/telephony functionality


Notifications


AccessibilityService


Background operation


Overlay functionality where required


Other Android system capabilities




The application should explain why each permission is required.


If a required permission is unavailable, the application should provide a clear error message and direct the user to the appropriate Android Settings screen.



Error Handling


The application should gracefully handle:




USSD timeout


Carrier network failure


Invalid USSD option


Changed carrier menu


AccessibilityService unavailable


Accessibility node not found


SIM unavailable


Wrong SIM


M-Pesa message parsing failure


Unmatched transaction


Database errors


Application restart during execution




Errors should never silently mark a transaction as successful.



Security and Reliability


The application should:




Avoid exposing sensitive transaction information unnecessarily.


Validate transaction data before automation.


Validate the expected USSD state before each action.


Prevent duplicate transaction execution.


Maintain transaction state persistently.


Recover safely after application restarts where possible.


Never assume an operation succeeded without confirmation.





Development Principles


1. Isolated Features


New features should be implemented independently whenever practical.


Existing Feature
       ↑
       │ controlled interface
       │
New Feature Module



Avoid directly modifying stable functionality unless the change is required for integration.


2. Single Responsibility


Each component should have one primary responsibility.


For example:


USSDService
    → observes USSD interface

USSDStateDetector
    → determines current USSD state

MenuMatcher
    → identifies trained option

USSDExecutor
    → performs the required action

TransactionManager
    → updates transaction state



3. Fail Safely


If the application cannot confidently identify the correct carrier menu or transaction state, it should stop rather than perform an incorrect action.


4. Confirm Before Completion


A transaction should only become COMPLETED after successful confirmation of the final required operation.



Project Goals


The long-term goal of Cruze Auto is to provide a fast, reliable and configurable Android automation platform for USSD-based transactions while maintaining a clear separation between:


Transaction Detection
        ↓
Product Matching
        ↓
USSD Training
        ↓
USSD Execution
        ↓
Carrier Verification
        ↓
Transaction Completion



The system should remain stable as new features are introduced and should minimize regressions to existing functionality.



Build


Open the project in Android Studio and allow Gradle to synchronize the project.


Recommended environment:


Android Studio
Kotlin
Gradle
Android SDK
JDK 17+



Build the debug version using Android Studio or Gradle.


Example:


./gradlew assembleDebug



For Windows:


gradlew.bat assembleDebug




Testing


Before releasing a new version, test:




Simple USSD execution.


Advanced USSD execution.


Training.


Changed carrier menus.


M-Pesa SMS detection.


Product matching.


Phone-number replacement.


Dual-SIM operation.


Successful transactions.


Failed transactions.


USSD timeout.


Accessibility permission changes.


Application restart during execution.


Airtime checking.


Dashboard counters.


Font-size changes throughout the entire application.


Dark mode.


Database persistence.





Status


Development


Cruze Auto is actively being developed with emphasis on:




Reliable Advanced USSD automation


Accurate training and menu mapping


Fast transaction execution


Robust AccessibilityService integration


Transaction verification


Modular feature architecture


Performance and stability





License


This project is proprietary software unless otherwise specified by the project owner.


Unauthorized copying, redistribution, modification, or commercial use is not permitted without permission from the project owner.



Author


Cruze Labs


Project:


Cruze Auto

