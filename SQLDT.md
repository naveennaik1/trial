# Software Architecture & Design - SQL Dependency Tracker

## Introduction
SQL Dependency Tracker is a tool designed to visualize and manage database object dependencies. It allows user to dynamically explore and document all your database object dependencies, using a range of graphical layouts. It integrates with SQL Server Management Studio, so that user can explore a database by right-clicking in the Object Explorer. It helps users to understand how different objects within a SQL Server database relates to each other, making it easier to analyse the impact of changes, troubleshoot issues, and manage database structures. 

## Architectural & component-level design
# -----
# insert flowchart here
# -----
![alt text](login_img.png)
#### Explanation: 

There are two methods to create dependency diagrams: 
Dependency Tracker application: Create a project, construct the diagram, and customize its appearance. 
Utilizing SQL Server Management Studio (SSMS): Right-click on the desired database and select the option to generate a dependency diagram. 

The system consists of three primary projects: UI, Engine, and Graph. 

UI (User Interface): This project is built using WinForms, this component handles user interaction through forms, controls, layouts etc. 

Engine: The core component responsible for server and network connections, dependency identification, diagram generation, and interaction with the Graph component. 

Graph: This component focuses on the visual representation of the dependency diagram. Leveraging the System.Drawing namespace, it creates nodes, edges, and customizes their appearance using elements like brushes, pens, and text. 

### Code Structure

SQL Dependency Tracker contains following code Structure: 

**Testing Folder:** Contains all tests related to the product, including unit tests and integration tests. This folder also includes projects for testing Engine.Tests, CommandLine.Tests, and Logging.Tests. 

**CommandLine Project:** Manages the logic for creating dependency diagram through a command-line interface. It uses the Runtime.CompilerServices package, which allows compiler writers to specify attributes in metadata that affect the runtime behavior of the Common Language Runtime. 

**Engine Project:** The core project responsible for connecting to the database, retrieving database objects, and identifying dependencies between objects. The Engine project is integrated with the UI and other graph components. 

**Graph Project:** Contains the code for the visual representation of the dependency diagram. It includes interfaces and classes like Entity, Node, and Edges, which define the properties for drawing the diagram. 

**Logging Project:** Handles the creation of informational and error logs for the project. It uses the Serilog package, a diagnostic logging library for .NET applications that is easy to set up, has a clean API, and runs on all recent .NET platforms. 

**SQLDependencyTrackerSSMSIntegration Project:** Contains the logic for integration with SQL Server Management Studio (SSMS). This includes the right-click feature that helps create a diagram of the selected databases in SSMS and the view dependency tab suggested by SQL Prompt, which displays the diagram within the tabs provided by the SQL Prompt suggestion box in the SSMS query window. 

**SSMSPackage Project:** Dependent on the SQLDependencyTrackerSSMSIntegration project, it contains Dlls for different SSMS versions. 

**UI Project:** Contains all the UI components where the user interacts, primarily using WinForms. It includes UI elements and their actions for forms, layouts, data, controls, etc. 

**UsageReporting Project:** Contains the code to report product and feature usage. It uses Redgate’s redgate.analytics.appinsights package, which helps to track the most used features by end users. 



### Code flow Diagram for “Add Selection to Project” feature
# -----
# insert flowchart here
# -----

#### Explanation: 
Below is the code flow illustrating the process when a user connects to an instance, selects the desired database objects to include in the dependency diagram, and clicks the `"Add Selection to Project"` button. 

On Clicking the `“Add Selection to Project”` button it calls `btnAddSelectedToProject_Click()` function from `AddObjectsToProjectForm.cs` file. 

The process begins by disabling the button to prevent multiple clicks and then it invokes the `GetLoginCredentials()` function and stores the result in a collection variable. 

Next, the `SaveSelectionToDatabaseObjectIDCollection()` function from `ObjectSelectControl.cs` is called with a list of instances or databases on the network (i.e., the collection). This function takes two parameters: the collection and parentNode. 

Within this function, all the nodes (such as tables, roles, stored procedures, etc.) are iterated through. If `GetNodeSelected(node)` returns 1, the node is added to the collection list passed as an argument. 

Once the collection object is populated with data from the selected objects or nodes, the `AddServersDatabasesAndObjectsToProject(collection, idList)` function is invoked. This is the main function responsible for creating the diagram based on the selected objects from a particular database. 

It calls `AddServersDatabasesAndObjectsToProjectInternal()` using the `ExecuteBusyOperation()` function. Here, it first checks if the collection is not null. If true, it merges the collection login credentials into the existing `LoginCredentials` using `UI.Project.LoginCredentials.MergeRetainingEnabledState(collection)` method. 

Afterward, the `UI.Project.Diagram.Augment()` function from ProgramDiagram.cs is called with the following arguments: `Network`, `collection`, `idList`, `AugmentType`, and `status`. 

Inside the `Augment` function, the `LoadObjects()` method is first invoked with `ShowConstraintsAsDependencies` set to true. This further calls `LoadObjects()` from `Network.cs` in the engine project with LoadBehaviour as `NotLoadedAndUnresolved`, and also calls the `ProxyAllObjects()` function, which proxies both resolved and unresolved objects. 
 
The Augment function then removes any existing edges by calling `Controller.RemoveEdges()`. 

It then checks the `AugmentType`. If set to AllObject, it calls `MapNodesWithRetinue (credentials, null, status)`. If set to `SelectedObjects` or `SelectedObjectsAndRedoDependencies`. it calls `MapNodesWithRetinue (credentials, idList, status)`.  

This function iterates over all selected nodes and invokes `MapNode()` to identify dependencies, followed by calling `CreateNode()` with the specific object as a parameter, thus creating a node. 
 
Similarly, for edges, it calls the `MapEdges(status)` function, which in turn calls `MapEdgesIncoming(obj)` and `MapEdgesOutgoing(obj)` to set edges for each node. 
Using these nodes and edges, the dependency diagram is created. 

Finally, `PopulateTabsFromDatabases()` is called to display other non-selected nodes, allowing users to make additional selections. 


### Constraints and Risks

•	SQL Dependency Tracker assumes the user has a basic understanding of SQL Server objects and dependencies. 
•	It also assumes a well-defined database schema with proper relationships between objects (tables, views, stored procedures, etc.). 
The tool likely assumes you're using a supported database platform (typically •	Microsoft SQL Server). 
•	Complex database structures with many objects and intricate relationships might overwhelm the tool, impacting performance and accuracy. 
•	Constraints related to the licensing fees for the tool and any additional costs for support or upgrades. 
•	SQL Dependency tracker might miss certain dependencies due to limitations in its detection algorithms or edge cases in database schema design. This could lead to unexpected errors when modifying database objects. 
•	Analysing large and complex databases can be resource-intensive, potentially impacting database performance during dependency tracking. 
•	Reliance on the tool could lead to vendor lock-in, making it difficult or expensive to switch to an alternative solution in the future. 