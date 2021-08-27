# AWSAmplify_Basics

The open-source Amplify Framework provides the following products to build fullstack iOS, Android, Flutter, Web, and React Native apps:

- `Amplify CLI` - Configure all the services needed to power your backend through a simple command line interface
- `Amplify Libraries` - Use case-centric client libraries to integrate your app code with a backend using declarative interfaces
- `Amplify UI Components` - UI libraries for React, React Native, Angular, Ionic and Vue

In the following tutorial, we’ll define a GraphQL schema that we can use to provision backend resources, store data locally, sync to a cloud database, as well as receive updates over a realtime subscription.

`GraphQL` is a data language that was developed to enable apps to fetch data from APIs. It has a declarative, self-documenting style. In a GraphQL operation, the client specifies how to structure the data when it is returned by the server. This makes it possible for the client to query only for the data it needs, in the format that it needs it in.

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
`eq ne le lt ge gt contains notContains beginsWith between`
Numbers
`eq ne le lt ge gt between`
Lists
`contains notContains`

For example: here we are returning queries which have a high priority
Amplify.DataStore.query(Todo.self,
                       where: Todo.keys.priority.eq(Priority.high)) { result in
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

#### Provision backend
DataStore is now persisting locally, next step is to connect to cloud. We will create an AWS AppSync API and configure DataStore to synchronize data to it.
1. Configure Amplify to manage cloud resources - run `amplify configure` - this creates a new IAM User
2. Initialize the Amplify backend - `amplify init`
3. Push your new backend to the cloud - ` amplify push`
4. Answer no to "Do you want to generate code for your newly created GraphQL API?"

#### Enable cloud syncing
In order to enable cloud syncing you need to configure your application to use the Amplify API category.
Replace configure function in TodoApp.swift to:
```
func configureAmplify() {
   let models = AmplifyModels()
   let apiPlugin = AWSAPIPlugin(modelRegistration: models)
   let dataStorePlugin = AWSDataStorePlugin(modelRegistration: models)
   do {
       try Amplify.add(plugin: apiPlugin)
       try Amplify.add(plugin: dataStorePlugin)
       try Amplify.configure()
       print("Initialized Amplify");
   } catch {
       assert(false, "Could not initialize Amplify: \(error)")
   }
}
```
