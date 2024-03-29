---
layout: basic
title: Mountebank Mocks with F#
---

This post describes how to use Mountebank imposters as Mock Objects/Test Spies via HTTP with F#.

<p class="message">As a learning exercise, for mountebank, but also for F#, I wrote a few F# functions to communicate with Mountebank imposters via HTTP and a few passing tests. The rest of the post describes what I learned so far.</p>

## What is Mountebank?

Mountebank is a tool (and currently the only one) which provides multi-protocol, multi-language, on-demand, [Test Doubles](http://xunitpatterns.com/Test%20Double.html) over the wire, named *imposters*.

Imposters are normally created during the [fixture setup](http://xunitpatterns.com/fixture%20setup.html) phase and disposed in the [fixture teardown](http://xunitpatterns.com/fixture%20teardown.html) phase.

According to the [official website](http://www.mbtest.org/), Mountebank currently supports imposters for:

* http
* https
* tcp
* smtp

## Smtp example

Imposters can act as Mocks, as well as Stubs. For Mocks, the Mountebank website contains a [mocking example](http://www.mbtest.org/docs/api/mocks) for the STMP protocol.

### Scenario: SMTP client transmits a mail message; verify the subject of the message.

The Mountebank server runs at 192.168.1.3 on port 2525 in a Unix-like OS. During the setup phase an imposter is created via HTTP POST specifying the SMTP protocol and port (4547) in the request body:

``` text
POST http://192.168.1.3:2525/imposters/ HTTP/1.1
Host: 192.168.1.3:2525
Accept: application/json
Content-Type: application/json

{
  "port": 4547,
  "protocol": "smtp"
}
```

In F# this can be written using the [FSharp.Data](http://fsharp.github.io/FSharp.Data/index.html) [Http](http://fsharp.github.io/FSharp.Data/library/Http.html) module as:

```fsharp
let create protocol host port =
    Http.Request(
        "http://" + host + ":2525/imposters/",
        headers =
            ["Content-Type", HttpContentTypes.Json;
             "Accept"      , HttpContentTypes.Json],
        httpMethod = "POST",
        body = TextRequest
                   (sprintf @"{ ""port"": %i, ""protocol"": ""%s"" }"
                       port protocol))
```

The response is:

``` text
HTTP/1.1 201 Created
Location: http://192.168.1.3:2525/imposters/4547
Content-Type: application/json; charset=utf-8
Content-Length: 167
Date: Mon, 21 Apr 2014 20:42:57 GMT
Connection: keep-alive

{
  "protocol": "smtp",
  "port": 4547,
  "requests": [],
  "stubs": [],
  "_links": {
    "self": {
      "href": "http://192.168.1.3:2525/imposters/4547"
    }
  }
}
```

Now we can send a mail message via SMTP to the Mountebank imposter using port 4547:

``` text
From: "Customer Service" <code@nikosbaxevanis.com>
To: "Customer" <nikos.baxevanis@gmail.com>
Subject: Thank you for your order

Hello Customer,
Thank you for your order from company.com.  Your order will
be shipped shortly.

Your friendly customer service department.
```

In F# this can be written as:

```fsharp
let expectedSubject = "Thank you for your order"
(new SmtpClient("192.168.1.3", 4547)).Send(
    new MailMessage(
        "code@nikosbaxevanis.com",
        "nikos.baxevanis@gmail.com",
        expectedSubject,
        "Hello Customer, Thank you for your order from company.com."))
```

To get the captured requests from the imposter we can issue a `GET` or a `DELETE` request. Normally, this happens during the fixture teardown phase via `DELETE`:

``` text
DELETE http://192.168.1.3:2525/imposters/4547 HTTP/1.1
Content-Type: application/json
Host: 192.168.1.3:2525
```

<p class="message">According to <a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.7">Hypertext Transfer Protocol -- HTTP/1.1</a> for HTTP method definitions, it is OK to issue a DELETE to also get the captured requests from the imposter: <i>"A successful response SHOULD be 200 (OK) if the response includes an entity describing the status" -- Hypertext Transfer Protocol -- HTTP/1.1, 9.7 DELETE</i>.</p>

The response is:

``` text
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 810
Date: Mon, 21 Apr 2014 20:41:10 GMT
Connection: keep-alive

{
"protocol": "smtp",
"port": 4547,
"requests": [
  {
    "requestFrom": "192.168.1.4",
    "envelopeFrom": "code@nikosbaxevanis.com",
    "envelopeTo": [
      "nikos.baxevanis@gmail.com"
    ],
    "from": {
      "address": "code@nikosbaxevanis.com",
      "name": ""
    },
    "to": [
      {
        "address": "nikos.baxevanis@gmail.com",
        "name": ""
      }
    ],
    "cc": [],
    "bcc": [],
    "subject": "Thank you for your order!",
    "priority": "normal",
    "references": [],
    "inReplyTo": [],
    "text": "Hello Customer, Thank you for your order from company.com.\n",
    "html": "",
    "attachments": []
  }
],
"stubs": [],
"_links": {
  "self": {
    "href": "http://192.168.1.3:2525/imposters/4547"
  }
}
```

In F# the `requests` JSON property can be decomposed and extracted using the FSharp.Data [JSON Parser](http://fsharp.github.io/FSharp.Data/library/JsonValue.html) and Http modules as:

```fsharp
let getCapturedRequests (spy : HttpResponse) =
    let getRequests jsonValue =
        match jsonValue with
        | FSharp.Data.Record properties ->
            match properties with
            | [| protocol; port; requests; stubs; _links; |] ->
                match snd requests with
                | FSharp.Data.Array elements ->
                    match elements |> Seq.toList with
                    | head :: tail -> Some(elements)
                    | [] -> None
                | _ -> None
            | _ -> None
        | _ -> None
    let response =
        Http.Request(
            spy.Headers.Item "Location",
            headers = [
                "Content-Type", HttpContentTypes.Json;
                "Accept", HttpContentTypes.Json ],
            httpMethod = "DELETE")
    match response.Body with
    | Text json ->
        JsonValue.Parse json
        |> getRequests
    | _ -> None
```

The signature of `GetCapturedRequests` function is:

```fsharp
spy : HttpResponse -> JsonValue [] option
```

The value of the `subject` property can be similarly decomposed and extracted with Pattern Matching:

```fsharp
match GetCapturedRequest imposter "subject" with
| Some actual -> expectedSubject = actual
| None ->
    raise <| InvalidOperationException(
        "No property named 'subject' found in captured requests.")
```

With all the above a test using [xUnit.net](https://github.com/xunit/xunit) and [composed assertions with Unquote](http://blog.ploeh.dk/2014/03/21/composed-assertions-with-unquote/) could be written as:

```fsharp
let verify = Swensen.Unquote.Assertions.test

[<Fact>]
let sendMailTransmitsCorrectSubject () =
    let expectedSubject = "Thank you for your order!"
    let mountebankHost = "192.168.1.3"
    let imposterPort = 4547
    let spy = Imposter.Create "smtp" mountebankHost imposterPort

    (new SmtpClient(mountebankHost, imposterPort)).Send(
        new MailMessage(
            "code@nikosbaxevanis.com",
            "nikos.baxevanis@gmail.com",
            expectedSubject,
            "Hello Customer, Thank you for your order from company.com."))

    verify <@
            match SmtpSpy.GetCapturedRequest spy "subject" with
            | Some actual -> expectedSubject = actual
            | None        ->
                raise <| InvalidOperationException(
                    "No property named 'subject' found in captured requests.") @>
```

<p class="message">The Mountebank website uses the <a href="http://xunitpatterns.com/Mock%20Object.html">Mock Object</a> terminology when verifying <a href="http://xunitpatterns.com/indirect%20output.html">indirect outputs</a>. However, the examples shown here don't setup expectations - instead they only verify <i>captured</i>-indirect outputs of the SUT; thus the <a href="http://xunitpatterns.com/Test%20Spy.html">Test Spy</a> terminology is used in code.</p>

### Scenario: SMTP client transmits correct number of requests.

In this case, it's only necessary to verify that the SMTP request on the imposter was made only once:

```fsharp
[<Fact>]
let sendMailTransmitsCorrectNumberOfSmtpRequests () =
    let expectedNumberOfRequests = 1
    let mountebankHost = "192.168.1.3"
    let imposterPort = 4546
    let spy = Imposter.Create "smtp" mountebankHost imposterPort

    (new SmtpClient(mountebankHost, imposterPort)).Send(
        new MailMessage(
            "code@nikosbaxevanis.com",
            "nikos.baxevanis@gmail.com",
            "Thank you for your order!",
            "Hello Customer, Thank you for your order from company.com."))

    verify <@
            match Imposter.GetCapturedRequests spy with
            | Some actual -> expectedNumberOfRequests = (actual |> Seq.length)
            | None ->
                raise <| InvalidOperationException(
                    sprintf "Expected %i calls but received none."
                        expectedNumberOfRequests) @>
```

The complete source code is available on this [gist](https://gist.github.com/moodmosaic/11169180) - any comments or suggestions are always welcome.

You may also read the next post, [Mountebank stubs with F#](http://nikosbaxevanis.com/blog/2014/05/02/mountebank-stubs-with-f-number).
