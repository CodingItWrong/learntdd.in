### Specify adding a post [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/71b7fa4bfd998768c889877d9fb5b19b74dfbe1d)

#### Acceptance Tests/CreatingAPostTest.m

```diff
{% raw %}+#import <KIF/KIF.h>
+
+@interface CreatingAPostTests : KIFTestCase
+
+@end
+
+@implementation CreatingAPostTests
+
+- (void)testItCanCreateAPost
+{
+    NSString *testMessage = @"Hello, test!";
+    
+    [tester enterText:testMessage intoViewWithAccessibilityLabel:@"New Message Field"];
+    [tester tapViewWithAccessibilityLabel:@"Add Post"];
+    
+    UITableViewCell *postCell = [tester waitForCellAtIndexPath:[NSIndexPath indexPathForRow:0 inSection:0]
+                        inTableViewWithAccessibilityIdentifier:@"Posts Table"];
+    
+    XCTAssertTrue([postCell.textLabel.text isEqualToString:testMessage],
+                  @"Expected cell label to be '%@', was '%@'", testMessage, postCell.textLabel.text);
+}
+
+@end{% endraw %}
```

To start out, we write an acceptance test for the entire feature we want to build. This test specifies that the user will enter a message into a field, tap an Add button, and then see that message in the first cell of a table.

The first error we get is that there is no message field:

Red: Failed to find accessibility element with the label "New Message Field"


### Add new message field [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/c30af43f582799b8c7f02befca11a966b24897e3)


![Message Field](message-field.png)

As part of adding the message field, we go ahead and set up the table view controller we'll be using. We set the accessibility label on the field to "New Message Field" so the acceptance test can find it.

The next error we get is similar: now we can't find the Add Post button:

Red: Failed to find accessibility element with the label "Add Post"


### Add Add Post button [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/3cd865fb5b24274d0f87498ebad70ab416721c93)


![Add Button](add-button.png)

We simply set the accessibility label on the button to "Add Post" so the test can find it.

Next, the test can't find the table to look in:

Red: Could not find element with accessibilityIdentifier == "Posts Table"


### Set table accessibility identifier [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/8dead2618d1f04a0f0d829e037665f22a97ebfb0)

#### Learn TDD in Cocoa Touch/PostsViewController.swift

```diff
{% raw %}+import UIKit
+
+class PostsViewController: UITableViewController {
+    
+    override func viewDidLoad() {
+        tableView.accessibilityIdentifier = "Posts Table"
+    }
+
+}{% endraw %}
```

Interface Builder doesn't provide a way to set the accessibility identifier on the table, so we need to set it in our code instead. We create a PostsViewController class that subclasses UITableViewController, then we set the table's accessibility identifier in the `viewDidLoad` method.

Next, the acceptance test can find the table, but it expects there to be a row added to the table, and there isn't one:

Red: Row 0 is not found in section 0 of table view


### Display contents of post store in table view [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/c1a836fbf7ae8be24230272c28adb63a1ace1751)

#### Learn TDD in Cocoa Touch/PostsViewController.swift

```diff
{% raw %} 
 class PostsViewController: UITableViewController {
     
+    var store: PostStore!
+    
     override func viewDidLoad() {
         tableView.accessibilityIdentifier = "Posts Table"
+        store = PostStore()
+    }
+    
+    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
+        return store.count
+    }
+    
+    
+    @IBAction func addPost(sender: AnyObject) {
+        store.create()
+        tableView.reloadData()
     }
 
 }{% endraw %}
```

We create a `PostStore` to hold the posts that we want to display in the table. We set the Add button to call a `create()` method on the PostStore, and we set the table to check the `PostStore`'s `count` property when determining the number of rows to display. Because the acceptance error we're trying to correct is the missing row, that's all the `PostStore` functionality that we need at this point.

Red: Use of undeclared identifier ‘PostStore’

We get an error that the `PostStore` we've referred to doesn't exist.


### Add post store class [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/b839ca71fae7c451cb6c65fb4c18c5e4e2afd6cf)

#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %}+class PostStore {
+
+}{% endraw %}
```

To make sure we don't implement more than the tests are driving us to, we do the minimum necessary to get past the current "undeclared identifier": we declare a `PostStore` class. After this, we get several errors preventing the code from compiling. Let's take them one at a time:

First Red: Value of type ‘PostStore’ has no member ‘count’


### Add count property [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/bc3f3b943a478b9c93428fec001943b0df06e51a)

#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %} class PostStore {
 
+    var count: Int {
+        get {
+            return 0
+        }
+    }
+
 }{% endraw %}
```

We declare a `count` property to fix the error that one is missing. Because we know its value will ultimately be dynamically determined from the number of posts in the store, we make it a calculated property. But for now, we just return a hard-coded zero value to get the code to compile.

