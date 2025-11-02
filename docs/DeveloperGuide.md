---
layout: default.md
title: "SummonersBook Developer Guide"
pageNav: 3
---

# SummonersBook Developer Guide

1. [Acknowledgements](#acknowledgements)
2. [Setting up, getting started](#setting-up-getting-started)
3. [Design](#design)
  - [Architecture](#architecture)
  - [UI component](#ui-component)
  - [Logic component](#logic-component)
  - [Model component](#model-component)
  - [Storage component](#storage-component)
  - [Common classes](#common-classes)
4. [Implementation](#implementation)
  - [Filter Feature](#filter-feature)
  - [Auto-Grouping Feature](#auto-grouping-feature)
  - [Viewing Person Details Feature](#viewing-person-details-feature)
  - [Ungrouping Teams Feature](#ungrouping-teams-feature)
  - [CommandResult Enhancement for Modal Windows](#commandresult-enhancement-for-modal-windows)
  - [Data Import / Export Feature](#data-import--export-feature)
5. [Documentation, logging, testing, configuration, dev-ops](#documentation-logging-testing-configuration-dev-ops)
6. [Appendix: Requirements](#appendix-requirements)
  - [Product scope](#product-scope)
  - [User stories](#user-stories)
  - [Use cases](#use-cases)
  - [Non-Functional Requirements](#non-functional-requirements)
  - [Glossary](#glossary)
7. [Appendix: Instructions for manual testing](#appendix-instructions-for-manual-testing)
  - [Launch and shutdown](#launch-and-shutdown)
  - [Deleting a person](#deleting-a-person)
  - [Editing a person](#editing-a-person)
  - [Filtering persons](#filtering-persons)
  - [Finding persons by name](#finding-persons-by-name)
  - [Adding and deleting performance statistics](#adding-and-deleting-performance-statistics)
  - [Creating teams manually](#creating-teams-manually)
  - [Viewing person details](#viewing-person-details)
  - [Auto-grouping persons into teams](#auto-grouping-persons-into-teams)
  - [Ungrouping teams](#ungrouping-teams)
  - [Recording team wins and losses](#recording-team-wins-and-losses)
  - [Viewing team details](#viewing-team-details)
  - [Importing and exporting data](#importing-and-exporting-data)
  - [Saving data](#saving-data)

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

SummonersBook was developed based on [AddressBook Level 3 (AB3)](https://github.com/se-edu/addressbook-level3) by the SE-EDU initiative.
Certain sections of the code, documentation structure, and testing conventions were reused or adapted from the original AB3 project.

It makes use of the following open-source libraries:

- [JavaFX](https://openjfx.io/) (v17.0.7) — for building the graphical user interface
- [Jackson Databind](https://github.com/FasterXML/jackson-databind) (v2.7.0) — for JSON data serialization and deserialization
- [Jackson Datatype JSR310](https://github.com/FasterXML/jackson-modules-java8) (v2.7.4) — for Java 8 date and time (JSR-310) support in JSON
- [JUnit 5 (Jupiter)](https://junit.org/junit5/) (v5.4.0) — for unit testing

We thank the original AB3 contributors and the authors of the above libraries for making this project possible.

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

**`Main`** (consisting of classes [
`Main`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/Main.java) and [
`MainApp`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/MainApp.java)) is in
charge of the app launch and shut down.

* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues
the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API
  `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using
the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component
through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the
implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [
`Ui.java`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`,
`StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures
the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that
are in the `src/main/resources/view` folder. For example, the layout of the [
`MainWindow`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/ui/MainWindow.java)
is specified in [
`MainWindow.fxml`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [
`Logic.java`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API
call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of
PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates
   a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
2. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which
   is executed by the `LogicManager`.
3. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take
   several interactions (between the command object and the `Model`) to achieve.
4. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:

* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a
  placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse
  the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a
  `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser`
  interface so that they can be treated similarly where possible e.g, during testing.

### Model component

**API** : [
`Model.java`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which
  is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to
  this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a
  `ReadOnlyUserPref` objects.
* stores all `Team` objects (which are contained in a `UniqueTeamList` object).
* provides a list of unassigned persons required for team formation commands like group.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they
  should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which
`Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person`
needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>

### Storage component

**API** : [
`Storage.java`](https://github.com/AY2526S1-CS2103T-F08b-1/tp/blob/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,

* can save both address book data and user preference data in JSON format, and read them back into corresponding
  objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only
  the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects
  that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### Filter Feature

#### Implementation

The filter feature is implemented through the `FilterCommand` and `FilterPredicate` classes.
It allows users to display only the persons whose attributes match the given criteria — such as name, role, rank, or champion.

When the user executes a command such as:

`filter rk/Gold rl/Mid`

the app filters the list of persons based on the provided criteria.
In this example, the result will include all persons whose **rank** is Gold **and** whose **role** is Mid.

This functionality is supported by three key components:

- **`FilterCommand`** — represents the command that performs filtering.
- **`FilterPredicate`** — encapsulates the logical conditions for filtering.
- **`Model#updateFilteredPersonList(Predicate<Person>)`** — applies the predicate to the main person list, updating the UI display.

Each field type (e.g. rank, role, champion) within the same category uses **OR** logic:
> Example: `filter rk/Gold rk/Silver` → persons with rank Gold **or** Silver.

Across different field types, conditions are combined using **AND** logic:
> Example: `filter rk/Gold rk/Silver rl/Mid` → persons who are (Gold **or** Silver) **and** play Mid.

---

#### Example Usage

**Step 1.**
When the app starts, all persons are displayed.
No filtering has been applied yet.

---

**Step 2.**
The user executes `filter rk/Gold`.
The `FilterCommandParser` creates a `FilterCommand` containing a `FilterPredicate` that checks each person’s rank.
`Model#updateFilteredPersonList(predicate)` is called, and the UI updates to show only matching entries.

---

**Step 3.**
The user then runs `filter rk/Gold rl/Mid`.
Now, the displayed list includes only persons whose rank is Gold **and** whose role is Mid.

---

<box type="info" seamless>

**Note:**
If no valid parameters are provided, all persons are shown.
The `list` command can also be used to reset the view and display everyone.

</box>

---

#### Sequence of Operation

The following sequence diagram illustrates how a filter command flows through the app:

<puml src="diagrams/FilterCommandExecutionSequenceDiagram.puml" alt="FilterCommandSequenceDiagram" />
<puml src="diagrams/FilterCommandParserSequenceDiagram.puml" alt="FilterCommandSequenceDiagram" />

---

#### Design Considerations

**Aspect: How filtering is executed**

- **Alternative 1 (current implementation):**
  Apply filtering by setting a predicate using `Model#updateFilteredPersonList(predicate)`.
    - *Pros:* Simple and efficient. Leverages JavaFX’s built-in `FilteredList`.
    - *Cons:* Each new filter replaces the previous one.

- **Alternative 2:**
  Maintain a list of active filters that are combined dynamically.
    - *Pros:* Enables cumulative filtering (e.g. filter by role, then by rank later).
    - *Cons:* Adds complexity and state management overhead.


### Auto-Grouping Feature

#### Implementation

The auto-grouping feature is implemented through the `GroupCommand`, `GroupCommandParser`, and `TeamMatcher` classes.
It automatically creates balanced teams from all unassigned persons, taking into account **person roles, ranks, and champions**.

When the user executes:

`group`

the application attempts to form as many full teams as possible while respecting role requirements (Top, Jungle, Mid, ADC, Support) and avoiding duplicate champions within the same team.

This functionality is supported by the following key components:

- **`GroupCommand`** — represents the command that performs auto-grouping. Validates input and orchestrates team formation.
- **`GroupCommandParser`** — parses user input for the group command. Ensures no arguments are provided.
- **`TeamMatcher`** — contains the core algorithm for forming balanced teams from unassigned persons.
- **`Model#getUnassignedPersonList()`** — retrieves the list of persons not currently assigned to any team.
- **`Model#addTeam(Team)`** — adds newly formed teams to the model and updates the UI.

---

#### TeamMatcher Algorithm

The `TeamMatcher` class implements a sophisticated role-based matching algorithm:

1. **Group by Role**: All unassigned persons are grouped by their roles (Top, Jungle, Mid, ADC, Support).

2. **Sort by Rank**: Within each role group, persons are sorted by rank in descending order (highest to lowest). This uses the `Rank` class's natural `Comparable` ordering.

3. **Iterative Team Formation**:
   - The algorithm attempts to form teams by selecting one person from each role.
   - For each role, it selects the highest-ranked available person who doesn't have a champion conflict.
   - A champion conflict occurs when a person plays the same champion as someone already selected for the current team.

4. **Champion Conflict Resolution**:
   - If the highest-ranked person for a role has a champion conflict, the algorithm tries the next person in that role.
   - If no person can be found without a conflict, team formation stops.

5. **Continuation**: The algorithm continues forming teams until it cannot form a complete team of 5 persons.

The key methods implementing this logic are `TeamMatcher#formTeams()` and `TeamMatcher#selectPersonWithoutChampionConflict()`.

---

#### Sequence Diagrams

The following sequence diagrams illustrate the execution of the `group` command.

The first diagram shows the overall execution flow with Model interactions:

<puml src="diagrams/GroupCommandSequenceDiagram-Model.puml" alt="GroupCommandSequenceDiagram-Model" />

The second diagram shows the detailed TeamMatcher algorithm for forming teams:

<puml src="diagrams/GroupCommandSequenceDiagram-TeamMatcher.puml" alt="GroupCommandSequenceDiagram-TeamMatcher" />

<box type="info" seamless>

**Note:** The diagrams show the flow for a successful team formation. Error cases (e.g., insufficient persons) would result in a `CommandException` being thrown from `GroupCommand` before reaching `TeamMatcher`.

</box>

---

#### Example Usage

**Step 1.**
The user has a list of unassigned persons with various roles and ranks.
The application currently shows all persons without any teams assigned.

**Step 2.**
The user executes the `group` command.
- `GroupCommandParser` validates that no arguments were provided.
- `GroupCommand` is created and executed.
- `GroupCommand#execute()` fetches all unassigned persons via `Model#getUnassignedPersonList()`.

**Step 3.**
`TeamMatcher#matchTeams()` is called to form balanced teams:
- Persons are grouped by role.
- Each role group is sorted by rank.
- Teams are formed iteratively while avoiding champion conflicts.

**Step 4.**
Teams are successfully formed according to role, rank, and champion constraints.
`Model#addTeam(team)` is called for each team, updating the application state and UI.
A success message shows the number of teams created and remaining unassigned persons.

---

<box type="info" seamless>

**Note:**
If there are insufficient persons to form a full team (i.e., missing a required role), the command will throw a `CommandException` with:

`No teams could be formed. Ensure there is at least one unassigned person for each role (Top, Jungle, Mid, ADC, Support).`

Any leftover unassigned persons remain in the pool and can be used in future auto-grouping operations.

</box>

---

#### Design Considerations

**Aspect: Team formation algorithm**

- **Alternative 1 (current implementation):**
  Uses a greedy algorithm that prioritizes rank within each role and checks for champion conflicts.
    - *Pros:* Simple, predictable, and efficient (O(n log n) for sorting + O(n) for team formation).
    - *Pros:* Ensures teams are balanced by rank since highest-ranked players are selected first.
    - *Cons:* May not find an optimal solution if champion conflicts are complex. The algorithm stops when it cannot form a complete team, even if rearranging persons might allow more teams.

- **Alternative 2:**
  Use a backtracking algorithm to explore all possible team combinations and find the maximum number of teams.
    - *Pros:* Guarantees finding the optimal solution (maximum number of teams).
    - *Cons:* Exponential time complexity (O(n!)). Impractical for large person lists.
    - *Cons:* Overly complex for the use case. Coaches can manually adjust teams if needed.

- **Alternative 3:**
  Use a constraint satisfaction problem (CSP) solver.
    - *Pros:* Can handle complex constraints elegantly.
    - *Cons:* Requires external library and adds significant complexity.
    - *Cons:* Overkill for this domain where greedy approach works well.

### Viewing Person Details Feature

#### Implementation

The person details viewing feature is implemented through the `ViewCommand`, `ViewCommandParser`, `PersonDetailWindow`, and enhanced `CommandResult` classes.
It allows users to view comprehensive information about a person in a dedicated modal window, including performance statistics visualized with JavaFX charts.

When the user executes:

`view INDEX`

the application opens a new window displaying the person's details, win/loss record, and performance trends over their last 10 matches.

This functionality is supported by the following key components:

- **`ViewCommand`** — represents the command that retrieves a person by index and triggers the detail window.
- **`ViewCommandParser`** — parses the user input and validates the index format.
- **`CommandResult`** — encapsulates the command result and signals to the UI that a person detail window should be shown. Uses the factory method `CommandResult.showPersonDetail()`.
- **`PersonDetailWindow`** — the JavaFX controller that manages the modal window displaying person details and performance charts.
- **`MainWindow#handlePersonDetail()`** — the UI event handler that creates and shows the `PersonDetailWindow` when triggered by a `CommandResult`.

---

#### Architecture and Design Pattern

The `PersonDetailWindow` follows a **hybrid declarative-imperative UI pattern**:

1. **Declarative FXML Structure**: The window's layout, labels, and chart components are defined in `PersonDetailWindow.fxml`. This separates presentation from logic (Separation of Concerns principle).

2. **Imperative Data Binding**: The Java controller (`PersonDetailWindow.java`) populates the FXML components with data from the `Person` object through the `setPerson()` method.

3. **Dynamic Chart Population**: Performance charts are dynamically populated with data series through private helper methods that transform `Stats` data into JavaFX `XYChart.Series`.

This design is pragmatic and maintainable: static structure is in FXML (easy to modify layouts), while dynamic data binding is in Java (type-safe and testable).

**Key methods**:
- `displayPersonDetails()` — binds basic person information to labels.
- `displayCharts()` — populates all four performance charts.
- `createChartSeries()` — transforms raw statistics into chart data (package-private for testing).
- `configureAxes()` — configures X-axis bounds to show match numbers correctly.

---

#### Sequence Diagram

The following sequence diagram illustrates how the `view` command interacts with the UI to display the person detail window:

<puml src="diagrams/ViewCommandSequenceDiagram.puml" alt="ViewCommandSequenceDiagram" />

<box type="info" seamless>

**Note:** The diagram shows how `CommandResult` acts as the bridge between Logic and UI layers. The command creates a `CommandResult` with the person data, and `MainWindow` detects this and opens the `PersonDetailWindow` accordingly, maintaining separation of concerns.

</box>

---

#### Example Usage

**Step 1.**
The user views the person list and identifies a person they want detailed information about.

**Step 2.**
The user executes `view 1` to view the first person.
- `ViewCommandParser` parses the index argument using `ParserUtil.parseIndex()`.
- `ViewCommand` is created with the target index.

**Step 3.**
`ViewCommand#execute()` validates the index and retrieves the person:
- Checks that the index is within bounds of the filtered person list.
- Retrieves the `Person` object at the specified index.
- Creates a `CommandResult` using the factory method `CommandResult.showPersonDetail()`, passing the person object.

**Step 4.**
`MainWindow` receives the `CommandResult`:
- Detects `isShowPersonDetail()` is true.
- Extracts the person using `getPersonToShow()`.
- Creates a new `PersonDetailWindow` and calls `setPerson(person)`.
- Shows the window with `show()`.

**Step 5.**
`PersonDetailWindow` displays the information:
- `setPerson()` triggers data population.
- Basic details are bound to labels via `displayPersonDetails()`.
- Charts are populated via `displayCharts()`, which creates data series for up to the latest 10 matches.
- The window appears centered on screen with all information visible.

---

#### Design Considerations

**Aspect: Modal window vs in-app panel**

- **Alternative 1 (current implementation):**
  Use a separate modal window to display person details.
    - *Pros:* Doesn't clutter the main UI. Users can keep the window open while working with the main app.
    - *Pros:* Allows multiple detail windows to be open simultaneously (future enhancement).
    - *Cons:* Window management (position, size) must be handled carefully.

- **Alternative 2:**
  Display details in a panel within the main window (e.g., sidebar or bottom panel).
    - *Pros:* No window management needed. Simpler implementation.
    - *Cons:* Takes up space in the main UI, reducing visibility of the person/team lists.
    - *Cons:* Can only view one person's details at a time.

**Aspect: Chart data range**

- **Alternative 1 (current implementation):**
  Show up to the latest 10 matches in performance charts.
    - *Pros:* Focuses on recent performance trends. Charts remain readable.
    - *Cons:* Older data is not visible in the charts (though still stored).

- **Alternative 2:**
  Show all matches in the charts.
    - *Pros:* Complete historical view.
    - *Cons:* Charts become cluttered for persons with many matches. Performance may degrade.

- **Alternative 3:**
  Allow users to configure the range (e.g., last 5, 10, 20 matches).
    - *Pros:* Flexible to user needs.
    - *Cons:* Adds UI complexity. Requires additional state management.

### Ungrouping Teams Feature

#### Implementation

The team ungrouping feature is implemented through the `UngroupCommand` and `UngroupCommandParser` classes.
It allows users to disband one specific team or all teams at once, returning all team members to the unassigned pool.

When the user executes:

`ungroup INDEX` or `ungroup all`

the application removes the specified team(s) from the model, making all affected persons available for future team formation.

This functionality is supported by the following key components:

- **`UngroupCommand`** — represents the command that removes teams. Supports two modes: single team removal and remove all.
- **`UngroupCommandParser`** — parses the user input, distinguishing between an index and the "all" keyword (case-insensitive).
- **`Model#deleteTeam(Team)`** — removes a team from the model and triggers UI updates.
- **`Model#getFilteredTeamList()`** — retrieves the current list of teams being displayed.

---

#### Two Modes of Operation

`UngroupCommand` supports two distinct modes through constructor overloading:

1. **Single Team Removal**: `new UngroupCommand(Index targetIndex)`
   - Removes the team at the specified index in the displayed team list.
   - The `removeAll` flag is set to `false`.

2. **Remove All Teams**: `new UngroupCommand()`
   - Removes all teams from the model.
   - The `removeAll` flag is set to `true` and `targetIndex` is `null`.

This design follows the **Factory Pattern** concept where different constructors create objects with different behaviors.

---

#### Sequence Diagram

The following sequence diagram illustrates the execution of the `ungroup` command for removing a single team:

<puml src="diagrams/UngroupCommandSequenceDiagram-Model.puml" alt="UngroupCommandSequenceDiagram-Model" />

<box type="info" seamless>

**Note:** The diagram shows the single team removal flow. For `ungroup all`, the flow is similar but `executeRemoveAll()` is called instead, which iterates through a defensive copy of the team list.

</box>

---

#### Example Usage Scenario 1: Remove Single Team

**Step 1.**
The user views the team list in the team panel and sees multiple teams.

**Step 2.**
The user executes `ungroup 2` to disband the second team.
- `UngroupCommandParser` parses "2" as an index using `ParserUtil.parseIndex()`.
- Creates `new UngroupCommand(Index.fromOneBased(2))`.

**Step 3.**
`UngroupCommand#execute()` validates and removes the team:
- Retrieves the filtered team list from the model.
- Validates the index is within bounds (throws `CommandException` if invalid).
- Calls `executeRemoveSingle()` helper method.
- Gets the team at the specified index.
- Calls `model.deleteTeam(team)` to remove it.
- Returns a success message showing which team was removed.

---

#### Example Usage Scenario 2: Remove All Teams

**Step 1.**
The user has multiple teams and wants to reset and start over.

**Step 2.**
The user executes `ungroup all`.
- `UngroupCommandParser` detects the "all" keyword (case-insensitive check).
- Creates `new UngroupCommand()` (no-argument constructor).

**Step 3.**
`UngroupCommand#execute()` removes all teams:
- Validates that there are teams to remove (throws `CommandException` if none).
- Calls `executeRemoveAll()` helper method.
- Creates a defensive copy of the team list using `List.copyOf()` to avoid concurrent modification.
- Iterates through the copy and calls `model.deleteTeam(team)` for each team.
- Returns a success message showing how many teams were removed.

---

#### Design Considerations

**Aspect: Single command vs separate commands**

- **Alternative 1 (current implementation):**
  Use one `UngroupCommand` that supports both single and all removal via constructor overloading.
    - *Pros:* User-friendly with intuitive syntax (`ungroup 1` vs `ungroup all`).
    - *Pros:* Reduces code duplication. Both modes share validation logic.
    - *Cons:* Command class handles two distinct behaviors, slightly violating Single Responsibility Principle.

- **Alternative 2:**
  Create separate commands: `UngroupCommand` for single removal and `UngroupAllCommand` for removing all.
    - *Pros:* Stricter adherence to Single Responsibility Principle.
    - *Cons:* More classes to maintain. Introduces code duplication for shared logic.
    - *Cons:* Requires separate command words (e.g., `ungroup` and `ungroupall`), less intuitive for users.

**Aspect: Handling concurrent modification**

- **Alternative 1 (current implementation):**
  Create a defensive copy of the team list before iteration using `List.copyOf()`.
    - *Pros:* Safe and prevents `ConcurrentModificationException`.
    - *Pros:* Simple and clear code.
    - *Cons:* Minor memory overhead for copying the list.

- **Alternative 2:**
  Use an iterator with explicit removal.
    - *Pros:* No memory overhead for copying.
    - *Cons:* More complex code. Easy to introduce bugs.

- **Alternative 3:**
  Collect team IDs first, then remove by ID.
    - *Pros:* Avoids concurrent modification.
    - *Cons:* Requires additional ID lookup logic. More complex.

### CommandResult Enhancement for Modal Windows

#### Implementation

The `CommandResult` class has been enhanced to support triggering modal windows for displaying person details and team statistics.
This allows commands to not only return feedback text but also signal the UI to show specific windows with context data.

#### Key Fields Added

1. **`showPersonDetail`** (boolean) — indicates whether the person detail window should be shown.
2. **`personToShow`** (Person) — the person whose details should be displayed (null if not applicable).
3. **`showTeamStats`** (boolean) — indicates whether the team stats window should be shown.
4. **`teamToShow`** (Team) — the team whose stats should be displayed (null if not applicable).

These fields extend the original `CommandResult` design which only supported `showHelp` and `exit` flags.

#### Factory Methods

To maintain clean command code and adhere to the **Dependency Inversion Principle**, `CommandResult` provides static factory methods:

- **`CommandResult.showPersonDetail(String message, Person person)`**
  - Creates a result configured to open the person detail window.
  - Used by `ViewCommand` to trigger the display.

- **`CommandResult.showTeamStats(String message, Team team)`**
  - Creates a result configured to open the team stats window.
  - Reserved for future team statistics features (e.g., `viewteam` command).

These factory methods make the intent explicit and prevent construction errors.

#### Integration with UI

The `MainWindow` class handles `CommandResult` objects returned by command execution:

1. **Check result flags**: After executing a command, `MainWindow` checks `isShowPersonDetail()` or `isShowTeamStats()`.

2. **Extract context data**: If a flag is set, it extracts the relevant object using `getPersonToShow()` or `getTeamToShow()`, which return `Optional<Person>` or `Optional<Team>`.

3. **Create and show window**: It creates the appropriate window instance (`PersonDetailWindow` or similar), passes the context data, and shows the window.

This design maintains **separation of concerns**: commands decide *what* to show, UI decides *how* to show it.

#### Design Considerations

**Aspect: How to signal UI actions from commands**

- **Alternative 1 (current implementation):**
  Extend `CommandResult` with flags and optional data fields.
    - *Pros:* Centralized result handling. All commands return the same type.
    - *Pros:* UI layer handles all window management. Commands remain decoupled from JavaFX.
    - *Cons:* `CommandResult` class grows as more UI actions are added.

- **Alternative 2:**
  Use an event bus or observer pattern where commands emit events.
    - *Pros:* More extensible. New events can be added without modifying `CommandResult`.
    - *Cons:* Adds complexity. Requires event bus infrastructure.
    - *Cons:* Harder to trace control flow during debugging.

- **Alternative 3:**
  Commands directly call UI methods (e.g., `ui.showPersonDetail(person)`).
    - *Pros:* Simple and direct.
    - *Cons:* Violates layered architecture. Commands become tightly coupled to UI.
    - *Cons:* Makes testing commands difficult (need to mock UI).

### Data Import / Export Feature
The **Import/Export** feature enables users to back up, share, and restore player and team data in CSV format.  
It enhances data portability, supports external analysis and allows synchronisation across different SummonersBook instances.

#### Implementation

This feature is implemented through two primary command classes:

| Command | Description |
|----------|-------------|
| `ExportCommand` | Exports player or team data into a structured CSV file. |
| `ImportCommand` | Imports player data from a CSV file. |

Both commands delegate CSV parsing and file I/O handling to utility classes within the `storage` package.

#### Export Workflow

1. The user executes `export players`/`export players to CUSTOM_PATH`/`export teams`/`export teams to CUSTOM_PATH`.
2. The command validates:
- That the target (`players` or `teams`) is specified.
- That the custom file path (if provided) ends with `.csv`.
3. Determines the output destination:
- Default paths:
  - `data/players.csv`
  - `data/teams.csv`
- Custom path
4. Retrieves relevant data from the `Model`.
5. Converts objects into CSV format and writes them to the file.
6. Returns a success message with the destination path.

**Example Output**
`Exported players to data/players.csv`

##### CSV Schema

| Data Type | Columns |
|------------|----------|
| Player | `Name,Role,Rank,Champion,Wins,Losses` |
| Team | `TeamId,Top,Jungle,Mid,Adc,Support,Wins,Losses,WinRate%` |


#### Import Workflow

1. The user executes `import players from FILE_PATH`.
2. The command validates that:
- The file exists.
- The file extension is `.csv`.
3. Parses the header row to detect supported formats:
- Basic: `Name,Role,Rank,Champion`
- Extended: `Name,Role,Rank,Champion,Wins,Losses`
4. For each row:
- Validates **Role** and **Rank** (must match defined enums).
- Skips **duplicate** entries (same Name + Role combination).
- Skips **invalid** entries (unrecognized enums, missing columns, or malformed data).
- Logs skipped rows for debugging.
- Constructs `Player` objects for valid rows and adds them to the `Model`.
5. Displays a summary message after completion.

**Example Output**
`Imported 10 players, skipped 1 duplicate, 2 invalid row(s).`

#### Design Considerations

| Aspect | Alternatives | Decision / Rationale |
|---------|---------------|----------------------|
| **File Format** | 1. CSV (✓ chosen)  <br> 2. JSON | CSV is lightweight, human-readable, and compatible with spreadsheet tools. |
| **Error Handling** | 1. Skip invalid rows (✓ chosen) <br> 2. Abort on first error | Skipping invalid rows ensures partial progress and robustness against formatting issues. |
| **Validation** | Validate enums for `Role` and `Rank`. | Prevents injection of invalid or malformed data into the system model. |
| **File Location** | 1. Default `/data/` directory (✓ chosen) <br> 2. Custom paths | Offers both consistency and flexibility for users. |
| **Round-Trip Accuracy** | N/A | Exported player CSVs can be re-imported to yield identical data, ensuring lossless serialization. |


#### Example Workflows

**Export players → Import players**
> export players
`Exported players to data/players.csv`

> import players from data/players.csv
`Imported 20 players, skipped 0 duplicates, 0 invalid row(s).

**Export teams to custom path`**
> export teams to data/myTeams.csv
`Exported teams to data/myTeams.csv`

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
SummonersBook is designed specifically for **League of Legends (LoL) esports coaches and team managers** who:

- Manage multiple players and teams in **competitive or training settings**
- Need to **form balanced 5v5 teams** quickly based on rank, role, and champion pool
- **Track player performance** across scrims and tournaments over time
- Prefer a **lightweight desktop application** over complex web dashboards
- Are **comfortable typing commands** (similar to Slack bots or Discord commands) for faster workflow
- Have **basic technical familiarity** (e.g. can install Java or use a terminal) but do not need programming experience

**Value proposition:** manage people and create balanced teams faster and more efficiently than a typical spreadsheet or mouse/GUI-driven app.

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a … | I want to …                                 | So that I can…                                  |
|----------|---------|----------------------------------------------|---------------------------------------------------|
| `* * *`  | coach   | add new people with their in-game names      | track and manage them in the system               |
| `* * *`  | coach   | update a person’s details                    | always have accurate and current information      |
| `* * *`  | coach   | record each person’s primary roles           | assign them to suitable teams                     |
| `* * *`  | coach   | record each person’s preferred champions     | avoid role duplication and build effective teams  |
| `* * *`  | coach   | filter people by role, rank, or score rating | quickly find suitable players for forming teams   |
| `* * *`  | coach   | create practice teams of 5 people            | simulate real match conditions                    |
| `* * *`  | coach   | record wins and losses for teams             | track team performance over time                  |
| `* *`    | coach   | balance teams automatically by rank          | ensure matches are fair and competitive           |
| `* *`    | coach   | see role distribution in each team           | avoid having duplicate roles in the same lineup   |
| `* *`    | coach   | view detailed team information               | review team composition and statistics            |
| `* *`    | coach   | import players from CSV files                | quickly add multiple players from external sources|
| `* *`    | coach   | export players and teams to CSV files        | backup data or share with others                  |

### Use cases

(For all use cases below, the **System** is the `SummonersBook` and the **Actor** is the `user`, unless specified otherwise)

---

**<u>Use case: UC1 - Add a person</u>**

**MSS:**
1. User requests to add a person by providing the necessary details (name, rank, role, champion).
2. SummonersBook validates the inputs.
3. SummonersBook creates and stores the person entry.
4. SummonersBook confirms that the person has been added.  
   <br>
   Use case ends.

**Extensions:**
* 2a. Missing or invalid fields.
  * 2a1. SummonersBook shows an error message indicating the missing or invalid fields.  
    <br>
    Use case ends.

---

**<u>Use case: UC2 - View a person</u>**

**MSS:**
1. User requests to view a person by specifying the index.
2. SummonersBook displays the person’s details.  
   <br>
   Use case ends.

**Extensions:**
* 2a. The given index is invalid.
  * 2a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC3 - Delete a person</u>**

**MSS:**
1. User requests to delete a person identified by an index in the displayed list.
2. SummonersBook deletes the specified person.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The command format is invalid (e.g., missing or non-numeric index).
  * 1a1. SummonersBook shows an error message explaining the correct command format.  
    <br>
    Use case ends.
* 1b. The list is empty.  
  <br>
  Use case ends.
* 1c. The given index is invalid (out of range).
  * 1c1. SummonersBook shows an error message.  
    <br>
    Use case resumes at Step 1.

---

**<u>Use case: UC4 - Find persons</u>**

**MSS:**
1. User requests to find persons by specifying one or more keywords in their names (search is case-insensitive).
2. SummonersBook searches the person list and displays all persons whose names contain at least one of the given keywords.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The command format is invalid (e.g., missing keywords or incorrect syntax).
  * 1a1. SummonersBook shows an error message explaining the correct command format.  
    <br>
    Use case ends.
* 2a. No persons match any of the given keywords.
  * 2a1. SummonersBook shows “No persons found.”  
    <br>
    Use case ends.

---

**<u>Use case: UC5 - Auto-group persons (create teams)</u>**

**MSS:**
1. User requests to automatically group persons into balanced teams.
2. SummonersBook creates balanced teams based on rank, role, and champion pool.
3. SummonersBook displays the newly formed teams.  
   <br>
   Use case ends.

**Extensions:**
* 2a. There are not enough persons to form a complete team.
  * 2a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC6 - Manually create a team</u>**

**MSS:**
1. User requests to create a team by specifying five person indices.
2. SummonersBook creates the team with the specified persons.
3. SummonersBook confirms that the team has been created.  
   <br>
   Use case ends.

**Extensions:**
* 1a. Fewer or more than five indices are provided.
  * 1a1. SummonersBook shows an error message indicating exactly five indices are required.  
    <br>
    Use case ends.
* 1b. Duplicate indices are provided.
  * 1b1. SummonersBook shows an error message about duplicate indices.  
    <br>
    Use case ends.
* 1c. One or more indices are invalid (out of range).
  * 1c1. SummonersBook shows an error message.  
    <br>
    Use case ends.
* 2a. One or more persons are already in another team.
  * 2a1. SummonersBook shows an error message indicating which person is already assigned.  
    <br>
    Use case ends.
* 2b. The selected persons have duplicate roles or champions.
  * 2b1. SummonersBook shows an error message specifying the duplication type.  
    <br>
    Use case ends.

---

**<u>Use case: UC7 - Ungroup teams</u>**

**MSS:**
1. User requests to ungroup either a specific team or all teams.
2. SummonersBook disbands the requested team(s).
3. SummonersBook confirms that the team(s) have been ungrouped.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The given index is invalid.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC8 - View a team</u>**

**MSS:**
1. User requests to view a team by specifying its index.
2. SummonersBook displays the team’s members and detailed statistics in a popup window.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The given index is invalid.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC9 - Filter persons</u>**

**MSS:**
1. User requests to filter the list of persons based on one or more criteria such as role, rank, champion, or average score.
2. SummonersBook displays the persons that match the criteria.  
   <br>
   Use case ends.

**Extensions:**
* 1a. No flags are provided.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.
* 1b. Flags are provided but no values are given.
  * 1b1. SummonersBook shows the full list of persons.  
    <br>
    Use case ends.
* 1c. Invalid value for any flag (role, rank, champion, or score).
  * 1c1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC10 - Add performance values</u>**

**MSS:**
1. User requests to add performance values (CPM, GD15, KDA) to a person identified by index.
2. SummonersBook updates the person’s statistics accordingly.
3. SummonersBook confirms that the values have been recorded.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The given index is invalid.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.
* 1b. Not all flags or values are provided.
  * 1b1. SummonersBook shows an error message explaining missing inputs.  
    <br>
    Use case ends.
* 1c. Invalid value for CPM, GD15, or KDA.
  * 1c1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC11 - Delete latest performance values</u>**

**MSS:**
1. User requests to delete the latest set of performance values for a person by specifying the index.
2. SummonersBook updates the person’s stats accordingly.
3. SummonersBook confirms that the latest performance values have been removed.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The given index is invalid.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC12 - Record a team win</u>**

**MSS:**
1. User requests to record a win for a team by specifying its index.
2. SummonersBook increments the win count for the team and all its members.
3. SummonersBook confirms the updated win/loss record.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The given index is invalid.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC13 - Record a team loss</u>**

**MSS:**
1. User requests to record a loss for a team by specifying its index.
2. SummonersBook increments the loss count for the team and all its members.
3. SummonersBook confirms the updated win/loss record.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The given index is invalid.
  * 1a1. SummonersBook shows an error message.  
    <br>
    Use case ends.

---

**<u>Use case: UC14 - Import persons from CSV</u>**

**MSS:**
1. User requests to import persons from a CSV file by providing a file path.
2. SummonersBook reads the CSV file and validates each row.
3. SummonersBook adds valid persons to the system.
4. SummonersBook displays a summary showing imported count, duplicates skipped, and invalid rows.  
   <br>
   Use case ends.

**Extensions:**
* 1a. The file path is invalid or the file does not exist.
  * 1a1. SummonersBook shows *“Failed to import: file not found.”*  
    <br>
    Use case ends.
* 2a. The CSV file has invalid headers.
  * 2a1. SummonersBook shows an error about invalid CSV format.  
    <br>
    Use case ends.
* 2b. The CSV file is empty.
  * 2b1. SummonersBook shows an error message for empty file.  
    <br>
    Use case ends.
* 2c. Some rows contain invalid data.
  * 2c1. SummonersBook imports valid rows and reports invalid rows with sample errors.  
    <br>
    Use case resumes at Step 4.
* 2d. Some rows are duplicates of existing persons.
  * 2d1. SummonersBook skips duplicate rows and reports the count.  
    <br>
    Use case resumes at Step 4.

---

**<u>Use case: UC15 - Export persons or teams to CSV</u>**

**MSS:**
1. User requests to export persons or teams, optionally specifying a custom file path.
2. SummonersBook writes the data to a CSV file at the specified or default path.
3. SummonersBook confirms the export location.  
   <br>
   Use case ends.

**Extensions:**
* 1a. There are no persons or teams to export.
  * 1a1. SummonersBook shows *“No data available to export.”*  
    <br>
    Use case ends.
* 2a. The file path is invalid or cannot be written to.
  * 2a1. SummonersBook shows *“Failed to export”* with details.  
    <br>
    Use case ends.

---

**<u>Use case: UC16 - Help</u>**

**MSS:**
1. User requests help.
2. SummonersBook displays a list of available commands and usage examples.  
   <br>
   Use case ends.

---

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2. Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be
   able to accomplish most of the tasks faster using commands than using the mouse.
4. Data entered by the user should be **saved automatically** and persist between sessions.
5. The system should handle invalid input gracefully by showing an error message without crashing.
6. System should be secure against unintended file modifications (i.e., only application data files are read/written).

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Balanced Team:**  
  A team automatically created by SummonersBook’s grouping algorithm to ensure fair matchups across all teams.  
  Each balanced team includes one player per unique role — **Top, Jungle, Mid, ADC, and Support** — with players sorted by rank and assigned so that overall skill levels between teams remain comparable.  
  The algorithm also ensures that no two players in the same team share the same champion.

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

### Launch and shutdown

1. Initial launch

    1. Download the jar file and copy into an empty folder

    2. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be
       optimum.

2. Saving window preferences

    1. Resize the window to an optimum size. Move the window to a different location. Close the window.

    2. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

### Deleting a person

1. Deleting a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    2. Test case: `delete 1`<br>
       Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message.
       Timestamp in the status bar is updated.

    3. Test case: `delete 0`<br>
       Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

    4. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.

### Editing a person

1. Editing a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    2. Test case: `edit 1 rk/Diamond`<br>
       Expected: First person's rank is updated to Diamond. Details of the edited person shown in the status message.

    3. Test case: `edit 1 n/NewName rl/Top rk/Gold c/Darius`<br>
       Expected: First person's name, role, rank, and champion are all updated. Success message shows updated details.

    4. Test case: `edit 1 t/`<br>
       Expected: First person's tags are cleared. Success message confirms tags removed.

    5. Test case: `edit 0 rk/Gold`<br>
       Expected: No person is edited. Error message "Invalid command format!" shown.

    6. Test case: Edit a person who is in a team<br>
       Expected: Person cannot be edited. Error message indicates the person must be removed from their team first.

    7. Other incorrect edit commands to try: `edit`, `edit x` (where x is larger than the list size), `edit 1` (no fields provided)<br>
       Expected: Similar error messages for invalid format or parameters.

### Filtering persons

1. Filtering by role and rank

    1. Prerequisites: Have multiple persons with different roles and ranks.

    2. Test case: `filter rl/Mid`<br>
       Expected: Only persons with role "Mid" are displayed. Success message shows number of persons listed.

    3. Test case: `filter rk/Diamond rk/Master`<br>
       Expected: Persons with rank Diamond OR Master are displayed.

    4. Test case: `filter rl/Support rk/Gold`<br>
       Expected: Persons who are Support AND Gold rank are displayed.

    5. Test case: `filter rl/Top rl/Jungle rk/Diamond rk/Platinum`<br>
       Expected: Persons who are (Top OR Jungle) AND (Diamond OR Platinum) are displayed.

    6. Test case: `filter` (no parameters)<br>
       Expected: Error message "Invalid command format!" shown.

### Finding persons by name

1. Finding persons with keyword search

    1. Prerequisites: Have multiple persons with different names.

    2. Test case: `find john`<br>
       Expected: All persons with "john" in their name are displayed (case-insensitive, whole word match).

    3. Test case: `find alex david`<br>
       Expected: Persons with "alex" OR "david" in their names are displayed.

    4. Test case: `find` (no keywords)<br>
       Expected: Error message "Invalid command format!" shown.

### Adding and deleting performance statistics

1. Adding performance statistics to a person

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    2. Test case: `addStats 1 cpm/8.5 gd15/450 kda/4.2`<br>
       Expected: Performance statistics are added to the first person. Success message confirms stats were added. Person's performance score is updated.

    3. Test case: `addStats 1 cpm/9.8 gd15/-200 kda/2.5`<br>
       Expected: Stats added successfully with negative gold difference (fell behind in lane).

    4. Test case: `addStats 0 cpm/8.0 gd15/100 kda/3.0`<br>
       Expected: No stats added. Error message "Invalid command format!" shown.

    5. Test case: `addStats 1 cpm/8.0`<br>
       Expected: No stats added. Error message indicates all three fields (cpm, gd15, kda) are required.

    6. Other incorrect addStats commands to try: `addStats 1 cpm/abc gd15/100 kda/3.0`, `addStats 1 cpm/8.0 gd15/100.5 kda/3.0`<br>
       Expected: Error messages for invalid numeric values or format violations.

2. Deleting the most recent performance statistics

    1. Prerequisites: Have at least one person with performance statistics recorded.

    2. Test case: `deleteStats 1`<br>
       Expected: The most recent performance entry for the first person is deleted. Success message confirms deletion. Person's performance score is recalculated.

    3. Test case: Delete stats from a person with no statistics<br>
       Expected: Error message indicating the person has no statistics to delete.

    4. Test case: `deleteStats 0`<br>
       Expected: No stats deleted. Error message "Invalid command format!" shown.

### Creating teams manually

1. Manually creating a team with specific persons by index

    1. Prerequisites: Have at least 5 unassigned persons with unique roles and unique champions. Use `list` to see indices.

    2. Test case: `makeGroup 1 2 3 4 5` (assuming these indices exist and persons are unassigned with unique roles and champions)<br>
       Expected: A new team is created with these 5 members. Success message shows team composition with member names and roles.

    3. Test case: `makeGroup 1 2 3 4 100` (where 100 is larger than the list size)<br>
       Expected: Team not created. Error message "The person index provided is invalid" shown.

    4. Test case: `makeGroup 1 1 2 3 4` (duplicate index)<br>
       Expected: Team not created. Error message "Duplicate index numbers found in the input" shown.

    5. Test case: Try to create a team with a person already in another team (e.g., `makeGroup 1 2 3 4 5` where person at index 1 is already in a team)<br>
       Expected: Team not created. Error message "Player is already in another team" with player details shown.

    6. Test case: Try to create a team with persons having duplicate roles<br>
       Expected: Team not created. Error message indicates duplicate roles are not allowed.

    7. Test case: Try to create a team with fewer than 5 indices (e.g., `makeGroup 1 2 3`)<br>
       Expected: Team not created. Error message "Exactly 5 index numbers must be provided" shown.

    8. Test case: Try to create a team with more than 5 indices (e.g., `makeGroup 1 2 3 4 5 6`)<br>
       Expected: Team not created. Error message "Exactly 5 index numbers must be provided" shown.

    9. Test case: Try to create a team with persons having the same champion<br>
       Expected: Team not created. Error message indicates duplicate champions are not allowed within a team.

### Viewing person details

1. Viewing a person's detailed information

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    2. Test case: `view 1`<br>
       Expected: A new window opens displaying detailed information about the first person, including basic details (name, role, rank, champion, tags), win/loss record, and performance charts. Success message shown in the result display with the person's name.

    3. Test case: `view 0`<br>
       Expected: No window opens. Error message "Invalid command format!" shown. The person index provided is invalid.

    4. Test case: `view x` (where x is larger than the list size)<br>
       Expected: No window opens. Error message "The person index provided is invalid" shown.

    5. Other incorrect view commands to try: `view`, `view abc`, `view -1`<br>
       Expected: Similar error messages for invalid format or invalid index.

2. Viewing person details with performance data

    1. Prerequisites: The person has performance statistics recorded (use `addStats` to add data if needed).

    2. Test case: View a person with at least 10 matches of statistics<br>
       Expected: All four charts (Performance Score, CS per Minute, KDA, Gold Diff @15) show up to 10 data points. The X-axis shows match numbers correctly.

    3. Test case: View a person with fewer than 10 matches<br>
       Expected: Charts show all available data points. The X-axis adjusts to show only the relevant match numbers.

### Auto-grouping persons into teams

1. Auto-grouping with sufficient persons

    1. Prerequisites: Have at least 5 unassigned persons with unique roles (Top, Jungle, Mid, ADC, Support) and unique champions.

    2. Test case: `group`<br>
       Expected: One or more teams are created. Success message shows the number of teams created, their members (formatted by role), and the number of remaining unassigned persons. Verify teams were created by viewing the team panel on the right.

    3. Test case: Have exactly 10 unassigned persons (2 per role) with unique champions, then run `group`<br>
       Expected: 2 teams are created with 0 persons remaining unassigned.

    4. Test case: Have 12 unassigned persons (mixed roles) then run `group`<br>
       Expected: As many complete teams as possible are created. The success message indicates how many persons remain unassigned.

2. Auto-grouping with insufficient persons

    1. Test case: Have only 4 unassigned persons with 4 different roles, then run `group`<br>
       Expected: No teams created. Error message indicates insufficient persons for all required roles.

    2. Test case: Have 5 unassigned persons but missing one required role (e.g., no Support), then run `group`<br>
       Expected: No teams created. Error message indicates at least one person per role is required.

    3. Test case: Run `group` when all persons are already assigned to teams<br>
       Expected: Error message "No unassigned persons available to form teams."

3. Auto-grouping with champion conflicts

    1. Prerequisites: Have 10 unassigned persons (2 per role) where 2 persons play the same champion and have the same role.

    2. Test case: Run `group`<br>
       Expected: One team is created with the higher-ranked person of the duplicate champion. The second person with the duplicate champion remains unassigned (along with 4 others).

### Ungrouping teams

1. Ungrouping a single team

    1. Prerequisites: Have at least one team created. Verify by viewing the team panel.

    2. Test case: `ungroup 1`<br>
       Expected: The first team is disbanded. Success message shows which team was removed. All 5 members of the team become unassigned and appear in the person list again.

    3. Test case: `ungroup 0`<br>
       Expected: No team is disbanded. Error message "Invalid command format!" shown.

    4. Test case: `ungroup x` (where x is larger than the number of teams)<br>
       Expected: No team is disbanded. Error message "The team index provided is invalid" shown.

    5. Other incorrect ungroup commands to try: `ungroup`, `ungroup abc`, `ungroup -1`<br>
       Expected: Similar error messages for invalid format or invalid index.

2. Ungrouping all teams

    1. Prerequisites: Have multiple teams created. View them in the team panel.

    2. Test case: `ungroup all`<br>
       Expected: All teams are disbanded. Success message shows "Successfully removed X team(s). All persons are now unassigned." Verify the team panel is empty. Verify with `list` that all persons are back in the unassigned pool.

    3. Test case: `ungroup ALL` (case-insensitive)<br>
       Expected: Same as above. The command is case-insensitive.

    4. Test case: Run `ungroup all` when there are no teams<br>
       Expected: Error message "No teams to remove."

### Recording team wins and losses

1. Recording a win for a team

    1. Prerequisites: Have at least one team created. View the team panel to see teams.

    2. Test case: `win 1`<br>
       Expected: The 1st team's win count is incremented. Success message shows "Team 1 has won a match! Their stats have been updated to W:X-L:Y." All members of the team also have their individual win counts incremented.

    3. Test case: `win 0`<br>
       Expected: No team record is updated. Error message "Invalid command format!" shown.

    4. Test case: `win x` (where x is larger than the number of teams)<br>
       Expected: No team record is updated. Error message "The team index provided is invalid" shown.

2. Recording a loss for a team

    1. Prerequisites: Have at least one team created. View the team panel to see teams.

    2. Test case: `lose 1`<br>
       Expected: The 1st team's loss count is incremented. Success message shows "Team 1 has lost a match. Their stats have been updated to W:X-L:Y." All members of the team also have their individual loss counts incremented.

    3. Test case: `lose 0`<br>
       Expected: No team record is updated. Error message "Invalid command format!" shown.

### Viewing team details

1. Viewing a team's detailed information

    1. Prerequisites: Have at least one team created. View the team panel.

    2. Test case: `viewTeam 1`<br>
       Expected: A new window opens displaying detailed information about the first team, including all member details and team win/loss record. Success message shown in the result display.

    3. Test case: `viewTeam 0`<br>
       Expected: No window opens. Error message "Invalid command format!" shown.

    4. Test case: `viewTeam x` (where x is larger than the number of teams)<br>
       Expected: No window opens. Error message "The team index provided is invalid" shown.

### Importing and exporting data

1. Importing players from CSV

    1. Prerequisites: Create a CSV file at `data/test_import.csv` with valid player data:
       ```
       Name,Role,Rank,Champion
       TestPlayer1,Mid,Gold,Ahri
       TestPlayer2,Top,Silver,Garen
       ```

    2. Test case: `import players from data/test_import.csv`<br>
       Expected: Players are imported. Success message shows "Imported X players, skipped Y duplicates, Z invalid row(s)."

    3. Test case: `import players from nonexistent.csv`<br>
       Expected: No players imported. Error message "Failed to import: file not found."

    4. Test case: Import a CSV with invalid data (e.g., invalid rank values)<br>
       Expected: Valid rows are imported, invalid rows are reported. Message shows count of invalid rows with sample error messages.

2. Exporting players and teams to CSV

    1. Prerequisites: Have some players and teams in the system.

    2. Test case: `export players`<br>
       Expected: Player data is exported to `data/players.csv`. Success message shows "Exported player data to data/players.csv."

    3. Test case: `export teams`<br>
       Expected: Team data is exported to `data/teams.csv`. Success message shows "Exported team data to data/teams.csv."

    4. Test case: `export players to/custom_path.csv`<br>
       Expected: Player data is exported to the specified custom path. Success message confirms the export location.

### Saving data

1. Dealing with missing/corrupted data files

    1. Test case: Delete the `data/addressbook.json` file, then launch the application<br>
       Expected: Application starts with sample data populated.

    2. Test case: Corrupt the `data/addressbook.json` file by adding invalid JSON syntax, then launch the application<br>
       Expected: Application starts with an empty data set (no players or teams).

    3. Test case: Manually edit `data/addressbook.json` to add a person with invalid field values (e.g., invalid rank), then launch the application<br>
       Expected: Application starts with an empty data set, discarding the corrupted data.

2. Data persistence

    1. Test case: Add a player, then exit and relaunch the application<br>
       Expected: The added player persists and appears in the player list.

    2. Test case: Create teams, then exit and relaunch the application<br>
       Expected: Teams persist and appear in the team panel with all members intact.
