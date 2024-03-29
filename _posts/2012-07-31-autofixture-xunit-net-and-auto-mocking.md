---
layout: basic
title: AutoFixture, xUnit.net, and Auto Mocking
---

<p class="message">The features discussed in this post are available when using <a href="https://github.com/AutoFixture/AutoFixture">AutoFixture</a> <a href="http://blog.ploeh.dk/2010/10/08/AutoDataTheoriesWithAutoFixture.aspx">decleratively</a> with the xUnit.net <a href="http://nuget.org/packages/AutoFixture.Xunit/">extension</a>. In addition, it is required to have at least one of the extensions for Auto Mocking using <a href="http://nuget.org/packages/AutoFixture.AutoMoq">AutoMoq</a>, <a href="http://nuget.org/packages/AutoFixture.AutoRhinoMocks">AutoRhinoMocks</a>, <a href="http://nuget.org/packages/AutoFixture.AutoFakeItEasy">AutoFakeItEasy</a>, <a href="http://nuget.org/packages/AutoFixture.AutoNSubstitute">AutoNSubstitute</a>, or <a href="http://nuget.org/packages/AutoFixture.AutoFoq">AutoFoq</a>.</p>

To install AutoFixture with xUnit.net data theories, run the following command in the Package Manager Console:

``` text
PM> Install-Package AutoFixture.Xunit
```

In the test method below we would like to use xUnit.net data theories to provide:

 1. Auto-generated data specimens for `d`, `s`, `i`, `a`.
 2. Auto-generated data specimens for `_`, `s`, `i`, `a`, and inline values for `d`.
 3. Auto-generated data specimens for `_`, `_`, `i`, `a`, and inline values for `d`, `s`.

``` csharp
public void TestMethod(
    double d, string s, IInterface i, AbstractType a)
{
    Assert.True(
        d != 0    &&
        s != null &&
        i != null &&
        a != null
        );
}
```

<p class="message">That fact that argument <code>i</code> is an interface and argument <code>a</code> is an abstract class prevents us from using <code>[InlineData]</code>, <code>[AutoData]</code>, or <code>[InlineAutoData]</code> attributes.</p>

AutoFixture allows us to easily define custom DataAttribute derived types:

 * By deriving from AutoDataAttribute and passing to the base constructor a customization that enables auto mocking with Moq, Rhino Mocks, FakeItEasy, or NSubstitute.
 * By deriving from CompositeDataAttribute and passing to the base constructor an array of DataAttribute derived type instances, e.g. an instance of `InlineDataAttribute` and an instance of `AutoDataAttribute` derived type.

## Auto Mocking using Moq

To install AutoFixture with Auto Mocking using Moq, run the following command in the Package Manager Console:

``` text
PM> Install-Package AutoFixture.AutoMoq
```

We decorate the test method with AutoMoqData and InlineAutoMoqData attributes:

``` csharp
[Theory]
[AutoMoqData]
[InlineAutoMoqData(2)]
[InlineAutoMoqData(3, "foo")]
public void WithMoq(
    double d, string s, IInterface i, AbstractType a)
{
    Write(d, s, i, a);
}
```

### Output

``` text
AutoMoqData:
  d = 1
  s = s05080d05-b874-4182-bba5-18678ad9b134
  i = Castle.Proxies.IInterfaceProxy
  a = Castle.Proxies.AbstractTypeProxy

InlineAutoMoqData:
  d = 2
  s = sce87888d-8cc8-4f69-927c-c4b1347d3147
  i = Castle.Proxies.IInterfaceProxy
  a = Castle.Proxies.AbstractTypeProxy

InlineAutoMoqData:
  d = 3
  s = foo
  i = Castle.Proxies.IInterfaceProxy
  a = Castle.Proxies.AbstractTypeProxy
```

### Source code

``` csharp
internal class AutoMoqDataAttribute : AutoDataAttribute
{
    internal AutoMoqDataAttribute()
        : base(new Fixture().Customize(new AutoMoqCustomization()))
    {
    }
}

internal class InlineAutoMoqDataAttribute : CompositeDataAttribute
{
    internal InlineAutoMoqDataAttribute(params object[] values)
        : base(new DataAttribute[] {
            new InlineDataAttribute(values), new AutoMoqDataAttribute() })
    {
    }
}
```