First Red: Value of type `PostStore` has no member `create`

With that, the compiler is satisfied with our `count` property, and the next error is that `PostStore` also needs a `create()` method.


### Add create function [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/d160651d63941d7ded4a8c9fde31b6bb3db232d6)

#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %}         }
     }
 
+    func create() {
+        
+    }
+    
 }{% endraw %}
```

We add a `create()` function that the compiler is asking for. With this, the compiler is happy with our `create()` function, because it doesn't take any parameters or return a value.

With this, we're past the compilation errors, and back to a failing acceptance test assertion:

Red: Row 0 is not found in section 0 of table view

This is the same failed assertion we saw before. We've made the number of rows in the table driven off of the `PostStore`, but because the `PostStore` always returns a count of 0, the row we need never appears. We need the `PostStore`'s `count` to return 1 after `create()` is called.


### Specify count should be 1 after creating [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/68a0086270aa5223f77401164db937c51784846c)

#### Unit Tests/PostStoreTests.swift

```diff
{% raw %}+import XCTest
+@testable import Learn_TDD_in_Cocoa_Touch
+
+class PostStoreTests: XCTestCase {
+    
+    var store: PostStore!
+    
+    override func setUp() {
+        super.setUp()
+        store = PostStore()
+    }
+    
+    override func tearDown() {
+        super.tearDown()
+    }
+    
+    func testItReturnsACountOfOneAfterCreating() {
+        store.create()
+        
+        let count = store.count
+        
+        XCTAssertEqual(count, 1, "Expected store.count() to be 1 but was \(count)")
+    }
+}{% endraw %}
```

This is a behavioral need, so instead of implementing the code to fix this directly, we step down from the acceptance test level and write a unit test for the `PostStore`. Our goal is to reproduce the acceptance test failure at the unit level, so that we know exactly what this unit needs to do to accomplish our overall goal.

Red: Expected store.count() to be 1 but was 0

This failed assertion is exactly the same problem we see at the acceptance level, so now we're ready to implement this behavior.


### Increase post count when `create()` is called [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/6d3f61b6775f8b90e3b6c8bdb862306a0ba5fe7f)

#### Learn TDD in Cocoa Touch/Post.swift

```diff
{% raw %}+class Post {
+    
+}{% endraw %}
```


#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %} class PostStore {
 
+    private var posts = [Post]()
+
     var count: Int {
         get {
-            return 0
+            return posts.count
         }
     }
 
     func create() {
-        
+        let post = Post()
+        posts.append(post)
     }
     
 }{% endraw %}
```

To get the post count to increase when `create()` is called, we _could_ just add an Int property and increment it. However, taking such a small step doesn't add a lot of value. It's often more useful to "write the code we wish we had." In this case, we know the `PostStore` is going to have a list of `Post` objects, so it seems safe to go ahead and have the `PostStore` keep an array of `Post`s, add a new `Post` every time `create()` is called, and delegate the `PostStore`'s `count` property to the array's `count`. The compiler requires us to go ahead and create a `Post` class, but we don't add anything else to it at this point: we wait until the tests drive us to add data and functionality to it.

This gets our unit test passing, and moves our acceptance test past its current error to a new one.

Inner green
Outer red: UITableView…failed to obtain a cell from its dataSource

The table view is trying to create a cell, which means we succeeded in increasing the row count when the user clicks "Add." But now the table view needs to know how to create a cell.


### Set up cell for table view [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/fd35fb6cb63faab1ccb3b8a1e8916ee1c2913a13)

#### Learn TDD in Cocoa Touch/PostsViewController.swift

```diff
{% raw %}         return store.count
     }
     
+    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
+        let cell = tableView.dequeueReusableCellWithIdentifier("Cell", forIndexPath: indexPath)
+        return cell
+    }
     
     @IBAction func addPost(sender: AnyObject) {
         store.create(){% endraw %}
```

We do the minimum necessary to get past the current acceptance test error: we implement `tableView(_:cellForRowAtIndexPath:)` to dequeue a cell with the identifier "Cell" (we only need one kind of cell), and we mark the prototype cell in our storyboard with that identifier.

Red: Expected cell label to be 'Hello, test!', was '(null)'

Now the cell is retrieved, but our assertion about the content of the cell's label is failing. Because we didn't customize the cell at all, it doesn't have a label by default.


### Pass message text along into post [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/00cf721166976df31b260ab2d09741d62c8f8d73)

#### Learn TDD in Cocoa Touch/PostsViewController.swift

