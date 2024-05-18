[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-7f7980b617ed060a017424585567c406b6ee15c891e84e1186181d67ecf80aa0.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=10944706)
# Assignment 3

**Due Date: 11:59 PM, May 27th (Monday) via Gradescope**

## Acknowledgements
This document is derived from
https://jlpteaching.github.io/comparch/modules/dino%20cpu/assignment3/ which
is originally derived from ECS 154B WQ 2023, which was previously derived from
ECS 154B WQ 2019.

## Introduction

In the last assignment, you implemented a full single cycle RISC-V CPU.
In this assignment, you will be extending this design to be a 5 stage pipeline instead of a single cycle.
You will also be implementing full forwarding for ALU instructions and hazard detection.
The simple in-order CPU design is based closely on the CPU model in Patterson and Hennessey's Computer Organization and Design.


## GitHub Codespaces / CSIF machines

The GitHub Classroom page for the class is located at [https://github.com/ECS154B-SQ24](https://github.com/ECS154B-SQ24)

The assignment 3 template repo is located at [https://github.com/ECS154B-SQ24/Assignment3](https://github.com/ECS154B-SQ24/Assignment3)

Follow the following link to access assignment 2: [https://classroom.github.com/a/jVrCgIMX](https://classroom.github.com/a/jVrCgIMX)

The above link will automatically create a repo in the GitHub Classroom page that only you have the access to.

In the event that the template repo is updated, your own repo won’t be automatically updated. You don’t need to keep track of the template repo, unless we found an error in the assignment, in which case, we will make an announcement on Canvas and provide ways to update your repo.

### Using the CSIF machines


Apptainer is installed on most CSIF machines. So, if you are using one of the
CSIF machines either locally or remotely, things should just work. However, if
you run into any problems, post on
[Piazza](https://piazza.com/class/let902t7oig1lr) or come to office hours.

To run the dinocpu container using apptainer, run the following command in the
dinocpu folder,

```bash
apptainer run --bind $(pwd):/home/sbt-user --workdir /home/sbt-user docker://jlpteaching/dinocpu-wq23
```

The command will pull the jlpteaching/dinocpu-wq23 image from the Docker Hub
Container Image Library, and run sbt, the Scala interactive tool. The command
line should look like this,

```bash
Unable to find image 'jlpteaching/dinocpu-wq23:latest' locally
latest: Pulling from jlpteaching/dinocpu-wq23
e96e057aae67: Pull complete
bdb9413ca2c5: Pull complete
ce5c4157a592: Pull complete
ae906d0f6b61: Pull complete
899b16822c39: Pull complete
Digest: sha256:b520941b695d9d4c1e8c72cd0143c3f300655790e7ca928e59042b32a7eb90b4
Status: Downloaded newer image for jlpteaching/dinocpu-sq23:latest
downloading sbt launcher 1.8.0
copying runtime jar...
[info] [launcher] getting org.scala-sbt sbt 1.2.7  (this may take some time)...
[info] [launcher] getting Scala 2.12.7 (for sbt)...
[info] Loading settings for project sbt-user-build from plugins.sbt ...
[info] Loading project definition from /home/sbt-user/project
[info] Loading settings for project root from build.sbt ...
[info] Set current project to dinocpu (in build file:/home/sbt-user/)
sbt:dinocpu>
```

The `sbt:dinocpu>` line indicates that sbt recognizes the `dinocpu` project.

The images are relatively large files. As of the beginning of the quarter, the
image is 700 MB. We have tried to keep the size as small as possible. Thus,
especially if we update the image throughout the quarter, you may find that the
disk space on your CSIF account is full. If this happens, you can remove some
of the scala cache to free up space.

To remove the scala cache, you can run the following command in the dinocpu
folder,

```bash
rm -rf ? # yes, sbt generated a folder named "?"
```

To find out how much space the Singularity containers are using, you can you
`du` (disk usage):

```bash
du -sh ?
```

Let us know if you would like more details on this method via
[Piazza](https://piazza.com/class/luhhz7vcascf5).

## How this assignment is written

The goal of this assignment is to implement a pipelined RISC-V CPU which can execute all of the RISC-V integer instructions.
Like the previous assignment, you will be implementing this step by step starting with a simple pipeline in [Part 1](#part-i-re-implement-the-cpu-logic-and-add-pipeline-registers).
After that, you will add to your design to implement forwarding and hazard detection.

## I/O constraint

We are making one major constraint on how you are implementing your CPU.
**You may not modify the I/O for any module.**
This is the same constraint that you had in Lab 2.
We will be testing your data path, your hazard detection unit, and our forwarding unit in isolation.
Therefore, you **must keep the exact same I/O**.
You will get errors on Gradescope (and thus no credit) if you modify the I/O.

## Goals

- Learn how to implement a pipelined CPU.
- Learn what information must be stored in pipeline registers.
- Learn which combinations of instructions cause hazards, and which can be overcome with forwarding.

# Pipelined CPU design

Below is a diagram of the pipelined DINO CPU.
This diagram includes all control wires unlike the diagram in Assignment 2 in addition to all of the MUXes needed.

**Notice: Please be aware that in some of the connections we have not specified what bits of a signal must be connected to the input of a module. While working on this assignment, please make sure you specify the proper bits, wherever needed in your implementation.**

The pipelined design is based very closely on the single cycle design.
You may notice there are a few minor changes (e.g., the location of the PC MUX).
You can take your code from the Assignment 2 as a starting point, or you can use the code provided in `src/main/scala/single-cycle/cpu.scala`, which is my solution to Assignment 2.

![Pipelined CPU](_assets/pipelined_a3.svg)

## Testing your pipelined CPU

Just like with the last lab, run the following command to go through all tests once you believe you're done.

```
sbt:dinocpu> Lab3 / test
```

You can also run the individual tests for each part with `testOnly`.
The command to run the tests for each part are included in each part below.

## Debugging your pipelined CPU

When you see something like the following output when running a test:

```
- should run branch bne-False *** FAILED ***
```

This means that the test `bne-False` failed.

For this assignment, it would be a good idea to single step through each one of the failed tests.
You can find out more information on this in the [DINO CPU documentation](../documentation/single-stepping.md) and in the video [DinoCPU - Debugging your implementation](https://video.ucdavis.edu/playlist/dedicated/0_8bwr1nkj/0_kv1v647d).

You may also want to add your own `printf` statements to help you debug.
Details on how to do this were are in the [Chisel notes](../documentation/chisel-notes/printf-debugging.md).

# Part I: Re-implement the CPU logic and add pipeline registers

In this part, you will be implementing a full pipelined processor with the exception of forwarding and hazards.
After you finish this part, you should be able to correctly execute any *single instruction* application.

**This is the biggest part of this assignment, and is worth the most points.**
Unfortunately, there isn't an easy way to break this part down into smaller components.
I suggest working from left to right through the pipeline as shown in the diagram above.
We have already implemented the instruction fetch (IF) stage for you.

Each of the pipeline registers is defined as a `Bundle` at the top of the CPU definition.
For instance, the IF/ID register contains the `PC` and `instruction` as shown below.

```scala
// Everything in the register between IF and ID stages
class IFIDBundle extends Bundle {
  val instruction = UInt(32.W)
  val pc          = UInt(32.W)
}
```

We have also grouped the control into three different blocks to match the book.
These blocks are the `EXControl`, `MControl`, and `WBControl`.
We have given you the signals that are needed in the EX stage as an example of how to use these bundles.

```scala
class EXControl extends Bundle {
  val aluop             = UInt(3.W)
  val op1_src           = UInt(1.W)
  val op2_src           = UInt(2.W)
  val controltransferop = UInt(2.W)
}
```

You can also create registers for the controls, and in the template we have split these out into other `StageReg`s.
We have given you the control registers.
However, each control register simply holds a set of bundles.
You have to set the correct signals in these bundles.

Note that to access the control signals, you may need an "extra" indirection.

As the definition of StageReg is shown below, if you want to access an input to a StageReg, you need to access it via reg.io.in.<input> and similarly, if you want to access an output to a StageReg, you need to acccess it via reg.io.data.<output>. And also, for every register that is defined as a StageReg, you need to set reg.io.flush and reg.io.valid fields.

```scala
/** A [[Bundle]] which adds a flush and `valid` bit to a data bundle.

    These values are observed by a StageReg during modification of its contents to
    either flush its contents to 0 when flush is high and write if valid is high. */ 
class StageRegIO[+T <: Data](gen: T) extends Bundle { 
    /* Inputted data to the stage register */ 
    val in = Input(gen)

    /** A bit that flushes the contents of a [[StageReg]] with 0 when high. */ 
    val flush = Input(Bool())

    /** A bit that writes the contents of @in to a [[StageReg]]. */ 
    val valid = Input(Bool())

    /** The outputted data from the internal register */ 
    val data = Output(gen)
}
```


See the example below:

```scala
class EXControl extends Bundle {
  val aluop             = UInt(3.W)
  val op1_src           = UInt(1.W)
  val op2_src           = UInt(2.W)
  val controltransferop = UInt(2.W)
}

class IDEXControl extends Bundle {
  val ex_ctrl  = new EXControl
  val mem_ctrl = new MControl
  val wb_ctrl  = new WBControl
}

val id_ex_ctrl  = Module(new StageReg(new IDEXControl))

...

id_ex_ctrl.io.in.ex_ctrl.aluop := control.io.aluop
```

Specifically in `id_ex_ctrl.io.in.ex_ctrl.aluop` you have to specify `ex_ctrl.aluop` since you are are getting a signal out of the `ex_ctrl` part of the `IDEXControl` bundle.

This pipeline register/bundle isn't complete.
It's missing *a lot* of important signals, which you'll need to add.

Again, I suggest working your way left to right through the pipeline.
For each stage, you can copy the datapath for that stage from the previous lab in `src/main/scala/single-cycle/cpu.scala`.
Then, you can add the required signals to drive the datapath to the register that feeds that stage.
Throughout the given template code in `src/main/scala/pipelined/cpu.scala`, we have given hints on where to find the datapath components from Lab 2.
We have also already instantiated each of the pipeline registers for you as shown below.

```scala
val if_id       = Module(new StageReg(new IFIDBundle))

val id_ex       = Module(new StageReg(new IDEXBundle))
val id_ex_ctrl  = Module(new StageReg(new IDEXControl))

val ex_mem      = Module(new StageReg(new EXMEMBundle))
val ex_mem_ctrl = Module(new StageReg(new EXMEMControl))

val mem_wb      = Module(new StageReg(new MEMWBBundle))
val mem_wb_ctrl = Module(new StageReg(new MEMWBControl))
```

For the `StageReg`, you have to specify whether the inputs are valid via the `valid` signal.
When this signal is high, this tells the register to write the values on the `in` lines to the register.
Similarly, there is a `flush` signal that when high will set all of the register values to `0` flushing the register.
In Part III, when implementing the hazard unit, you will have to wire these signals to the hazard detection unit.
For Part I, all of the registers (including the control registers) should always be `valid` and not `flush` as shown below.

```scala
if_id.io.valid := true.B
if_id.io.flush := false.B
```

For Part I, you **do not** need to use the hazard detection unit or the forwarding unit.
These will be used in later parts of the assignment.
You also do not need to add the forwarding MUXes or worry about the PC containing any value except `PC+4`, for the same reason.

**Important**: Remember to remove the `*.io := DontCare` at the top of the `cpu.scala` file as you flesh out the I/O for each module.

## Testing your basic pipeline

You can run the tests for this part with the following commands:

```
sbt:dinocpu> Lab3 / testOnly dinocpu.RTypeTesterLab3
sbt:dinocpu> Lab3 / testOnly dinocpu.ITypeTesterLab3
sbt:dinocpu> Lab3 / testOnly dinocpu.UTypeTesterLab3
sbt:dinocpu> Lab3 / testOnly dinocpu.MemoryTesterLab3
```

Don't forget about [how to single-step through the pipelined CPU](https://github.com/jlpteaching/dinocpu/blob/main/documentation/single-stepping.md) and [DinoCPU - Debugging your implementation](https://video.ucdavis.edu/playlist/dedicated/0_8bwr1nkj/0_kv1v647d).

**Hint-1**: `auipc1` and `auipc3` actually execute two instructions (the first is a `nop`) so even though this section is about single instructions, you still need to think about the value of the `PC`.
Note: These instructions *don't* require forwarding.

**Hint-2**: You may need to include other modules to properly drive the pipelined CPU. We strongly encourage you to analyze the [data and control path](../documentation/pipelined.png) we provided to make sure you have included all the modules.


------------------------------------------------------------------------------------------------------------
# **End of Part 3.1**

# Grading for Part 3.1
Grading will be done automatically on Gradescope.
See [the Submission section](#Submission) for more information on how to submit to Gradescope.

| Name     | Percentage |
|----------|------------|
| Part I   | 30%        |

# Submission

**Warning**: read the submission instructions carefully.
Failure to adhere to the instructions will result in a loss of points.

## Code portion

You will upload the file that you changed to Gradescope on the Assignment 3.1 part of the assignment.

- `src/main/scala/pipelined/cpu.scala`

Once uploaded, Gradescope will automatically download and run your code.
This should take less than 5 minutes.
For each part of the assignment, you will receive a grade.
If all of your tests are passing locally, they should also pass on Gradescope unless you made changes to the I/O, **which you are not allowed to do**.

Note: There is no partial credit on Gradescope.
Each part is all or nothing.
Either the test passes or it fails.

## Academic misconduct reminder

You are to work on this project **individually**.
You may discuss *high level concepts* with one another (e.g., talking about the diagram), but all work must be completed on your own.

**Remember, DO NOT POST YOUR CODE PUBLICLY ON GITHUB!**
Any code found on GitHub that is not the base template you are given will be reported to SJA.
If you want to sidestep this problem entirely, don't create a public fork and instead create a private repository to store your work.
GitHub now allows everybody to create unlimited private repositories for up to three collaborators, and you shouldn't have *any* collaborators for your code in this class.

------------------------------------------------------------------------------------------------------------


# Part II: Implementing forwarding

There are three steps to implementing forwarding.

1. Add the forwarding MUXes to the execute stage, as seen below.
2. Wire the forwarding unit into the processor.
3. Implement the forwarding logic in the forwarding unit (components/forwarding.scala file).

![Forwarding MUXes](_assets/forwarding_a3.svg)

For #3, you may want to consult Section 4.7 of Patterson and Hennessy.
Specifically, Figure 4.53 will be helpful.
Think about the conditions you want to forward and what you want to forward under each condition.
`when/elsewhen/otherwise` statements will be useful here.

After this, you can remove the `forwarding.io := DontCare` from the top of the file.

## Testing your forwarding unit

With forwarding, you can now execute applications with multiple R-type and/or I-type instructions!
The following tests should now pass.

```
sbt:dinocpu> Lab3 / testOnly dinocpu.ITypeMultiCycleTesterLab3
sbt:dinocpu> Lab3 / testOnly dinocpu.RTypeMultiCycleTesterLab3
```

Don't forget about [how to single-step through the pipelined CPU](../documentation/single-stepping.md) and [DinoCPU - Debugging your implementation](https://video.ucdavis.edu/playlist/dedicated/0_8bwr1nkj/0_kv1v647d).

# Part III: Implementing branching and flushing

There are five steps to implementing branches and flushing.

1. Add MUXes for PC stall and PC from taken
2. Add code to bubble for ID/EX and EX/MEM
3. Add code to flush IF/ID
4. Connect the taken signal to the hazard detection unit
5. Add the logic to the hazard detection unit for when branches are taken. (components/hazaard.scala)
(Hints given in the pipelined/cpu.scala file)

Before you dive into this part, give some thought to what it means to bubble ID/EX and EX/MEM, how you will implement bubbles, and what it means to flush IF/ID.
Section 4.8 of Patterson and Hennessy will be helpful for understanding this part.

## Testing branching and flushing

With the branch part of the hazard detection unit implemented, you should now be able to execute branch and jump instructions!
The following tests should now pass.

```
sbt:dinocpu> Lab3 / testOnly dinocpu.BranchTesterLab3
sbt:dinocpu> Lab3 / testOnly dinocpu.JumpTesterLab3
```

Don't forget about [how to single-step through the pipelined CPU](../documentation/single-stepping.md) and [DinoCPU - Debugging your implementation](https://video.ucdavis.edu/playlist/dedicated/0_8bwr1nkj/0_kv1v647d).

# Part IV: Hazard detection

For the final part of the pipelined CPU, you need to detect hazards for certain combinations of instructions.
There are only three remaining steps!

1. Wire the rest of the hazard detection unit.
2. Modify the PC MUX.
3. Add code to *bubble* in IF/ID.

Again, section 4.8 of Patterson and Hennessy will be helpful here.

After this, you can remove the `hazard.io := DontCare`line from the top of the file.

## Testing your hazard detection unit

With the full hazard detection implemented, you should now be able to execute any RISC-V application!
The following tests should now pass.

```
sbt:dinocpu> Lab3 / testOnly dinocpu.MemoryMultiCycleTesterLab3
sbt:dinocpu> Lab3 / testOnly dinocpu.ApplicationsTesterLab3
```

Don't forget about [how to single-step through the pipelined CPU](../documentation/single-stepping.md) and [DinoCPU - Debugging your implementation](https://video.ucdavis.edu/playlist/dedicated/0_8bwr1nkj/0_kv1v647d).

## Full application traces

To make debugging easier, below are links to the full application traces from the solution to Lab 3.
To check your design, you can use the singlestep program.
If you run `print inst` at the prompt in the singlestep program it will print the current PC and the instruction at that PC (in the fetch stage).

- [Fibonacci](https://gist.github.com/powerjg/258c7941516f9c66471cd98f9f179d06), which computes the nth Fibonacci number. The initial value of `t1` contains the Fibonacci number to compute, and after computing, the value is found in `t0`.
- [Natural sum](https://gist.github.com/powerjg/974a97de1a54bd85002fc32efe3358c8), which computes the sum of numbers from 1 to 10 and stores the result (55) in the data memory at address 0x400 (and it can be found in `t0`).
- [Multiplier](https://gist.github.com/powerjg/fbfc2c993e53ba058e27a10703362f27), which multiplies two numbers initially in registers `t0` and `t1`. It stores the result of multiplication in the data memory at address 0x500 (and it can be found in `t0`).
- [Divider](https://gist.github.com/powerjg/4b836d4c0b6adb7dd4f450b1aadda279), which divides the value in `t0` by the value in `t1` and the results can be found in `t2` and stored in data memory at address 0x450.

# Grading

Grading will be done automatically on Gradescope.
See [the Submission section](#Submission) for more information on how to submit to Gradescope.

| Name     | Percentage |
|----------|------------|
| Part I   | 30%        |
| Part II  | 15%        |
| Part III | 15%        |
| Part IV  | 20%        |
| Oral     | 20%        |

# Submission

**Warning**: read the submission instructions carefully.
Failure to adhere to the instructions will result in a loss of points.

## Code portion

You will upload the three files that you changed to Gradescope on the Assignment 3.2 assignment.

- `src/main/scala/components/forwarding.scala`
- `src/main/scala/components/hazard.scala`
- `src/main/scala/pipelined/cpu.scala`

Once uploaded, Gradescope will automatically download and run your code.
This should take less than 5 minutes.
For each part of the assignment, you will receive a grade.
If all of your tests are passing locally, they should also pass on Gradescope unless you made changes to the I/O, **which you are not allowed to do**.

Note: There is no partial credit on Gradescope.
Each part is all or nothing.
Either the test passes or it fails.

## Academic misconduct reminder

You are to work on this project in pairs. You should try to find a partner
ASAP. **Remember to submit both of your assignments via your own GradeScope
account**. You may discuss high level concepts with one another (e.g., talking
about the diagram). The oral questions will be asked in the TA discussion
session to determine whether you did and understood the assignment. Those
questions will be asked individually.

**Remember, DO NOT POST YOUR CODE PUBLICLY ON GITHUB!** Any code found on
GitHub that is not the base template you are given will be reported to SJA. 
If you want to sidestep this problem entirely, don’t create a public fork and 
instead create a private repository to store your work. GitHub now allows 
everybody to create unlimited private repositories for up to three
collaborators, and you should have only one other collaborator for your code in
this class.

## Hints

* Start early! There is a steep learning curve for Chisel, so start early and ask questions on Discord and in discussion.
* If you need help, come to office hours for the TAs, or post your questions on Piazza.
* See [common errors](https://github.com/ECS154B-WQ23/dinocpu-assignment1/blob/main/documentation/common-errors.md) for some common errors and their solutions.

## Single stepper

You can also use the [single stepper](https://github.com/ECS154B-WQ23/dinocpu-assignment1/blob/main/documentation/single-stepping.md) to step through the execution one cycle at a time and print information as you go.
Details on how to use the single stepper can be found in the [documentation](https://github.com/ECS154B-WQ23/dinocpu-assignment1/blob/main/documentation/single-stepping.md).

## `printf` debugging

This is the best style of debugging for this assignment.

- Use `printf` when you want to print *during the simulation*.
  - Note: this will print *at the end of the cycle* so you'll see the values on the wires after the cycle has passed.
  - Use `printf(p"This is my text with a $var\n")` to print Chisel variables. Notice the "p" before the quote!
  - You can also put any Scala statement in the print statement (e.g., `printf(p"Output: ${io.output})`).
  - Use `println` to print during compilation in the Chisel code or during test execution in the test code. This is mostly like Java's `println`.
  - If you want to use Scala variables in the print statement, prepend the statement with an 's'. For example, `println(s"This is my cool variable: $variable")` or `println("Some math: 5 + 5 = ${5+5}")`.
