# *What is movi\_i32?*

movi\_i32 is one of Qemu intermediate representations (IRs), defined in file *tcg/tcg-opc.h*:

    DEF(movi_i32, 1, 0, 1, 0)

According with the syntax of defining an IR (*tcg/tcg-opc.h*):

    /*
     * DEF(name, oargs, iargs, cargs, flags)
     */

movi\_i32 has one **constant** input temporary, and one output temporary. The semantic is to move the constant input temporary to the output. 

# *How Qemu allocates register for this IR?*

The corresponding source code defined in *tcg/tcg.c*:

    static void tcg_reg_alloc_movi(TCGContext *s, const TCGArg *args)
    {
        TCGTemp *ots;
        tcg_target_ulong val;
    
        ots = &s->temps[args[0]];
        val = args[1];
    
        if (ots->fixed_reg) {
            /* for fixed registers, we do not do any constant
               propagation */
            tcg_out_movi(s, ots->type, ots->reg, val);
        } else {
            /* The movi is not explicitly generated here */
            if (ots->val_type == TEMP_VAL_REG)
                s->reg_to_temp[ots->reg] = -1;
            ots->val_type = TEMP_VAL_CONST;
            ots->val = val;
        }
    }

The big picture is to assign the input constant to the output. Details are explained below.

1.  `args[0]` is the **index** of the output temporary
2.  `args[1]` is the **value** of the constant input temporary
3.  Via the index - `args[0]`, Qemu can accesses the content of the output temporary (`&s->temps[args[0]]`), which `temps[]` is an array to store all temporaries of IRs that are being used in the current translation block.
4.  Temporary is defined as a `structure`, one of its member is `fixed_reg`. Most temporaries don't use this member with a few exceptions, indicating the temporary has a fixed register in the host machine. 
    
    For example, `env` temporary uses this member. For the rest, this fixed\_reg is set to 0.
5.  If the output temporary `fixed_reg` is nonzero, then Qemu will generate a **host instruction** via function: `tcg_out_movi(s, ots->type, ots->reg, val);`
    -   `ots` is the output temporary
    -   `ots->type` indicates the temporary type, either 32 bit or 64 bit width
    -   `ots->reg` indicates the **index** of the register that the temporary occupies **currently** in the host machine (since it has a fixed\_reg, which one is it?)
    -   `s` is the TCGContext (will not talk about it in details here)
    -   The semantic of this function is to issue an host instruction that assigning the value `val` to the register (`ots->reg`), which is corresponding to the semantic of the IR.
6.  If the output temporary's fixed\_reg is zero, then 
    -   If `ots->val_type` is `TEMP_VAL_REG`, indicating that the temporary currently occupies a register in the host machine
        -   `s->reg_to_temp[ots->reg] = -1;` frees the register
        -   Array `reg_to_temp[]` is a member of the TCGContext indicating which registers in the host machine correspond to which temporaries.
    -   Update val\_type to TEMP\_VAL\_COSNT, indicating the current value of the temporary is stored in the member `val` other that a register nor a memory in host.
    -   Assign the input constant val to the `val` member of the temporary.
    
    <font color = "blue">In this case, what it does is a constant propagation without issuing any host instruction.</font>

# *A concrete example*

Consider an assembly instruction running in the guest machine of Qemu. 

    IN: 
    0x000fe070:  mov    $0xe5852,%edx

It assigns the constant value `0xe5852` to register edx.

*So what IRs are translated based on the instruction?*

    OP:
     ---- 0xfe070
     movi_i32 tmp0,$0xe5852
     mov_i32 edx,tmp0

Qemu translates the guest instruction into two IRs:
1.  Assign constant `0xe5852` to a temporary `tmp0`
2.  Assign `tmp0` to `edx`

Let's focus on the first IR, since it is a `movi_i32`. 

Via gdb, we can see the index of the output temporary - `args[0]` is

    (gdb) p args[0]
    $38 = 7

Then Qemu can access the content of the output temporary.

    (gdb) p *ots 
    $39 = {base_type = TCG_TYPE_I32, type = TCG_TYPE_I32, val_type = 2, reg = 0, 
      val = 0, mem_reg = 5, mem_offset = 8, fixed_reg = 0, mem_coherent = 0, 
      mem_allocated = 1, temp_local = 0, temp_allocated = 0, next_free_temp = 0, 
      name = 0x8026e302 "edx"}

-   The output temporary is a **global** temporary edx (specified by the name member)
-   The val\_type is 2, indicating it is a `TEMP_VAL_MEM`, which indicates its current value was stored in memory of the host.
    
    defined in tcg.h
    
        #define TEMP_VAL_DEAD  0
        #define TEMP_VAL_REG   1
        #define TEMP_VAL_MEM   2
        #define TEMP_VAL_CONST 3
-   fixed\_reg is set to 0, Qemu will not issue any **host** instruction
-   ots->val\_type is **NOT** `TEMP_VAL_REG`, Qemu need not to free any host register
-   The key codes that Qemu executed in run-time are:
    
        ...
        ots->val_type = TEMP_VAL_CONST;
        ots->val = val;
        ...

After the allocation finished executing, the output temporary became:

    {base_type = TCG_TYPE_I32, type = TCG_TYPE_I32, val_type = 2, reg = 0, 
      val = 0xe5852, mem_reg = 5, mem_offset = 8, fixed_reg = 0, mem_coherent = 0, 
      mem_allocated = 1, temp_local = 0, temp_allocated = 0, next_free_temp = 0, 
      name = 0x8026e302 "edx"}

# Conclusion

For this particular case, Qemu did not issue any host instruction, it only did a constant propagation.

(*All codes presented should be found in* [stable-1.0](http://git.qemu.org/?p=qemu.git;a=tree), *and assumes both guest and host machine are in x86 platform.*)