```diff
{% raw %}     
     var store: PostStore!
     
+    @IBOutlet var messageField: UITextField!
+    
     override func viewDidLoad() {
         tableView.accessibilityIdentifier = "Posts Table"
         store = PostStore()
...
     
     override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
         let cell = tableView.dequeueReusableCellWithIdentifier("Cell", forIndexPath: indexPath)
+        let post = store.find(indexPath.row)
+        
+        cell.textLabel?.text = post.message
+        
         return cell
     }
     
     @IBAction func addPost(sender: AnyObject) {
-        store.create()
-        tableView.reloadData()
+        if let message = messageField.text {
+            store.create(["message": message])
+            tableView.reloadData()
+        }
     }
 
 }{% endraw %}
```

To get the right text to show up in the cell, when a message is saved we need to retrieve the text from the text field, and pass it into the `PostStore`'s `create()` method. Then, when we're displaying the cell, we need to retrieve a `Post` from the `PostStore` and use its `message` value for the cell's label.

To accomplish this, we add a dictionary parameter to `create()`, and we add a `find()` method to retrieve a `Post`.

First Red: Value of type `PostStore` has no member `find`

The first compilation error we get is that there is no `find()` method on `PostStore`.


### Add find method [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/a40f122b643af578fd3eae33f49d0a7946af328a)

#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %}         posts.append(post)
     }
     
+    func find(index: Int) -> Post {
+        return Post()
+    }
+    
 }{% endraw %}
```

Rather than dropping down to the unit level yet, we fix the compilation error right away, because until our code compiles we can't get any other feedback from tests. We simply add a `find()` method that takes an `Int` parameter and returns a `Post`. To do the minimum necessary to satisfy the compiler, we just instantiate and return a new empty `Post`.

Red: Value of type `Post` has no member `message`

Now our `Post` is being returned to the view controller, but when it tries to access the `message` property, there isn't one.


### Add message field to post [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/47bf7df35ec1f224c4641e1a84cd7bed88df70c7)

#### Learn TDD in Cocoa Touch/Post.swift

```diff
{% raw %} class Post {
     
+    var message: String?
+    
 }{% endraw %}
```

We add a simple `message` property to `Post` that will store our message string. This gets us to our next unrelated compilation error:

Red: Argument passed to a call that takes no arguments

Previously `create()` didn't have any arguments because all it needed to do was result in an incremented `count`. Now that our view controller is passing a dictionary into it with the message string, we need to update our other code to match.


### Add post data param [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/c8f8fe42b2cbbf7b10a6ae72ed324ad29d26cda1)

#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %}         }
     }
 
-    func create() {
+    func create(postData: [String:String]) {
         let post = Post()
         posts.append(post)
     }{% endraw %}
```


#### Unit Tests/PostStoreTests.swift

```diff
{% raw %}     }
     
     func testItReturnsACountOfOneAfterCreating() {
-        store.create()
+        store.create([:])
         
         let count = store.count
         {% endraw %}
```

We add a dictionary argument to the `create()` method, as well as updating all references to it (in this case, just one in the `PostStoreTests`). This gets us past our last compilation error, and now we get an acceptance test failure:

Red: Expected cell label to be 'Hello, test!', was '(null)'

Like last time, this is the same acceptance test failure we were getting previously. We've added the parameters and properties to pass the message along from the text field into the cell label, but we haven't actually persisted it: in our `find()` method, we just return an empty `Post`.


### Add store unit test for find [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/cd6b14aae38590ea05257bee1112d65bf4d8aaea)

#### Unit Tests/PostStoreTests.swift

```diff
{% raw %}         
         XCTAssertEqual(count, 1, "Expected store.count() to be 1 but was \(count)")
     }
+    
+    func testItAllowsRetrievingAPostByRow() {
+        let testMessage = "test message"
+        store.create(["message": testMessage])
+        
+        let post = store.find(0)
+        XCTAssertEqual(post.message, testMessage)
+    }
 }{% endraw %}
```

We add a `PostStore` unit test showing the behavior we need: when `create()` is called with a certain message key, the post we later `find()` should have that same message.

Red: (“nil”) is not equal to (“Optional(“test message”)”)

With this unit test failure, we've reproduced the problem we're seeing at the acceptance test level.


### Save and retrieve post with data [<span class="octicon octicon-mark-github"></span>](https://github.com/learn-tdd-in/cocoa-touch/commit/fa0ff2e91abae0831999ec8216e852ca00b8c6a2)

#### Learn TDD in Cocoa Touch/PostStore.swift

```diff
{% raw %} 
     func create(postData: [String:String]) {
         let post = Post()
+        post.message = postData["message"]
         posts.append(post)
     }
     
     func find(index: Int) -> Post {
-        return Post()
+        return posts[index]
     }
     
 }{% endraw %}
```

We check for the "message" key in the dictionary passed into `create()` and save it on the `Post` that's created. Now we actually need to retrieve that same `Post` in the `find()` method, and we do so. This causes the unit test to pass, and our acceptance test is passing as well!

Inner green
Outer green

We've successfully used Test-Driven Development to drive out our feature.
