Rust interface for Apple's C language extension of blocks.

On macOS and iOS, blocks use the system runtime. Other platforms use
`libBlocksRuntime` by default. Enable the `gnustep_runtime` Cargo feature
to use the blocks runtime provided by GNUstep's `libobjc2` instead; this
requires a GNUstep runtime with blocks support, not GCC's `libobjc`.

The optional `objc` feature provides the Objective-C `@?` encoding for
`&Block` and `&mut Block` through `objc` 0.2.7. Rust's orphan rules prevent
the older experimental implementations for raw pointers and `Option` of
block references; those implementations remain in the merged history.

For more information on the specifics of the block implementation, see
Clang's documentation: http://clang.llvm.org/docs/Block-ABI-Apple.html

## Invoking blocks

The `Block` struct is used for invoking blocks from Objective-C. For example,
consider this Objective-C function:

``` objc
int32_t sum(int32_t (^block)(int32_t, int32_t)) {
    return block(5, 8);
}
```

We could write it in Rust as the following:

``` rust
unsafe fn sum(block: &Block<(i32, i32), i32>) -> i32 {
    block.call((5, 8))
}
```

Note the extra parentheses in the `call` method, since the arguments must be
passed as a tuple.

## Creating blocks

Creating a block to pass to Objective-C can be done with the `ConcreteBlock`
struct. For example, to create a block that adds two `i32`s, we could write:

``` rust
let block = ConcreteBlock::new(|a: i32, b: i32| a + b);
let block = block.copy();
assert!(unsafe { block.call((5, 8)) } == 13);
```

It is important to copy your block to the heap (with the `copy` method) before
passing it to Objective-C; this is because our `ConcreteBlock` is only meant
to be copied once, and we can enforce this in Rust, but if Objective-C code
were to copy it twice we could have a double free.
