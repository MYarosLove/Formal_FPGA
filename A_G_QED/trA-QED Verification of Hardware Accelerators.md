Abstract—We present A-QED (Accelerator-Quick Error Detection), a new approach for pre-silicon formal verification of stand-alone hardware accelerators. A-QED relies on bounded model checking -- however, it does not require extensive design-specific properties or a full formal design specification. While A- QED is effective for both RTL and high-level synthesis (HLS) design flows, it integrates seamlessly with HLS flows. Our A- QED results on several hardware accelerator designs demonstrate its practicality and effectiveness: 1. A-QED detected all bugs detected by conventional verification flow. 2. A-QED detected bugs that escaped conventional verification flow. 3. A-QED improved verification productivity dramatically, by 30X, in one of our case studies (1 person-day using A-QED vs. 30 person-days using conventional verification flow). 4. A-QED produced short counterexamples for easy debug (37X shorter on average vs. conventional verification flow).

I. INTRODUCTION
Pre-silicon verification is used to detect logic design flaws (logic bugs) before integrated circuits (ICs) are manufactured. Several industrial reports highlight significant challenges associated with existing pre-silicon verification methodologies (e.g., [Foster 15]). This paper is about pre-silicon verification of stand-alone hardware accelerators. Unlike general-purpose processors, hardware accelerators implement a (set of) specific function(s) (e.g., encryption, 3D Rendering, or Deep Neural Network inference [Cong 17, Zhou 18, Hao 19]) and are widely used for building energy-efficient (heterogeneous) System-on-Chips (SoCs) [Cong 12, Cota 15]. While many publications target processor verification (too many to enumerate), very few address accelerator verification. Hardware accelerator verification remains highly challenging because: 

1. Unlike processors with a detailed specification (the ISA or the Instruction Set Architecture), hardware accelerators often lack precise descriptions of their functionality and interfaces; 
2. SoCs integrate a wide variety of functions and there can be many design variants (employing a rich diversity of design techniques) even for the same hardware accelerator function (e.g., various energy and execution time targets). Each design variant must be verified thoroughly and quickly; and, 
3. Accelerator verification lacks decades of rich experience unlike processor verification. 

We present A-QED (Accelerator-Quick Error Detection), a new formal technique for pre-silicon verification of stand-alone hardware accelerators. A-QED is inspired by Symbolic QED [Lin 15, Singh 18] (which targets designs containing processor cores) and other self-consistency techniques for processors (e.g., [Jones 96]). A-QED relies on Bounded Model Checking (BMC) [Clarke 01] to symbolically analyze input sequences of increasing lengths for self-consistency, i.e., whether an operation with the same inputs always results in the same outputs. A-QED targets stand-alone verification of hardware accelerators (i.e., A-QED does not require the accelerator to be integrated inside a larger SoC). In addition to design reuse, a stand-alone technique has several benefits, including better scalability and better bug visibility (i.e., bugs that may be difficult or impossible to reach in a specific SoC may be triggered by a short trace during stand-alone analysis).
A-QED is readily applicable for an important and commonly- used class of accelerators known as Loosely-Coupled Accelerators (LCAs) [Cong 12, 17, Zhou 18, Hao 19]. Unlike tightly-coupled accelerators (e.g., those directly integrated within a processor pipeline), LCAs are generally placed on the SoC’s on-chip interconnection network, outside of the processor core(s). This separation provides several advantages: 1. LCAs can substantially improve energy and execution time by offloading complete tasks [Cong 12]; 2. LCAs can directly access memory with high bandwidth [Cota 15]; and, 3. LCAs can be reused more easily across different SoCs (making independent verification of stand-alone LCA designs crucial). Our A-QED approach in this paper targets stand-alone LCAs that perform non-interfering operations, i.e., operations which always generate the same result for a given input (not to be confused with combinational circuits). Many LCAs belong to this category (formal model in Sec. III).
An orthogonal trend in hardware accelerator design is the use of High-Level Synthesis (HLS) for design productivity. In HLS, the design is described in a high-level language (e.g., C/C++ or domain- specific language), and translated to RTL (e.g., Verilog). While A-QED can be used for both HLS and RTL designs, A-QED leverages HLS automation to considerably reduce A-QED setup time.
We applied A-QED to multiple accelerator designs: a memory-controller unit design for a CGRA (coarse-grained reconfigurable architecture) as well as HLS designs. The memory-controller unit design study allowed an apples-to-apples comparison of A-QED vs. conventional verification flow. Our study shows: 1. A-QED detected all bugs detected by the conventional verification flow. A-QED detected more bugs that escaped the conventional flow. 2. A-QED enabled a 30-fold improvement in verification productivity (1 person-day using A-QED vs. 30 person-days using the conventional flow), stemming from multiple aspects of A-QED: (a) A-QED does not require extensive design- specific properties or assertions or a full functional specification (that are often generated manually and are error- prone); (b) progress in BMC tools; (c) (optional) A-QED-HLS integration. 3. A-QED generated short counterexamples for easy debug: 37- fold shorter on average (6 cycles on average using A-QED vs. 224 using the conventional verification flow).
In addition, we introduce the formal basis for A-QED that enables a crisp understanding of its effectiveness as well as a thorough characterization of logic bugs detected by A-QED.
The rest of this paper is organized as follows. Sec. II provides an overview of the accelerator model targeted by A-QED in this paper. Sec. III presents a formal model of such accelerators. Sec. IV details how A-QED leverages HLS. Results are presented in Sec. V, followed by related work in Sec. VI. Sec. VII concludes this paper.
II. ACCELERATOR MODEL TARGETED IN THIS PAPER
Various accelerator models exist, based on various design characteristics: e.g., programmable vs. fixed-function architectures, asynchronous vs. synchronous communication with the host (e.g., processor core(s)), LCA vs. tightly-coupled [Patel 08, Cascaval 10, Cota 15]. While our general A-QED approach may be adapted to any accelerator, we focus on a specific model in this paper (formally defined in Sec. III). First, we informally explain the characteristics of accelerators that fit our model: 
a) The accelerator is an LCA. (Sec. I, Fig. 1). Since an LCA is connected to the SoC interconnect, it can directly access system components such as memories.
b) A handshake protocol is used to communicate between the LCA and the host (e.g., processor core). This protocol must define when the inputs to/outputs from the accelerator are valid, and also when the accelerator and the host are each ready to receive inputs. 
c) The LCA execution is non-interfering; i.e., the result produced by the accelerator for a given input is independent of any other inputs received (earlier or later). LCAs should not be confused with combinational circuits – they are complex sequential circuits. A. Motivating example We present a bug scenario to motivate A-QED. Fig. 2 shows an LCA design where four buffers forward inputs to execution units (f(x)), each of which takes several cycles to compute a result. Due to a bug, the clock_enable signal is disconnected from Buffer 4, which causes the design to produce incorrect outputs.

A. Motivating example
We present a bug scenario to motivate A-QED. Fig. 2 shows an LCA design where four buffers forward inputs to execution units (f(x)), each of which takes several cycles to compute a result. Due to a bug, the clock_enable signal is disconnected from Buffer 4, which causes the design to produce incorrect outputs.


IV. A-QED SETUP
A-QED uses BMC to detect bugs. BMC takes as inputs a model (the design) and a set of properties, and symbolically analyzes input sequences to search for counterexamples to the properties. For BMC, we need to know how to apply legal (symbolic) inputs to the accelerator, how to analyze the accelerator outputs, and what to check to evaluate a property. We focus on FC and RB here. As noted in Sec. III, SAC is not our focus. This section details A-QED setup when leveraging HLS. While HLS makes A-QED easy to use, it is not required. A-QED can be applied for RTL designs as well, but that requires additional time and effort.
A. Setup with High-Level Synthesis
A high-level description of an accelerator (e.g., in C++) defines its operation within a function prototype. The variables of the prototype, provided as values or references (i.e., pointers), define the arguments passed to a call of the function. Hence, the result of each operation executed by the accelerator function depends only on these arguments (assuming no global variables). The arguments not only contain inputs but also (references to) variables where the result is stored. From the function definition, the inputs and outputs of the accelerator can be identified as: a) input variables, which become symbolic inputs for BMC; and b) results of the function call (values returned, plus updated variables passed as references) which are used for property checking. Constraints on the valid range of input values for the accelerator can be directly obtained from the valid range of each high-level input.
B. FC Checking
To apply A-QED to an accelerator, we generate the corresponding A-QED module. For the LCA model (Sec. II), the A-QED module interfaces with the accelerator during BMC – this enables stand- alone accelerator verification. For high-level designs, A-QED leverages HLS to not only synthesize the A-QED module (for BMC), but also to connect various signals between the accelerator and the A-QED module. 
The A-QED module is used during pre- silicon verification only – it is not included in the final design. The A-QED module (Fig. 4) contains two functions. The first, aqed_in, monitors input sequences to the accelerator. It labels a certain input as I\p&q or the “original.” At some later point in the input sequence, BMC issues the same original I\p&q to the accelerator again and A-QED labels it as “duplicate” or Ir]s . The exact positions I\p&q and Ir]s in the input sequence are controlled by the BMC tool. The second function, aqed_out, analyzes the outputs produced by the accelerator (to check for FC). 
An accelerator (especially an LCA) might receive multiple inputs in a batch (that may be processed by the accelerator in parallel). As long as the execution is noninterfering (Sec. II), such an accelerator is still a valid instance of the model in Sec. III. A-QED can then analyze single- (batch size = 1) or multiple-input (batch size > 1) batches. I\p&q and Ir]s can belong to the same or different (single- or multiple input-) batches. In Fig. 4, bmc_mem is a global memory for accelerator inputs and outputs (the BMC places symbolic values in this memory). A detailed explanation of the pseudo-code in Fig. 4 is available in [RESULTS 20].

	#PARAMETER MAX_BATCH_SIZE
	#PARAMETER MAX_BATCH_COUNT
	#PARAMETER IN_SIZE
	#PARAMETER OUT_SIZE
	\\ Data type definition result {dup_done; fc_check; orig_labeled; orig_done};
	\\ Pseudo-code assumes accelerator function with 2 inputs
	\\ and 2 outputs, i.e., IN_SIZE = OUT_SIZE = 2
	\\ Initialization of global state variables
		orig_val[IN_SIZE] ¬ 0; 
		orig_out[OUT_SIZE] ¬ 0; 
		ORIG_BATCH ¬ FF; 
		DUP_BATCH ¬ FF;
		orig_labeled ¬ 0; 
		dup_labeled ¬ 0;
		batch_ct ¬ 0; 
		out_batch_ct ¬ 0; 
		ORIG_IDX ¬ 0; 
		DUP_IDX ¬ 0;
		orig_done ¬ 0; 
		mem_ptr ¬ 0;
		dup_done ¬ 0; 
		fc_check ¬ 0;

	\\ Global Memory
	bmc_mem[MAX_BATCH_SIZE*IN_SIZE*MAX_BATCH_COUNT]
	\\ Pseudo-code assumes accelerator writes output result
	\\ at the corresponding input location (in bmc_mem)
	
	aqed_in (mem2acc, size, is_orig, is_dup, orig_idx, dup_idx) {
		label_orig ¬ (is_orig) && (orig_idx < size) && !orig_labeled;
		label_dup ¬ (is_dup) && (dup_idx < size) && !dup_labeled &&
		((orig_labeled && (*(mem2acc + dup_idx*IN_SIZE) == orig_val[0]) &&
		(*(mem2acc + 1 + dup_idx*IN_SIZE) == orig_val[1])) || (label_orig &&
		(*(mem2acc + orig_idx*IN_SIZE) == *(mem2acc + dup_idx*IN_SIZE)
		&& *(mem2acc + 1 + orig_idx*IN_SIZE) == *(mem2acc + 1 +
		dup_idx*IN_SIZE))));

	if (label_orig) {
		orig_labeled ¬ 1; orig_val[0] ¬ *(mem2acc + orig_idx*IN_SIZE); 
		orig_val[1] ¬ *(mem2acc + 1 + orig_idx*IN_SIZE);
		ORIG_BATCH ¬ batch_ct; ORIG_IDX ¬ orig_idx;
		}

	if (label_dup) {
		dup_labeled ¬1; DUP_BATCH ¬ batch_ct; DUP_IDX ¬ dup_idx;
		}
		batch_ct ¬ batch_ct + 1; 
	}

	aqed_out (acc2mem) {
		orig_done ¬ orig_labeled && (out_batch_ct >= ORIG_BATCH);
		if (orig_done && (out_batch_ct == ORIG_BATCH) && !dup_done) {
			orig_out[0] ¬ *( acc2mem + ORIG_IDX*OUT_SIZE);
			orig_out[1] ¬ *( acc2mem + 1 + ORIG_IDX*OUT_SIZE);
		}
		if (orig_labeled && dup_labeled && (out_batch_ct == DUP_BATCH) && !dup_done) {
			dup_done ¬ 1;
			dup_0 ¬ *(acc2mem + DUP_IDX*OUT_SIZE);
			dup_1 ¬ *(acc2mem + 1 + DUP_IDX*OUT_SIZE);
		fc_check ¬ ((orig_out[0] == dup_0) && (orig_out[1] == dup_1)); 
	}
	if (out_batch_ct > DUP_BATCH) {
		dup_done ¬ 1; 
	}
	out_batch_ct ¬ out_batch_ct + 1;
	return (dup_done, fc_check, orig_labeled, orig_done); 
	}
	aqed_top(batch_size, is_orig, orig_index, is_dup, dup_index) {
	result output;
	current_batch ¬ &bmc_mem[MAX_BATCH_SIZE*IN_SIZE*mem_ptr];
	aqed_in(current_batch, batch_size, is_orig, is_dup, orig_index, dup_index);
	acc(current_batch, batch_size);
	output ¬ aqed_out(current_batch);
	mem_ptr ¬ mem_ptr + 1;
	return output; 
	}

