Tutorials
=========

Articles
--------

.. note::

   We're working on some in-depth long-form tutorials, please keep an eye on this space.
   
Videos
------

We'll be producing short screencasts to complement the reference guides. To begin, here's a quick, broad introduction to Freya and developing an API.

* `Introduction to Freya (YouTube) <https://www.youtube.com/watch?v=TYvUovTP7qk>`_

Freya Hello World
-----------------

    open System
    open System.IO

    open Freya.Core

    let helloWorld =
        freya {
            let text = "Hello World"B
            let! stae = Freya.getState

            state.Environment.["owin.ResponseStatusCode"] <- 200
            state.Environment.["owin.RepsonseReasonPhrase"] <- "Awesome"
            state.Environment.["owin.ResponseBody"] :?> Stream
            |> fun x -> x.Write (text, 0, text.Length)

    open Microsoft.Owin.Hosting

    type Exercise() =
        member __.Configuration() =
            OwinAppFunc.ofFreya helloWorld

    [<EntryPoint>]
    let run _ =
        let url = "http://localhost:8080"
        let _ = WebApp.Start<Exercise> (url)
        Console.WriteLine("Serving site at " + url)
        Console.WriteLine("Press ENTER to cancel")
        let _ = Console.ReadLine
