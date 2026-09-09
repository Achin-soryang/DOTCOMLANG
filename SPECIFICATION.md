? Wait DOTCOMLANG V3.2.6

Final Deterministic Specification


---

1. Language Identity



DOTCOMLANG is a symbol-oriented, stack-based esoteric programming language.

Outside comments, DOTCOMLANG source may contain only:

". , ' " : ; " ~ ^ ° ∼`

The characters "~" and "∼" are distinct:

"~" = U+007E

"∼" = U+223C


Unicode normalization MUST NOT modify DOTCOMLANG source characters before lexical processing.

Whitespace separates tokens and has no executable meaning.

Outside comments, any character not belonging to the DOTCOMLANG character set or whitespace is a LEXICAL ERROR.

Comment contents are exempt from the executable-character restriction.


---

2. Lexical Structure



DOTCOMLANG is whitespace-delimited.

Every executable token and every numeric operand MUST be separated from adjacent tokens by whitespace.

No independent instruction tokens may be concatenated.

Valid:

.,.
.,.. ,...,
.,.,.
~'.,

Invalid:

.,..,...,

because ".,.." and ",...," are not separated by whitespace.

There is no implicit token boundary without whitespace.


---

3. Token Categories



The executable token stream contains:

3.1 Instruction tokens

These represent executable DOTCOMLANG operations.

3.2 Numeric operand tokens

Numeric operands occur only immediately after:

".,.." — PUSH

"∼.,." — CALL


A numeric operand is not executable.

Every whitespace-separated token in the executable token stream, including numeric operands, receives a zero-based token address.


---

4. Number Literals



4.1 Positive integers

A positive integer "N" is:

, + N dots + ,

with exactly two commas.

Examples:

,.,       = 1
,..,      = 2
,...,     = 3
,....,    = 4

A normal positive number MUST contain exactly two commas.

Therefore:

,.,.,     = NOT a number
,.,.,.,   = NOT a normal number

The token:

,.,.,

is SUBTRACT.

The token:

,.,.,.,

is ZERO.

4.2 Zero

Zero has the reserved representation:

,.,.,.,

The ZERO token takes precedence over ordinary numeric-literal recognition.

4.3 Negative literals

Negative source literals are not supported.

Negative integers may exist at runtime as results of arithmetic operations.


---

5. Integer Domain



DOTCOMLANG integers are mathematically unbounded signed integers.

A conforming implementation MUST behave as though arbitrary-precision signed integer arithmetic is used.

Finite internal representations are permitted only when they produce exactly the same language-defined mathematical results.

Arithmetic overflow is therefore not a DOTCOMLANG language condition.


---

6. Program Structure



A valid program MUST contain exactly one START:

.,.

START MUST be the first executable token.

START is a structural marker and is not a legal CALL target.

A valid program MUST contain exactly one HALT:

°.,.

Missing START, multiple START tokens, or missing HALT produces:

PROGRAM STRUCTURE ERROR

HALT is global and terminates the entire interpreter regardless of call-frame or structured-control state.


---

7. Token Addressing



Every whitespace-separated executable token receives a zero-based token address.

Address "0" is the first token.

Numeric operands also receive addresses.

Example:

Address| Token
0| ".,."
1| ".,.."
2| ",...,"
3| "∼.,."
4| ",.....,"
5| ";'.,"

The corresponding token stream is:

.,. .,.. ,..., ∼.,. ,....., ;'.,

For an operand-taking instruction:

.,.. NUMBER
∼.,. ADDRESS

the operand occupies the next token address.

Therefore, if the instruction begins at address "N", its operand is at "N + 1".


---

8. Instruction Pointer



The interpreter maintains an instruction pointer, "IP".

For an instruction without an operand:

IP := IP + 1

For an instruction with one operand:

IP := IP + 2

Control-flow instructions may replace the normal next IP with another address.

A numeric operand token is never independently executed.


---

9. Stack Model



DOTCOMLANG uses a LIFO data stack containing arbitrary-precision signed integers.

For binary operations:

[a][b]

means:

"a" = left operand

"b" = right operand


Therefore:

a - b
a / b
a % b

are evaluated in that order.

All runtime preconditions MUST be checked before the instruction modifies relevant machine state.

If a runtime error occurs, the instruction MUST NOT partially modify machine state.


---

10. START



Canonical token:

.,.

START establishes the beginning of the executable program.

Stack effect:

0 → 0

START performs no runtime computation.

START is not a legal CALL target.


---

11. PUSH



Canonical token:

.,..

Syntax:

.,.. NUMBER

PUSH reads the immediately following numeric operand and pushes its value.

Stack effect:

0 → 1

If the following token is not a valid numeric operand:

PROGRAM STRUCTURE ERROR


---

12. Arithmetic



12.1 ADD

Canonical token:

.,.,.

Pops "a" and "b" and pushes:

a + b

Stack effect:

2 → 1

Requires verification.

12.2 SUBTRACT

Canonical token:

,.,.,

Pops "a" and "b" and pushes:

a - b

Stack effect:

2 → 1

Requires verification.

12.3 MULTIPLY

Canonical token:

.,.,,

Pops "a" and "b" and pushes:

a × b

Stack effect:

2 → 1

All signed integer values are valid.

Requires verification.

12.4 DIVIDE

Canonical token:

..,..

Pops "a" and "b" and computes:

trunc(a / b)

where truncation is toward zero.

If "b = 0":

DIVISION BY ZERO

Examples:

-7 / 2   = -3
7 / -2   = -3
-7 / -2  = 3

Stack effect:

2 → 1

Requires verification.

12.5 MODULO

Canonical token:

.,..,.,

Pops "a" and "b" and computes:

a % b = a - trunc(a / b) × b

"b" MUST NOT be zero.

If "b = 0":

MODULO BY ZERO

Examples:

-7 % 2   = -1
7 % -2   = 1
-7 % -2  = -1

Stack effect:

2 → 1

Requires verification.


---

13. Comparison Operations



All comparison operations produce exactly one Boolean:

0 = false
1 = true

No other value is produced by a comparison.

13.1 EQUAL

:'".,

Pushes "1" when "a = b", otherwise "0".

Stack effect:

2 → 1

Requires verification.

13.2 GREATER

:'.,.,

Pushes "1" when "a > b", otherwise "0".

Stack effect:

2 → 1

Requires verification.

13.3 LESS

:,'.,.

Pushes "1" when "a < b", otherwise "0".

Stack effect:

2 → 1

Requires verification.


---

14. INPUT



Canonical token:

:'.,

INPUT requests an integer from the input source.

Accepted input grammar:

[+|-]?[0-9]+

Only ASCII decimal digits are accepted.

Examples:

7
+7
-7
007

Surrounding whitespace is ignored.

Malformed input does not modify the stack and causes INPUT to request another value.

If EOF occurs before a valid integer is supplied:

INVALID INPUT

The stack remains unchanged.

INPUT does not require verification.


---

15. OUTPUT



Canonical token:

;' .,

The canonical token, with no internal whitespace, is:

;'.,

";'.," is the ONLY OUTPUT token.

OUTPUT reads the top data-stack value and displays it.

OUTPUT does not remove the value.

Stack effect:

1 → 1

An empty stack causes:

STACK UNDERFLOW

OUTPUT does not require verification.

The sequence:

;'. ,

is NOT an alternative spelling of OUTPUT because it contains a whitespace boundary.


---

16. IF



Canonical token:

^.,.

IF consumes the top data-stack value.

The value MUST be exactly "0" or "1".

Any other value produces:

INVALID CONDITION

Stack effect:

1 → 0

If the condition is "1", execution enters the IF body.

If the condition is "0":

if an ELSE exists, execution jumps to that ELSE body;

otherwise execution jumps to the matching ENDIF.


An IF creates structured-control state in the current call frame.


---

17. ELSE



Canonical token:

^..,.,

ELSE belongs to the nearest unmatched IF at the same structural level.

When the true branch reaches ELSE, execution jumps to the corresponding ENDIF.

When the false branch reaches ELSE through the IF false-path jump, execution begins executing the ELSE body.

ELSE does not modify the data stack.

Stack effect:

0 → 0

An unmatched ELSE reached during execution produces:

UNMATCHED ELSE

An unmatched ELSE in source structure produces:

PROGRAM STRUCTURE ERROR


---

18. ENDIF



Canonical token:

^..,.

ENDIF closes the current active IF structure.

Stack effect:

0 → 0

An unmatched ENDIF produces:

PROGRAM STRUCTURE ERROR

An ENDIF reached without the corresponding active IF in the current call frame produces:

UNMATCHED ENDIF


---

19. WHILE START



Canonical token:

°.,..

WHILE START examines the top data-stack value without removing it.

The value MUST be exactly "0" or "1".

Any other value produces:

INVALID CONDITION

Stack effect:

1 → 1

If the condition is "1", a loop-control frame is established in the current call frame and execution enters the loop body.

If the condition is "0", execution jumps to the token immediately following the matching WHILE END.

The condition remains on the data stack.


---

20. WHILE END



Canonical token:

°..,.

WHILE END closes the current active WHILE structure.

When reached during an active loop, execution jumps to its matching WHILE START.

The loop condition remains on the data stack.

When the loop terminates, the final condition remains on the stack.

If WHILE END is reached without a matching active WHILE in the current call frame:

UNMATCHED WHILE END


---

21. Structured Control State



IF and WHILE execution uses a structured-control stack separate from:

data stack

call stack


Every call frame contains its own structured-control state.

The initial program execution context contains one main call frame.

The main call frame has:

no return address

empty structured-control state


A CALL creates a new call frame.

The new frame begins with an empty structured-control state.

Structured structures cannot cross call-frame boundaries.


---

22. CALL



Canonical token:

∼.,.

Syntax:

∼.,. ADDRESS

ADDRESS is a non-negative numeric literal.

CALL does not modify the data stack.

CALL first validates:

1. the operand is a valid numeric literal;


2. the target address exists;


3. the target is an executable instruction token;


4. the target is not a numeric operand;


5. the target is not START;


6. the target is a legal subroutine entry point.



Only after all checks succeed does CALL create a new call frame.

The new frame contains:

return address

empty structured-control state


The return address is:

CALL instruction address + 2

Execution then transfers to the target address.


---

23. Legal CALL Targets



A CALL target MUST:

1. exist;


2. be an executable instruction token;


3. not be a numeric operand;


4. not be START;


5. be a legal top-level entry point.



A legal top-level entry point is an executable instruction whose static structural depth is zero and which is not a structural-closing token.

Therefore CALL MUST NOT target:

START

numeric operands

instructions inside an IF body

instructions inside an ELSE body

instructions inside a WHILE body

ELSE

ENDIF

WHILE END


CALL MAY target:

PUSH

arithmetic instructions

comparison instructions

INPUT

OUTPUT

IF

WHILE START

CALL

RETURN

VERIFY

HALT


provided the target satisfies all other CALL rules.

An invalid target produces:

INVALID ADDRESS


---

24. Static Structural Depth



Before execution, the interpreter performs structural matching after comments are removed.

Each executable token receives a static structural depth.

Example:

IF
instruction
ENDIF

has:

IF           depth 0
instruction  depth 1
ENDIF        depth 0

A token inside the body of an IF or WHILE therefore has depth greater than zero.

CALL targets MUST have static structural depth zero.

This prevents CALL from entering the middle of an existing structured-control region.


---

25. CALL and Structured Control



A callee never inherits the caller's active IF or WHILE structures.

Example:

caller:

IF
CALL F
ENDIF

When F begins:

caller structural state = saved
callee structural state = empty

The callee may create its own IF or WHILE structures.

Those structures belong only to the callee frame.

When RETURN occurs:

1. the callee's structured-control state MUST be empty;


2. the callee frame is removed;


3. the caller's saved structural-control state is restored;


4. execution resumes at the saved return address.



A callee MUST NOT return with unmatched IF or WHILE structures.

If it does:

UNTERMINATED STRUCTURE AT RETURN


---

26. RETURN



Canonical token:

∼..,.

RETURN removes the current non-main call frame and transfers execution to its saved return address.

RETURN does not modify the data stack.

If the call stack contains no caller frame:

RETURN WITHOUT CALL

Before returning, the current frame's structured-control state MUST be empty.

Otherwise:

UNTERMINATED STRUCTURE AT RETURN


---

27. Subroutines



DOTCOMLANG has no separate function-declaration syntax.

A subroutine is a programmer-defined region of top-level executable code reached by CALL and normally terminated by RETURN.

The formal semantic object is:

CALL target + execution region + RETURN

rather than a separately declared function object.

Recursion is permitted.

The data stack is shared between caller and callee.

Each CALL receives a fresh structured-control state.


---

28. COMMENTS



The comment delimiter is:

`.,.

It consists of the three characters:

`
.
,
.

The delimiter is whitespace-delimited.

When encountered outside a comment, comment mode begins.

Everything after that delimiter is ignored until the next whitespace-delimited:

`.,.

token.

Comment contents may contain arbitrary characters, whitespace, and text.

Comment contents have no executable meaning.

A delimiter inside a comment ends the comment.

An unterminated comment produces:

PROGRAM STRUCTURE ERROR


---

29. Comment Processing Order



DOTCOMLANG processing occurs in this conceptual order:

source characters
↓
recognize comment regions
↓
remove comment regions from executable source
↓
validate executable characters
↓
split executable source by whitespace
↓
recognize executable tokens and operands
↓
perform structural validation
↓
execute

Comment contents cannot produce:

lexical errors;

stack operations;

control structures;

CALL targets;

verification instructions.


Comment processing is a lexical phase and does not create runtime machine state.


---

30. Verification



Verification is mandatory after every successful:

ADD

SUBTRACT

MULTIPLY

DIVIDE

MODULO

EQUAL

GREATER

LESS


The verification state is:

NORMAL
AWAITING_VERIFY

After a successful operation requiring verification:

NORMAL → AWAITING_VERIFY

The next executable non-comment instruction MUST be:

~'.,


---

31. VERIFY



Canonical token:

~'.,

VERIFY is valid when the machine is in AWAITING_VERIFY.

The verifier independently recomputes the pending result using a separate verification code path.

The verifier receives the saved operation and operands.

The verifier result is compared with the primary result.

If equal:

AWAITING_VERIFY → NORMAL

and execution continues.

If different:

VERIFY FAILED

and execution terminates.

The produced result remains on the normal data stack.

A verifier that always reports success without recomputation is non-conformant.

Primary and verification calculations MUST be distinct code paths.

Verification is a reliability mechanism, not cryptographic proof.


---

32. VERIFY in NORMAL State



If VERIFY executes while the machine is in NORMAL:

no stack modification occurs;

no call-stack modification occurs;

verification state remains NORMAL;

no error occurs.


VERIFY is therefore a no-op in NORMAL.


---

33. Verification and Comments



Comments do not satisfy a pending verification requirement.

If the machine is AWAITING_VERIFY, comments may occur before VERIFY.

Example:

ADD
`.,. ignored text .,.
~'.,