Fig. 4. Pseudo code for A-QED functions targeting FC. Actual implementations will vary depending on the accelerator and the HLS tool used, see [RESULTS 20].

To check for FC, the BMC tool searches for a counterexample to the following property: dup_done → fc_check In Fig. 4, dup_done is true if outputs for both I\p&q and Ir]s have been generated, and fc_check is true if both these outputs match. Some accelerator designs may require further A-QED module customization. For instance, an AES implementation (in Sec. V.B) uses a common key across an input batch. Details of such customization can be found in [RESULTS 20].

C. RB Checking
Checking for RB involves monitoring signals related to the ready-valid protocol, which can be synthesized together with the A-QED module using HLS: input-ready (rdin) and host-ready (rdh), and the related sequences of captured inputs ( C&4 (sR , in) ) and outputs (C\]5 (sR , in)) for an input sequence in received in some state sR . A counterexample to RB is a counterexample to either part (1) or (2) of Def. 3 (Sec. III). Checking for part (1) is simple: check that signal rdin does not stay low indefinitely. A counterexample to part (2) states that, after the accelerator has received k valid inputs starting from the initial state (|C&4 (s&4&5 , in)| = k), it fails to produce the expected k valid outputs regardless of how many times the host is ready (rdh is high) to accept outputs. To check for part (2), the BMC tool searches for counterexamples to the following property:

