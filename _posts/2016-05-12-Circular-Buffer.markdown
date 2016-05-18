---
title:  Circular Buffer
date:   2016-05-12 23:50:00
description: What goes around comes around
---

Circular buffer/Ring buffer/Cyclic buffer store data in a FIFO (First In First Out) queue. The code in my github repo is a small example of how it should function. It could certainly be optimized for larger spaces. However, for this case the circular buffer has a pre-determined size. It uses byte array to store data and pointers to depict the circular behavior. Circular buffers could have a lot of uses. One that I can think on top of my head is cache memory. 

{% highlight C %}

uint8_t maxSize;
uint8_t *origBufferAddress;
uint8_t *readPtr;
uint8_t *writePtr;

void initialize(size_t size)
{
    uint8_t buffer[size];
    
    memset(buffer, 0, sizeof(buffer));
    maxSize = size;
    
    //Initializes all the pointers as well.
    origBufferAddress = &buffer[0];
    writePtr = &buffer[0];
    readPtr = &buffer[0];
    return;
}

{% endhighlight %}

So **origBufferAddress** pointer always keeps track of the starting address of the buffer. As the **readPtr** and **writePtr** navigate through the buffer incrementally, **origBufferAddress** is used to reset their position in the buffer. 

{% highlight C %}

uint8_t readByte()
{
    if (readPtr - origBufferAddress == maxSize)
    {
        readPtr = origBufferAddress;
    }
    uint8_t retVal = *readPtr;
    readPtr = readPtr + 1;
    return retVal;
}

void writeByte(uint8_t byte)
{
    if (writePtr - origBufferAddress == maxSize)
    {
        writePtr = origBufferAddress;
    }
    
    *writePtr = byte;
    writePtr = writePtr + 1;
}

{%endhighlight c %}

As the **readPtr** reads through the buffer, the **maxSize** ensures that it doesn't read beyond the max bound of the buffer. This is done by taking the difference of the original buffer address and the current address of the read pointer. The difference is essentially the current index of the buffer from where the **readPtr** is retrieving data. If the difference is still within the bounds of the buffer, the **readPtr** retrieves the byte and carries on to read the neighbour byte.

The same logic applies for the **writePtr** as well. The **writePtr** writes the byte and moves on to the neighbour byte  .


**MISTAKE #1**

{% highlight C %}

struct Node{
    uint8_t val;
    struct Node *prev;
};

{% endhighlight %}

I actually started off with idea of using LinkedList to implement a circular buffer. Linkedlist has the obvious advantage of dynamic memory allocation. However, I had to think about the overhead of using singly/doubly and start pointers. Adding series of checks to prevent memory leaks and preserve the circular attribute of the buffer seemed a little excessive for a small example like this.


**MISTAKE #2**

I was foolish enough to forget this line of code. 

{% highlight C %}

memset(buffer, 0, sizeof(buffer));

{% endhighlight %}
