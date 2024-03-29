---
layout: basic
title: Auto-Mocking with Foq and AutoFixture
---

A new Auto-mocking extension have been recently added to [AutoFixture](http://github.com/autofixture/autofixture) allowing the mock instances to be created by [Foq](http://foq.codeplex.com/).

This brings the total number of Auto-mocking extensions for AutoFixture up to five:

* [AutoFixture.AutoMoq](http://nuget.org/packages/AutoFixture.AutoMoq)
* [AutoFixture.AutoRhinoMocks](http://nuget.org/packages/AutoFixture.AutoRhinoMocks)
* [AutoFixture.AutoFakeItEasy](http://nuget.org/packages/AutoFixture.AutoFakeItEasy)
* [AutoFixture.AutoNSubstitute](http://nuget.org/packages/AutoFixture.AutoNSubstitute)
* [AutoFixture.AutoFoq](http://nuget.org/packages/AutoFixture.AutoFoq)

## Install

To install AutoFixture with Auto Mocking using Foq, run the following command in the Package Manager Console:

``` text
PM> Install-Package AutoFixture.AutoFoq
```

To use it, add an `AutoFoqCustomization` to the `Fixture` instance:

``` fsharp
let fixture = Fixture().Customize(AutoFoqCustomization())
```

## Usage

In the test below the mocked instance is created automatically by Foq:

``` fsharp
[<Fact>]
let AutoMockInterfaceAndSetupExpectations() =
    // Fixture setup
    let fixture = Fixture().Customize(AutoFoqCustomization())
    let dummy = obj()
    // Exercise system
    let sut = fixture.Create<IInterface>()
    sut.MakeIt(dummy) |> ignore
    // Verify outcome
    Mock.Verify(<@ sut.MakeIt(dummy) @>, Times.Once)
    // Teardown
```

The above test can be written declaratively using [AutoData](http://blog.ploeh.dk/2010/10/08/AutoDataTheoriesWithAutoFixture.aspx) theories:

``` fsharp
[<Theory>][<AutoFoqData>]
let AutoMockInterfaceAndSetupExpectationsDeclaratively
  (sut: IInterface, dummy: obj) =
    sut.MakeIt(dummy) |> ignore
    Mock.Verify(<@ sut.MakeIt(dummy) @>, Times.Once)
```

The `[AutoFoqData]` attribute is defined as:

``` fsharp
type AutoFoqDataAttribute() =
    inherit AutoDataAttribute(
        Fixture().Customize(AutoFoqCustomization()))
```

An automatically published release created from the latest successful build can be also downloaded from [here](http://autofixture.codeplex.com/releases/).
