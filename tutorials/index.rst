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

Freya Hello World (from Introduction to Freya video)
----------------------------------------------------

.. code-block:: powershell

   Install-Package Freya
   Install-Package Microsoft.Owin.SelfHost

.. code-block:: fsharp

    module MyWebSite
    
    open System
    open System.IO
    open Freya.Core
    open Microsoft.Owin.Hosting

    let helloWorld =
        freya {
            let text = "Hello World"B
            let! state = Freya.State.get

            state.Environment.["owin.ResponseStatusCode"] <- 200
            state.Environment.["owin.RepsonseReasonPhrase"] <- "Awesome"
            state.Environment.["owin.ResponseBody"] :?> Stream
                |> fun x -> x.Write (text, 0, text.Length)
            }

    type Exercise() =
        member __.Configuration() =
            OwinAppFunc.ofFreya helloWorld

    [<EntryPoint>]
    let main _ =
        let url = "http://localhost:8080"
        WebApp.Start<Exercise> (url) |> ignore
        Console.WriteLine("Serving site at " + url)
        Console.WriteLine("Press ENTER to cancel")
        Console.ReadLine |> ignore

Simple Static File Server
-------------------------

.. code-block:: fsharp

    module SelfHostWebSite
    
    open Freya.Core
    open Microsoft.Owin.Hosting
    open System.IO
    open System
    open Freya.Lenses.Http

    let fileServer root = freya {
      let! reqPath = Freya.Lens.get Request.Path_      
      let path =
        let relPath = reqPath.Substring(Path.GetPathRoot(reqPath).Length).TrimStart() 
        match relPath with
        | ""  -> Path.Combine(root, "index.html")
        | q   -> Path.Combine(root, q)
      let (code, phrase, bytes) =
        try
          match File.Exists path with
          | true  -> (200, "OK"                     , File.ReadAllBytes(path))
          | _     -> (404, "File Not Found"         , [||])
        with _    -> (501, "Internal Server Error"  , [||] )
      do! Freya.Lens.setPartial Response.StatusCode_ code
      do! Freya.Lens.setPartial Response.ReasonPhrase_ phrase
      do! Freya.Lens.map Response.Body_ (fun s -> s.Write(bytes, 0, bytes.Length); s)
      }

    type Startup() =
      member __.Configuration() =
        OwinAppFunc.ofFreya (fileServer @"C:\MyWebSite\StaticContent\")

    [<EntryPoint>]
    let main _ =
      try
        let url = "http://localhost:8080"
        let _ = WebApp.Start<Startup> (url)
        [ 
          "Serving site at " + url
          "Press ENTER to cancel"
        ] |> List.iter Console.WriteLine
        let _ = Console.ReadLine()
        0
      with x ->
        [
          "\n"
          x.Message
          "\n"
        ] |> List.iter Console.WriteLine
        let _ = Console.ReadLine()
        1
