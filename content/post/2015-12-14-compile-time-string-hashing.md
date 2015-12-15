+++
date = "2015-12-14T22:29:27-06:00"
draft = true
title = "C++11 Compile-Time String Hashing"
+++

We can write a simple djb string hashing function that can be called at runtime.

{{< highlight cpp >}}
uint32_t hash(const char *str)
{
    uint32_t hash = 5381;

    while (*str)
        hash = hash * 33 + *str++;

    return hash;
}
{{< /highlight >}}

C++11 added constexpr functions which can execute at compile-time but C++11 constexprs do not allow loops (C++14 does).  The djb function must be rewritten as recursive to satisfy the constexpr constraint.

{{< highlight cpp >}}
inline constexpr uint32_t hash(const char *str, uint32_t hash=5381)
{
    return *str == 0 ? hash : hash(str + 1, hash * 33 + *str);
}
{{< /highlight >}}

The constexpr can now be called to hash a string literal.  However, assembly output shows the constexpr is still generating a runtime function call.

{{< highlight cpp >}}
uint32_t sid = hash("this is a string test!");
{{< /highlight >}}
{{< highlight nasm >}}
	mov	edx, 5381				; 00001505H
	lea	rcx, OFFSET FLAT:$SG63574
	call	?hash@@YAIPEBDI@Z			; hash
	mov	DWORD PTR sid$[rsp], eax
{{< /highlight >}}

To force the constexpr to be evaluated at compile-time it needs to be called from a compile-time only context.  One solution is to use the constexpr function call as a template parameter.

{{< highlight cpp >}}
template<uint32_t v>
struct const_value {
	static const uint32_t value = v;
};

#define SID(x) const_value<hash(x)>::value

uint32_t sid = SID("this is a string test!");
{{< /highlight >}}

{{< highlight nasm >}}
	mov	DWORD PTR sid$[rsp], 1910223762		; 71dbb392H
{{< /highlight >}}

The template forces the evaluation at compile-time leaving only the string hash literal compiled into the executable.