is valid.

The comment does not satisfy verification.

The first executable non-comment instruction following the pending operation MUST be VERIFY.

Therefore:

ADD
`.,. ignored text .,.
;' .,

produces:

MISSING VERIFY


---

34. Verification and CALL



CALL cannot satisfy a pending verification requirement.

Therefore:

ADD
CALL ...

produces:

MISSING VERIFY

The same rule applies to every executable instruction except VERIFY.

CALL cannot bypass mandatory verification.


---

35. HALT



Canonical token:

°.,.

HALT terminates the entire interpreter normally.

HALT is global.

If HALT is encountered while AWAITING_VERIFY, it does not satisfy verification.

The result is:

MISSING VERIFY

Otherwise HALT terminates execution normally even if:

call frames exist;

IF structures are active;

WHILE structures are active.


HALT does not require RETURN.


---

36. Structural Validation



Before execution, the interpreter MUST validate:

exactly one START;

exactly one HALT;

START is first;

valid IF/ELSE/ENDIF matching;

valid WHILE START/WHILE END matching;

required numeric operands;

comment termination;

legal executable characters;

numeric-literal grammar;

valid token formation.


Structural validation occurs before execution.

Malformed unreachable code therefore still produces a PROGRAM STRUCTURE ERROR.


---

37. IF Matching



IF structures are matched using nearest-unmatched structural matching.

For every IF:

its ELSE, if present, is the nearest unmatched ELSE at the same nesting level;

its ENDIF is the nearest unmatched ENDIF.


Nested IF structures are resolved from the innermost level outward.

Example:

IF
IF
ELSE
ENDIF
ELSE
ENDIF

The first ELSE belongs to the inner IF.

The second ELSE belongs to the outer IF.


---

38. WHILE Matching



WHILE START and WHILE END are matched using nearest-unmatched structural matching.

Nested loops match in standard stack order.

A WHILE END cannot close a loop belonging to another CALL frame.


---

39. Runtime Preconditions and Atomicity



An instruction checks all required conditions before modifying relevant state.

Binary arithmetic

The interpreter checks:

at least two data-stack values;

right operand is nonzero for DIVIDE/MODULO.


Only then are operands consumed.

IF

The interpreter checks:

stack is non-empty;

condition is 0 or 1.


Only then is the condition consumed.

WHILE START

The interpreter checks:

stack is non-empty;

condition is 0 or 1.


Only then is loop-control state changed.

OUTPUT

An empty stack causes STACK UNDERFLOW without modifying the stack.

CALL

The interpreter validates the operand and target before creating the call frame.


---

40. Runtime Errors



A conforming implementation recognizes at least:

LEXICAL ERROR
PROGRAM STRUCTURE ERROR
STACK UNDERFLOW
INVALID CONDITION
DIVISION BY ZERO
MODULO BY ZERO
INVALID INPUT
MISSING VERIFY
VERIFY FAILED
INVALID ADDRESS
RETURN WITHOUT CALL
UNTERMINATED STRUCTURE AT RETURN
UNMATCHED ELSE
UNMATCHED ENDIF
UNMATCHED WHILE END

Errors terminate execution.


---

41. Error Precedence



Errors are resolved in this order.

Phase 1 — Lexical validation

Invalid executable characters produce:

LEXICAL ERROR

Phase 2 — Structural validation

Invalid source structure produces:

PROGRAM STRUCTURE ERROR

before execution begins.

Phase 3 — Runtime execution

The first runtime error reached in execution order terminates execution.

Instruction preconditions are checked before mutation.

No later error is evaluated after an earlier error terminates execution.


---

42. Abstract Machine State



The interpreter maintains:

DATA STACK

CALL STACK

INSTRUCTION POINTER

VERIFICATION STATE

CURRENT CALL FRAME


Each call frame contains:

RETURN ADDRESS, except the main frame;

STRUCTURED-CONTROL STATE.


The main frame has:

no return address;

empty structured-control state.


Comment processing is completed before runtime execution.


---

43. Canonical Stack Effects



Instruction| Stack Effect
START| 0 → 0
PUSH n| 0 → 1
ADD| 2 → 1
SUBTRACT| 2 → 1
MULTIPLY| 2 → 1
DIVIDE| 2 → 1
MODULO| 2 → 1
EQUAL| 2 → 1
GREATER| 2 → 1
LESS| 2 → 1
INPUT| 0 → 1
OUTPUT| 1 → 1
IF| 1 → 0
ELSE| 0 → 0
ENDIF| 0 → 0
WHILE START| 1 → 1
WHILE END| 0 → 0
CALL| 0 → 0
RETURN| 0 → 0
VERIFY| 0 → 0
HALT| 0 → 0


---

44. Canonical Token Reference



The following table is authoritative.

Every canonical token contains no internal whitespace.

.,.       START

.,..      PUSH
.,.,.     ADD
,.,.,     SUBTRACT
.,.,,     MULTIPLY
..,..     DIVIDE
.,..,.,   MODULO

:'".,     EQUAL
:'.,.,    GREATER
:,'.,.    LESS

:'.,      INPUT
;'.,      OUTPUT

^.,.      IF
^..,.,    ELSE
^..,.     ENDIF

°.,..     WHILE START
°..,.     WHILE END

∼.,.      CALL
∼..,.     RETURN

`.,.      COMMENT

