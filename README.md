# SAT-MapIt: A SAT-based Modulo Scheduling Mapper for Coarse Grain Reconfigurable Architectures


### Abstract:

Coarse-Grain Reconfigurable Arrays (CGRAs) are emerging low-power architectures aimed at accelerating compute-intensive application loops.
The acceleration that a CGRA can ultimately provide, however, heavily depends on the quality of the mapping, i.e. on how effectively the loop is compiled onto the given platform. State of the Art compilation techniques achieve mapping through modulo scheduling, a strategy which  attempts to minimize the II (Iteration Interval) needed to execute a loop, and they do so usually through well known graph algorithms, such as Max-Clique Enumeration.


We address the mapping problem through a SAT formulation, instead, and thus explore the solution space more effectively than current SoA tools.
To formulate the SAT problem, we introduce an ad-hoc schedule called the Kernel Mobility Schedule (KMS), which we use in conjunction with  the data-flow graph and the architectural information of the CGRA in order to create a set of boolean statements that describe all constraints to be obeyed by the mapping for a given II. We then let  the SAT solver efficiently navigate this complex space. As in other SoA techniques, the process is iterative: if a valid mapping does not exist for the given II, the II is increased and a new KMS and set of constraints is generated and solved.

Our experimental results show that SAT-MapIt obtains better results compared to SoA alternatives in 47.72% of the benchmarks explored: sometimes finding a lower II, and others even finding a valid mapping when none could previously be found.


## Usage:
To use our tool two things needs to be done:
1) Include our custom pass to LLVM
2) Set up the python script responsible of the mapping 

### Step 1
1) Compile LLVM by following the guide at: https://llvm.org/docs/GettingStarted.html. Additionally, add the following flag when compiling: `-DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt;polly"`
2) Copy the `DFG` folder to `/path/to/llvm-project/llvm/lib/Transforms`
3) Add the line: `add_subdirectory(DFG)` to the `CMakeLists.txt` file in the `Transforms` directory
4) Compile again LLVM

Notice that those are the steps needed to compile and add a custom pass to LLVM.

To use the pass do:
```
clang -isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX12.3.sdk -O3 -fno-unroll-loops -fno-vectorize -fno-slp-vectorize -S -c -emit-llvm my_code.c
opt -load /path/to/llvm-project/build/lib/LLVMDFG.dylib -enable-new-pm=0  -printDFG < my_code.ll > /dev/null
```

If the instruction selection phase is not commented in the LLVM pass code then you could get some errors at this point.

On Ubuntu the flow is similar (No need to use `-isysroot` and you don't have the *.dylib* extension, but *.so*)


### Step 2
SAT-MapIt usage.

1) Install python dependency: `python3 -m install z3-solver`


2) After using the LLVM pass, change the content of two variables in the main.py file:
	- The variable `path` needs to be updated with the path that contains all the files generated by the LLVM pass
	- The variable `benchmark` needs to be updated with the name of the benchmark

	For example, file generated by the LLVM are in folder:
	```
	/home/my/path/to/files/:
		-sqrt321_nodes
		-sqrt321_edges
		-sqrt321_constants
		-sqrt321_livein_edges
		-sqrt321_liveinnodes
		-sqrt321_liveout_edges
		-sqrt321_liveoutnodes
		-sqrt321_loop_graph.dot
	```
	
	Then:
	`path="/home/my/path/to/files/"`
	`benchmark="sqrt321"`
	
	To change the CGRA dimensions update:
	`n_cols = 4`
	`n_rows = 4`
	The number of internal registers of PEs can be changed by updating the variable `n_regs = 20`, but failure of the register allocation phase are not implemented yet.
	To change the topology of the CGRA the method `isNeighbor` in `mapper.py` needs to be updated.

3) Execute the script and wait for the output: `python3 main.py`



The output of the script includes many debug informations. The tool is still not reliable enough to be used as a black box, so a manual check is always needed for now. Note that there are many TODOs to implement, so the assembly generated is always checked to fix small mistakes, like branch destinations

