---
title: "Handling SMO events"
description: Handle SMO event types by subscribing to them using an event handler and the ServerConnection object.
author: markingmyname
ms.author: maghan
ms.reviewer: randolphwest
ms.date: 01/18/2024
ms.service: sql
ms.topic: "reference"
ms.custom:
  - ignite-2024
helpviewer_keywords:
  - "events [SMO]"
  - "SQL Server Management Objects, events"
  - "SMO [SQL Server], events"
  - "events [SMO], about events"
monikerRange: "=azuresqldb-current || =azure-sqldw-latest || >=sql-server-2016 || >=sql-server-linux-2017 || =azuresqldb-mi-current || =fabric"
---
# Handle SMO events

[!INCLUDE [SQL Server Azure SQL Database Azure SQL Managed Instance Synapse Analytics FabricSQLDB](../../../includes/applies-to-version/sql-asdb-asdbmi-asa-fabricsqldb.md)]

There are server event types that can be subscribed to by using an event handler and the <xref:Microsoft.SqlServer.Management.Common.ServerConnection> object.

Many of the instance classes in [!INCLUDE [ssNoVersion](../../../includes/ssnoversion-md.md)] Management Objects (SMO) can trigger events when certain actions on the server occur.

These events can be handled programmatically by setting up an event handler and subscribing to the associated events. This type of event handling is transient because all the subscriptions are removed when the SMO client program exits.

## ConnectionContext event handling

The <xref:Microsoft.SqlServer.Management.Common.ServerConnection> object supports several event types. The event property must be set to an instance of an appropriate event handler, and the event handler object must be defined as a protected function that handles the event.

## Event subscription

You handle events by writing an event handler class, creating an instance of it, assigning the event handler to the parent object, and then subscribing to the event.

An event handler class must be written to handle events. The event handler class can contain more than one event handler function, and must be installed for the events to be handled. The event handler functions receive information about the event from the *ServerEventNotificatificationArgs* parameter that can be used to report information about the event.

The types of database and server events that can be handled are listed in the <xref:Microsoft.SqlServer.Management.Smo.DatabaseEventSet> class and the <xref:Microsoft.SqlServer.Management.Smo.ServerEventSet>class.

## Examples

To use any code example that is provided, you have to choose the programming environment, the programming template, and the programming language in which to create your application. For more information, see [How to Create a Visual C# SMO Project in Visual Studio .NET](../how-to-create-a-visual-csharp-smo-project-in-visual-studio-net.md).

## Register event handlers and subscribe to event handling in .NET

The following code samples show how to set up the event handler, and how to subscribe to the database events, in .NET.

## [Visual Basic](#tab/vbnet)

### Visual Basic

This code example shows how to set up the event handler, and how to subscribe to the database events.

```vbnet
'Create an event handler subroutine that runs when a table is created.
Private Sub MyCreateEventHandler(ByVal sender As Object, ByVal e As ServerEventArgs)
    Console.WriteLine("A table has just been added to the AdventureWorks2022 database.")
End Sub

'Create an event handler subroutine that runs when a table is deleted.
Private Sub MyDropEventHandler(ByVal sender As Object, ByVal e As ServerEventArgs)
    Console.WriteLine("A table has just been dropped from the AdventureWorks2022 database.")
End Sub

Sub Main()
    'Connect to the local, default instance of SQL Server.

    Dim srv As Server
    srv = New Server

    'Reference the AdventureWorks2022 database.
    Dim db As Database
    db = srv.Databases("AdventureWorks2022")

    'Create a database event set that contains the CreateTable event only.
    Dim databaseCreateEventSet As New DatabaseEventSet
    databaseCreateEventSet.CreateTable = True

    'Create a server event handler and set it to the first event handler subroutine.
    Dim serverCreateEventHandler As ServerEventHandler
    serverCreateEventHandler = New ServerEventHandler(AddressOf MyCreateEventHandler)

    'Subscribe to the first server event handler when a CreateTable event occurs.
    db.Events.SubscribeToEvents(databaseCreateEventSet, serverCreateEventHandler)

    'Create a database event set that contains the DropTable event only.
    Dim databaseDropEventSet As New DatabaseEventSet
    databaseDropEventSet.DropTable = True

    'Create a server event handler and set it to the second event handler subroutine.
    Dim serverDropEventHandler As ServerEventHandler
    serverDropEventHandler = New ServerEventHandler(AddressOf MyDropEventHandler)

    'Subscribe to the second server event handler when a DropTable event occurs.
    db.Events.SubscribeToEvents(databaseDropEventSet, serverDropEventHandler)

    'Start event handling.
    db.Events.StartEvents()

    'Create a table on the database.
    Dim tb As Table
    tb = New Table(db, "Test_Table")

    Dim mycol1 As Column
    mycol1 = New Column(tb, "Name", DataType.NChar(50))
    mycol1.Collation = "Latin1_General_CI_AS"
    mycol1.Nullable = True

    tb.Columns.Add(mycol1)
    tb.Create()

    'Remove the table.
    tb.Drop()

    'Wait until the events have occurred.
    Dim x As Integer
    Dim y As Integer
    For x = 1 To 1000000000
        y = x*2
    Next

    'Stop event handling.
    db.Events.StopEvents()
End Sub
```

## [C#](#tab/csharp)

### C#

This code example shows how to set up the event handler, and how to subscribe to the database events.

```csharp
//Create an event handler subroutine that runs when a table is created.
private void MyCreateEventHandler(object sender, ServerEventArgs e)
{
    Console.WriteLine("A table has just been added to the AdventureWorks2022 database.");
}

//Create an event handler subroutine that runs when a table is deleted.
private void MyDropEventHandler(object sender, ServerEventArgs e)
{
    Console.WriteLine("A table has just been dropped from the AdventureWorks2022 database.");
}

public void Main()
{
    //Connect to the local, default instance of SQL Server.
    var srv = new Server();

    //Reference the AdventureWorks2022 database.
    Database db = srv.Databases["AdventureWorks2022"];

    //Create a database event set that contains the CreateTable event only.
    var databaseCreateEventSet = new DatabaseEventSet
    {
        CreateTable = true
    };

    //Create a server event handler and set it to the first event handler subroutine.
    var serverCreateEventHandler = new ServerEventHandler(MyCreateEventHandler);

    //Subscribe to the first server event handler when a CreateTable event occurs.
    db.Events.SubscribeToEvents(databaseCreateEventSet, serverCreateEventHandler);

    //Create a database event set that contains the DropTable event only.
    var databaseDropEventSet = new DatabaseEventSet
    {
        DropTable = true
    };

    //Create a server event handler and set it to the second event handler subroutine.
    var serverDropEventHandler = new ServerEventHandler(MyDropEventHandler);

    //Subscribe to the second server event handler when a DropTable event occurs.
    db.Events.SubscribeToEvents(databaseDropEventSet, serverDropEventHandler);
    
    //Start event handling.
    db.Events.StartEvents();

    //Create a table on the database.
    var tb = new Table(db, "Test_Table");

    var myCol1 = new Column(tb, "Name", DataType.NChar(50))
    {
        Collation = "Latin1_General_CI_AS",
        Nullable = true
    };

    tb.Columns.Add(myCol1);
    tb.Create();

    //Remove the table.
    tb.Drop();
    
    //Wait until the events have occurred.
    int x;
    int y;
    for (x = 1; x <= 1000000000; x++)
    {
        y = x * 2;
    }

    //Stop event handling.
    db.Events.StopEvents();
}
```

---