~'.,      VERIFY
^'.,      VERIFY FAILED

°.,.      HALT

,.,.,.,   ZERO

"VERIFY FAILED" is an interpreter-generated terminal condition, not a programmer instruction.

Every canonical token is written exactly as above.


---

45. Canonical Example



This program computes:

3 + 7

then verifies and outputs the result.

.,.
.,.. ,...,
.,.. ,.......,
.,.,.
~'.,
;' .,
°.,.

The canonical OUTPUT token must actually be written without internal whitespace:

.,.
.,.. ,...,
.,.. ,.......,
.,.,.
~'.,
;'.,
°.,.

Execution:

PUSH 3
PUSH 7
ADD
VERIFY
OUTPUT
HALT

Result:

10


---

46. Conditional Example



Conceptually:

PUSH n
PUSH 3
GREATER
VERIFY
IF
...
ELSE
...
ENDIF
HALT

The comparison creates a Boolean value.

VERIFY must occur before IF executes.

IF then consumes the verified Boolean.


---

47. CALL Example



Suppose the token stream is:

0  .,.
1  .,.,
2  ,...,
3  ∼.,.
4  ,.....,
5  ;'.,

The CALL instruction at address "3" uses address "4" as its operand.

Its return address is:

5

because:

CALL address = 3
operand address = 4
return address = 5

The target specified by the CALL operand must be a legal top-level executable instruction.


---

48. CALL Frame Example



A caller may have an active IF:

IF
CALL F
ENDIF

The caller's structural-control state is stored in its call frame.

When F begins:

caller structural state = saved
callee structural state = empty

F may safely execute:

IF
...
ENDIF
RETURN

The inner IF belongs only to F.

When RETURN occurs:

callee structural state = empty
caller structural state = restored

Execution resumes at the saved return address.

Structured-control state therefore cannot leak between call frames.


---

49. Determinism Guarantee



A conforming DOTCOMLANG V3.2.6 implementation MUST agree on:

lexical validity;

token boundaries;

numeric literals;

integer domain;

arithmetic results;

division rounding;

modulo;

Boolean values;

stack effects;

INPUT syntax;

EOF handling;

IF behavior;

ELSE matching;

WHILE behavior;

comment processing;

verification rules;

CALL addressing;

legal CALL targets;

CALL/RETURN structural state;

return addresses;

START behavior;

HALT behavior;

runtime error conditions.


The same valid DOTCOMLANG V3.2.6 program supplied the same valid input MUST therefore produce the same language-defined behavior on independent conforming implementations.


---

50. Language Philosophy



DOTCOMLANG intentionally remains:

symbol-oriented;

stack-based;

strongly constrained;

unusual in appearance;

mandatory-verification based;

variable-free in the core language;

capable of loops;

capable of recursion;

based on explicit control structures.


These properties are part of the language design and are not implementation errors.

The purpose of V3.2.6 is to remove interpretive ambiguity while preserving those properties.


---

51. Implementation Safety



DOTCOMLANG's syntax, obscurity, and dual-path verification do not constitute cryptographic security or mathematical proof of correctness.

Implementations used with untrusted programs should independently consider:

memory limits;

execution limits;

call-stack limits;

denial-of-service resistance;

secure input handling;

testing;

formal verification where appropriate;

cryptography where appropriate.


These implementation concerns do not alter DOTCOMLANG V3.2.6 semantics.
DOTCOMLANG V3.2.6 — Security Hardening Addendum

52. Security Model



DOTCOMLANG is designed to execute programs inside a restricted abstract machine.

