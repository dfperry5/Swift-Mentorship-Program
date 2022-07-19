# The Marvel Public API

### Get Your Keys
1. Navigate to https://developer.marvel.com/account, create a free account and retrieve your public and private keys. The folks at Marvel are kind enough to give you a maximum of 3000 calls / day.

2. Next, lets check out [Marvel's interactive documentation](https://developer.marvel.com/docs). We're going to focus on the `/v1/public/character` API, in this section.
    - This API wil return you a list of comic book characters, with a good amount of detail about each character. There is also a lot of documentation on the structure of the response which will be important when we begin to create `Models`, or objects in our Swift Code that represent this API Response. But, more on that later.
    - ![The Documentation](images/Characters-API-Screenshot.png)


3. Let's investigate this API a little more. As you scroll down, you'll see a list of parameters.
    - ![API Parameters](images/Parameters.png)
    - This tells you what you can pass in to change or update the response. The one missing is your API Secret / Public - the tool is automatically adding it for you. So try it out, and have fun!

4. Now, lets make this call from your application.


## Connecting To Marvel API

1. Launch your project. Once XCode opens up, go ahead and create a folder/group named `Services` at the same level as your `ContentView.swift`. After this, create another folder/group inside of `Services` and name it `MarvelService`. Now, we will need to create 3 Swift files:
    - MarvelService.swift
    - MarvelService+Config.swift
    - MarvelService+Models.swift

2. Let's begin filling out these files. First, lets start with MarvelService.swift.
    - For now, just create a basic struct in the file named Marvel Service. It should look like:
        - ```swift
                public struct MarvelService {}
            ```
    - For now, we will leave this as it.

3. Next, let's update the file `MarvelService+Config.swift`.
    - In this file. we're going to store configurations, and extend the Marvel Service struct you just defined. The configuration will hold 3 values: `privateKey`, `publicKey`, and `URL`.
        - The `privateKey` and `publicKey `should be what you retrieved from the Marvel Developer site. You should not commit these values to your repo.
        - the `URL` will be the URL of the API we want to call - in this case: "https://gateway.marvel.com:443/v1/public/characters"
    - Now, we're going to do this with some code, like below. First, declare a public extension to the MarvelService struct. Then, declare a public enum inside of it named `Config`. Inside that enum, declare the three valuese we mentioned above (privateKey, publicKey, URL) as public static constants, using the `let` keyword. 
        - ```swift
                public extension MarvelService {
                    enum Config {
                        public static let publicKey = "Your Public Key Here"
                        public static let privateKey = "Your Private Key Here"
                        public static let URL = "https://gateway.marvel.com:443/v1/public/characters"
                    }
                }
            ```

4. Finally, let's fill out the MarvelService+Models.swift file. Before we get to that though, revisit the [Marvel API Documentation](https://developer.marvel.com/docs#!/public/getCreatorCollection_get_0) for the `/v1/public/characters` API.
    - The documentation shows the basic structure of the response with some definitions of the values and what they represent. In our models file, we will need to create these structures (seen below).
        - ![Marvel Response Types](images/Response-Types.png)
    - In the code, we will define each entry (`CharacterDataWrapper`, `CharacterDataContainer`, `Character`, and eventually a flavor of `Image`). All of these will conform to the `Codable` protocol which means that Swift will automatically translate JSON into a Swift object.

5. Let's add code for the first 3 types - `CharacterDataWrapper`, `CharacterDataContainer`, and `Character`. It'll look something like this:
    - ```swift
      struct Character: Codable {
            public let id: Int
            public let name: String
            public let description: String
            public let thumbnail: MarvelImage
        }

        struct CharacterDataContainer: Codable {
            public let offset: Int
            public let limit: Int
            public let total: Int
            public let count: Int
            public let results: [Character]
        }

        struct CharacterDataWrapper: Codable {
            public let code: Int
            public let status: String
            public let data: CharacterDataContainer
        }
        ```
    - Note the use of `MarvelImage`, where you may expect `Image` to be - the reason is that `Image` is a reserved word in Swift. We will use `MarvelImage` to represent the Image type in the Marvel response.

6. Now, adding in the MarvelImage struct will be a bit more tricky. First, define the struct.
    - ```swift
            struct MarvelImage: Codable { }
        ```
    - Now, one of the attributes of the `MarvelImage` (or just Image in the Marvel Developer docs) is named `extension`. This is, once again, a protected word in Swift. So we will need to use coding keys to get around this. This will end up looking like the code below - add this snippet inside the `Models` enum, where you added different types.
    - ```swift
        struct MarvelImage: Codable {
            public let path: String
            public let imageExtension: String
            enum CodingKeys: String, CodingKey {
                case path
                case imageExtension = "extension"
            }
        }
        ```
    - Notice the use of Coding Keys - where we define that the `imageExtension` attribute of the `MarvelImage` struct - corresponds to the `extension` attribute on the `Image` in the Marvel documentation.

## Calling the Service

1. Let's return to the `MarvelService.swift` file. We are now going to add in additional functions to actually make the request. Let's create two different functions.
    - Inside your empty `MarvelService` struct, create the private function `buildUrlString`, that returns a string. The function should look like below:
        - ```swift
                private func buildUrlString() -> String {}
            ```
    - Create a second function, this one will be public and called `getCharacters`, it will be asynchronous, it can throw an error, and it wll return an instance of `Models.CharacterDataWrapper`. The function signature should look like below:
        - ```swift
                public func getCharacters() async throws -> Models.CharacterDataWrapper {}
            ```

2. Now, lets fill out the `buildUrlString` function. The purpose of this function is to add in the additional URL parameters required for the service to work. Visit the Marvel Developer documentation for [Authorization](https://developer.marvel.com/documentation/authorization). 
    - Generally, we would try to do the client-side authorization, but that is a bit more of an involved process. Instead, we will follow the `Authentication for Server-Side Applications`. The process described here is creating a timestamp URL Parameter (`ts`), the public API Key `apikey`, and a hash value (`hash`) that is 'a md5 (a hasing algorithim) digest of the ts parameter, your private key and your public key (e.g. md5(ts+privateKey+publicKey)'.

3. Let's create the timestamp (`ts` value). Inside the `buildUrlString` function, add this line:
    - ```swift
            let timeStamp =  Int64((Date().timeIntervalSince1970 * 1000.0).rounded())
        ``` 
    - This converts the Day to an epoch (IE miliseconds since 1970), and rounds it to make sure there are no decimals present which would mess up the URL String.

4. Next, define the hash String. This will just be a string combination of the timestamp you just created, the private key, and the public key. Define this varibale as `hashString`, like below:
    - ```swift
            let hashString = timeStamp.description + Config.privateKey + Config.publicKey
        ```

5. We are now ready to create the hash value. First, we must import CryptoKit to get access to the md5 alogirthm. At the top of the file import this library like below:
    - ```swift
            import CryptoKit
        ```
    - Now, add a third line to the `buildUrlString` function that creates the md5 hash.
        - ```swift
                let hashValue = Insecure.MD5.hash(data: hashString.data(using: .utf8) ?? Data())
            ```

6. We're almost done with this function! We now need to retrieve the digest from the hash. This is done with a single line, like below:
    - ```swift
            let hash = hashValue.map { String(format: "%02hhx", $0) }.joined()
        ```
    - Finally, let's put it all together to define a URL String.
        - ```swift
                return Config.URL + "?ts=\(timeStamp.description)&apikey=\(Config.publicKey)&hash=\(hash)"
            ```

7. The final `buildUrlString` function looks like: 
    - ```swift
            private func buildUrlString() -> String {
                let timeStamp =  Int64((Date().timeIntervalSince1970 * 1000.0).rounded())
                let hashString = timeStamp.description + Config.privateKey + Config.publicKey
                let hashValue = Insecure.MD5.hash(data: hashString.data(using: .utf8) ?? Data())
                let hash = hashValue.map { String(format: "%02hhx", $0) }.joined()
                
                return Config.URL + "?ts=\(timeStamp.description)&apikey=\(Config.publicKey)&hash=\(hash)"
            }
        ```

8. Now, it's time to put this together to build and make the request. As this is an asynchrous function, we must wrap everything in a `do/catch` block. Go ahead and fill out the catch block with a log for the error and a fatalError.
    - Now youre code should look like this:
        - ```swift
                  public func getCharacters() async throws -> Models.CharacterDataWrapper {
                    do {
                    } catch {
                        print("Error - ! \(error)")
                        fatalError("Couldn't create response")
                    }
                }
            ```

9. Let's begin filling out the `do` piece of the function. First, let's get the URL. To do this, we will use our  `buildUrlString` function we just finished writing. Wrap this in a guard statement, to make sure the URL is created. If it fails, throw another fatalError.
    - ```swift
            public func getCharacters() async throws -> Models.CharacterDataWrapper {
                do {
                    guard let url = URL(string: self.buildUrlString()) else {
                        fatalError("Missing URL")
                    }
                } catch {
                    print("Error - ! \(error)")
                    fatalError("Couldn't create response")
                }
            }
        ```

10. Now, let's build the request, using the built-in Swift URLRequest, and providing it with the URL created in the previous step. 
    - ```swift
            public func getCharacters() async throws -> Models.CharacterDataWrapper {
                do {
                    guard let url = URL(string: self.buildUrlString()) else {
                        fatalError("Missing URL")
                    }
                    let request = URLRequest(url: url)

                } catch {
                    print("Error - ! \(error)")
                    fatalError("Couldn't create response")
                }
            }
        ```

11. Now lets make the request, using Apples Async / Await framework and URLSession. Pass in the request you just crated to the shared URLSession's data functionm like below:
    - ```swift
            let (data, response) = try await URLSession.shared.data(for: request)
        ```
        - Notice that we're using try (cause the function can throw an error) and the `await` keyword which signals that we should wait for this function to return, and is what makes the `async` keyword in the function signature necessary.

12. At this point, we are now ready to parse the response, decode it, and return it as the `CharacterDataWrapper`. We are first going to check the response objects status code. If that is not 200, throw an error. After that, lets use the JSONDecoder to convert the data to a Swift structure, `CharacterDataWrapper`. Your code in the function should now look like below:
    - ```swift
            public func getCharacters() async throws -> Models.CharacterDataWrapper {
                do {
                    guard let url = URL(string: self.buildUrlString()) else {
                        fatalError("Missing URL")
                    }
                    let request = URLRequest(url: url)
                    let (data, response) = try await URLSession.shared.data(for: request)
                    guard (response as? HTTPURLResponse)?.statusCode == 200 else {
                        fatalError("Error while fetching data")
                    }
                    let wrapper = try JSONDecoder().decode(Models.CharacterDataWrapper.self, from: data)
                    return wrapper
                } catch {
                    print("Error - ! \(error)")
                    fatalError("Couldn't create response")
                }
            }
        ```

13. Voila! You have created a function to call the Marvel API, used extensions, private and public keys, CryptoKit, and perhaps most interestingly async/await! Next we will call this from the basic UI!