(cnt_rdh ≥ τ) ⋀ (cnt_in ≥ in_min) → rdy_out

Parameters τ and in_min are design-specific constants and cnt_rdh and cnt_in are auxiliary signals to monitor the host-ready signal and captured inputs. The value of cnt_rdh is the number of cycles the host has been ready (rdh is high) to accept an output since it sent a certain input I. The value of cnt_in is the number of inputs captured by the accelerator since it captured input I. Parameter τ is the expected maximum number of cycles the accelerator takes to produce the output for a given input. It is a concrete implementation of parameter n in our formal model. In practice, some accelerators require more than one input to be provided before producing any outputs. This is handled by setting parameter in_min (this low-level detail was omitted from the formal model for simplicity, but it can easily be added). Finally, property rdy_out holds if the output for input I has been generated (and if further design-specific conditions hold, if any). We check whether rdy_out holds in the cycle where the preconditions hold, i.e., the host allowed the accelerator a sufficient number of cycles to produce the output for input I (cnt_rdh ≥ τ) and it provided the accelerator with a sufficient number of captured inputs ( cnt_in ≥ in_min ). If rdy_out does not hold given these preconditions, then the accelerator is unresponsive with bound τ.

V. RESULTS
We demonstrate the effectiveness of A-QED for various designs (details in [RESULTS 20]). We did not artificially inject bugs. All A-QED results were generated using Cadence JasperGold version 2016.09p002 on an Intel Xeon E5-2640 v3 with 128GB of DRAM.
A. Memory-Controller Unit Case Study
We present a case study for a memory-controller unit (~17,000 flip-flops, ~97,600 gates) targeting CGRA-based accelerators. We had unique access to a tracked repository of various versions (SystemVerilog RTL) of this design (and bugs detected for each version using conventional verification). This allows an apples-to- apples comparison of A-QED vs. conventional verification flow.
The memory-controller unit supports several configurations (e.g., double buffer, line buffer, FIFO). The conventional simulation- based flow verified each configuration separately using well-crafted test patterns and full-fledged applications (e.g., point-wise multiplication, dilated convolution).
We applied A-QED for each configuration (except three, which involved interfering operations not supported by our accelerator model, as explained earlier). We created working C++ models for each configuration in consultation with the designers. For each such C++ model, we created A-QED module C++ functions (Fig. 4) and used HLS (Catapult) to generate the A-QED module RTL. For each configuration, we instantiated an RTL wrapper containing its A-QED module and the memory-controller (with its configuration bits hard-coded). For A-QED setup customization for this design, e.g., when a configuration does not provide a ready signal for the ready-valid protocol (Sec. III) or when in_min needs to be incorporated for RB checking (Sec. IV.C), please refer to [RESULTS 20].
The results of this study are presented in Table 1 and Fig. 5.

