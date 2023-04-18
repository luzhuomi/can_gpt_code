# can_gpt_code
Code might not even compile, just to test see whether can GPT code.

# What is it?
Here is mine attempt to pretend to be a dumb and lazy developer who wants GPT-4 to write code for him.

# Setting up
I decided to rust because
1. it probably has far less "resources" for GPT to train from.
2. it is one of the most "restrictive" language, which requires good understanding of software design principles, compiler design, the limitation of type system and memory model.

I defined the data structure (an AST of regex, `Ext` stands for external syntax tree), gave it to GPT and asked it to use the `combine` library to write a parser for regex. 

```rust
pub enum Ext {
    Empty,
    GrpNonMarking(Rc<Ext>),
    Grp(Rc<Ext>),
    Or(Vec<Ext>),
    Concat(Vec<Ext>),
    Opt(Rc<Ext>, bool),
    Plus(Rc<Ext>, bool),
    Star(Rc<Ext>, bool),
    Bound(Rc<Ext>, u64, Option<u64>, bool), // inner regex, lower bound, uppper bound, greedy flag
    Carat,
    Dollar,
    Dot,
    Any(HashSet<char>),
    NoneOf(HashSet<char>),
    Escape(char), // escaped character
    Char(char) // non-escaped character
}
```

# The result... Not promising! 

It generates a bunch of codes, which do not compile. With 31 compilation errors initially.
I copied and pasted back the compilation error back to the chat window and asked. (Sometimes, I pretend to be less lazy to fix some trivial error, e.g. GPT suggests some function which was defined in the wrong module. I googled and figure it out and fixed it.)

## Things I noticed

1. GPT tends to get stuck in a simple task that we human think that is straight forward. In particular, from chats number 15 to 36, (screenshots are found in the `chat_history` sub folder), it was struggling to fix a simple parser that parse natural number. 

From time to time it seems going around the same error. I also need to purposely pick the *right* error so that it should stay focus.

2. It seems that GPT does not understand the overall algorithm anymore after a long conversation.
After it managed to get pass the `natural` function error, it faced (is still facing) the another bigger problem. Due to some of the changes it made, it rewrote the function sigature of `postfix` from `postfix<Input>() -> impl Parser<Input, Output = Ext>` to `postfix<Input>(a: Ext) -> impl Parser<Input, Output = Ext>`. 

This is an algorithm design issue. The former expects that what `postfix()` parsed should be independent from what appears before, the latter expects the otherwise. From the rest of the code it generated I noticed semantics disconnection, e.g. parsers `atom()` and `base()` are not called anywwhere. The changes made to `postfix()` impact its callers and callers of callers, i.e.  `concat()` and `or()`. 

At this stage, GPT simply kicks the ball back to my court and ask me to choose. I am quite pesmisstic that GPT will ever manage to get this right. 

# Summary
Is that it? Probably I will continue to play with it.

## How does it perform as a productivity tool? 
Do I really need GPT for this task? Hell no. I manage to code up mine without AI in 3 days, despite that I just picked rust last month. My working (manual) version can be found in `rs-pderiv` repo. While this AI version is gonna take quite a while. 

## Does it helps me to learn to be a better programmer? 
It does suggest some API usages that I've probably never read up before hence I went to read up. 
But it generates (non-functional) solutions too fast, not good for beginners. 
It assumes that the users have the understanding of the algorithm, the compiler's behavior to guide it.
Perhaps that's what AI fans' are saying what the "prompt engineering" will be.  


