---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<img src="images/ArchitectureDiagram.png" width="450" />

The ***Architecture Diagram*** given above explains the high-level design of the App. Given below is a quick overview of each component.

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document can be found in the [diagrams](https://github.com/se-edu/addressbook-level3/tree/master/docs/diagrams/) folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.

</div>

**`Main`** has two classes called [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java). It is responsible for,
* At app launch: Initializes the components in the correct sequence, and connects them up with each other.
* At shut down: Shuts down the components and invokes cleanup methods where necessary.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

The rest of the App consists of four components.

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

Each of the four components,

* defines its *API* in an `interface` with the same name as the Component.
* exposes its functionality using a concrete `{Component Name}Manager` class (which implements the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component (see the class diagram given below) defines its API in the `Logic.java` interface and exposes its functionality using the `LogicManager.java` class which implements the `Logic` interface.

![Class Diagram of the Logic Component](images/LogicClassDiagram.png)

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

The sections below give more details of each component.

### UI component

![Structure of the UI Component](images/UiClassDiagram.png)

**API** :
[`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter`, `ViewPatientBox` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class.

The `UI` component uses JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* Executes user commands using the `Logic` component.
* Listens for changes to `Model` data so that the UI can be updated with the modified data.

### Logic component

![Structure of the Logic Component](images/LogicClassDiagram.png)

**API** :
[`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

1. `Logic` uses the `AddressBookParser` class to parse the user command.
1. This results in a `Command` object which is executed by the `LogicManager`.
1. The command execution can affect the `Model` (e.g. adding a patient).
1. The result of the command execution is encapsulated as a `CommandResult` object which is passed back to the `Ui`.
1. In addition, the `CommandResult` object can also instruct the `Ui` to perform certain actions, such as displaying help to the user.

Given below is the Sequence Diagram for interactions within the `Logic` component for the `execute("delete 1")` API call.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

### Model component

![Structure of the Model Component](images/ModelClassDiagram.png)

**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

The `Model`,

* stores a `UserPref` object that represents the user’s preferences.
* stores the address book data.
* exposes an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* does not depend on any of the other three components.

### Storage component

![Structure of the Storage Component](images/StorageClassDiagram.png)

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

The `Storage` component,
* can save `UserPref` objects in json format and read it back.
* can save the address book data in json format and read it back.

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### Archive-related features

**How**<br>

![image](images/ArchiveCommandLogic.png)

After `Logic` component receives the instruction from `Ui` component, the `LogicManager` calls the `AddressBookParser`
which correctly identifies that the `ArchiveCommand` is needed in the current situation. The `ArchiveCommandParser`
checks the arguments and then creates an `ArchiveCommand` object. Next, the archiving mechanism is facilitated by the
`ArchiveCommand` object which extends `Command`. It overrides `Command#execute` in order to return
a `CommandResult` with a success message stating that the command has been successfully executed. Then, the patient to
be archived is correctly identified from the index provided and passed to the `Model` component from the `Logic`
component. The `UnarchiveCommand` works similar to the implementation of the `ArchiveCommand`.

![image](images/ArchiveCommandModel.png)

The indicator for whether a patient is archived is implemented as a boolean in the `Patient` class. Therefore, when the
`ArchiveCommand` is executed, `Model` component receives the instruction and patient to be archived from `Logic`.
Then, the `UniquePersonList` in the `AddressBook` is accessed through the `ModelManager` and the specified patient is
archived by removing them from the list, changing their `isArchived` status to true and then adding them back to the
`UniquePersonList`. The `UnarchiveCommand` works similarly except for changing the `isArchived` to false.

**Why**<br>
It was implemented in this way in order to make it easier to use the `Model#updateFilteredPersonList` in order to
display all archived patients using a suitable predicate when the `ArchiveListCommand` is called just like how the
original `ListCommand` works. Moreover, the existing `DeleteCommand` and `FindCommand` could be used with the
archived list just like how they worked with the main list.

Another possible implementation was to create 2 `UniquePersonList` objects in AddressBook, one for the main list and
one for the archived list. However, this idea was scrapped as that would involve more changes throughout the component
and involve changes in unrelated classes. This way, the changes were contained and it made more sense for the `Patient`
object to contain the information of whether or not they were archived.

### Viewing all information regarding a patient

Given below is the Sequence Diagram for interactions within the `Logic` component for the `executeCommand("view 1")` API call.

![image](https://user-images.githubusercontent.com/48408342/114147903-c3b5ca80-994b-11eb-9bf3-add78bd3fca7.png)

**How**

The viewing mechanism is facilitated by the `ViewPatientCommand` which extends `Command`. It mainly overrides `Command#execute` in order to return a `CommandResult` with a `Patient` attribute. When `MainWindow#executeCommand` is ran:
1. The command is parsed into a `CommandResult` by the `LogicManager` and passed into `MainWindow#processResult`.
2. The `CommandResult` will then trigger `MainWindow#handlePatientViewBox` since it has a patient.
3. `MainWindow#handlePatientViewBox` handles the construction of the `StackPane` containing all the patient information. It clears the `viewPatienBoxPlaceholder` in case there are Javafx nodes from the previous patient, and adds a new `ViewPatientBox` to it.
4. The constructor of `ViewPatientBox` takes in a `Person` object and extracts information such as their name, phone, address, email, tags, appointments and medical records and adds the information to the labels and Panes which will be displayed in the `ViewPatientBox` UI.

**Why**

Given below is the Sequence Diagram for interactions within the `Logic` component for the `processResult(CommandResult)` API call.

![image](https://user-images.githubusercontent.com/48408342/114144410-e47c2100-9947-11eb-895f-afd00657b5af.png)

Since `MainWindow#processResult` dictates what to show on the UI depending on the `CommandResult` as seen above, we can easily allow `MainWindow#processResult` to call `MainWindow#handlePatientViewBox` 
when it detects that a patient is present in `CommandResult`. `MainWindow#handlePatientViewBox` can then simply contruct the `StackPane` containing the patient's information.

### Creating an appointment for a patient

The implementation of creating an appointment borrows from the implementation of the `EditCommand`, since adding an appointment can be viewed as editing a patient with a new list of appointments
which includes the new appointment to be added. This avoids unnecessary repetition of implementation code.

Note: Whenever the appointments of a patient is requested, we look through the list to delete appointments that are more than a day old. 

### Creating/viewing a medical record of a patient 

Given below is the Sequence Diagram for interactions within the `Logic` component for the `executeCommand("mrec 1")` API call. The interactions for creating a 
medical record are similar to those for viewing one, which will not be elaborated on here. 

**How**

The creation of a medical record is facilitated by the `OpenMedicalRecordCommand` while viewing is facilitated by `ViewMedicalRecordCommand`, 
both of which extend `Command`. Both commands inform `MainWindow` to open a separate secondary window for the editing/viewing of the medical record. When `MainWindow#executeCommand` is ran:
1. The command is parsed into a `CommandResult` by the `LogicManager` and passed into `MainWindow#processResult`.
2. The `CommandResult` will then trigger `MainWindow#handleEdit` which tells `MainWindow` to create a separate `EditorWindow` for creating/viewing the medical record.
3. `EditorWindow` handles the construction of the new window for editing/viewing the medical record. If appropriate, such as when creating a new medical record, it allows editing of the relevant text fields
4. If the medical record has been saved, `EditorWindow` itself will create a relevant `SaveMedicalRecordCommand`
5. `EditorWindow` itself has a copy of the `commandExecutor` passed by `MainWindow`, and will execute the `SaveMedicalRecordCommand` created to save the medical record under the relevant
patient.

**Why**

One of the core functionalities of this feature is to allow usage of other commands in the `MainWindow` even while the `EditorWindow` containing the medical record is open. To facilitate this,
we give the `EditorWindow` a copy of the `MainWindow#executeCommand` method during its creation, so that after opening the record, saving can be initiated anytime by the `EditorWindow`
(but still handled by `MainWindow`) while the `MainWindow` resumes its role in the parsing and execution of commands.

In this way, the opening and saving of medical records is decoupled, allowing multiple records to be opened before one is closed, and there is no strict time dependency between the commands.

_{more aspects and alternatives to be added}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

**Target user profile**:

* Clinics where patient information is managed in a written medium such as pen and paper
* has many patients to manage
* has a need to edit and maintain patients personal and medical information
* well versed in CLI
* prefers typing to mouse interactions


**Value proposition**:

* Ease the job of clinics with a centralised record of its patients
* Doctors can easily access patient's personal and medical information without having to go through stacks of paper
* Helps the clinic doctor to keep track of his appointments
* For those proficient in typing, ease management of assets


### Introduction to DG
The purpose of this DG is to serve as a place for future designers to familarise themself with the product. Included below are user stories that show the importance of the features added.  Also included are use cases and non functional requirements. We have also included a guide to manual testing but designers are encouraged to test further.

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                                    | I want to …​                     | So that I can…​                                                        |
| -------- | ------------------------------------------ | ------------------------------ | ---------------------------------------------------------------------- |
| `* * *`  | user                                       | add patient's contact.         | store their contacts and personal information in DocBob           |
| `* * *`  | user                                       | delete patient's contact       | remove unwanted patients from DocBob                                |
| `* * *`  | user                                       | view all information related to a patient | get a quick overview of that patient and find out any information about them needed              |
| `* * *`  | user                                       | find a patient by name         | locate details of patients without having to go through the entire list|
| `* * *`  | user                                       | list out all my patients       | see a list of all my patients and their nearest upcoming appointments |
| `* * *`  | user                                       | edit my patient's information  | update it or fix mistakes I made when entering their data|
| `* * *`  | user                                       | add appointments to a patient  | schedule my next appointment with that patient|                    
| `* * *`  | user                                       | list out all appointments with a particular patient  | view my upcoming appointments with them and find out relevant information like date and time of each appointment|
| `* *`    | user                                       | view all my upcoming appointments | see my all my upcoming appointments and know when my next appointment is |
| `* *`    | user                                       | create a medical record for a patient | organise and save my patient's medical information after an appointment with them|
| `* *`    | user                                       | write my medical records freely in sections | organise and arrange their information in the format I prefer|
| `* *`    | user                                       | view the medical records belonging to a patient | refer to the medical information I previously keyed in for them|
| `* * `   | user                                       | archive a patient | remove patients from my main list while still keeping their information saved somewhere|
| `* *`    | user                                       | unarchive a patient | add pre-existing patients from my archives back to my main list|
| `* *`    | user                                       | list out patients archived     | see patients in the archived list
| `* `     | user                                       | clear the patients in DocBob    | delete all existing patient information and start anew|
| `*`      | user                                       | exit the app                    | close DocBob and use my computer for other stuff|
| `***`   | user                                        | see all the commands available | know what commands are available in DocBob and how to use them                |



### Use cases

(For all use cases below, the **System** is `DocBob` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Adding a new patient**

**MSS**

  1. User chooses to add a patient.
  2. User enters the requested details of patient.
  3. DocBob adds the client and displays the new log of clients.
    
    Use case ends.

**Extensions**

  *2a. DocBob detects an error in the format of the entered data.
  *    2a1. Bob requests for the correct format of the data.
  *    2a2. User enters new data.
  *    Steps 2a1-2a2 are repeated until the data entered are correct.

      Use case resumes from step 3.
      Use case ends.

**Use case: Listing all patients**

**MSS**

1. User requests to list all patients.
2. DocBob shows a list of patients.

    `Use case ends.`

**Extensions**

  *2a. The list is empty

      Use case ends.

**Use case: Deleting a patient's information**

**Pre-requisite:** Use 'list' to list out the index of all the patients

**MSS**

1. User requests to delete a specific patient in the list.
2. DocBob deletes the patient.

      `Use case ends.`

**Extensions**

  *1a. The given index is invalid
  *    1a1. Bob shows an error message.

      Use case resumes at step 1.

**Use case: View a patient's information**

**MSS**

1. User request to view patients information.
2. DocBob provides patient's information
    
    `Use case ends.`

**Extensions**

  *1a. the given index is invalid
  *    1a.1 DocBob shows an error message
  
   `Use case resumes at step 1.`
  
**Use case: Find a patient**

**Pre-requisite:** Finding for patient either in an archived or unarchived list

**MSS**

1. User finds for a patient using a keyword which is case insensitive
2. DocBob returns a patients

   `Use case ends`

**Extensions**

  *1a1. the given keyword is a partial word
  *     1a1.1 DocBob shows an error message
  
    `Use case resumes at step 1'
        
  *1a2. the given patient is in the list user is currently in
  *     1a2.1 DocBob a message to indicate user is in wrong list

    `Use case resumes at step 1'
        
**Use case: Edit a patient's information**

**MSS**

1. User request to edit information by presenting new information and specifying which patient
2. DocBob presents the new patient with updated information

**Extensions**

  *1a1. there is no patient to edit information
  *     1a1.1 DocBob shows an error message
  
        `Use case resumes at step 1
        
  *1a2. no parameters were input to be editted
  *     1a2.1 DocBob shows an error message
  
        `Use case resumes at step 1

  *1a3. there is no corresponding field to be editted
  *     1a2.1 DocBob shows an error message
  
       `Use case resumes at step 1

**Use case: Adding a new appointment to a patient**

**MSS**

  1. User chooses to add an appointment to a patient.
  2. User enters the requested details of the appointment and the patient.
  3. DocBob adds the appointment to the patient and displays updated information.
    
    `Use case ends.`

**Extensions**

  *2a. DocBob detects an error in the format of the entered data.
  *    2a1. Bob requests for the correct format of the data.
  *    2a2. User enters new data.
  *    Steps 2a1-2a2 are repeated until the data entered are correct.
  
      `Use case resumes from step 3.`
      `Use case ends.`

**Use case: Listing all appointments**

**MSS**

1. User requests to list all appointment.
2. DocBob shows a list of appointment.

    `Use case ends.`

**Extensions**

  *2a. The list is empty

      Use case ends.

**Use case: Creating a new medical record**

**MSS**

  1. User chooses to add a medical to a patient.
  2. User enters the requested index of patient and types in the information in the text editor
  3. Bob adds the medical record to the patient
    
    `Use case ends.`

**Extensions**

  *2a. Bob detects an error in the format (index out of bounds) of the entered data.
  *    2a1. Bob requests for the correct format of the data.
  *    2a2. User enters new data.
  *    Steps 2a1-2a2 are repeated until the data entered are correct.
  
      `Use case resumes from step 3.`
      `Use case ends.`

**Use case: View a patient's medical record**

**MSS**

1. User request to view patients medical record.
2. DocBob provides patient's medical record in edit mode
    
     `Use case ends.`

**Extensions**

  *1a. the given index is invalid
  *    1a.1 DocBob shows an error message
  
       `Use case resumes at step 1.`

*1b. current date is more than a day after medical record's creation
*    1b.1 DocBob provides patient's medical record in view mode

     `Use case ends.`
  
**Use case: Adding a patient to archive**

**MSS**

  1. User chooses to add a patient to archive.
  2. User enters the requested index of patient to add to archive.
  3. Bob adds the client and displays the new log of clients.
    
      `Use case ends.`

**Extensions**

  *2a. Bob detects an error in the format (index out of bound) of the entered data.
  *    2a1. Bob requests for the correct format of the data.
  *    2a2. User enters new data.
  *    Steps 2a1-2a2 are repeated until the data entered are correct.
  
      `Use case resumes from step 3.`
      `Use case ends.`

**Use case: Unarchiving a patient**

**Pre-requisite:** Use 'archivelist' to list out the index of all the archived patients

**MSS**

1. User requests to unarchive a specific patient in the archieve list.
2. DocBob unarchive the patient.

      `Use case ends.`

**Use case: Listing all patients in archive list**

**MSS**

1. User requests to list all patients that have been archived.
2. DocBob shows a list of patients.

      `Use case ends.`

**Extensions**

  *2a. The list is empty

       Use case ends.

**Use case: Clear patients in from DocBob**

**MSS**

1. User requests to clear patient in DocBob.
2. DocBob clears patients.

**Use case: Exit DocBob program**

**MSS**

1. User requests to exit DocBob.
2. DocBob terminates program.

**Use case: Get list of commands**

**MSS**

1. User requests list of commands from DocBob.
2. DocBob provides list of commands.


### Non-Functional Requirements

| Category of Non Functional Requirements | Non Functional Requirement                                                                                    | 
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Domain rules                            | at least 1 patient added |
| Constraints                             | System should be compatible with previous version and easily scalable to add new functionalities  |
| Constraints                             | System should give obvious advantage to fast typers |
| Constraints                             | System should not have elements of social networking i.e. communication between devices |
| Technical requirement                   | System should be able to work on any processor i.e. 32 bit or 64 bit |
| Technical requirement                   | System should be scalable enough to be able to keep up with new processors (more than 64) |
| Technical requirement                   | System should be able to work on an operating system (OS) i.e. MacOS, Windows, Linux etc.|
| Technical requirement                   | System should be able to work on older version of different OS i.e. Windows 7, OS x 10.3 (panther) etc.|
| Performance requirement                 | System should have at most a tolerable lag time |
| Performance requirement                 | Lag time of system should not be too long and result in an off putting experience|
| Quality requirement                     | System should be straightforward enough that a novice should at the very least be able to add, edit and delete users|
| Quality requirement                     | System should be have features that make sense so that as time pass, the users can do more with the software|
| Quality requirement                     | System should be robust enough to handle manner of edge cases i.e. throw appropriate exceptions that guide the user|
| Process requirement                     | the project should adhere to the schedule pre decided (subject to minor adjustments within margin of error)|
| Notes about project scope               | the program should not have to handle printing printing of the screen |
| Notes about project scope               | the program should not have to handle uploading information to private servers |
| Notes about project scope               | the program should not have to handle encryption and decryption of information passed into it |
| Miscellaneous                           | the program should not contain any offending imaginary |
| Miscellaneous                           | the program should not contain any vulgar words/language |
| Miscellaneous                           | the program should not contain any political imagery |




### Glossary

* **Mainstream OS**: Windows, Linux, Unix, OS-X
* **MSS**: Main success story which describes the most straightforward interaction for a given use case, where nothing goes wrong
* **Case insensitive**: Capitilisation of letters do not matter.  For example shrek,SHREK,ShErK are considered the same.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Adding a patient

1. Adding a patient

   1. Test case: `add n/NAME y/DATEOFBIRTH g/GENDER p/PHONENUMBER e/EMAIL a/ADDRESS b/BLOODTYPE h/HEIGHT w/WEIGHT [t/TAG]`<br>
      Expected: Patient with appropriate information (with a tag) added

   1. Test case: `add n/NAME y/DATEOFBIRTH g/GENDER p/PHONENUMBER e/EMAIL a/ADDRESS b/BLOODTYPE h/HEIGHT w/WEIGHT`<br>
      Expected: Patient with appropriate information (without a tag) added

   1. Other incorrect delete commands to try: `add`, `add x`, `...` (where x is not in the above specified format)<br>
      Expected: Similar to previous.

1. Adding a patient in archieved list

   1. Prerequisites: List all patients using the `archivelist` command. Multiple patients in the archieved list.

   1. Test case: `archive 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.
      
   1. Other incorrect delete commands to try: `archive`,<br>
      Expected: Similar to previous.

1. Adding a medical record to a patient

   1. Test case: `mrec 1`<br>
      Expected: Opens a notepad to add medical record to the patient with index 1
      
  1. Other incorrect delete commands to try: `mrec`, `mrec x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Deleting a patient

1. Deleting a patient while all patients are being shown

   1. Prerequisites: List all patients using the `list` command. Multiple patients in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No patient is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. Deleting a patient in archieved list

   1. Prerequisites: List all patients using the `archivelist` command. Multiple patients in the archieved list.

   1. Test case: `unarchive 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `unarchive 0`<br>
      Expected: No patient is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `unarchive`, `unarchive x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_