Observation 1: A-QED significantly improved bug coverage vs. conventional verification flow. With BMC as its backbone, A-QED finds the shortest sequence to trigger and detect bugs (within the BMC bound). This is in sharp contrast to conventional verification flows, where testbenches are highly dependent on the expertise of verification engineers. For the memory-controller unit, A-QED detected all logic bugs (Fig. 5) detected by the conventional verification flow for the studied configurations. A-QED uniquely detected additional (13%) bugs that were not detected by the conventional flow. These additional bugs represent difficult corner- case scenarios. For example, one such bug (triggered by a complex condition) caused a crash (after 70 cycles) during an application run (after the design was verified using conventional flow). In contrast, A-QED detected it in 1 second with a 6-cycle counterexample. A- QED detected one bug using RB and the remaining using FC. 
Observation 2: A-QED (with HLS support) dramatically improves verification productivity. As Table 1 shows, the setup effort improves (i.e., reduces) 30-fold: 1 person-day using A-QED vs. 1 person-month using conventional verification flow. 
Observation 3: A-QED detects bugs quickly (≤ 2 sec. runtime) with short counterexamples (nearly 40-fold shorter on average vs. conventional verification flow) enabling quick debug.
The Vivado HLS tool was used for these designs. Examples of bugs detected include various array indexing errors and incorrect FIFO sizing. Details on abstracted versions of the designs used in Table 2 (for BMC scalability) as well as A-QED module customization (e.g., to support a common key across a batch for AES) can be obtained from [RESULTS 20]). 
Observation 4: A-QED successfully detected bugs in various HLS accelerators. FC bugs (across different designs) were detected using the same FC universal property (Sec. III).

VI. RELATED WORK
There are numerous publications on pre-silicon verification: simulation-based approaches, formal approaches, and combinations thereof. Many of these publications focus on processor cores. The difficulties with verification of stand-alone hardware accelerators vs. processor verification are highlighted in Sec. I and are also well- explained in [Huang 18]. In contrast, A-QED enables verification of stand-alone hardware accelerators in an effective and practical way (especially in the context of HLS designs). As discussed in Sec. I, A-QED is inspired by Symbolic QED [Lin 15, Singh 18], which targets designs containing processor cores. However, there are important differences between A-QED vs. Symbolic QED: (a) A- QED targets stand-alone accelerators without any processor core; (b) A-QED leverages HLS to simplify the setup process; and (c) A- QED does not require a partitionable register file or memory for accelerators supporting the formal model in Sec. III.
A-QED is applicable to both RTL and HLS designs. For RTL, A-QED can leverage the Instruction-Level Abstraction (ILA) approach [Huang 18] to further improve verification productivity.
Unlike conventional BMC, A-QED does not require design- specific properties or a full specification (that are often created manually). While some tools try to automate formal property generation, it is often difficult to identify the “right” set. Attempts to generate design-specific properties from a specification can be problematic, since a complete specification may not be available (or the specification itself may be buggy [Singh 19]). As part of A-QED, checking for SAC may be necessary – however, it doesn’t require a complete specification and can be largely automated (e.g., techniques in [Reid 16]). Finally, A-QED can be used in conjunction with many (commercial and academic) BMC engines.

A-QED is also distinct from simulation-based pre-silicon verification approaches including those that leverage HLS (e.g., [Campbell 19, Chi 19]: A-QED uses BMC, and therefore (a) is significantly more thorough, and (b) finds short counterexamples. As is well-known, BMC-based techniques can face scalability challenges (e.g., design size, BMC bound). This aspect of A-QED is discussed in Sec. VII.

VII. CONCLUSION
A-QED is a highly effective and practical approach for pre-silicon verification of stand-alone hardware accelerators. It leverages BMC but bypasses major BMC challenges (e.g., creation of design-specific properties). A-QED is especially attractive for HLS-based accelerator designs because it largely automates the verification setup process while avoiding extensive (manual) efforts in understanding RTL designs.
A-QED creates several promising research directions: 
1) Extension of A-QED beyond the LCA model (including the handshake protocol); 
2) Verifying designs that execute interfering operations (and not just non-interfering ones); 
3) Improving the scalability of A-QED through techniques such as abstraction (e.g., [Andraus 04]), concolic execution (e.g., [Sen 05]), and symbolic starting states (e.g., [Fadiheh 18] for processor cores); 
4) More detailed case studies to demonstrate the effectiveness of A-QED; and, 
5) A-QED for post-silicon validation and debug. The use of A- QED-inspired techniques for accelerator hardware security (similar to Symbolic QED-inspired techniques for detecting security vulnerabilities in processors [Fadiheh 19]) is another interesting direction for future work. 
ACKNOWLEDGEMENT 
This work was supported in part by the DARPA POSH program.