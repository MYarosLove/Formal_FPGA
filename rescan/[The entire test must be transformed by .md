[The entire test must be transformed by QED for this to work. If some QED checks are left out, then this cannot be guaranteed. For example, if some Normal checks and Store checks are omitted, an error caused by a bug inside the core may propagate to an uncore component.]

[Bug scenarios are in the appendix. The bug scenarios were simulated by modifying the RTL of the OpenSPARC T2 SoC design so that, for each bug scenario, if the bug activation criterion is satisfied, the bug effect is simulated.] [Partial instantiation 1 is the largest that will fit into the BMC tool; all designs also contain the crossbar that connects the components together.] 

[These Inst_min and Inst_max parameters do not affect the bug traces found by Symbolic QED shown later; they are only used to create the QED tests for detecting bugs.]



Effective Post-Silicon Validation of System-on-Chips Using Quick Error Detection