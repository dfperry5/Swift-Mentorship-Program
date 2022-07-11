# App Protocol

## Declaration
```swift 
    protocol App 
```

## Usage
```swift
    @main
    struct MyApp: App {
        var body: some Scene {
            WindowGroup {
                Text("Hello, world!")
            }
        }
    }
```

Declare a struct, 'MyApp' that conforms to the 'App' Protocol. This defines the entry point into your swiftUI Application.

## The 'body' variable
### Declaration
```swift
    var body: Self.Body { get }
```
### Description
For any app that you create, provide a computed body property that defines your app’s scenes, which are instances that conform to the Scene protocol.

Generally an attribute on a View (or on the App) instance.

## Scene Protocol
### Declaration
```swift
protocol Scene
```

### Description
An app is created by combining one or more instances that conform to the Scene protocol. A scene is a container for a view heirarchy. The system decides when and how to present the scene in the users interface in a platform appropiate way. 

