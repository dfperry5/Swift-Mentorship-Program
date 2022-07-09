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