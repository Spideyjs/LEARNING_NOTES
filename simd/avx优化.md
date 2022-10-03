## AVX is not faster than SSE

[c - AVX is not faster than SSE? - Stack Overflow](https://stackoverflow.com/questions/32675422/avx-is-not-faster-than-sse)

I have large block of data to calculate:

```c
static float source0[COUNT];
static float source1[COUNT];
static float result[COUNT];    /* result[i] = source0[i] * source1[i]; */

s0 = (size_t)source0;
s1 = (size_t)source1;
r = (size_t)result;
```

They are all 32-byte aligned.

The related SSE code:

```c
for(i = 0; i < COUNT; i += 16)
{
    __asm volatile
    (
        "movntdqa xmm0, [%0]\n\t"
        "movntdqa xmm1, [%1]\n\t"
        "mulps xmm1, xmm0\n\t"
        "movntps [%2], xmm1"
        : : "r"(s0 + i), "r"(s1 + i), "r"(r + i) : "xmm0", "xmm1"
    );
}
```

The related AVX code:

```c
for(i = 0; i < COUNT; i += 32)
{
    __asm volatile
    (
        "vmovapd ymm0, [%0]\n\t"
        "vmovapd ymm1, [%1]\n\t"
        "vmulps ymm1, ymm1, ymm0\n\t"
        "vmovntps [%2], ymm1"
        : : "r"(s0 + i), "r"(s1 + i), "r"(r + i) : "ymm0", "ymm1"
    );
}
```

The result is that AVX code used time is always nearly the same as SSE code. But they are much faster then normal C code. I think the major reason is that "vmodapd" does not support "NT" version, until AVX2 extension. This causes too much d-cache pollution.

**The data block size is 32MB**

### answer

- Yeah, you're completely memory bound. See this: [stackoverflow.com/a/18159503/922184](http://stackoverflow.com/a/18159503/922184) 

- Is it *absolutely necessary* for you to store all multiplication results before proceeding? You could 1) **do extra work with the data you stream (additions/masks/comparisons?)**, which would increase the cost of a loop iteration and thus reduce your memory bandwidth requirements, or 2) You could **cut up your data in less-than-cache-sized chunks.**
- 2
- 3
- 4
- 5

[c - Why vectorizing the loop does not have performance improvement - Stack Overflow](https://stackoverflow.com/questions/18159455/why-vectorizing-the-loop-does-not-have-performance-improvement/18159503#18159503)

非顺序访存

## Loop unrolling Loop tiling

完成**循环展开**以消除循环的开销。它(通常)仅对相当小的循环有用，其中迭代次数很少并且在编译时是已知的。它主要由编译器完成。 在较旧的时候，当计算机速度较慢而编译器更加原始时，程序员会进行手动循环展开，但现在程序员这样做是不寻常的 - 除非是非常严格的嵌入式系统。 

**循环平铺**通常用非常大的数据集完成。目标是：在一些新数据中进行分页之前，将一些数据加载到高速缓冲存储器中并对其执行所有操作。 根据正在执行的操作和数据的内部组织，一个简单的循环可能跳转到不同的数据页面，导致大量的缓存未命中(和页面加载)。仔细规划执行顺序可以显着改善某些问题的运行时间。 虽然编译器可能会执行循环切片，但有时程序员可能会手动执行此操作，并且可能比编译器做得更好。 通常，不要尝试进行这些类型的优化，因为它们会给代码增加很多复杂性(和错误)，并且通常只能提供适度的性能提升。但是，如果您的代码很慢并且分析表明特定类型的瓶颈，那么应该考虑循环平铺之类的东西，这可能会带来很大的性能提升。