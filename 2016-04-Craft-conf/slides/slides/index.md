- title : Computation expressions in context
- description : Monoids, monads and computation expressions
- author : Andrea Magnorsky
- theme : solarized
- transition : default

***
- data-background : images/fsharp.png
- data-background-size : 900px

' change back image?
##  Computation expressions in context

### FP User Group + Craft Conf - April 2016

***
- data-background: images/roundcrisis.png
- data-background-size: 900px

## Andrea Magnorsky

#### @SilverSpoon
#### [roundcrisis.com](roundcrisis.com)

![wd](images/wd-logo.gif)

***
- data-background: images/otter-question.jpg

' introduce comp expressions

## Computation Expressions

---

    let getHtml(url:string) =  
      let req = WebRequest.Create url
      let response = req.GetResponse()
      use streatm = response.GetResponseStream()
      use reader = new StreamReader(streatm)
      
      reader.ReadToEnd().Length

---

    let getHtmlA(url:string) =  
      async{
          let req = WebRequest.Create url
          let! response = req.AsyncGetResponse() // ding!
          use streatm = response.GetResponseStream()
          use reader = new StreamReader(streatm)
          return reader.ReadToEndAsync().Length // ding!
          }

---

    let getHtmlA(url:string) =  
      //async{
          let req = WebRequest.Create url
          let (*! *) response = req.AsyncGetResponse() 
          use streatm = response.GetResponseStream()
          use reader = new StreamReader(streatm)
          (*return *) reader.ReadToEndAsync().Length 
      //    }



***

    let maybe = new MaybeBuilder()
    let addNUmbers  =
        maybe {
            let x = 12
            let! y = Some 11
            let! z = Some 30
            return x + y + z
        }

' Block of code  that represent a computation with a non standard aspect
' code inside the block is reinterpreted using a computation builder

---

' record of operations that define the semantics and syntax

![builder](images/otter-builder.jpg)

---

  type SimplestBuilder () =

      member this.Bind(x,f) = 
        printfn "Bind begin  %A %s " (DateTime.Now.TimeOfDay)  (x.ToString()) 
        let y = f x
        printfn "Bind end  %A %s " (DateTime.Now.TimeOfDay)  (x.ToString()) 
        y
      member this.Return(x) = 
          printfn "Return %A %s"  (DateTime.Now.TimeOfDay) (x.ToString())
          x

---

## async

    open System

    let sleepWorkflow  = async{
        printfn "Starting at %O" DateTime.Now.TimeOfDay
        do! Async.Sleep 2000
        printfn "Finished at %O" DateTime.Now.TimeOfDay
        }

    Async.RunSynchronously sleepWorkflow      


---

## mbrace

    let job =
        cloud {
            return sprintf
            "run on worker '%s' " Environment.MachineName }
        |> runtime.CreateProcess

---

    let makeJob i =
        cloud {
            if i % 8 = 0 then failwith "fail"
            let primes = Sieve.getPrimes 1000000
            return sprintf
            "calculated %d primes %A on machine '%s'"
                primes.Length
                primes
                Environment.MachineName
        }

    let jobs2 =  
        [ for i in 1 .. 10 ->
             makeJob i |> cluster.CreateProcess ]


[example source](https://github.com/mbraceproject/MBrace.StarterKit/blob/master/HandsOnTutorial/3-cloud-parallel-cpu-intensive.fsx)


***

' read -> maths heavy
' first person to describe monads for functional programming was Eugenio Moggi, paper called ["Notions of computation and monads"](http://www.disi.unige.'it/person/MoggiE/ftp/ic91.pdf) this paper is maths heavy tho I am sure some of you will be interested in looking at this.

### Notions of computation and monads (1991)
![Moggi](images/Eugenio_Moggi.jpg)

---

' Philip Wadler, these papers made monads in programming widely known.

### Comprehending Monads (1990)
### Monads for functional programming (1995)

![wadler](images/wadler.gif)

' purity vs impurity and how when trying to keep pure certain things become harder than they would be in imperative code
' F# is an impure language
' Pure language: it has referential transparency, so, you can replace any expression with its value without changing the meaning of the program.

