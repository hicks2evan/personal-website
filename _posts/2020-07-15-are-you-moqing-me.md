---
title: "Are you moqing me?"
subtitle: "C# unit testing with moq"
published: false
image_url: "featured/2020-07-15.png"
---
When I was looking for my first job in software I interviewed with a company who had me build a simple web application as a take-home section of the interview. I did what I had learned to do in my classes and in my personal projects- fork their interview repo, run a scaffolding generator ([Yeoman](https://yeoman.io/) for Angular, which I would still recommend) and code, code, code!

![Hand-drawn Yeoman logo](/assets/images/posts/2020-07-15/1.png)

The interviewers reviewed my project with me on the phone. They liked how I had things organized and how I had reused my Angular components throughout the site- but then they had a question. What’s in the test directory? How does this test code work? Can you tell us the difference between a unit test and an integration test?

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
    private shoes;
    private nose;
    private mouth;
    private hat;

    public MrPotatoHead(Shoe[] shoes, Nose nose, Mouth mouth, Hat hat) {
        this.shoes = shoes;
        this.nose = nose;
        this.mouth = mouth;
        this.hat = hat;

        Console.WriteLine("I'm a new potato!");
    }
}
{% endhighlight csharp %}
