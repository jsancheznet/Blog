---
title: "Avoiding double allocation for structs and data"
layout: post
---

Let's say you want to allocate a struct that contains a pointer to
other data that also needs allocation. The natural way to do it is
calling malloc twice, one for the container and one for the data.

{% highlight c++ %}
/* The natural way to allocate a struct and its data */
struct sample_type
{
    float A;
    uint32_t B;
    uint8_t C;
    bool D;
    bool E;
    bool F;
    bool G;
};

struct sample_struct
{
    uint32_t Count;
    sample_type *Data;
};

uint32_t Count = 5;
sample_struct *Container = (sample_struct*)malloc(sizeof(sample_struct));
Container->Count = Count;
Container->Data = (sample_type*)malloc(sizof(sample_type) * Count);

// Do something to the data
for(int i = 0; i < Container->Count; i++)
{
    Container->Data[i] = {};
}
{% endhighlight %}

Something doesn't feel right, these two allocations are going to be
used together, one after the other but there is no guarantee they will
be contiguous in memory, a cache miss is likely to happen everytime
you access the data.

Here is one way to use just a single allocation for the container and
its data.


{% highlight c++ %}
/* Using a single allocation */

struct sample_type
{
    float A;
    uint32_t B;
    uint8_t C;
    bool D;
    bool E;
    bool F;
    bool G;
};

struct sample_struct
{
    uint32_t Count;
    sample_type *Data;
};

uint32_t Count = 5;
sample_struct *Container = (sample_struct*)malloc( sizeof(sample_struct) + sizeof(sample_type) * Count);
Container->Count = Count;
Container->Data = (sample_type*)(Container + 1);

// Do something to the data
for(int i = 0; i < Container->Count; i++)
{
    Container->Data[i] = {};
}
{% endhighlight %}


If you ask me the second version is better, a single call to malloc
and things that belong together are adjacent in memory.

There is something you _need_ to be aware of while using this trick,
Data should always be the last member of the struct.

{% highlight c++ %}
struct sample_struct
{
    uint32_t Count;
    sample_type *Data;
    float Other; // WRONG, writing to this variable will overwrite values in the Data array and vice versa.
};

struct sample_struct
{
    uint32_t Count;
    float Other; // CORRECT, Remember to add struct members before the Data array.
    sample_type *Data;
};
{% endhighlight %}
