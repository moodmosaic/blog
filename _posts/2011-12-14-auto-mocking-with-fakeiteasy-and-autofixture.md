---
layout: basic
title: Auto-Mocking with FakeItEasy and AutoFixture
---

<p>AutoFixture can become an auto-mocking container using <a title="The simplest mocking library for .NET 3.5 and Silverlight with deep C# 3.0 integration." href="http://code.google.com/p/moq/" target="_blank">Moq</a>&nbsp;(described <a title="AutoFixture as an auto-mocking container." href="http://blog.ploeh.dk/2010/08/19/AutoFixtureAsAnAutomockingContainer.aspx" target="_blank">here</a>) and <a title="Rhino Mocks is a dynamic mock object framework for the .Net platform. Its purpose is to ease testing by allowing the developer to create mock implementations of custom objects and verify the interactions using unit testing." href="http://hibernatingrhinos.com/open-source/rhino-mocks" target="_blank">Rhino Mocks</a> (described <a title="Rhino Mocks-based auto-mocking with AutoFixture." href="http://blog.ploeh.dk/2010/11/13/RhinoMocksbasedAutomockingWithAutoFixture.aspx" target="_blank">here</a>).</p>
<p>In addition to the above auto-mocking features it is now possible to use <a title="A .Net dynamic fake framework for creating all types of fake objects, mocks, stubs etc. Easier semantics, all fake objects are just that - fakes - the use of the fakes determines whether they're mocks or stubs." href="http://code.google.com/p/fakeiteasy/" target="_blank">FakeItEasy</a>.</p>
<p>To install AutoFixture with Auto Mocking using FakeItEasy, run the following command in the Package Manager Console:</p>

``` text
PM> Install-Package AutoFixture.AutoFakeItEasy
```

<p>To use it, add an AutoFakeItEasyCustomization to the Fixture instance:</p>

``` csharp
var fixture = new Fixture()
    .Customize(new AutoFakeItEasyCustomization());
```

<p>Here is a typical usage inside a test method which will automatically create mocked instances using FakeItEasy:</p>

``` csharp
var fixture = new Fixture()
    .Customize(new AutoFakeItEasyCustomization());
var result = fixture.CreateAnonymous<IInterface>();
```

<p>To <em>explicitly </em>use FakeItEasy inside a test you need to <a title="AutoFixture Freeze." href="http://blog.ploeh.dk/2010/03/17/AutoFixtureFreeze.aspx" target="_blank">Freeze</a> it first:</p>

``` csharp
[Fact]
public void FixtureCanFreezeFake()
{
    // Fixture setup
    var fixture = new Fixture()
        .Customize(new AutoFakeItEasyCustomization());
    var dummy = new object();
    var fake = fixture.Freeze<Fake<IInterface>>();
    fake.CallsTo(x => x.MakeIt(dummy))
        .Returns(null);
    // Exercise system
    var result = fixture.CreateAnonymous<IInterface>();
    result.MakeIt(dummy);
    // Verify outcome
    A.CallTo(() => result.MakeIt(dummy)).MustHaveHappened();
    // Teardown
}
```

<p>The above example can be made even more elegant by using <a title="AutoData Theories with AutoFixture." href="http://blog.ploeh.dk/2010/10/08/AutoDataTheoriesWithAutoFixture.aspx" target="_blank">AutoData</a> theories:</p>

``` csharp
[Theory, AutoFakeItEasyData]
public void FixtureCanFreezeFake([Frozen]Fake<IInterface> fake, IInterface sut)
{
    var dummy = new object();
    sut.MakeIt(dummy);
    A.CallTo(() => sut.MakeIt(dummy)).MustHaveHappened();
}
```

**Update (2014/02/28)**: It is also possible to use the `[Frozen]` attribute on the `IInterface` directly, as [Mark Seemann](http://blog.ploeh.dk/) commented [here](https://github.com/AutoFixture/AutoFixture/issues/251#issuecomment-36285413).

<p>Below is the code for the AutoFakeItEasyDataAttribute class:</p>

``` csharp
public class AutoFakeItEasyDataAttribute : AutoDataAttribute
{
    public AutoFakeItEasyDataAttribute()
        : base(new Fixture().Customize(new AutoFakeItEasyCustomization()))
    {
    }
}
```

<p>An automatically published release created from the latest successful build can also be downloaded from&nbsp;<a title="AutoFixture - Downloads" href="http://autofixture.codeplex.com/releases/view/78605" target="_blank">here</a>.</p>
