Machine language is the sequence of bits that directly controls a processor, causing it to add, compare, move data from one place to another, and so forth at appropriate times. 

Aseembly languages were invented to allow operations to be expressed with mnemonic abbreviations. Translating from mnemonics to machine language became the job of a systems program known as *assembler*. Programming then is a machine-centered enterprise: each different kind of computer had to be programmed in its own assembly language, and programmers thought in terms of the instructions that the machine would actually execute.

High-level programming languages came into stage in the mid-1950s. Translating from a high-level language to aseembly or machine language is the job of a systems program known as *compiler*. Compilers are substantially more complicated than assemblers because the one-to-one correspondence between source and target operations no longer exists when the source is a high-level language.

Why are there so many high-level programming languages?
- evolution: computer science is a young discipline, we are constantly finding better ways to do things
- special purposes: some languages were designed for a specific problem domain
- personal preference: different people like different things, much of the parochialism of programming is simply a matter of taste

What makes a language successful?
- expressive power
- easy of use for the novice
- easy of implementation
- standardization: standardization of both the language and a broad set of libraries is the only truly effective way to ensure the portability of code across platforms
- open source
- excellent compilers
- economics, patronage, and inertia

# The Programming Language Spectrum
The *declarative* languages focus on *what* the computer is to do, and the *imperative* languages focus on *how* the computer should do it.

*Functional* languages employ a computational model based on the recursive definition of functions. They take their inspiration from the *lambda calculus*, a formal computational model developed in the 1930s.

*Dataflow* languages model computation as the flow of information among primitive functional *nodes*. They provide an inherently parallel model: nodes are triggered by the arrival of input tokens, and can operate concurrently.