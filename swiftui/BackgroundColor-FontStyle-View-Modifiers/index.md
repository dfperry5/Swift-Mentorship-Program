# Intro to View Modifiers, Background Color, and Font Styling

## Intro to View Modifiers

## Background Color

Let's add a background color to our entire application! 

1. First things first, navigate to the top of level of your project, and find `SwiftUI-Sample-Marvel/SwiftUI_Sample_MarvelApp`. Right now, the file should look like this:
    - ```swift
            import SwiftUI
            @main
            struct SwiftUI_Sample_MarvelApp: App {
                var body: some Scene {
                    WindowGroup {
                        ContentView()
                    }
                }
            }
        ```
2. So, backgrounds in SwiftUI are added by being adding the `background(alignment:content:)` view modifier to the top-level view, in this case `ContentView`. The background modifier takes in a specific alignment (defaulting to center), and a specific view. So, lets start with defining the `view` we want for our background.
    - Inside the `SwiftUI_Sample_MarvelApp` struct, create a variable "backgroundGradient" and define it as below:
        - ```swift
              let backgroundGradient = LinearGradient(
                    colors: [Color.purple, Color.green],
                    startPoint: .top, endPoint: .bottom
                )
        ```
    - Next, use that inside the `.background(alignment:content:)` view modifier to the `ContentView()`, and align it to the center like below:
        - ```swift
            var body: some Scene {
                WindowGroup {
                    ContentView()
                        .background(alignment: .center) {
                            backgroundGradient
                        }
                }
            }
        ``` 

3. Run the app and checkout the results! It probably won't look quite right.
    -  ![Background - First Try](images/Background-Screenshot-1.jpg)
    - What happened? Well, view modifiers only target what you've given it! So there is some weirdness because it is only the background of the `ContentView` View, which only takes up a very small part of the screen. Since the WindowGroup doesn't take on view modifiers - we are going to use something else, a Z-Stack!

4. Use a ZStack, by wrapping everything inside the WindowGroup with it, like below:
    - ```swift
        var body: some Scene {
            WindowGroup {
                ZStack {
                    ContentView()
                    .background(alignment: .center) {
                        backgroundGradient
                    }
                }
            }
        }
    ```

5.  Now, remove the background modifier from ContentView() and add backgroundGradient above the ContentView(), like below:
    - ```swift
            var body: some Scene {
                WindowGroup {
                    ZStack {
                        backgroundGradient
                        ContentView()
                    }
                }
            }
    ```

6. Now run the app again! Congrats! You made a colorful background!
    - ![Background - Full Frame](images/Full-Background.jpg)

7. See this final [code here](https://github.com/dfperry5/SwiftUI-Sample-Marvel/commit/f8e456014b70d571948c6a1803748f3c839d2f63).

7. Now, lets learn about a Z-Stack. A Z-Sack is a View where each individual, successive view child view has a higher Z-Axis value than the one preceding it. This means, that in our example if you think about the Z-STack as the depth of the view (going into the screen) - then the `backgroundGradient` we build is the first level, then the `ContentView()` sits "on top" of that gradient.