More information on AutoFixture with Auto Mocking using Moq can be found at [AutoFixture as an auto-mocking container](http://blog.ploeh.dk/2010/08/19/AutoFixtureAsAnAutomockingContainer.aspx) blog post.

## Auto Mocking using Rhino Mocks

To install AutoFixture with Auto Mocking using Rhino Mocks, run the following command in the Package Manager Console:

``` text
PM> Install-Package AutoFixture.AutoRhinoMocks
```

We decorate the test method with AutoRhinoMockData and InlineAutoRhinoMockData attributes:

``` csharp
[Theory]
[AutoRhinoMockData]
[InlineAutoRhinoMockData(2)]
[InlineAutoRhinoMockData(3, "foo")]
public void WithRhinoMocks(
    double d, string s, IInterface i, AbstractType a)
{
    Write(d, s, i, a);
}
```

### Ouput

``` text
AutoRhinoMockData:
  d = 1
  s = s01172643-5162-474f-bcfa-319ef40a8272
  i = Castle.Proxies.IInterfaceProxy201f58a5559841bfb41895881489658c
  a = Castle.Proxies.AbstractTypeProxy5a86172bbda14cc098b4c675c5b7e555

InlineAutoRhinoMockData:
  d = 2
  s = sd3b9c112-5181-4daa-9b2f-333c7498044a
  i = Castle.Proxies.IInterfaceProxy201f58a5559841bfb41895881489658c
  a = Castle.Proxies.AbstractTypeProxy5a86172bbda14cc098b4c675c5b7e555

InlineAutoRhinoMockData:
  d = 3
  s = foo
  i = Castle.Proxies.IInterfaceProxy201f58a5559841bfb41895881489658c
  a = Castle.Proxies.AbstractTypeProxy5a86172bbda14cc098b4c675c5b7e555
```

### Source code

``` csharp
internal class AutoRhinoMockDataAttribute : AutoDataAttribute
{
    internal AutoRhinoMockDataAttribute()
        : base(new Fixture().Customize(new AutoRhinoMockCustomization()))
    {
    }
}

internal class InlineAutoRhinoMockDataAttribute : CompositeDataAttribute
{
    internal InlineAutoRhinoMockDataAttribute(params object[] values)
        : base(new DataAttribute[] {
            new InlineDataAttribute(values), new AutoRhinoMockDataAttribute() })
    {
    }
}
```

More information on AutoFixture with Auto Mocking using Rhino Mocks can be found at [Rhino Mocks-based auto-mocking with AutoFixture](http://blog.ploeh.dk/2010/11/13/RhinoMocksbasedAutomockingWithAutoFixture.aspx) blog post.

## Auto Mocking using FakeItEasy

To install AutoFixture with Auto Mocking using FakeItEasy, run the following command in the Package Manager Console:

``` text
PM> Install-Package AutoFixture.AutoFakeItEasy
```

We decorate the test method with AutoFakeItEasyData or InlineAutoFakeItEasyData attributes:

``` csharp
[Theory]
[AutoFakeItEasyData]
[InlineAutoFakeItEasyData(2)]
[InlineAutoFakeItEasyData(3, "foo")]
public void WithFakeItEasy(
    double d, string s, IInterface i, AbstractType a)
{
    Write(d, s, i, a);
}
```

### Output

``` text
AutoFakeItEasyData:
  d = 1
  s = s2b4c118b-d18b-4782-992a-d4138c65fd2a
  i = Faked IInterface
  a = Faked AbstractType

InlineAutoFakeItEasyData:
  d = 2
  s = se8da7e09-ea4f-4d16-86bd-4229802e5f5d
  i = Faked IInterface
  a = Faked AbstractType

InlineAutoFakeItEasyData:
  d = 3
  s = foo
  i = Faked IInterface
  a = Faked AbstractType
```

### Source code

``` csharp
internal class AutoFakeItEasyDataAttribute : AutoDataAttribute
{
    internal AutoFakeItEasyDataAttribute()
        : base(new Fixture().Customize(new AutoFakeItEasyCustomization()))
    {
    }
}

internal class InlineAutoFakeItEasyDataAttribute : CompositeDataAttribute
{
    internal InlineAutoFakeItEasyDataAttribute(params object[] values)
        : base(new DataAttribute[] {
            new InlineDataAttribute(values), new AutoFakeItEasyDataAttribute() })
    {
    }
}
```

More information on AutoFixture with Auto Mocking using FakeItEasy can be found at [Auto-Mocking with FakeItEasy and AutoFixture](http://nikosbaxevanis.com/2011/12/14/auto-mocking-with-fakeiteasy-and-autofixture/) blog post.

## Auto Mocking using NSubstitute

To install AutoFixture with Auto Mocking using NSubstitute, run the following command in the Package Manager Console:

``` text
PM> Install-Package AutoFixture.AutoNSubstitute
```

We decorate the test method with AutoNSubstituteData or InlineAutoNSubstituteData attributes:

``` csharp
[Theory]
[AutoNSubstituteData]
[InlineAutoNSubstituteData(2)]
[InlineAutoNSubstituteData(3, "foo")]
public void WithNSubstitute(
    double d, string s, IInterface i, AbstractType a)
{
    Write(d, s, i, a);
}
```

### Output

``` text
AutoNSubstituteData:
  d = 1
  s = sf7ce77e0-c2b3-4749-b9ec-78f3aba0254a
  i = Castle.Proxies.IInterfaceProxy
  a = Castle.Proxies.AbstractTypeProxy

InlineAutoNSubstituteData:
  d = 2
  s = sf7ba31be-3136-4b50-a51f-3763f1d06004
  i = Castle.Proxies.IInterfaceProxy
  a = Castle.Proxies.AbstractTypeProxy

InlineAutoNSubstituteData:
  d = 3
  s = foo
  i = Castle.Proxies.IInterfaceProxy
  a = Castle.Proxies.AbstractTypeProxy
```

### Source code

``` csharp
internal class AutoNSubstituteDataAttribute : AutoDataAttribute
{
    internal AutoNSubstituteDataAttribute()
        : base(new Fixture().Customize(new AutoNSubstituteCustomization()))
    {
    }
}

internal class InlineAutoNSubstituteDataAttribute : CompositeDataAttribute
{
    internal InlineAutoNSubstituteDataAttribute(params object[] values)
        : base(new DataAttribute[] {
            new InlineDataAttribute(values), new AutoNSubstituteDataAttribute() })
    {
    }
}
```

More information on AutoFixture with Auto Mocking using NSubstitute can be found at [NSubstitute Auto-mocking with AutoFixture](http://blog.ploeh.dk/2013/01/09/NSubstituteAuto-mockingwithAutoFixture/) blog post.
