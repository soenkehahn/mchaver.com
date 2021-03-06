---
title: Parser Combinators and Test Driven Parsing
---

[Source Code](https://github.com/mchaver/mchaver.com/tree/master/tutorials/projects/haskell/attoparsec/1-attoparsec-template-project)

[Attoparsec](http://hackage.haskell.org/package/attoparsec-0.13.0.2) is a Haskell parser combinator library for parsing `ByteString` and `Text`. We can use it to make small parsers and combine these small parsers to create more complex ones. The advantage of this style is that we can individually test each parser, making it easier to debug and build, as well as make the individual pieces reusable.

We are going to focus on parsing `Text` from `Data.Text`. The top of our parsing file looks like this.

```haskell
{-# LANGUAGE OverloadedStrings #-}

module Parser where

import Data.Attoparsec.Text
import Data.Text (Text)

```

We want string literals to be treated as `Text`, we want to load the attoparsec parser for `Text` and the `Text` type. Here is our first parser.

```haskell
parseHelloWorld :: Parser ()
parseHelloWorld = do
  _ <- string "Hello World!"
  return ()
```

Look at the type signature for this function: `Parser ()`. Though you cannot tell by looking at it, it takes a `Text` and returns `()`. `Parser` is a type synonym for a more complex type that takes `Text`, manages some state, and returns a parsed value of your choice. Running a parser will return `Either Text a`, where `Text` is an error message and `a` is whatever we decide to return. In this case it is `()`. `string` takes a `Text` and returns it in a `Parser`, but we are ignoring the value returned from `string` because our parser returns `()`. In a later lesson we will explore the Parser type declaration.

Now we will make a test to prove to ourselves that the parser works. We will use two packages to help us with testing. `hspec` helps automatically find files for testing and provides some basic testing functions. We can test `attoparsec` with just `hspec`, but `hspec-attoparsec` provides some convenient functions for specifically testing attoparsec.

```haskell
{-# LANGUAGE OverloadedStrings #-}

module ParserSpec (main, spec) where

import Data.Text (Text)
import Parser

import Test.Hspec
import Test.Hspec.Attoparsec

main :: IO ()
main = hspec spec

spec :: Spec
spec = do
  describe "Parser" $ do
    it "should parse the phrase 'Hello World!'" $
      parseHelloWorld `shouldSucceedOn` ("Hello World!" :: Text)
```

You can run this from the command line with `cabal test`, and you should see all tests pass. `shouldSucceedOn` runs a parser from the left hand side with the input on the right hand side and allows the test to pass if the parser returns `Right`. Do you want to see what a failed test looks like? Just change the last line to:

```haskell
      parseHelloWorld `shouldSucceedOn` ("Hello World" :: Text)
```

You will see a "not enough input" error. The `string` function could not find the last character `!`. Let's slightly change the `parseHelloWorld` function so we can show off some more functionallity.

```haskell
parseHelloWorld2 :: Parser Text
parseHelloWorld2 = do
  result <- string "Hello World!"
  return result
```

Now the parse will return the result from the `string` function. We can verify this with another test.

```haskell
    it "should parse the phrase 'Hello World!' and return 'Hello World!'" $
      ("Hello World!" :: Text) ~> parseHelloWorld2 `shouldParse` ("Hello World!" :: Text)
```

The `(~>)` function feeds a `Text` value to be parsed by `parseHelloWorld2` from the left hand side and `shouldParse` says not only should the parser succeed, but its result should match the right hand side. There are two ways this can fail. `parseHelloWorld2` could fail to parse the left hand side, or it could succeed in parsing it but it does not return the expectation on the right hand side. Try deleting something from the left or right hand side and see what happens when you run the test.

Sometimes we want to be able to prove that a parser will fail, just use `shouldFailOn`.

```haskell
    it "should fail to parse any other string" $ do
      parseHelloWorld `shouldFailOn` ("Goodnight Everyone!" :: Text)
```

For failed cases we do not need to inspect the returned result because we just get an error message, the left hand side from `Either Text a` that the parser returns. Finally, here is what a parser combinator looks like. We will split up the original function and glue it together in another one.

```haskell
parseHello :: Parser Text
parseHello = string "Hello"

parseWorld :: Parser Text
parseWorld = string " World!"

parseHelloWorld3 :: Parser Text
parseHelloWorld3 = do
  hello <- parseHello
  world <- parseWorld
  return $ T.concat [hello,world]
```

Add a new test for it.

```haskell
  describe "A Parser Combinator" $ do
    it "shoul parse 'Hello World!'" $
      ("Hello World!" :: Text) ~> parseHelloWorld3 `shouldParse` ("Hello World!" :: Text)
```

Try making your own parser combinator with various `string` functions. You can use this [Attoparsec Template Project](https://github.com/mchaver/mchaver.com/tree/master/tutorials/projects/haskell/attoparsec/1-attoparsec-template-project) to get started. In the command line move to the project and run the following:

```bash
cabal sandbox init
cabal install --enable-tests
cabal test
```

References:

[Hackage :: attoparsec](http://hackage.haskell.org/package/attoparsec-0.13.0.2/docs/Data-Attoparsec-Text.html)

[Hackage :: hspec](http://hackage.haskell.org/package/hspec-2.2.3)

[Hackage :: hspec-attoparsec](http://hackage.haskell.org/package/hspec-attoparsec-0.1.0.2)
