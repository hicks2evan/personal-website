---
title: "Are you Moqing me?"
subtitle: "C# unit testing with moq"
published: true
image_url: "featured/2020-07-15.png"
code_sample: true
---
When I was looking for my first job in software I interviewed with a company who had me build a simple web application as a take-home section of the interview. I did what I had learned to do in my classes and in my personal projects- fork their interview repo, run a scaffolding generator ([Yeoman](https://yeoman.io/) for Angular, which I would still recommend) and code, code, code!

![Hand-drawn Yeoman logo](/assets/images/posts/2020-07-15/1.png)

The interviewers reviewed my project with me on the phone. They liked how I had things organized and how I had reused my Angular components throughout the site- but then they had a question. **What’s in the test directory? How does this test code work? Can you tell us the difference between a unit test and an integration test?**

Like many students, I had been given a few privileged opportunities to be exposed to software testing:
1. I had taken a course called “Software Engineering I” at my university which covered testing as a topic along with UML, non-technical requirements, and the like.
2. I had probably read a Medium article espousing TDD as the *only way* to write quality software.
3. I had an internship at an organization that *had* unit tests, even if my intern project *did not*.


> *“Well, a unit test is designed to test just a section of the code, when the integration test is responsible for testing how things wire together, right?”*


> *“Ok. For the sections of the code that aren’t being tested in a unit test, how do you simulate them? How do you make sure the unit of code you are testing has what it needs to do its job?”*


My test directory didn’t have anything but the default yeoman-generated [Mocha](https://mochajs.org/) tests. I hadn’t run npm test in my development. And not too surprisingly, I didn’t end up getting an offer.

# So, what were they asking about?

If you’ve been working in software for some time, or you are a student who has prepared for this type of interview better than I had, you have probably seen an illustration like this:

![Puzzle pieces with disguises](/assets/images/posts/2020-07-15/2.png)

But that puzzle piece illustration is boring. I actually like to think of it more as a [Mr. Potato head doll](https://www.amazon.com/Mr-Potato-Head-Story-Classic/dp/B003C1MW4Q). Your unit tests are all about running the code that can be represented as just one piece of Mr. Potato head. Like his mustache, or his plump potato body.

![Disassembled Mr. Potato Head](/assets/images/posts/2020-07-15/3.png)

In object oriented land, that one Mr. Potato piece is a *class*. An instance of the class or an object is constructed in your code at runtime with everything that it needs.

{% highlight csharp %}
public class MrPotatoHead {
    private IHat hat;
    private IArm[] arms;

    public MrPotatoHead(IHat hat, IArm leftArm, IArm rightArm) {
        this.hat = hat;
        this.arms = new IArm[] {leftArm, rightArm};

        Console.WriteLine("I'm a new potato!");

        hat.Tip();
    }

    public void Wave() {
        arms[1].Wave();
    }
}
{% endhighlight csharp %}

In this case, Mr. Potato Head’s body needs a left arm, right arm, and a hat. It also could need things like eyes, a nose, ears, a mustache, and a pair of shoes. In UML-speak:

![Mr. Potato Head UML Class Diagram](/assets/images/posts/2020-07-15/4.png)

The point they were trying to get at in my interview was that in a unit test you should only be *instantiating* the code you are testing, and *mocking* the code that you aren’t testing. 

A mock simulates the functionality of the code your unit test depends on, but doesn’t implement it. Kind of like sticking a shoelace in the hole where Mr. Potato Head’s left arm goes. It isn’t a left arm, it’s just able to connect to the potato body *like a left arm*.

![Mr. Potato Head other objects inserted](/assets/images/posts/2020-07-15/5.png)

So how is this handled in code? Most unit tests use some kind of mocking framework. In C# you can use moq to mock classes based off of what methods are laid out in their interfaces. Just like inserting a shoelace in Mr. Potato Head’s arm hole, if LeftArm implements the IArm interface, then I can mock IArm and pass it as a parameter to the MrPotatoHead class to test.

{% highlight csharp %}
[TestClass]
public class MrPotatoHeadTest
{
    private readonly MrPotatoHead mrPotatoHead;
    private readonly Mock<IArm> leftArm = new Mock<IArm>();
    private readonly Mock<IArm> rightArm = new Mock<IArm>();
    private readonly Mock<IHat> hat = new Mock<IHat>();

    public MrPotatoHeadTest() {
        mrPotatoHead = new MrPotatoHead(
            hat.Object, 
            leftArm.Object, 
            rightArm.Object
        );
    }

    [TestMethod]
    public void MrPotatoHeadWaves_RightArmShouldWave()
    {
        mrPotatoHead.Wave();

        leftArm.Verify(l => l.Wave(), Times.Never());
        rightArm.Verify(r => r.Wave(), Times.Once());
    }
}
{% endhighlight csharp %}

# Why is this useful?

Next question is how this is going to provide value to us in our testing. The first easy answer is that it lets the test focus on just one section of the code. Let’s say you introduced a bug that is causing the wrong value to be returned to the user from the database. There are a lot of classes in-between, how do you tell where the issue is?

![Mr. Potato Head browsing website, error loading potato](/assets/images/posts/2020-07-15/6.png)

Speaking of that database, maybe we don’t want our test code (which we run regularly to get feedback on the small incremental code we have written) to have to integrate with external systems. Mocking allows us to set up scenarios with our interfaces as a point of delineation.

{% highlight csharp %}
[TestMethod]
public void CreatePotato_Success_ReturnsId()
{
    //Setup mock provider
    int returnId = 12345;
    dbProvider.Setup(p => p.CreatePotato(It.IsAny<IPotato>())).Returns(returnId);

    //Call real controller code
    int result = potatoController.CreateNewPotato("Lieutenant J.D. Potato");

    //Check what happened
    Assert.AreEqual(returnId, result);
    dbProvider.Verify(p => p.CreatePotato(It.IsAny<IPotato>()), Times.Once);
}
{% endhighlight csharp %}

In this code we mock a database provider, and any scenario we want for some state of data that could be returned from the database can be configured through this mock. No test data setup or teardown.

Finally, mocking doesn’t just let you specify how your simulated dependencies should behave. It also lets you check in on how the code you are testing is behaving with those dependencies. Let’s say your code also has to write files to some persistent file store. 

![Website to profile json file to potato profile file store](/assets/images/posts/2020-07-15/7.png)

How can you check in your test if the contents of the file your code writes are correct? You could just instantiate your file writer, but then you are testing two pieces of functionality at once, not good!

In Moq you can also verify that methods are called on your mock with parameters that meet certain criteria, as well as how many times they are called.

{% highlight csharp %}
[TestMethod]
public void WritePotatoProfile_Success_ReturnsId()
{
    //Setup mock provider
    int returnId = 12345;
    filestoreProvider.Setup(p => p.WritePotatoFile(It.IsAny<string>())).Returns(returnId);

    //Call real controller code
    int result = potatoController.WriteProfile(
        "Darth Potato", 
        new string[] {"the force", "fast food"}, 
        "I am a sith lord potato lookin' for love."
    );

    //Check what happened
    Assert.AreEqual(returnId, result);
    filestoreProvider.Verify(p => p.WritePotatoFile(It.Is<string>(s => 
        s.Contains("the force") && 
        s.Contains("Darth Potato"))), Times.Once);
}
{% endhighlight csharp %}

So if you are expecting your code to write 5 files containing potato dating bios, you can precisely verify that it is.

# What could go wrong?
## 1. Your unit tests only test the happy path
![Mr. Potato Head fallen apart](/assets/images/posts/2020-07-15/8.png)

In general your tests should probably test the happy path, an edge case scenario, and an exception case at least. Writing tests for your exception scenarios will help you better understand how your code responds to exceptions and verify that it behaves correctly. In Moq, you can mock exceptions to simulate the expected exceptions in your system.

{% highlight csharp %}
[TestMethod]
public void CreatePotato_Failure_ReturnsNegative()
{
    //Setup mock provider with exception
    dbProvider.Setup(p => p.CreatePotato(It.IsAny<IPotato>())).Throws(
        new System.Exception("database timed out"));

    //Call real controller code
    int result = potatoController.CreateNewPotato("Lieutenant J.D. Potato");

    //Check what happened
    Assert.AreEqual(-1, result);
}
{% endhighlight csharp %}

You can even assert on the contents of the exception, this may be a bit extreme but if somebody changed an exception message, it would be caught by the test!

{% highlight csharp %}
[TestMethod]
public void WritePotatoProfile_Failure_ThrowsException()
{
    string expectedException = "Something bad happened when trying to write the file.";

    //Setup mock provider
    filestoreProvider.Setup(p => p.WritePotatoFile(It.IsAny<string>())).Throws(
        new System.Exception("file contained bad data"));

    //Call real controller code
    var exception = Assert.ThrowsException<System.Exception>(() => potatoController.
    WriteProfile(
        "Darth Potato", 
        new string[] {"the force", "fast food"}, 
        "I am a sith lord potato lookin' for love."
    ));

    //Check what happened
    Assert.AreEqual(expectedException, exception.Message);
}
{% endhighlight csharp %}

## 2. Your tests don’t assert anything meaningful

Your unit tests should provide value and increase confidence in the quality of your software. This is more of an issue of assertions than mocking, but your tests should assert something meaningful about the contents of what you are testing.

{% highlight csharp %}
[TestMethod]
public void MeaninglessTest()
{
    //Setup mock provider
    int potatoId = 12345;
    Potato returnPotato = new Potato();
    dbProvider.Setup(p => p.GetPotato(potatoId)).Returns(returnPotato);

    //Call real controller code
    IPotato result = potatoController.GetPotato(potatoId);

    //No assertions
    //What happened? Who knows? The test passes.
}
{% endhighlight csharp %}

These assertions do little more than the compiler is doing on its own, and will pass even if the code doesn’t work as designed. Not good. A test is only as good as its assertions.

## 3. Your test code is more *code* than test
![Mr. Potato Heads side by side stolen arm](/assets/images/posts/2020-07-15/9.png)

Your test code doesn’t need super smart implementation like your code does. This sounds obvious, but it is a pretty easy mistake. The most heinous example is copying implementation code to help you calculate what you expect for your test. Imagine you had a silly little method to calculate some number.

{% highlight csharp %}
public double CalculatePotatoNumber(int input) {
    double someSillyNumber = 12345 * (Math.Floor(input + input * (0.9)));
    return someSillyNumber;
}
{% endhighlight csharp %}

Then you make a test helper method to help you calculate the expected result.

{% highlight csharp %}
[TestMethod]
public void CalculatePotatoNumber_100()
{
    double expectedPotatoNumber = GetExpectedPotatoNumber(100);
    double result = mrPotatoHead.CalculatePotatoNumber(100);
    Assert.AreEqual(expectedPotatoNumber, result);
}

private double GetExpectedPotatoNumber(int input) {
    double someSillyNumber = 12345 * (Math.Floor(input + input * (0.9)));
    return someSillyNumber;
}
{% endhighlight csharp %}

Really, if the PotatoNumber formula was typed in incorrectly, it won’t have been caught. It seems like we did something smart and made something reusable, but that isn’t always the goal in tests. The purpose would have been better served by just hard coding a few expected outcomes.

When I first came out of school, I felt like code was only “good” if it did something. Hard-coded values were “bad”. In this case it is actually much better to hard-code what you expect and validate against that. If you have a bug and your test is dumb, it will catch it instead of replicating it.

## 4. Your dependencies are really hard to mock

When does mocking code break down? Usually when the code you are trying to test has a lot of responsibility and calls its dependencies in a lot of different ways. Or maybe your code depends on a class that is really hard to mock… like some kind of in house static class for bitmap formatting. Gnarly, dude! In my experience, this is usually more of a code organization issue. Common appearances in the wild:

* *I need to mock a gazillion methods in my utility class for any class that depends on it to work!*- Read up on SOLID design principles, specifically [single-responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle).
* *I can’t mock a private method?*- Why is the method that your code needs private? Should there be another point of entry for your test code?
* *I can’t mock a static method? Oh no!*- Read [this stack exchange](https://softwareengineering.stackexchange.com/questions/148049/how-to-deal-with-static-utility-classes-when-designing-for-testability?rq=1), think about whether or not your static method really needs to be mocked. You may just want to let your static dependency do its job.

In object-oriented development, if your code is easily mocked and tested, you will also find that it becomes more organized, composable, and self-contained.

# You don’t need to be a TDD guru to write meaningful unit tests
![Mr. Potato Head mountain guru](/assets/images/posts/2020-07-15/10.png)

Your code base doesn’t need to have 100% code coverage, even if it does it probably won’t have 100% scenario coverage. Those are just statistics for your management to put in a slide deck. The biggest takeaway I had from learning how to use Moq for my C# code was simple:

> *Writing code with your mocks and tests in mind helps you write cleaner code.*

> *Covering your code base with meaningful unit tests will give you increased confidence to continue to make changes as your code base grows.*

> *Breath in. Breath out.*