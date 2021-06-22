# AWSAmplify_Application

## Getting Started
### Setup fullstack project
Install the following:
- Node.js v12.x or later
- npm v5.x or later
- git v2.14.1 or later

1. Create a new xcode project(SwiftUI) - Here we are creating a Todo
2. Add Amplify to the project
Amplify is distributed through Swift Package Manager, which is integrated in xcode
Go to your project terminal and run command
`amplify init --quickstart --frontend ios`
3. Add Source Packages - File > Swift Packages > Add Package Dependency
4. Amplify iOS GitHub repo URL (https://github.com/aws-amplify/amplify-ios)
5. Chose the latest version and select Amplify, AWSAPIPlugin, and AWSDataStorePlugin libraries as of now

### Generate Model Files
1. Replace contents of `amplify/backend/api/amplifyDatasource/schema.graphql` to
```
 enum Priority {
   LOW
   NORMAL
   HIGH
 }

 type Todo @model {
   id: ID!
   name: String!
   priority: Priority
   description: String
 }
 ```
- `id` an auto-generated identifier field for a Todo item
- `name` a non-optional string field that is the title of the Todo item
- `priority` an optional enumeration type field that indicates the importance of a Todo item; the value of priority can be only LOW, NORMAL, or HIGH
- `description` an optional string field that holds more information about a Todo item

2. Generate the classes for these models 
`amplify codegen models`

### Integrate in your app
Add dataStore plugin and configure amplify to app
Make following changes to `TodoApp.swift`
```
import SwiftUI
import Amplify
import AWSDataStorePlugin

@main
struct TodoApp: App {
     // add a default initializer and configure Amplify
     public init() {
         configureAmplify()
     }

     func configureAmplify() {
     let dataStorePlugin = AWSDataStorePlugin(modelRegistration: AmplifyModels())
     do {
         try Amplify.add(plugin: dataStorePlugin)
         try Amplify.configure()
         print("Initialized Amplify");
     } catch {
         // simplified error handling for the tutorial
         print("Could not initialize Amplify: \(error)")
     }
  }
}
```

#### Create 
```
import SwiftUI
import Amplify

struct ContentView: View {

    var body: some View {
        Text("Hello, World!")
            .onAppear {
                self.performOnAppear()
        }
    }

    func performOnAppear() {
       let item = Todo(name: "Build iOS Application",
                       description: "Build an iOS application using Amplify")

        Amplify.DataStore.save(item) { result in
           switch(result) {
           case .success(let savedItem):
               print("Saved item: \(savedItem.name)")
           case .failure(let error):
               print("Could not save item to DataStore: \(error)")
           }
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

How to create an item:
`let item = Todo(name: "Build iOS Application",
                       description: "Build an iOS application using Amplify")`
`let item = Todo(name: "Finish quarterly taxes",
               priority: .high,
               description: "Taxes are due for the quarter next week")`

#### Query 
If we have some data in DataStore, you can run queries to retrieve those records. The following code shows this:
```
func performOnAppear() {
   Amplify.DataStore.query(Todo.self) { result in
       switch(result) {
       case .success(let todos):
           for todo in todos {
               print("==== Todo ====")
               print("Name: \(todo.name)")
               if let priority = todo.priority {
                   print("Priority: \(priority)")
               }
               if let description = todo.description {
                   print("Description: \(description)")
               }
           }
       case .failure(let error):
           print("Could not query DataStore: \(error)")
       }
   }
}
```

You can also use Predicates to find certain records:

The following predicates are supported:
Strings
eq ne le lt ge gt contains notContains beginsWith between
Numbers
eq ne le lt ge gt between
Lists
contains notContains

```
Amplify.DataStore.query(Todo.self,
                       where: Todo.keys.priority.eq(Priority.high)) { result in
   switch(result) {
   case .success(let todos):
       for todo in todos {
           print("==== Todo ====")
           print("Name: \(todo.name)")
           if let description = todo.description {
               print("Description: \(description)")
           }
           if let priority = todo.priority {
               print("Priority: \(priority)")
           }
       }
   case .failure(let error):
       print("Could not query DataStore: \(error)")
   }
}
```

#### Update
```
 Amplify.DataStore.query(Todo.self,
                         where: Todo.keys.name.eq("Finish quarterly taxes")) { result in
     switch(result) {
     case .success(let todos):
         guard todos.count == 1, var updatedTodo = todos.first else {
             print("Did not find exactly one todo, bailing")
             return
         }
         updatedTodo.name = "File quarterly taxes"
         Amplify.DataStore.save(updatedTodo) { result in
             switch(result) {
             case .success(let savedTodo):
                 print("Updated item: \(savedTodo.name)")
             case .failure(let error):
                 print("Could not update data in DataStore: \(error)")
             }
         }
     case .failure(let error):
         print("Could not query DataStore: \(error)")
     }
 }
```

#### Delete
```
 Amplify.DataStore.query(Todo.self,
                         where: Todo.keys.name.eq("File quarterly taxes")) { result in
     switch(result) {
     case .success(let todos):
         guard todos.count == 1, let toDeleteTodo = todos.first else {
             print("Did not find exactly one todo, bailing")
             return
         }
         Amplify.DataStore.delete(toDeleteTodo) { result in
             switch(result) {
             case .success:
                 print("Deleted item: \(toDeleteTodo.name)")
             case .failure(let error):
                 print("Could not update data in DataStore: \(error)")
             }
         }
     case .failure(let error):
         print("Could not query DataStore: \(error)")
     }
}
```
### Connect to cloud