The language itself provides no direct access to:

the operating system;

the filesystem;

networks;

processes;

environment variables;

hardware devices;

host memory;

host code execution.


A conforming implementation MUST NOT expose these capabilities to DOTCOMLANG programs through language-defined instructions.

Host applications MAY provide controlled input and output channels.


---

53. Program Immutability



After lexical and structural validation, the executable token stream becomes immutable.

A running DOTCOMLANG program MUST NOT modify:

executable tokens;

numeric operands;

token addresses;

structural matching information;

instruction definitions.


Self-modifying DOTCOMLANG programs are not supported.


---

54. Execution Isolation



DOTCOMLANG execution MUST occur inside an isolated interpreter state.

The program may access only:

its DATA STACK;

its CALL STACK;

its INSTRUCTION POINTER;

its VERIFICATION STATE;

its CALL-FRAME STRUCTURED-CONTROL STATE;

explicitly supplied INPUT;

the designated OUTPUT channel.


No other host state is part of the DOTCOMLANG abstract machine.


---

55. Resource Limits



Implementations SHOULD provide configurable limits for untrusted programs.

Possible limits include:

maximum source size;

maximum executable-token count;

maximum execution steps;

maximum call-stack depth;

maximum input size;

maximum output size;

maximum integer magnitude or storage size.


Reaching an implementation resource limit MUST terminate execution safely.

A recommended implementation error is:

RESOURCE LIMIT EXCEEDED

Resource limits MUST NOT silently produce incorrect arithmetic results.

When no resource limit is reached, the implementation MUST behave according to the mathematically unbounded language semantics.


---

56. Execution-Step Limit



An implementation MAY maintain an execution-step counter.

One execution step is counted whenever one executable instruction is processed.

Numeric operand tokens do not independently count as execution steps.

If the configured maximum number of execution steps is reached before normal termination:

RESOURCE LIMIT EXCEEDED

Execution terminates.

This prevents non-terminating programs from consuming unlimited host resources.


---

57. Call-Stack Protection



An implementation MAY impose a maximum CALL depth.

If a CALL would exceed the configured maximum call depth:

RESOURCE LIMIT EXCEEDED

The CALL MUST NOT create a partially initialized call frame.

Recursive programs remain valid DOTCOMLANG programs; the limit is an implementation safety mechanism.


---

58. Integer Resource Protection



DOTCOMLANG integers are mathematically unbounded.

Implementations MAY impose an internal resource limit on integer representation when executing untrusted programs.

If an integer operation would exceed the configured representation limit:

RESOURCE LIMIT EXCEEDED

The operation MUST NOT partially modify the data stack.

A conforming implementation MUST NOT report arithmetic overflow merely because its internal representation has finite capacity.


---

59. Input Isolation



INPUT receives data only from the interpreter's explicitly designated input source.

Input data is treated strictly as data.

Input characters MUST NOT be interpreted as:

DOTCOMLANG source code;

instruction tokens;

CALL addresses;

comments;

host commands.


For example, input resembling DOTCOMLANG source remains ordinary input data.


---

60. Output Isolation



OUTPUT writes only the canonical representation of the top data-stack integer to the designated output channel.

OUTPUT MUST NOT execute, interpret, or expand the produced value as:

host code;

shell commands;

filesystem paths;

network requests;

DOTCOMLANG source.


DOTCOMLANG output is data only.


---

61. No Ambient Authority



A DOTCOMLANG program receives no authority merely because it is executed.

Access to external resources MUST be explicitly provided by the host environment.

The absence of a language instruction for an external resource MUST be treated as intentional.

A conforming implementation MUST NOT introduce hidden instructions that expose host capabilities while claiming full DOTCOMLANG compatibility.


---

62. Deterministic Host Interaction



Language-defined execution MUST NOT depend on:

system time;

process identifiers;

memory addresses;

random values;

thread scheduling;

host-specific undefined behavior;

locale-dependent number formatting;

Unicode normalization performed by the host.


For identical valid source and identical valid input, a conforming implementation MUST produce identical language-defined behavior.


---

63. Canonical Source Handling



DOTCOMLANG source MUST be processed using the exact Unicode code points defined by this specification.

Unicode normalization MUST NOT be performed.

Visually similar Unicode characters MUST NOT be treated as equivalent.

In particular:

~ = U+007E

∼ = U+223C

These characters are distinct DOTCOMLANG characters.


---

64. Fail-Closed Behavior



If an implementation detects an internal invariant violation, corrupted execution state, impossible instruction-pointer state, or inconsistent structured-control state, it MUST terminate execution rather than continuing with undefined behavior.

Such a condition is an implementation failure and MUST NOT be converted into a fabricated DOTCOMLANG result.


---

65. Verification Security Boundary



DOTCOMLANG's mandatory verification system provides an independent recomputation mechanism.