---

Some ``not-useful-right-away`` info

* There is a strong link between monads and category theory
* Monads have 3 monadic laws that every monad must satisfy:
        - Left identity
        - Right identity and
        - Associativity

---

## Monads eh?

![meh](images/otter_notimpressed.jpg)

' lots of noise many monads tutorials
' f# not a pure language

***

# Monoids

'  Convert pairwise operations into work in collections
' Parallelization and Incrementalism

---

## Monoids

* Closures  $  a' \rightarrow  a' \rightarrow  a'  $ (example  int -> int -> int   )
* Identity   $ x + I  = x $
* Associativity  $ x + (y + z) = (x + y ) + z $

---

    type Colour = { r: byte; g: byte; b: byte; a: byte }

    let addTwo c1 c2 = {
        r = c1.r + c2.r
        g = c1.g + c2.g
        b = c1.b + c2.b
        a = c1.a + c2.a
    }
---

    type Monoid<'a> =
        { neutral : 'a
          op : 'a -> 'a -> 'a }

---

    let black = { r = 0uy; g = 0uy; b = 0uy; a = 0uy }

    let colourAdd : Monoid<Colour> = {
        neutral = black
        op = (addTwo)
    }

---

    let c1 = { black with g = 254uy }
    let c2 = { black with r = 254uy }

    let l = [ c1; c2; black ]
            |> List.reduce (addTwo)

---

' Oh, yes and you can property check that your type is a monoid!!

    type T = Colour

    let M = colourAdd
    let Z = M.neutral
    let (++) = M.op

    [<Property>]
    let `` Z is the neutral element`` (v : T) =
        Z ++ v = v && v ++ Z = v

    [<Property>]
    let ``The op is associative`` (a : T, b : T, c : T) =
        a ++ (b ++ c) = (a ++ b ++ c)


---

![fixer](images/otter_fixer.jpg)

' Because of closures-> We can convert pairwise operations into operations that work on collections
' Because of associativity -> We can implement divide and conquer algorithms that are great for
'                               Parallelization
'                               Incrementalism
' Because of identity -> we can actually perform certain of the above



***

- data-background : images/working.jpg

***

> A **monad** is a monoid in the category of endofunctors.

---

> A **monad** is like a burrito.

---

> Think of **monads** just like you would think about Legos.
![legos](images/legos.jpg)

---

' It all seemed pretty strange to me and I wound up understanding less and less

![scared-otter](images\otter_surprised.png)


---
' then I see this in the news!! Is there some sort of link between otters and monads?

![Terror](images/terror.png)

***

' Wadler talked about handling errors in his monad paper, MAYBE we should try to write some code and see how can we use this stuff

## Errors

    let division a b c d=
        match b with
        | 0 -> None
        | _ -> match c with
               | 0 -> None
               | _ -> match d with
                      | 0 -> None
                      | _ -> Some (((a/b)/c)/d)



---

### 1. Extract the core

     let divide a b =
        match b with
        | 0 -> None
        | _ -> Some (a/b )

---

### 2. Write the Builder

    type MaybeBuilder() =
        member __.Bind(value, func) =
            match value with
            | Some value -> func value
            | None -> None
        member __.Return value = Some value

---

### 3. Profit

    let maybe  = MaybeBuilder()

    let divisionM a b c d=
        maybe{
            let! x = divide a b
            let! y = divide x c
            let! z = divide y d
            return z
        }

' what is happening here ?
' Monads apply a function that returns a wrapped value to a wrapped value. Monads have a function >>= (pronounced “bind”) to do this.

---
' oh, oh! and you can also property check a monad (tho a little harder to read than the monoids)

    [<Test>]
    let ``monad laws``() =
        let ret (x: int) = choose.Return x
        let n = sprintf "Choice : monad %s"
        let inline (>>=) m f = choose.Bind(m,f)
        fsCheck "left identity" <|
            fun f a -> ret a >>= f = f a
        fsCheck "right identity" <|
            fun x -> x >>= ret = x
        fsCheck "associativity" <|
            fun f g v ->
                let a = (v >>= f) >>= g
                let b = v >>= (fun x -> f x >>= g)
                a = b

---

# Why learn this?
' how many of you read about DDD's ubiquitous language?

' Because understanding the abstraction and having a common language is worth it
' enables pure FP to represent non functions such as IO, state
' control side effect (via the type system)
' metaprogramming (haskell, c++ via templates)

***

## Computation Expressions

> Most monads are computation expressions, not all computation expressions are monads

---

## let, for and try .. with, but with a different semantics

***

## Monoids again


	  type Colour = { r : byte; g : byte; b : byte; a : byte }

	  let addColour c1 c2 = 
	      { r = c1.r + c2.r
	        g = c1.g + c2.g
	        b = c1.b + c2.b
	        a = c1.a + c2.a }

	  let neutral = { r = 0uy; g = 0uy; b = 0uy; a = 0uy }

---

	  type MonoidBuilder ()= 
	    member this.Zero() = neutral
	    member this.Combine(x, y) = addColour x  y    
	    member x.For(sequence, f) =
	        let combine a b = x.Combine(a, f b)
	        let Z = x.Zero()
	        Seq.fold combine Z sequence
	    member x.Yield (a) = a

---

	  let monoid = new MonoidBuilder()

	  let monoidAdd xs= monoid {
	       for x in xs do
	         yield x
	       }

***

### Computation expressions or workflows

Computation expressions have been available in F# since 2007 and they are fully documented in the [F# language specification][4]

* Abstract computations
* Handling of effects

---

## Maybe problems

* Slow
* Hard to debug
* Operator overuse

' nesting of expression

---

![otter](images/math_otter.png)


***

## Next

![rop](images/rop.png)

* [Railway Oriented Programming](http://fsharpforfunandprofit.com/rop/#monads)
* [Chessie](https://fsprojects.github.io/Chessie/)


***
- data-background: images/roundcrisis.png
- data-background-size: 900px

## Thanks

#### @SilverSpoon
#### [roundcrisis.com](roundcrisis.com)


***

### Resources

* [Abstraction, intuition, and the “monad tutorial fallacy”](https://byorgey.wordpress.com/2009/01/12/abstraction-intuition-and-the-monad-tutorial-fallacy/)
* [The "What are monads?" fallacy](http://two-wrongs.com/the-what-are-monads-fallacy)
* [Beyond Foundations of F# - Workflows](http://www.infoq.com/articles/pickering-fsharp-workflow)
* [Monads, Arrows and idioms](http://homepages.inf.ed.ac.uk/wadler/topics/monads.html)  papers here.
* [Why a monad is like a writing desk](http://www.infoq.com/presentations/Why-is-a-Monad-Like-a-Writing-Desk) Video
* [Understanding Monoids][3] F# for fun and profit on monoids
* [Understanding Monoids using F#](http://gettingsharper.de/2015/03/03/understanding-monoids-using-f/) From gettingsharp-er :)
* [Syntax Matters: Writing abstract computations in F#](http://tomasp.net/academic/papers/computation-zoo/syntax-matters.pdf) paper by Tomas Petricek and Don Syme about computation expressions
* [Monads for functional programming][2] P.Wadler paper

---

### Resources

* [F# language specification][4]
* [Try Joinads F# research extension for concurrent, parallel and asynchronous programming.](http://tryjoinads.org/index.html?computations/home.html)
* [Functors Applicatives and monads in pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
* [Monads explained from a maths point of view](https://www.youtube.com/watch?v=9fohXBj2UEI) Video
* [Comprehending Monads][1] P.Wadler paper
* [More freedom from side effects](http://www.davesquared.net/2013/11/freedom-from-side-effects-fsharp.html)


[1]:(http://ncatlab.org/nlab/files/WadlerMonads.pdf)
[2]:(http://homepages.inf.ed.ac.uk/wadler/papers/marktoberdorf/baastad.pdf)
[3]:(http://fsharpforfunandprofit.com/posts/monoids-without-tears/#series-toc)
[4]:http://fsharp.org/specs/language-spec/

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-46761189-1', 'auto');
  ga('send', 'pageview');

</script>
