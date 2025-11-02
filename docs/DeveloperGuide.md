---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# EduBase Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

[_AB3_](https://github.com/nus-cs2103-AY2526S1/tp).

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `deregister S00001`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `CourseListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/AY2526S1-CS2103T-T13-4/tp/blob/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete_course C0001")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete_course C0001` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCourseCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `MainParser` object which in turn creates a parser that matches the command (e.g., `DeleteCourseCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCourseCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `MainParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `RegisterCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `RegisterCommand`) which the `MainParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `RegisterCommandParser`, `DeleteCourseCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/AY2526S1-CS2103T-T13-4/tp/blob/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the course book data i.e., all `Course` objects (which are contained in a `CourseList` object).
* stores the currently 'selected' `Person`, `Course` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>`, `ObservableList<Course>` respectively that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

### Storage component

**API** : [`Storage.java`](https://github.com/AY2526S1-CS2103T-T13-4/tp/blob/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save course book data, address book data, and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `CourseBookStorage`, `AddressBookStorage`, and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

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

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `deregister S00005` command to delete the 5th person in the address book. The `deregister` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `deregister S00005` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `register n/David …​` to add a new person. The `register` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
    * Pros: Easy to implement.
    * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
    * Pros: Will use less memory (e.g. for `deregister`, just save the person being deleted).
    * Cons: We must ensure that the implementation of each individual command are correct.

### \[Proposed\] Add tag to courses

### Proposed Feature — Tags for Courses

#### Motivation
Currently, each course can only store fixed information such as course name, code, and number of students enrolled. However, users may wish to add **custom remarks** — for example, the **name of the teacher-in-charge**, **difficulty level**, or **semester offered**.
A flexible **tagging system** allows users to attach personalized tags to courses without altering the base data model.

#### User Stories
| Version | As a ...          | I want to ... | So that I can ... |
|----------|-------------------|----------------|-------------------|
| v1.3 | busy teacher      | add tags to my courses | remember who teaches each module |
| v1.3 | teacher           | remove tags from a course | keep my course list tidy |
| v1.4 | organized teacher | list all tags | quickly find courses with certain remarks |
| v1.4 | advanced teacher  | edit a tag | update outdated information |

---

#### Proposed Implementation

The `Tag` feature introduces a `Tag` class that represents a single tag, and integrates it into the `Course` model as a `Set<Tag>`.

Each course stores its own independent tag list. Tags can be added or removed through dedicated commands like `addtag` and `removetag`.

##### Class Structure

The following class diagram shows the key relationships among `Course` and `Tag`:

![](diagrams/CourseClassDiagram.puml)

##### Sequence of Operations

When a user executes the `addtag` command — e.g.,

`addtag 1 t/Dr Tan teaches this course`

the following steps occur:

1. The `LogicManager` receives the command text and passes it to the `CourseBookParser`.
2. The parser creates an `AddTagCommand` object with the parsed `Index` and `Tag`.
3. The `AddTagCommand` retrieves the specified course from the model using its index.
4. The command creates a new `Course` object with the updated tag set.
5. The `Model` replaces the old course with the new course in its course list.
6. The UI updates to reflect the change.

---

#### Example

**Command:**

`addtag 2 t/Dr Tan teaches this course`

**Expected behavior:**
Adds a new tag to the course in position 2. The course now displays: CS2103 Software Engineering [Tags: Dr Tan teaches this course]

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

Target users are **CLI Power Users** (Teachers) who:
* **Are Proficient Typists**: The application MUST prioritize keyboard speed and command-line efficiency.
* **Prefer CLI**: Usage must minimize or eliminate the need for mouse interaction, focusing on keyboard navigation and muscle memory.
* **Handle High Volume**: Require rapid tools for tasks like bulk student management and information retrival. 

**Value proposition**:

* **EduBase** maximizes administrative workflow speed for expert users by providing a **Typing-Optimized CLI**.
* **Typing-Optimized Efficiency**: Enables faster task completion (e.g., adding student details) compared to navigating complex GUI forms.
* **Keyboard-Driven Workflow**: Full system functionality is achieved solely through commands, eliminating GUI dependency.
* **Direct Access**: Provides immediate access to all critical educational administration functions(adding, editing and deleting of student/course data).


### User Stories

The following user stories are derived from the project's feature list, grouped by their level of operation (School Level being foundational, Course Level being management actions within a class).

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …  | I want to …                                               | So that I can…                                                |
| -------- | ------- | --------------------------------------------------------- |---------------------------------------------------------------|
| `***`    | teacher | create a new course                                       | manage a new subject or class offering.                       |
| `***`    | teacher | view a list of all existing courses                       | see all the courses available in the system.                  |
| `***`    | teacher | view detailed information of a course                     | check which students are enrolled and course details.         |
| `***`    | teacher | edit a course’s name or ID                                | update incorrect or outdated course information.              |
| `***`    | teacher | delete an existing course                                 | remove outdated or cancelled courses.                         |
| `***`    | teacher | find courses by name                                      | quickly locate a specific course among many.                  |
| `***`    | teacher | register a new student with their name, phone, and gender | add them to the school's central student database.            |
| `***`    | teacher | view all students                                         | get an overview of all registered students.                   |
| `***`    | teacher | find a student by name or ID                              | easily search for a specific student record.                  |
| `***`    | teacher | edit a student’s details                                  | update a student’s information if there are changes or typos. |
| `***`    | teacher | deregister a student                                      | remove them from the school database if they leave.           |
| `***`    | teacher | add an existing student to a specific course              | enroll them in my class roster.                               |
| `***`    | teacher | remove a student from a specific course                   | unenroll them when they drop the class.                       |
| `***`    | teacher | clear all existing data                                   | reset the database for a new academic term.                   |
| `**`     | teacher | use a help command                                        | learn how to use the application or find a command I forgot.  |
| `**`     | teacher | exit the application                                      | close EduBase safely after finishing my tasks.                |
| `*`      | teacher | list both all students and courses together               | view an overview of all current data in the system.           |


### Use cases

[cite_start]A **use case** describes an interaction between a user (or 'actor') and the system for a specific functionality[cite: 358]. [cite_start]It is a description of a set of action sequences that the system performs to provide an observable result of value to the actor[cite: 357]. [cite_start]Use cases are used to capture the functional requirements of the system[cite: 392].

(For all use cases below, the **System** is `EduBase (EB)[cite_start]` and the **Actor** is the `Teacher`, unless specified otherwise [cite: 361, 407])

---

**Use case: UC01 - Create Course**

**MSS:**
1.  Teacher commands to create a new course with a course name and a course ID.
2.  EB creates the course with the indicated course name and ID.
3.  EB indicates success.
    Use case ends.

**Extensions:**
* 2a. EB detects invalid data (e.g., course ID is already used, course ID is in an invalid format, or a required parameter is missing).
    * 2a1. EB shows an error message.
      Use case ends.

---

**Use case: UC02 - View All Courses**

**MSS:**
1.  Teacher commands to view all existing courses.
2.  EB retrieves and displays a formatted list of all created courses, showing each course's ID and name.
    Use case ends.

**Extensions:**
* 2a. EB detects that no courses have been created.
    * 2a1. EB shows an empty list or a "no courses found" message.
      Use case ends.

---

**Use case: UC03 - Delete Course**

**Preconditions:**
* The target course (identified by course ID) exists.

**MSS:**
1.  Teacher commands to delete a course using its course ID.
2.  EB deletes the course.
3.  EB indicates success.
    Use case ends.

**Extensions:**
* 1a. EB detects that the course ID is in an invalid format or does not exist.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB detects that the course has students enrolled in it.
    * 2a1. EB shows an error prompting the teacher to remove all students from the course before deletion.
      Use case ends.

---

**Use case: UC04 - Register New Student**

**MSS:**
1.  Teacher commands to register a new student with all required details (e.g., name, phone, gender).
2.  EB registers the student, assigns a unique student ID, and adds the student to the address book.
3.  EB indicates success.
    Use case ends.

**Extensions:**
* 2a. EB detects invalid data (e.g., the name contains digits or special symbols).
    * 2a1. EB shows an error message about the invalid data.
      Use case ends.

---

**Use case: UC05 - Deregister Student**

**Preconditions:**
* The target student (identified by student ID) exists in the address book.

**MSS:**
1.  Teacher commands to deregister a student from the address book using a valid student ID.
2.  EB removes the student from the address book database.
3.  EB displays a success message.
    Use case ends.

**Extensions:**
* 1a. EB detects an invalid format for the student ID or an ID that does not exist.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB detects that the student is still enrolled in one or more courses.
    * 2a1. EB shows an error message, possibly listing the courses.
      Use case ends.

---

**Use case: UC06 - Find Student by Student ID**

**Preconditions:**
* One or more students are registered in the address book.

**MSS:**
1.  Teacher commands to find one or more students using their student IDs.
2.  EB finds the students matching the provided student IDs.
3.  EB displays a filtered list of the found students.
    Use case ends.

**Extensions:**
* 1a. EB detects that no student ID was provided or that one or more IDs are in an invalid format.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB finds no students matching the provided IDs.
    * 2a1. EB displays an empty list or a "no students found" message.
      Use case ends.

---

**Use case: UC07 - Edit Course**

**Preconditions:**
* The target course (identified by its index in the displayed list) exists.

**MSS:**
1.  Teacher commands to edit a course (selected by its index) with a new course name and/or new course ID.
2.  EB updates the details of the selected course.
3.  EB shows a success message and displays the edited course.
    Use case ends.

**Extensions:**
* 1a. EB detects an invalid index (e.g., out of bounds) or that no new details were provided.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB detects that the new course ID is in an invalid format or is already used by another course.
    * 2a1. EB shows an error message.
      Use case ends.
* 3a. EB detects that the name is an invalid format.
    * 2a1. EB shows an error message.
      Use case ends.
---

**Use case: UC08 - Add Student To Course**

**Preconditions:**
* The target student (identified by student ID) is registered in the address book.
* The target course exists.

**MSS:**
1.  Teacher commands to add a student to a specific course using the student's ID.
2.  EB adds the student to the course roster.
3.  EB displays a success message.
    Use case ends.

**Extensions:**
* 1a. EB detects an invalid format for the student ID or a student ID that does not exist in the address book.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB detects that the student is already enrolled in the course.
    * 2a1. EB shows an error stating that the student is already in the course.
      Use case ends.

---

**Use case: UC09 - Remove Student From Course**

**Preconditions:**
* The target student (identified by student ID) is enrolled in the target course.

**MSS:**
1.  Teacher commands to remove a student from a course using the student's ID.
2.  EB removes the student from the course roster.
3.  EB displays a success message confirming the removal.
    Use case ends.

**Extensions:**
* 1a. EB detects an invalid format for the student ID.
    * 1a1. EB displays an error message.
      Use case ends.
* 2a. EB detects the student is not enrolled in this course (or the student ID does not exist).
    * 2a1. EB displays an error message.
      Use case ends.

---

**Use case: UC10 - Find Student By Name**

**Preconditions:**
* One or more students are registered in the address book.

**MSS:**
1.  Teacher commands to find students using one or more name keywords.
2.  EB finds the students whose names contain any of the provided keywords.
3.  EB displays a filtered list of the found students.
    Use case ends.

**Extensions:**
* 1a. EB detects no name keyword was provided.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB finds no students matching the provided keywords.
    * 2a1. EB displays an empty list or a "no students found" message.
      Use case ends.

---

**Use case: UC11 - Find Course By Name**

**Preconditions:**
* One or more courses exist in the system.

**MSS:**
1.  Teacher commands to find courses using one or more name keywords.
2.  EB finds the courses whose names contain any of the provided keywords.
3.  EB displays a filtered list of the found courses.
    Use case ends.

**Extensions:**
* 1a. EB detects no name keyword was provided.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB finds no courses matching the provided keywords.
    * 2a1. EB displays an empty list or a "no courses found" message.
      Use case ends.

---

**Use case: UC12 - Edit Student**

**Preconditions:**
* The target student (identified by index) exists in the displayed list.

**MSS:**
1.  Teacher commands to edit a student (selected by index) with new details (e.g., name, phone).
2.  EB updates the details of the selected student.
3.  EB displays the edited student and shows a success message.
    Use case ends.

**Extensions:**
* 1a. EB detects an invalid index or that no new field values were provided.
    * 1a1. EB shows an error message.
      Use case ends.
* 2a. EB detects this action would result in a duplicate student (if uniqueness is required).
    * 2a1. EB shows an error message.
      Use case ends.

---

**Use case: UC13 - Clear the storage**

**MSS:**
1.  Teacher commands to clear the storage.
2.  EB deletes all students and courses from the system.
3.  EB displays an empty list of students and courses and shows a success message.
    Use case ends.

**Extensions:**
* 1a. Teacher does not confirm the clear command (if a confirmation step is implemented).
    * 1a1. EB cancels the operation.
      Use case ends.
* 2a. EB detects the storage is already empty.
    * 2a1. EB shows a message indicating the storage is already empty.
      Use case ends.

### Non-Functional Requirements

🚀 Performance
1. All commands should execute and return results within 1 second on a standard system with at least 8GB RAM and a 2.0 GHz processor.

2. The system should support viewing for up to **100 students in a course** without exceeding **2 seconds** of response time.

3. The system shall support up to **50 concurrent courses** and **1000 registered students** without measurable degradation in performance.

⌨️ Usability
1. The system should be fully operable **using only keyboard input**, with no requirement for mouse or GUI interaction.

2. The system should provide **clear and specific error messages** for invalid inputs.

💾 Reliability & Data Persistence
1. The system shall persist all data updates (courses, students, sessions, attendance) to storage immediately, ensuring **no data loss** if the application closes unexpectedly.

2. Data must persist across sessions (saved to storage on each update).

3. The system shall ensure that no two entities (courses, students, sessions) share the same ID by enforcing **unique identifier generation**.

💻 Portability
1. The system shall run on **Windows, macOS, and Linux command-line environments** without requiring OS-specific changes to commands.

### Glossary

* **EB**: shortform for EduBase.
* **CLI**: A text-based system for interacting with a computer, where users type commands to execute programs or perform tasks.
* **Teacher**: Target user for EduBase. Responsible for creating courses and sessions, and managing students.
* **Student**: Contains details such as student_ID, name, gender and parents contact number.
* **Course ID**: Unique identifier automatically generated for each course (e.g., `C0001`).
* **Student ID**: Unique identifier automatically generated for each student (e.g., `S00001`).
* **Register Student**: The action of adding a new student to the school database.
* **Deregister Student**: The action of permanently removing a student from the school database.
* **Add Student to Course**: The action of enrolling a student (who is already registered) into a specific course.
* **Remove Student from Course**: The action of unenrolling a student from a specific course.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

    1. Download the jar file and copy into an empty folder

    2. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

2. Saving window preferences

    1. Resize the window to an optimum size. Move the window to a different location. Close the window.

    2. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.


### Deleting a person

1. Deleting a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    2. Test case: `deregister S00001`<br>
       Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

    3. Test case: `deregister S00001`<br>
       Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

    4. Other incorrect delete commands to try: `deregister`, `deregister x`, `...` (where x is a non-valid student ID)<br>
       Expected: Similar to previous.


### Saving data

1. Dealing with missing/corrupted data files

    1. Close the app if it is running.

    2. Navigate to the data folder (located at data/addressbook.json or data/coursebook.json).

    3. Delete the addressbook.json and/or coursebook.json file.

    4. Relaunch the app.