Verification MAY detect implementation errors in the primary arithmetic or comparison path.

However, verification MUST NOT be considered:

cryptographic authentication;

cryptographic integrity protection;

proof of interpreter correctness;

protection against a compromised interpreter;

protection against a malicious host.


A malicious or compromised interpreter can modify both execution paths.

Therefore verification is a reliability mechanism rather than a security boundary.


---

66. Optional Program Authentication



Program authentication is outside the DOTCOMLANG core language.

An implementation or distribution system MAY provide an external signed-program format.

Such a system MAY authenticate:

program source;

program version;

metadata;

publisher identity;

integrity hashes.


Authentication MUST occur outside the DOTCOMLANG instruction set.

Cryptographic signatures do not alter DOTCOMLANG execution semantics.


---

67. No Self-Escalation



A DOTCOMLANG program MUST NOT be able to grant itself additional privileges.

Execution of CALL, RETURN, INPUT, OUTPUT, arithmetic, comparison, or control-flow instructions MUST NOT increase the program's authority over the host environment.


---

68. Security Principle



DOTCOMLANG security follows four primary principles:

1. Isolation — programs operate inside the abstract machine.


2. Determinism — execution has no hidden external state.


3. Resource containment — untrusted programs cannot consume unlimited host resources when limits are configured.


4. Fail-closed execution — invalid or unsafe execution states terminate instead of producing undefined behavior.



These security properties supplement, but do not replace, operating-system sandboxing and normal application security practices.


---

69. Security Guarantee



DOTCOMLANG V3.2.6 does not claim to be impossible to crack, reverse engineer, or modify.

Instead, a conforming secure implementation SHOULD ensure that an untrusted DOTCOMLANG program cannot obtain unintended authority over the host system.

Security therefore depends on both:

the DOTCOMLANG language specification; and

the implementation's isolation and sandboxing.


A secure implementation MUST NOT rely solely on the obscurity of DOTCOMLANG's syntax.


---

70. Security Boundary



The DOTCOMLANG abstract machine is the primary language security boundary.

Everything outside the abstract machine is controlled by the host environment.

Consequently:

DOTCOMLANG program
→ restricted interpreter
→ controlled input/output
→ host environment

