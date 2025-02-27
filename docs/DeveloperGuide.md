---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

* {list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well}

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document can be found in the [diagrams](https://github.com/se-edu/addressbook-level3/tree/master/docs/diagrams/) folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** has two classes called [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java). It is responsible for,
* At app launch: Initializes the components in the correct sequence, and connects them up with each other.
* At shut down: Shuts down the components and invokes cleanup methods where necessary.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

The rest of the App consists of four components.

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.


**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

How the `Logic` component works:
1. When `Logic` is called upon to execute a command, it uses the `AddressBookParser` class to parse the user command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `AddCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to add a person).
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

The Sequence Diagram below illustrates the interactions within the `Logic` component for the `execute("delete 1")` API call.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in json format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th client in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new client. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the client was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how the undo operation works:

![UndoSequenceDiagram](images/UndoSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the client being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


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

* works as an insurance agent
* prefer desktop apps over other types
* has a need to manage a large number of his/her Singaporean clients and their details
* has a need to be reminded about his/her meeting timings with clients
* has a need to manage revenue and costs for his clients

**Value proposition**: manage clients faster than a typical mouse/GUI driven app and to also be able
to collate details of clients in one application.


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority   | As a …​                                    | I want to …​                     | So that I can…​                                                        |
| -----------| ------------------------------------------ | ------------------------------ | ---------------------------------------------------------------------- |
| `* * *`  | insurance agent                            | add clients' names to the client list        | connect with them easier on future meetups.              |
| `* * *`  | insurance agent whose clients live in Singapore                            | add the address of clients to the client list              | plan meetups with them in the future easily                                                                 |
| `* * *`  | insurance agent                            | add clients' client number to the client list               | contact them when I want to                                  |
| `* * *`  | insurance agent                            | keep track of clients' claim status         | update the client about it |
| `* * *`  | insurance agent                                     | add total revenue from each of my clients  |  determine the quality of my clients|
| `* * *`  | insurance agent                                     | add total costs from each of my clients         | determine how much to spend on gifts for clients.|
| `* * *`  | user                                                | save my total costs for future use         | save time on having to type them everytime.|
| `* * *`  | insurance agent                                     | keep notes about the client      | remember the health condition and ailments of all my clients.|

| `* * *`  | insurance agent                                     | add the time and place of my appointments for my respective clients in the contact page        | be punctual|
| `* * *`  | user                                     | save my contacts upon closing my address book        | save time on having to type them everytime |
| `* * *`  | organised user                                    | sort the clients in my address book by their first name         | locate a client easily|

| `* * *`  | organised and shrewd insurance agent                                     | sort my clients based on how much money I am making from them         | know which clients to prioritise|
| `* * *`  | insurance agent                                     | calculate the commissions I get from my client        | know the revenue obtained from the policy my client buys. |
| `* * *`  | organised insurance agent                                    | remember what insurance my client already has        |  sell the client insurance he/she does not have yet |

| `* * *`  | user with many contacts in the address book                                     | search for contacts in my contacts list whose name matches my input         |  navigate to the person I am looking for quickly |
| `* * *`  | user                                     | delete clients from my contact list         | remove a client from my contact list I no longer need to keep in contact with|
| `* * *`  | user                                    | use programs on Windows and Mac         | use it on all my laptops |
| `* * *`  | user                                    | exit the program safely        | free up resources on his computer |
| `* * *`  | new user                                     |  install the application        | I can use it |
| `* * *`  | insurance agent                                     | keep track of clients' claim status         | update the client about it |
| `* * *`  | insurance agent                                     | keep track of clients' claim status         | update the client about it |
| `* *`    | insurance agent                                      | keep track of the birthday of my clients   | maintain customer relations with them|
| `* *`    | insurance agent that labels my clients    | delete labels that I have assigned  | correct mislabels and inaccurate labels
| `* *`    | user                                       | import my contacts from a different source   | easily add multiple contacts at once
| `* *`    | user                                       | filter out the contacts in my address book based on address   | better plan out my appointments
| `* *`    | user                                       | see my contact list as a user interface   | read the list easily|
| `* *`    | experienced user                                      | back up my client data  | not lose the data|
| `* *`    | expert insurance agent                                      | give my clients a nickname   | more easily search for my regular contacts |
| `* *`    | insurance agent with a lot of clients                                       | create labels for my clients   |  classify and keep track of each of their characteristics|
| `* *`    | insurance agent                                      | see the labels attached to each client easily   |  quickly reference it when I am about to meet the client|
| `* *`    | user                                       | edit the information of entries in my address book   | ensure the information is accurate and up to date.|
| `* *`    | insurance agent with a lot of clients                                       | create labels for my clients   |  classify and keep track of each of their characteristics|
| `*`      | user | see how much space the program is using          | easily manage my computer memory |

*{More to be added}*

### Use cases

(For all use cases below, the **System** is the `InsurancePal` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Viewing help**

**MSS**

1. User requests to see the help menu
2. InsurancePal shows the user how to access the help webpage
3. User accesses the help webpage

   Use case ends.

**Use case: Add a client**

**MSS**

1. User requests to add a client
2. InsurancePal adds the client

   Use case ends.

**Extensions**

* 1a. The given details are of an invalid format

    * 1a1. InsurancePal shows an error message

      Use case ends.

**Use case: List all clients**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients

   Use case ends.

**Extensions**

* 2a. The list is empty
    * 2a1. InsurancePal displays an empty list of clients

      Use case ends.

**Use case: Edit a client**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to edit a specific client in the list
4. InsurancePal edits the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.

**Use case: Finding a client**

**MSS**

1. User requests to find a client using keywords
2. InsurancePal shows a list of clients whose names match at least one keyword

   Use case ends.

**Use case: Delete a client**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to delete a specific client in the list
4. InsurancePal delete the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.

**Use case: Clear the InsurancePal**

**MSS**

1. User requests to clear the InsurancePal
2. InsurancePal clears all entries in it

   Use case ends.

**Use case: Add reveneue to a client**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to add revenue to a specific client in the list
4. InsurancePal adds revenue to the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.

* 3b. The given revenue is of an invalid format
    * 3b1. InsurancePal shows an error message.

      Use case resumes at step 2.

**Use case: Add a claim to a client**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to add a claim to a specific client in the list
4. InsurancePal adds the claim to the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.
* 3b. The given claim is of an invalid format
    * 3b1. InsurancePal shows an error message.

      Use case resumes at step 2.

**Use case: Add a note to a client**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to add a note to a specific client in the list
4. InsurancePal adds a note to the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.

**Use case: Schedule a meeting**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to schedule a meeting with a specific client in the list
4. InsurancePal schedules a meeting with the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.

* 3b. The given meeting time is of an invalid format
    * 3b1. InsurancePal shows an error message.

      Use case resumes at step 2.

**Use case: Add insurance to a client**

**MSS**

1. User requests to list clients
2. InsurancePal shows a list of clients
3. User requests to add insurance to a specific client in the list
4. InsurancePal adds insurance to the client

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.
    * 3a1. InsurancePal shows an error message.

      Use case resumes at step 2.

* 3b. The given insurance is of an invalid format
    * 3b1. InsurancePal shows an error message.

      Use case resumes at step 2.

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2. Should be able to hold up to 1000 clients without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4. New users should be able to easily see where to find information on how to use the application.
5. Should inform user of the necessary amendments to make to their input when receiving a bad input.
6. Data should be transferable between different devices that are both running InsurancePal.
7. Each command should be successfully executed within 1 second.
8. Should not exit unexpectedly as a result of software implementation regardless of user input.
9. Should not modify information stored without explicit permission or instruction by the user.
10. Should not allow duplicate entries.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, OS-X
* **Notes**: A short paragraph that is written for a person to remind the user about details of that person
* **Client**: A person the user is selling or trying to sell insurance to


--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample clients. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a client

1. Deleting a client while all clients are being shown

   1. Prerequisites: List all clients using the `list` command. Multiple clients in the list.

   1. Test case: `delete 1`<br>
      Expected: First client is deleted from the list. Details of the deleted client shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No client is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
