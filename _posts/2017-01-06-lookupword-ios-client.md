---
title: "LookupWord - iOS client"
categories:
  - Tutorial
tags:
  - iOS
  - Alamofire
  - SwiftyJSON
  - RxSwift
  - swift
---

## An iOS app can browse the words you've saved by the lookup word server


For this project, we're going to build a very simple iOS app, grab the data from the server, and display it.

And I'm going to use couple of greatest 3rd-party tools in the field to make this app easy to build.

It's really a good way to practice these tools by working on a interesting project like this 😜

### Preparation


> 1. [Alamofire](https://github.com/Alamofire/Alamofire) - an HTTP networking library written in Swift
2. [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) - makes it easy to deal with JSON data in Swift
3. [RxSwift](https://github.com/ReactiveX/RxSwift) - the fantastic way to deal with data flow
4. [Realm](https://realm.io/docs/swift/latest/#installation) - the popular mobile database
4. [Carthage](https://github.com/Carthage/Carthage) - light weight package manager for Cocoa application



### Steps


> 1. Create a Xcode project
2. Set up the dependencies
3. Get the data from the server
4. Set up database and create models
5. Create views to display
6. Display the data


**Step 1 - Create a Xcode project**

Create a project, choose `Single View Application` as the template, and name it `LookupWordClient` or what ever you want, and choose `Swift` as the language, then you can left rest as default.

![](http://i.imgur.com/9GdIJU2.png)

Then you can press run▶️(or ⌘ + `R`) to run the application, the simulator will open and shows a blank screen, and you've completed first step.

------

**Step 2 - Set up the dependencies**

Second step, before we build our app, we need to deal with the dependencies, which is a little confusing for beginner, especially when there are some many different versions out there, you sometimes don't know which one should be used, even don't know there is an update. So, we need some tool for us to manage it, which is the package manager, there are couple out there, and for this tutorial, I'll use the [Carthage](https://github.com/Carthage/Carthage) to do it, which is relative new if compare with [Cocoapods](https://cocoapods.org/) but lighter weight, and current more stable than [Swift Package Manager](https://swift.org/package-manager/), which is the developing official package manager still in early stage at the moment I'm writing.

***Install [Carthage](https://github.com/Carthage/Carthage)***

We're going to use [Homebrew](http://brew.sh/), the tool is used in the previous tutorial, if you're not install yet, go to the site and see how to install.

Update the brew first:

```bash
brew update
```

and install [Carthage](https://github.com/Carthage/Carthage)

```bash
brew install carthage
```

***Create a Cartfile***

Open terminal and go to your project directory, in my case:

```
cd Documents/SwiftWithMe/LookupWordClient/
```

create a file called `Cartfile`

```bash
touch Cartfile
```

and use the editor you like to edit the file as:

```
github "Alamofire/Alamofire" ~> 4.0
github "SwiftyJSON/SwiftyJSON"
github "ReactiveX/RxSwift" ~> 3.0
github "realm/realm-cocoa"
```

then back to terminal, update carthage

```bash
carthage update
```

this will take quite amount of time, so let's wait a moment.

------

After the update is done, we need to set something up,

Drag the frameworks we need

![](http://i.imgur.com/yMVVt7i.png)

Drag to here, in the application targets `General` setting tab, to `Linked Frameworks and Libraries` where at the end of the page.

![](http://i.imgur.com/1h6153p.png)

Go to application targets `Build Phases` setting tab and add(`+`) `New Run Script Phase`

![](http://g.recordit.co/arqqCcSrvl.gif)  

And add this in the script field:

```bash
/usr/local/bin/carthage copy-frameworks
```

Add input files for each framework like the image

![](http://i.imgur.com/oBcUJfT.png)

Try to run▶️ it again, and if it success, then we're good to start to develop!

------

### Step 3 - Get the data from the server

Now we want to ask the vocabulary data from the server, and since the `ViewController` is the initial view controller, I'll start write code from here, use the `Alamofire` to send the HTTP request.

We need to `import Alamofire` and add some [example code](https://github.com/Alamofire/Alamofire), and the `Viewcontroller.Swift` will looks like:

> Please use your server app's URL to replace "https://boiling-ocean-81373.herokuapp.com/word"

```swift
import UIKit
import Alamofire


class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        Alamofire.request("https://boiling-ocean-81373.herokuapp.com/word").response { response in
            print("Request: \(response.request)")
            print("Response: \(response.response)")
            print("Error: \(response.error)")

            if let data = response.data, let utf8Text = String(data: data, encoding: .utf8) {
                print("Data: \(utf8Text)")
            }
        }

    }

}
```

Then you can run the application again, the console will print out the information like this:

```
Request: Optional(https://boiling-ocean-81373.herokuapp.com/word)
Response: Optional(<NSHTTPURLResponse: 0x6000002211c0> { URL: https://boiling-ocean-81373.herokuapp.com/word } { status code: 200, headers {
    Connection = "keep-alive";
    "Content-Length" = 506;
    "Content-Type" = "application/json; charset=utf-8";
    Date = "Sat, 07 Jan 2017 09:16:17 GMT";
    Server = Cowboy;
    Via = "1.1 vegur";
} })
Error: nil
Data: [{"id":1,"word":"happy"},{"id":2,"word":"cry"},{"id":3,"word":"strong"},{"id":4,"word":"great"},{"id":5,"word":"tackle"},{"id":6,"word":"create"},{"id":7,"word":"amazing"},{"id":8,"word":"agent"},{"id":9,"word":"sort"},{"id":10,"word":"make"},{"id":11,"word":"kick"},{"id":12,"word":"kickass"},{"id":13,"word":"brick"},{"id":14,"word":"head"},{"id":15,"word":"think"},{"id":16,"word":"option"},{"id":17,"word":"path"},{"id":18,"word":"parameter"},{"id":19,"word":"compile"},{"id":20,"word":"Philanthropy"}]

```

Which you can see, in the data, there are the words we've looked up, you get it from the server by ourselves. Now, the only thing we need to do is how can we display this pieces of information properly.

So, let's move to next step.

------

### Step 4 - Set up database and create models

We do can just extract the information from the data using JSON, and transform it to maybe dictionary of the word. But the would be a mess. So here we're going to build the model for it to make us access the information more easily.

Just like the server's data models, we need a word and a definition model to store the data, let's refer the [Realm's Document]() to see what is the right way to do it.

Create a new group(like new folder) called `Model`, and create two swift file `Word.swift` and `Definition.swift` respectively.

`Word.swift`

```swift
import Foundation
import RealmSwift

class Word: Object {
    dynamic var word = ""
    let definitions = List<Definition>()
}
```

`Definition.swift`

```swift
import Foundation
import RealmSwift

class Definition: Object {
    dynamic var definition = ""
    dynamic var example = ""
    dynamic var type = ""
    dynamic var word: Word? // Properties can be optional
}
```

Which the `Word` object has a word string property and a list of definitions, and the `Definition` also has definition string, example and type, also a word refer to the `Word` object.

Now, try to use the model to save the data from the internet.

Let's import `RealmSwift` and `SwiftyJSON`, and save every word we got from the server, the code is like below:

```swift
import UIKit
import Alamofire
import RealmSwift
import SwiftyJSON


class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        Alamofire.request("https://boiling-ocean-81373.herokuapp.com/word").response { response in
            print("Request: \(response.request)")
            print("Response: \(response.response)")
            print("Error: \(response.error)")

            guard let data = response.data else {
                print("Data: no data")
                return
            }

            let json = JSON(data: data)
            if let wordList = json.array {

                // Get the default Realm
                let realm = try! Realm()

                for word in wordList {
                    let newWord = Word()
                    newWord.word = word["word"].stringValue
                    try! realm.write {
                        realm.add(newWord)
                    }
                }

                let allWords = realm.objects(Word.self)

                print(allWords)
            }

        }

    }

}

```

then run it, will print out like:

```
Request: Optional(https://boiling-ocean-81373.herokuapp.com/word)
Response: Optional(<NSHTTPURLResponse: 0x608000026a40> { URL: https://boiling-ocean-81373.herokuapp.com/word } { status code: 200, headers {
    Connection = "keep-alive";
    "Content-Length" = 506;
    "Content-Type" = "application/json; charset=utf-8";
    Date = "Sat, 07 Jan 2017 10:06:10 GMT";
    Server = Cowboy;
    Via = "1.1 vegur";
} })
Error: nil
Results<Word> (
	[0] Word {
		word = happy;
		definitions = RLMArray <0x6080000f2480> (
		
		);
	},
	[1] Word {
		word = cry;
		definitions = RLMArray <0x6080000f2580> (
		
		);
	},
	[2] Word {
		word = strong;
		definitions = RLMArray <0x6080000f2700> (
		
		);
	},
	[3] Word {
		word = great;
		definitions = RLMArray <0x6080000f2880> (
		
		);
	}
)
```


That's great, but still has some problem like I don't want every time I run the app will add the word again, so we need to check if the word is exist in our database, also we need to ask for the definition of the word as well, so below is the program after we'd modified:

```swift
import UIKit
import Alamofire
import RealmSwift
import SwiftyJSON


class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        Alamofire.request("https://boiling-ocean-81373.herokuapp.com/word").response { response in

            guard let data = response.data else {
                print("Data: no data")
                return
            }

            let json = JSON(data: data)
            if let wordList = json.array {

                // Get the default Realm
                let realm = try! Realm()

                for word in wordList {
                    let wordString = word["word"].stringValue
                    let wordResult = realm.objects(Word.self).filter("word = '\(wordString)'")
                    let isNewWord = (wordResult.count == 0)
                    if isNewWord {
                        let newWord = Word()
                        newWord.word = wordString

                        Alamofire.request("https://boiling-ocean-81373.herokuapp.com/word/\(wordString)").response { response in

                            guard let data = response.data else {
                                print("Data: no data")
                                return
                            }
                            let json = JSON(data: data)
                            if let definitionList = json.array {

                                for definitionJson in definitionList {

                                    let newDefinition = Definition()
                                    newDefinition.definition = definitionJson["definition"].stringValue
                                    newDefinition.example = definitionJson["example"].stringValue
                                    newDefinition.type = definitionJson["type"].stringValue
                                    newDefinition.word = newWord

                                    try! realm.write {
                                        newWord.definitions.append(newDefinition)
                                        realm.add(newDefinition)
                                        print(newWord.definitions)
                                    }

                                }

                            }


                        }

                        try! realm.write {
                            realm.add(newWord)
                        }
                    }
                }

                let allWords = realm.objects(Word.self)

                print(allWords)
            }

        }

    }

}

```


By this point, if you want to see the data more clearly, I recommend you download the [Realm Browser](https://itunes.apple.com/hk/app/realm-browser/id1007457278?mt=12), then find the path of your `.realm` file.

To do that, set a breakpoint after `super.viewDidLoad()` and type the command in the console, which will give you a path to your `.realm` file in the Simulator directory:

```
po Realm.Configuration.defaultConfiguration.fileURL
```

After you've download the [Realm Browser](https://itunes.apple.com/hk/app/realm-browser/id1007457278?mt=12)

Go to terminal, type `open ` and the path like this:

```
open file:///Users/lee/Library/Developer/CoreSimulator/Devices/8A550E52-D4A5-4C52-9896-9C55361D5411/data/Containers/Data/Application/C864FAEB-6397-4148-8AC6-7952DF74F509/Documents/default.realm
```

It'll open the browser, which is a tool can manipulate the data and watch it changing in real-time.


### Step 5 - Create views to display

I'm glad that you follow through here, let's deal with the view now.

I'm going to both use lovely [UITableView](https://developer.apple.com/reference/uikit/uitableview) to display word list and the definition list.

And that don't concern about the layout, focus on the data flow first, because, these two aspect can be treated separately, for the clarity.

We're going the have a table view to display wall the words in the database, maybe the search function in the future, but let's start from the basic. So, what is the data every table view cell will needs? What word, of course, and for the basic, that is the only string we need.

### Step 6 - Display the data