A DOTCOMLANG program MUST NOT have an implicit path around the restricted interpreter to reach host resources.
"""

def validate_structure(tokens):
if not tokens or tokens[0]!=TOK_START: raise DotcomError("PROGRAM STRUCTURE ERROR","START must be first")
if tokens.count(TOK_START)!=1 or tokens.count(TOK_HALT)!=1: raise DotcomError("PROGRAM STRUCTURE ERROR","exactly one START/HALT")
i=0
while i < len(tokens):
tok=tokens[i]
if tok in (TOK_PUSH, TOK_CALL):
if i+1>=len(tokens): raise DotcomError("PROGRAM STRUCTURE ERROR","missing operand")
is_num,=is_number_token(tokens[i+1])
if not is_num: raise DotcomError("PROGRAM STRUCTURE ERROR","operand not number")
i+=2
else:
is_num,=is_number_token(tok)
if is_num: raise DotcomError("PROGRAM STRUCTURE ERROR",f"stray number {tok}")
i+=1
depth=0; depths=[]; if_to_else={}; if_to_endif={}; else_to_endif={}; while_to_end={}; if_stack=[]; while_stack=[]; else_map={}
for idx,tok in enumerate(tokens):
is_num,_=is_number_token(tok)
if is_num: depths.append(depth); continue
if tok==TOK_IF: depths.append(depth); if_stack.append(idx); depth+=1
elif tok==TOK_ELSE:
depth-=1; depths.append(depth); if_idx=if_stack.pop(); if_to_else[if_idx]=idx; else_map[idx]=if_idx; if_stack.append(idx); depth+=1
elif tok==TOK_ENDIF:
depth-=1; depths.append(depth); top=if_stack.pop()
if tokens[top]==TOK_IF: if_to_endif[top]=idx
else: else_to_endif[top]=idx; if_to_endif[else_map[top]]=idx
elif tok==TOK_WSTART: depths.append(depth); while_stack.append(idx); depth+=1
elif tok==TOK_WEND: depth-=1; depths.append(depth); start=while_stack.pop(); while_to_end[start]=idx
else: depths.append(depth)
return {"depths":depths,"if_to_else":if_to_else,"if_to_endif":if_to_endif,"while_to_end":while_to_end}

def run(source,input_values=None,limits=None):
if input_values is None: input_values=[]
if limits is None: limits={}
max_steps=limits.get("max_steps",100000); max_call=limits.get("max_call_depth",1000)
max_stack=limits.get("max_stack",10000); max_digits=limits.get("max_int_digits",10000)
tokens=lex(source); struct=validate_structure(tokens)
depths=struct["depths"]; if_to_else=struct["if_to_else"]; if_to_endif=struct["if_to_endif"]; while_to_end=struct["while_to_end"]
data_stack=[]; call_stack=[]; ctrl_stack=[]; ip=0; verify_state="NORMAL"; pending_op=None; pending_operands=None; pending_result=None; input_ptr=0; outputs=[]; steps=0
def trunc_div(a,b): return math.trunc(a/b)
def comp(op,a,b):
if op==TOK_ADD: return a+b
if op==TOK_SUB: return a-b
if op==TOK_MUL: return a*b
if op==TOK_DIV: return trunc_div(a,b)
if op==TOK_MOD: return a - trunc_div(a,b)*b
if op==TOK_EQ: return 1 if a==b else 0
if op==TOK_GT: return 1 if a>b else 0
if op==TOK_LT: return 1 if a<b else 0
def get_op(addr): ,v=is_number_token(tokens[addr]); return v
while True:
if ip>=len(tokens): raise DotcomError("PROGRAM STRUCTURE ERROR","IP OOB")
steps+=1
if steps>max_steps: raise DotcomError("RESOURCE LIMIT EXCEEDED","max steps")
tok=tokens[ip]
is_num,=is_number_token(tok)
if is_num: ip+=1; continue
if verify_state=="AWAITING_VERIFY" and tok!=TOK_VERIFY: raise DotcomError("MISSING VERIFY",f"got {tok}")
if tok==TOK_START: ip+=1
elif tok==TOK_PUSH:
data_stack.append(get_op(ip+1)); ip+=2
elif tok in VERIFY_REQUIRED:
if len(data_stack)<2: raise DotcomError("STACK UNDERFLOW")
b=data_stack[-1]; a=data_stack[-2]
if tok in (TOK_DIV, TOK_MOD) and b==0: raise DotcomError("DIVISION BY ZERO" if tok==TOK_DIV else "MODULO BY ZERO")
data_stack.pop(); data_stack.pop()
res=comp(tok,a,b); data_stack.append(res)
verify_state="AWAITING_VERIFY"; pending_op=tok; pending_operands=(a,b); pending_result=res; ip+=1
elif tok==TOK_VERIFY:
if verify_state=="AWAITING_VERIFY":
a,b=pending_operands
if comp(pending_op,a,b)!=pending_result: raise DotcomError("VERIFY FAILED")
verify_state="NORMAL"
ip+=1
elif tok==TOK_INPUT:
if input_ptr>=len(input_values): raise DotcomError("INVALID INPUT","EOF")
data_stack.append(input_values[input_ptr]); input_ptr+=1; ip+=1
elif tok==TOK_OUTPUT:
if not data_stack: raise DotcomError("STACK UNDERFLOW")
outputs.append(data_stack[-1]); ip+=1
elif tok==TOK_IF:
cond=data_stack[-1]
if cond not in (0,1): raise DotcomError("INVALID CONDITION")
data_stack.pop()
if cond==1: ctrl_stack.append({"type":"IF","start":ip,"state":"TRUE"}); ip+=1
else: ctrl_stack.append({"type":"IF","start":ip,"state":"FALSE"}); ip=if_to_else[ip]+1 if ip in if_to_else else if_to_endif[ip]+1
elif tok==TOK_ELSE:
top=ctrl_stack[-1]
if top["state"]=="TRUE": ip=if_to_endif[top["start"]]+1; ctrl_stack.pop()
else: top["state"]="TRUE"; ip+=1
elif tok==TOK_ENDIF: ctrl_stack.pop(); ip+=1
elif tok==TOK_WSTART:
cond=data_stack[-1]
if cond not in (0,1): raise DotcomError("INVALID CONDITION")
if cond==1:
if not (ctrl_stack and ctrl_stack[-1]["type"]=="WHILE" and ctrl_stack[-1]["start"]==ip): ctrl_stack.append({"type":"WHILE","start":ip})
ip+=1
else: ip=while_to_end[ip]+1
elif tok==TOK_WEND: ip=ctrl_stack[-1]["start"]
elif tok==TOK_CALL:
target=get_op(ip+1)
if target>=len(tokens) or is_number_token(tokens[target])[0] or tokens[target]==TOK_START or depths[target]!=0: raise DotcomError("INVALID ADDRESS")
if len(call_stack)>=max_call: raise DotcomError("RESOURCE LIMIT EXCEEDED","call depth")
call_stack.append({"return_addr":ip+2,"ctrl_stack":ctrl_stack}); ctrl_stack=[]; ip=target
elif tok==TOK_RET:
if not call_stack: raise DotcomError("RETURN WITHOUT CALL")
if ctrl_stack: raise DotcomError("UNTERMINATED STRUCTURE AT RETURN")
f=call_stack.pop(); ctrl_stack=f["ctrl_stack"]; ip=f["return_addr"]
elif tok==TOK_HALT: break
return outputs'''
