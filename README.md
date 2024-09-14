# Qemu_Understanding
## Aim:
We wish to turn some standard/generic library calls for a linux-user on the emulator to native calls i.e. native versions of the code for faster and more efficient execution of the program
 
Understanding:-
### Box 64:
In box 64 all the library calls are either emulated or used in their native version. This bridging between the emulated and native worlds is the special feature provided by the box64/86 emulator as it increases the speed of the functioning of the emulator as compared to only using the interpreter. Here we use a mixture of both dynarec(dynamic recompiler emulation technique) for emulation and native versions of the code for the library calls.
### Qemu:
Qemu makes the use of TCG-Tiny Code Generator(a jit compiler) for emulation. It emulates the target architecture on the host architecture by dynamically translating target ISA(Instruction Set Architecture) to  IR code(intermediate representation) and then the IR code into the host ISA.
TCG is used in the case when the target and the host architectures are different but in the case when both the host and target have the same architecture, we don't need to convert the target's ISA into host's ISA as both will have the same ISA. In this case we make the use of an accelerator like KVM which provides qemu with virtualizing capabilities. KVM is considered an accelerator because it prevents unneeded emulation of instructions when host and target share the same architecture. Only system level instructions might be emulated/intercepted. Use of KVM allows Qemu to provide the target architecture with almost native -like speed thus enhancing the capabilities of Qemu majorly. 
### Working of TCG:
The function tcg_cpu_exec is used to find and/or generate translated blocks. The TB(translation blocks) stores some state information about the virtual CPUs which allows for the target CPUs to achieve a good speed by using this info. If the state changes (e.g. privilege level), a new TB will be generated and the previous TB won’t be used anymore until the state matches the state recorded in the previous TB. 
The function tb_gen_code is responsible for generating IR code using the gen_intermediate_code function and then host architecture assembly instructions with tcg_gen_code function.
### TB: 
TB have 2 major parts, a prologue part in tb_start(made by func gen_tb_start) and an epilogue part tb_end(made by func gen_tb_end). The tb_end is of extreme importance as it is used as a placeholder for block chaining- an optimization feature of TCG that executes TBs to be called succesfully after execution without the need to get back to the Qemu code and look for the next TB to execute. 
Now, if we have the host assembly code, we can directly run it on our physical CPU. However, some parts of the generated code redirect to special handlers for IR operations that couldn't be translated into host assembly code.
We must not forget that TCG is a jit compiler for general purpose instructions found in the architecture. But once we reach system specific instructions, they remain emulated and emulation requires a lot more host instructions to be executed for a single target instruction. This is where we need the use of TCG Helpers. These are implemented inside the specific target emulation part of Qemu. In order to simulate system instructions, Qemu introduced the helper concept. The func tcg_gen_callN is responsible for the generation of a function call to our helper function. Qemu helpers deal with the implementation of guest memory access with support from virtual TLBs and the exceptions generated in the process too. Now that we finally have the host assembly code, out host/physical CPUs can run this code.Thus running a program which was intended to be run on the target architecture which was being emulated by Qemu.

## Question:
Native versions of the code are required for these libraries. The box64 is generally used as an emulator for games but qemu is used to emulate whole OSs. So when a linux user will make a standard library call, where will the native version of the code need to be present and how will its wrapping with code needs to be linked in order to tell Qemu that it doesn't need to translate the call into the host ISA and can simply use the native version of the code present?  

## Current Theoretical Solution Trajectory:
Need to make native libraries in qemu for linux-users and test that the wrapping of the native versions of these calls is compatible with emulation technique i.e. TCG used by qemu. With my current understanding of Qemu, I think these native library function codes(for example: in the case of memcpy- mem*, str* ) will need to be linked with the TB generated codes before the generation of the host architecture assmebly instructions as just like in the case of box64 for a library call, we can check for the native versions of the code to be present and if not present then proceed with the standard translation of the target ISA to host ISA(the general emulation technique). We'll have to make sure that a kind of wrap-up/tie-up takes place after the translation between the native versions and the translated codes as all the library function calls were part of the same program being tried to be run. It will just be that the fast execution of the native versions of the code(as compared to the translated part) will allow Qemu to shorten the runtime used by these library calls. 

## Requirements:
Need to understand where exactly will the native versions of the code be need to be present so that some methods can be thoughtout for the compatible linking or bridging between native versions of the generic library calls and the emulated part of these calls. 

## Links which helped me while studying about Qemu:
https://github.com/airbus-seclab/qemu_blog

https://gist.github.com/royrishi06/c87331d01e60a76b5008fd66fabea69e#file-qemu_gsoc

https://github.com/ptitSeb/box64

https://github.com/qemu/qemu

https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi
