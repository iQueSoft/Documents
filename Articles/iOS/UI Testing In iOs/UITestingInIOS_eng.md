\*\* UI Testing in iOS\*\*

Any software product creation starts from studying requirements and creating its
architecture. After that or even in the same time the development process
starts. An important part of the development process it is testing it at all
stages of its production. Quality assurance engineers (QA) are involved into
development process from the beginning of this process. As well as developers
they are studying requirements, also they are testing specification (in case if
it exist :) ). Creating test documentation and testing product throughout all
development process it’s also QA engineers tasks. To reduce the load on the QA
department, to detect new bugs which can appears in earlier checked parts of
code, to solve integration problems and the most important to improve the
quality of the final product the Continuous Integration (CI) is used.

The CI was added to Xcode starting from version 5. Like all other products
provided by Apple it is not hard to set up CI. It works out of the box.

The important part of CI is writing and support unit-tests. When you created a
new project in Xcode 5/6 the tests directory and one test file were
automatically created for you. But Xcode 5/6 was not allow us to test UI.

UI testing in iOS app development was still only QA department tasks, or you had
to use third-party solutions. Framework
[KIF](<https://github.com/kif-framework/KIF>) (KIF) is one of such solutions.

To have access to UI elements such as UIButton, UItextField, UIlabel using KIF
you should setup accessibility label in storyboard or .xib for each element you
needed to test.

To create mock object it is necessary add new target to project. This target
should be a copy of target you want to test, but instead of real objects it uses
mock ones.

The example of UI testing with KIF framework you can find
[here](<https://github.com/iQueSoft/iOSDemo_KIFSample>), (please see UI Tests
target) 

When you create a new project in Xcode 7 it optionally allow you to include unit
tests and UI tests into your new project. Into existing project you can add UI
Tests as follows: (File \> New \> Target \> Test \> Cocoa Touch UI Testing
Bundle). Inside UI Tests files as well as inside XCTests ones each method should
start from the word “test”.

To create new UI test you can follow the next steps: create empty method which
starts from the word “test”. Put cursor inside the test method and press record
button.

![](<PicStartTest.png>)

Now you can made test steps on simulator. And when you tap on any UI element
Xcode automatically creates new line of code in test method.

![](<picProcessTest.png>)

To stop test writing you should tap record button again.

![](<PicStopTest.png>)

To have test fully completed you should add assertions (XCTAsserts) in it.

If you want to create a new test method based on exist one, or to change
existing test you can set breakpoint inside this test method and run it. When
the test execution will be paused, you can tap the record button to continue
test creation.

To use mock object for testing you need to create an XCUIApplication object,
fill its launchArguments array  property by one or more strings. After that you
should send “launch” message to this object. For example,

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    XCUIApplication *app = [[XCUIApplication alloc] init];

    app.launchArguments = @[@"StringToUseMockObjects"];

    [app launch];
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Lines of code above you can add in the beginning of each test method, or in
setUp method, which is called before each test method execution in the whole
test file.

And then you can use the next if statement in all places where you need to
replace real object by mock one:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    if([[NSProcessInfo processInfo].arguments containsObject:@"StringToUseMockObjects"]) {

        // Use your mock object here

    }
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This if statement should be used in the target which is being tested, not in
UITests one. The example of using UITests you can find
[here](<https://github.com/iQueSoft/iOSDemo_UITests>). This example is the same
as above, except the UI Testing feature was used instead of KIF framework.

For my opinion it is simpler and more reliable to use built-in UI testing
feature then using third party solutions. Apple also give us ability of tests
debugging. This UI Testing feature was lacked for full CI provided by Apple.
With new Xcode 7 they have filled this gap.

For now the main disadvantage of UI testing in Xcode is that you can use it only
for iOS 9. But most of apps in App Store support iOS 7 and higher. However, the
number of devices with iOS 9 installed is constantly increasing. According to
the data [provided by Apple](<https://developer.apple.com/support/app-store/>)
on November 30, 2015 about 70% of devices have iOS 9 installed.

![](<iOS_usage_statistics.png>)

So ability of UI Testing in iOS 9 only will not be a big disadvantage soon :).
