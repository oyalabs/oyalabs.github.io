---
title: "LookupWord - Vapor Server"
---

## A server which record the vocabulary you've looked up

**Preparation**

> 1. [Vapor](https://vapor.codes/) - Swift web framework 
2. [PostgreSQL](https://www.postgresql.org/) - The database
3. [owlbot dictionary](https://owlbot.info/api/v1/dictionary/owl) - Where we get the definition and example
4. [Heroku](https://www.heroku.com/) - Cloud Application Platform


**Steps**

> 1. Build a server
2. Handling the HTTP request
3. Make a request to [owlbot dictionary](https://owlbot.info/api/v1/dictionary/owl)
4. Save the new word and the definition to database


------

**Step 1 - Build a server**
[Vapor introduce by Ray Wenderlich](https://videos.raywenderlich.com/screencasts/server-side-swift-with-vapor-getting-started/) - Help you to setup the basic Vapor project
create a new Vapor project

```shell
$ vapor new LookupWord
```

go to the directory

```shell
$ cd LookupWord 
```

run Xcode

```shell
$ vapor xcode
```

It'll take a while, and then press `y` to open Xcode

```shell
Select the `App` scheme to run.
Open Xcode project?
y/n> y
```

We want to see it work or not,
to do that, just change the target and Press run ▶️

![Change target and run!](http://g.recordit.co/kyCTBOKPLa.gif)

![it works](http://i.imgur.com/8YfKXMO.png)

Great, first step done!

------

**Step 2 - Handling the HTTP request***
Now, we need to do some programming.

Go back to Xcode, and go to `main.swift`

You can hold ⇧`Shift` and ⌘`Command` and press `O` to quick open a file.

I need you to replace the code to

```swift
import Vapor

let drop = Droplet()

drop.get("word", String.self) { req, word in
	
		// We need to add some code later, to get the definition.

		// return a json.
    return try JSON(node: [
        "word": word
        ])
}

drop.run()
```

After modified the file, we should run ▶️ the project again.


This code will handle a GET request to `0.0.0.0:8080/word/[the word you want to search]`.

For now, it'll return a JSON, for example:

Use your browser go to `http://0.0.0.0:8080/word/swift`
It'll return `{"newWord":"swift"}`

OK! we complete step 2!!

------

**Step 3 - Make a request to [owlbot dictionary](https://owlbot.info/api/v1/dictionary/owl)**

About [how to send a HTTP request and handle the response](https://grokswift.com/completion-handlers-in-swift/), you can check this out.

but here we need to get the definition from [owlbot dictionary](https://owlbot.info/api/v1/dictionary/owl), so we're going to add some code replace `// We need to add some code later, to get the definition.`, so the code in main.swift will looks like this:

```swift
import Vapor
import Foundation

let drop = Droplet()

drop.get("word", String.self) { req, word in

    // get the shared URLSession
    let session = URLSession.shared

    // define the URL
    let wordURLString: String = "https://owlbot.info/api/v1/dictionary/\(word)"
    guard let url = URL(string: wordURLString) else {
        print("Error: cannot create URL")
        return try JSON(node: [
            "error": "Error: cannot create URL"
            ])
    }

    // create the session task
    let task = session.dataTask(with: url, completionHandler: { (data, response, error) in

        // check for any errors
        guard error == nil else {
            print("error calling GET on /todos/1")
            print(error!)
            return
        }
        // make sure we got data
        guard let responseData = data else {
            print("Error: did not receive data")
            return
        }

        // transform to JSON object
        let json = try? JSONSerialization.jsonObject(with: responseData, options: [])
        // cast JSON to Array
        guard let jsonArray = json as? [Any] else {
            print("Error: wrong data type")
            return
        }

        // get each definition
        for jsonDict in jsonArray {

            if let wordDict = jsonDict as? [String:String]  {
                print("\n \(word) \n")
                let definition = wordDict["defenition"] ?? "no definition"
                let example = wordDict["example"] ?? "no example"
                let type = wordDict["type"] ?? "no type"

                print("definition : \(definition)")
                print("example : \(example)")
                print("type : \(type)")

            } else {
                print("Error: wrong data type")
            }

        }

    })
    task.resume()
    session.finishTasksAndInvalidate()
    
    return try JSON(node: [
        "word": word
        ])
}


drop.run()

```

Now, use browser go to `http://0.0.0.0:8080/word/success`, then you will see some information in the console:

```bash
GET /word/success

 success 

definition : the accomplishment of an aim or purpose.
example : "the president had some <b>success in</b> restoring confidence"
type : noun

 success 

definition : archaic
example : "the good or ill success of their maritime enterprises"
type : noun

```

![](http://i.imgur.com/BVdNqza.png)

------

**Step 4 - Save the new word and the definition to database**

We're in the final step now, and we'll create a Database in local environment for testing, and then deploy on the [Heroku](https://www.heroku.com/) platform, which provide a free space for us. And it also provide a database.

> also take a look at [vapor/postgresql](https://github.com/vapor/postgresql)

For setting up the database, here is another [tutorial](https://videos.raywenderlich.com/screencasts/server-side-swift-with-vapor-configuring-a-database) from Ray Wenderlich, which is very clear and helpful.

To install database, we need to open the terminal
First, install [Homebrew](http://brew.sh/) 

````bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
````

To update, run

```bash
brew update
```

And install [postgres](https://www.postgresql.org/)

```bash
brew postgre
```

To start the server

```bash
postgres -D /usr/local/var/postgres
```

Then create a database base on the user name

```bash
createdb `whoami`
```

Ok, to see if there a database you've create, 

```bash
psql
```

if you see something like this, you're good to go, 

```bash
psql (9.6.1)
Type "help" for help.

lee=#
```

type`\q` to quit.

So now, you have a database work on your computer, isn't it great?

------

Next, we need to configure Vapor to use it.
> 1. Import the package
2. configure the droplet
3. create a configuration file


First, check out the [provider page](https://github.com/vapor/postgresql-provider)

Copy the link of it and open the Xcode go to Package.swift add the dependencies
```swift
.Package(url: "https://github.com/vapor/postgresql-provider", majorVersion: 1, minor: 0)
```

So your `Package.swift` will looks like this:

```swift
import PackageDescription

let package = Package(
    name: "LookupWord",
    dependencies: [
        .Package(url: "https://github.com/vapor/vapor.git", majorVersion: 1, minor: 3),
        .Package(url: "https://github.com/vapor/postgresql-provider", majorVersion: 1, minor: 0)
    ],
    exclude: [
        "Config",
        "Database",
        "Localization",
        "Public",
        "Resources",
        "Tests",
    ]
)
```

Then we need to regenerate the Xcode dependencies, so go to terminal and run 

```bash
vapor xcode
```

To configure the droplet to use it, open the `main.swift` file.

import the `VaporPostgreSQL`

```swift
import VaporPostgreSQL
```

and add this code after `let drop = Droplet()`

```swift
try drop.addProvider(VaporPostgreSQL.Provider)
```

The `main.swift` will looks like this:

```swift
import Vapor
import VaporPostgreSQL
import Foundation

let drop = Droplet()
try drop.addProvider(VaporPostgreSQL.Provider)

// test the connection of database
drop.get("version") { req in
    if let db = drop.database?.driver as? PostgreSQLDriver {
        let version = try db.raw("SELECT version()")
        return try JSON(node: version)
    } else {
        return "No db connection"
    }

}

drop.get("word", String.self) { req, word in

    // get the shared URLSession
    let session = URLSession.shared

    // define the URL
    let wordURLString: String = "https://owlbot.info/api/v1/dictionary/\(word)"
    guard let url = URL(string: wordURLString) else {
        print("Error: cannot create URL")
        return try JSON(node: [
            "error": "Error: cannot create URL"
            ])
    }

    // create the session task
    let task = session.dataTask(with: url, completionHandler: { (data, response, error) in

        // check for any errors
        guard error == nil else {
            print("error calling GET on /todos/1")
            print(error!)
            return
        }
        // make sure we got data
        guard let responseData = data else {
            print("Error: did not receive data")
            return
        }

        // transform to JSON object
        let json = try? JSONSerialization.jsonObject(with: responseData, options: [])
        // cast JSON to Array
        guard let jsonArray = json as? [Any] else {
            print("Error: wrong data type")
            return
        }

        // get each definition
        for jsonDict in jsonArray {

            if let wordDict = jsonDict as? [String:String]  {
                print("\n \(word) \n")
                let definition = wordDict["defenition"] ?? "no definition"
                let example = wordDict["example"] ?? "no example"
                let type = wordDict["type"] ?? "no type"

                print("definition : \(definition)")
                print("example : \(example)")
                print("type : \(type)")

            } else {
                print("Error: wrong data type")
            }

        }

    })
    task.resume()
    session.finishTasksAndInvalidate()
    
    return try JSON(node: [
        "word": word
        ])
}


drop.run()
```


create a folder name it `secret` under `config`

and create a file called `postgresql.json`

> replace the user name for both "user" and "database"

```
{
    "host": "127.0.0.1",
    "user": "Your user name",
    "password": "",
    "database": "Your user name",
    "port": 5432
}
```

Then, you're good to go, run the application and go to

```
http://0.0.0.0:8080/version
```

then you'll see something like

```
[{"version":"PostgreSQL 9.6.1 on x86_64-apple-darwin16.1.0, compiled by Apple LLVM version 8.0.0 (clang-800.0.42.1), 64-bit"}]
```

------

Now we need to add two models for our database

Go to terminal, and to your project's directory,

then create two models file by following command:

```bash
touch Sources/App/Models/Word.swift
```


```bash
touch Sources/App/Models/Definition.swift
```

then regenerate the dependency again

```bash
vapor xcode
```

Then put the following code in the files respectively:

`Word.swift`

```swift
import Vapor
import Fluent
import Foundation

final class Word: Model {
    var id: Node?
    var exists: Bool = false

    var word: String


    init(word: String) {
        self.id = nil
        self.word = word

    }

    init(node: Node, in context: Context) throws {
        id = try node.extract("id")
        word = try node.extract("word")

    }

    func makeNode(context: Context) throws -> Node {
        return try Node(node: [
            "id": id,
            "word": word,
            ])
    }
}

extension Word: Preparation {
    static func prepare(_ database: Database) throws {
        try database.create("words", closure: { words in
            words.id()
            words.string("word")
        })
    }

    static func revert(_ database: Database) throws {
        try database.delete("words")
    }
}

extension Word {
    func definitions() throws -> Children<Definition> {
        return children()
    }
}

```

`Definition.swift`

```swift
import Vapor
import Fluent
import Foundation

final class Definition: Model {
    var id: Node?
    var exists: Bool = false

    var word_id: Node?
    var definition: String
    var example: String
    var type: String

    init(word_id: Node,definition: String, example: String, type: String) {
        self.id = nil
        self.word_id = word_id
        self.definition = definition
        self.example = example
        self.type = type

    }

    init(node: Node, in context: Context) throws {
        id = try node.extract("id")
        word_id = try node.extract("word_id")
        definition = try node.extract("definition")
        example = try node.extract("example")
        type = try node.extract("type")

    }

    func makeNode(context: Context) throws -> Node {
        return try Node(node: [
            "id": id,
            "word_id": word_id,
            "definition": definition,
            "example": example,
            "type": type,
            ])
    }
}

extension Definition: Preparation {
    static func prepare(_ database: Database) throws {
        try database.create("definitions", closure: { definitions in
            definitions.id()
            definitions.parent(Word.self, optional: false, unique: false, default: nil)
            definitions.string("definition")
            definitions.string("example")
            definitions.string("type")
        })
    }

    static func revert(_ database: Database) throws {
        try database.delete("definitions")
    }
}

extension Definition {
    func word() throws -> Parent<Word> {
        return try parent(word_id)
    }
}
```

Then add preparations to droplet:

```swift
let drop = Droplet()
try drop.addProvider(VaporPostgreSQL.Provider)
drop.preparations += Word.self
drop.preparations += Definition.self
```

And let's add some code to try it out, modify the main.swift as below:

> Different from the previous code, this become use drop.client to make a request

```swift
import Vapor
import VaporPostgreSQL
import HTTP

let drop = Droplet()
try drop.addProvider(VaporPostgreSQL.Provider)
drop.preparations += Word.self
drop.preparations += Definition.self


// test the connection of database
drop.get("version") { req in
    if let db = drop.database?.driver as? PostgreSQLDriver {
        let version = try db.raw("SELECT version()")
        return try JSON(node: version)
    } else {
        return "No db connection"
    }

}

//Redirect to word
drop.get() { req in

    // change to your URL
    return Response(redirect: req.uri.appendingPathComponent("word").path)
}

// Show all the words
drop.get("word") { req in
    return try JSON(node: Word.all().makeNode())
}

// Show single word
drop.get("word", String.self) { req, wordString in

    // Check if the word exist
    if let word = try Word.query().filter("word", wordString).first() {

        // if exist, show all the definition
        return try JSON(node: word.definitions().all().makeNode())

    } else {

        // create a new word and save
        var word = Word(word: wordString)
        try word.save()

        let wordDictResponse = try drop.client.get("https://owlbot.info/api/v1/dictionary/\(wordString)")

        print(wordDictResponse.json?.array ?? "no response")

        if let jsonArray = wordDictResponse.json?.array {

            for jsonDict in jsonArray {
                print(jsonDict)
                if let jsonDefinition = jsonDict as? JSON {
                    let definition = jsonDefinition["defenition"]?.string ?? "no definition"
                    let example = jsonDefinition["example"]?.string ?? " "
                    let type = jsonDefinition["type"]?.string ?? "no type"

                    //create Definition
                    var newDefinition = Definition(word_id: word.id!, definition: definition, example: example, type: type)
                    try! newDefinition.save()
                }

            }
        }

    }
    
    return try JSON(node: [
        "new_word": wordString
        ])
}


drop.run()

```

Hurrah!! now go test the application again by run it and use browser the lookup a word. 

e.g. `http://0.0.0.0:8080/word/happy`

if everything right, it'll return this first:

```
{"new word" : "happy"}
```

and if you try again, it'll show:

```
[{"definition":"feeling or showing pleasure or contentment.","example":"\"Melissa came in looking happy and excited\"","id":1,"type":"adjective","word_id":1},
{"definition":"fortunate and convenient.","example":"\"he had the happy knack of making people like him\"","id":2,"type":"adjective","word_id":1},
{"definition":"informal","example":"\"they tended to be grenade-happy\"","id":3,"type":"adjective","word_id":1}]
```

Which means, we are success!

> Now we've the function we need

> 1. Lookup a word by sending GET request
2. The server get the definition from [owlbot dictionary](https://owlbot.info/api/v1/dictionary/owl)
3. The server save the word and definition in database

------

**Deploy to Heroku**

However, now everything is local, in my computer, and it can't be asked the data by our mobile device.

So we need to deploy this online.

borrow the [deploy tutorial by Ray Wenderlich](https://videos.raywenderlich.com/screencasts/server-side-swift-with-vapor-deploying-to-heroku-with-postgresql) again.

First, create git local repository

```
git init
```

add all files

```
git add .
```

commit

```
git commit -m "init"
```

Then set up the Heroku

```
vapor heroku init
```

and keep the default setting

```bash
Would you like to provide a custom Heroku app name?
y/n>n
https://boiling-ocean-81373.herokuapp.com/ | https://git.heroku.com/boiling-ocean-81373.git

Would you like to provide a custom Heroku buildpack?
y/n>n
Setting buildpack...
Are you using a custom Executable name?
y/n>n
Setting procfile...
Committing procfile...
Would you like to push to Heroku now?
y/n>n
You may push to Heroku later using:
git push heroku master
Don't forget to scale up dynos:
heroku ps:scale web=1
```

build the database on Heroku

```
heroku addons:create heroku-postgresql:hobby-dev
```


```
heroku config
```

get something like

```
DATABASE_URL: postgres://yfghktvrmwrael:6e48ccb331711093e9ee11bc89d2ef49db4d2bde8a9b596f7b5275e8fb2c3bfc@ec2-107-20-149-243.compute-1.amazonaws.com:5432/d5cds2laqgtqqu
```

then set up the `Procfile`

```
vi Procfile
```

press `i` to insert

add this code in the end

```
--config:servers.default.port=$PORT --config:postgresql.url=$DATABASE_URL
```

It'll looks like this

```
web: App --env=production --workdir="./"
web: App --env=production --workdir=./ --config:servers.default.port=$PORT --config:postgresql.url=$DATABASE_URL
```

Finally go back to Xcode

edit the `Config/secret/postgresql.json` as below

> depend on what url you've got

```
{
    "url": "postgres://yfghktvrmwrael:6e48ccb331711093e9ee11bc89d2ef49db4d2bde8a9b596f7b5275e8fb2c3bfc@ec2-107-20-149-243.compute-1.amazonaws.com:5432/d5cds2laqgtqqu"
}
```

Then go back to terminal

add and commit the modifies by

```
git add .
git commit -m "modified Procfile"
```

Then, the very last step, deploy to Heroku

```
git push heroku master
```

then wait...

------

After wait for a long time to do it, let's try it out!

Congrats! you have your own server to record the word you've looked up.

Thanks for Swift with me.