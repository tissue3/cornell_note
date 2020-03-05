#### Abstract:

HLS is hindered by monolithic toolchains that combine essential passes such as data path generation with optimization passes. => I don't follow the drawbacks of HLS

#### Intro:

line 42-49 => What does it mean by reuse? I am confused since HLS actually uses LLVM, which "enables large scale reuse in software compilers", while HLS is not reusable? 

line 50-52 => An IR should capture constraints of various target and specify domain specific optimization. Why is it related to reusing?

line 85-100: This paragraph seems to state what is binding and scheduling. But what is the argument we are trying to make in this section? There is a limitation of this kind of design? If so, the limitation is not mentioned in Futil design principles I feel.

It seems figure 1 is still not mentioned or discussed?

#### Related Work:

LLVM: Same opinion as Rachit. Also, consider merging the first two paragraphs. All we need to illustrate is LLVM is not capable of embedding hardware structure like unrolling a loop.

HPVM: The way HPVM is introduced is a little confusing, which makes it seems like a same level IR as Futil rather than frontend. It might be better to mention the abstraction level of the IR.

#### Language:

line 296-297: The enable keyword can be used to “enable" various components in the structure. => Maybe explain what exactly "enable" means more.

It would also be better if the **Control Sub-language** paragraph has some sort of opening/summary for what is control flow and why it is needed. ( e.g move "makes it easier to convert frontend languages to Futil" here rather than at the end)

line 320-329: Why is it necessary to have *seq* in *while* and *if* control logic? I guess we can also have *par* or any control logic. We can replace (seq (enable A)...(enable b)) with comments to avoid confusion.

line 351-353: "Futil allows for the process of generating the global schedule to be broken up into several modular passes." This is not intuitive. How about give an example?

line 441: I don't follow how memory banking is represented in Futil. If this is going to be part of optimization pass, an example before and after optimization would be nice.

line 452-457: Operator chaining, to my understanding, requires data to be independent. I feel one cannot randomly chain a sequence of enables.

 



