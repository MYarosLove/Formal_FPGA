A. Bug Activation and Detection
Logic bugs in a processor can be modeled in two steps:
(1) a particular instruction sequence that activates the bug
and (2) a particular instruction sequence that makes the bug
observable in a program-visible state bit. In this bug model,
the bug activation criterion can be described by the set of states
that are reached by the design under verification at the end of
the bug activation sequence.
In a minimal QED bug trace, there is (at least) one failing
instruction in the sequence which propagates the error effect
into program-visible registers; the instructions before the (first)
affected instruction either contribute to the activation of the
bug or they can be omitted from the trace. By this definition,
the length of the error trace depends on how many instructions
are needed to activate the bug. However, in S2 QED, if the
solver is allowed to consider every possible initial state, it
will be able to start from a state that implicitly represents the
system after a bug activation sequence.
Hence, the error trace for every possible QED-discoverable
bug can be as short as one instruction provided that there is
no restriction on the initial state of the proof.
B. S2 QED Verification model
The verification model we present in this paper is based on
the idea that the original and EDDI-V-transformed sequence
can be executed in parallel on two independent instances of
the CPU. QED consistency in this case refers to a mapping
between the registers of the two CPU instances. There can be
an arbitrary mapping between the register names. For example,
1
} is the set of original registers (in
if O = {R01 , . . . , RN
2
} the duplicate register
instance 1) and D = {R02 , . . . , RN
set (in instance 2), and all registers behave in the same way,
one way of defining the correspondence function could be:
2
2
m(Ri1 ) = RN
−i . Note that in S QED also special registers
like the PC and the status registers can be mapped between
the two instances; however, usually every such special register
in CPU 1 must be mapped to the same register in CPU 2.
S2 QED can be enabled to also check the control flow if we
extend the check expression QED consistent registers() by
also comparing the values of the PC in the two instances after
a branch instruction.
In order to simplify the presentation of the basic idea,
let us for now consider a processor with a static (in-order)
pipeline. For example, let us consider a classical 5-stage RISC
pipeline with IF, ID, EX, MEM and WB stages. Fig. 4 shows
the property that needs to be proven in order to show the
absence of any QED-detectable bug. The property assumes
that both CPU instances fetch the same opcode at time point
tIF ; the instruction fetched in CPU 2 is the QED-duplicated
version of the instruction fetched in CPU 1, with the same
opcode but with different operands according to the register
mapping m. In each CPU instance, this “instruction under
verification” (IUV) passes through the pipeline and eventually
writes its results to the register file in the write-back stage at
time point tWB .
The S2 QED property of Fig. 4 makes no restrictions on
the initial state other than that CPU 2 executes a QED copy
of the instruction in CPU 1, and that the previous instruction
sequence produces a QED-consistent register file.
As an example, let us see how S2 QED detects the bug in
Fig. 1. When checking the S2 QED property of Fig. 4 on the
buggy pipeline the SAT solver produces one out of many
valid counterexamples. One possible error trace could, for
example, contain the first three instructions from this example
as “original sequence” on CPU 1, and their duplicated versions
on CPU 2. At time point tIF , the CPU 1 fetches the “instruction
under verification”, MOV R1, #3 from line 3 in Fig. 1,
and CPU 2 fetches the “duplicate” instruction MOV R17, #3
from line 6 (assuming the same register mapping as in the
QED example). In the counterexample, the initial state at tIF
also contains the instructions from lines 1 and 2 in the EX and
ID stages of CPU 1, and the instructions from lines 4 and 5 in
the EX and ID stages of CPU 2. The error trace shows that at
the later time point tWB −1, the instruction from line 2 writes-
back into the register file of CPU 1, and the instruction from
line 5 writes-back into the register file of CPU 2. At this time
point, both register files are still in QED consistency with each
other, as required by the assumption of the property. However,
the bug has been activated in the pipeline of CPU 1 (but not in
CPU 2), and the IUV from line 3 writes-back a corrupted value
into the register file at tWB , while the duplicate instruction
writes the correct values into its register file.
Theorem 1. The S2 QED property of Fig. 4 fails for all QED-
EDDI-V-detectable logic bugs (cf. Def. 1), for a given register
mapping m.
Proof. Assume there is a QED-EDDI-V-detectable bug in the
processor design. Then, there exists an instruction sequence 1
which starts from some QED-consistent initial state and pro-
duces a wrong result in the processor registers or memory
locations, and there also exists another instruction sequence 2
with the same opcodes which produces a different result (e.g.,
a correct result). If we compare the register sets after each
instruction of sequence 1 with the corresponding register sets
of sequence 2 according to the mapping function m, as a
result of Def. 1, we can identify one instruction (the IUV
from above) for which the registers/memory locations are still
QED-consistent before the execution of this instruction, but
not QED-consistent after the execution. This is the instruction
that propagates the error effect into the program-visible regis-
ters/memory locations. We call it the “observing instruction”
in the following.
The property of Fig. 4 fails for a processor design containing
the considered bug, because a counterexample exists that vio-
lates the property. This counterexample fetches the observing
instruction at tIF in CPU 1 and its non-observing duplicate in
CPU 2. There are no constraints on the initial state of the IPC
property other than that the instruction sequence preceding
the observing instruction does not create QED-inconsistent
register sets in CPU 1 and CPU 2 and that CPU 2 executes
the same instruction as CPU 1, however based on different
operands as given by m. The SAT solver implicitly enumerates
in CPU 1 all possible instructions of the ISA, under all possible
operand configurations and addressing modes. If a QED-
EDDI-V-discoverable bug exists, every instruction observing
the error effect of the activated bug causes the property to
fail.
Note that in S2 QED, actually only the instruction under
verification (IUV) needs to be duplicated. There is no reason
to also duplicate the opcodes of the preceding instructions.
This observation allows for a modification of the property that
reduces the proof complexity but does not impair the generality
of the proof result: One of the two CPU instances may be
restricted to start from an initial state that represents a pipeline
with a predetermined instruction sequence, for example, a
sequence of NOPs (flushed pipeline). Also, the instructions
that follow the IUV may be constrained to be NOP instructions
in that core.
Fig. 5 shows the S2 QED computational model. We unroll
the two instances of the processor for a number of time frames.
CPU 1 is constrained to start from a flushed-pipeline state Sref
and also fetches only NOPs in the time frames for t > 1.
CPU 2 is unconstrained with respect to its initial state and all
succeeding instructions. In this computational model, the SAT
solver compares the scenario 1 where the IUV executes in a
flushed-pipeline context with all possible scenarios 2 where the
IUV executes in an arbitrary context including the ones where
bugs are activated and propagated. (Should the bug occur in 
the flushed-pipeline context of CPU 1 then any other context
produces a different (correct) result in CPU 2 and the bug is
detected, also.) Constraining CPU 1 by fixing many inputs to
constants, as shown, significantly reduces proof times.
If a design passes the S2 QED property then this means that
there is no QED-EDDI-V-detectable logic bug (cf. Def. 1) in
the design, for QED tests of arbitrary length. This is a strong
statement that is very useful in post-silicon validation: It allows
to conclude that any failed QED test in post-silicon validation
is due to an electrical or other bug, not a logical design bug.
Furthermore, we can also make useful statements in pre-
silicon verification, as already mentioned earlier. Proving the
S2 QED property allows us to conclude for every instruction
in the ISA that its execution is independent of the state of the
processor when the instruction is loaded.