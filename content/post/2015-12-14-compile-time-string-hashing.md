+++
date = "2015-12-14T22:29:27-06:00"
title = "C++11 Compile-Time String Hashing"
+++

Here is a simple, lightweight implementation of compile-time string hashing in C++11.

I'll start with the basic djb hash function.  This function can be called at runtime to hash a string literal.

{{< highlight cpp >}}
uint32_t hash(const char *str)
{
    uint32_t hash = 5381;

    while (*str)
        hash = hash * 33 + *str++;

    return hash;
}
{{< /highlight >}}

In C++11, `constexpr` can be used to denote that a function can be evaluated at compile time.  But `constexpr` also has certain constraints which disallow looping and multiple return statements.  hash can be rewritten as a recursive function to satisfy these `constexpr` constraints.

{{< highlight cpp >}}
constexpr uint32_t hash(const char *str, uint32_t hash=5381)
{
    return *str == 0 ? hash : hash(str + 1, hash * 33 + *str);
}
{{< /highlight >}}

Now the constexpr hash function can be called to hash a string literal.  This does not guaruntee it is evaluated at compile-time.  In fact, assembly output shows the constexpr still generates a runtime function call.

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

This forces the hash function to be evaluated at compile-time leaving only the string hash value compiled into the executable.
