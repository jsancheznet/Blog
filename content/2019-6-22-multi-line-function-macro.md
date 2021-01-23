---
title: "Making multi-line macros behave as a functions"
layout: post
---

When you need a muli-line macro to have its own scope and behave as a
regular function would, the naive way is to simply wrap the macro
statements inside curly brackets. Now, time has passed and suddenly
program compilation is failing, and as usual the compiler output is
not helping. The naive macro implementation has come back to bite you.


Here's the naive implementation, let's see how to fix it.

{% highlight c++ %}
#define MyMultipleLineMacro(Expr) \
{                                 \
    DoStuffRightHere(Expr);       \
    DoMoreStuff;                  \
}                                 \

void main()
{
    if(1)
        MyMultipleLineMacro(Input); // Fails to compile!
    else
        SomeOtherOperation;
}
{% endhighlight %}

Let's run the preprocessor and see how the macro was expanded:
{% highlight c++ %}
void main()
{
    if(1)
        {
            DoStuffRightHere(Input);
            DoMoreStuff;
        }; // Fails to compile!
    else
        SomeOtherOperation;
}
{% endhighlight %}
To solve this problem, wrap the macro inside a do{}while(0) loop instead.
{% highlight c++ %}
#define MyMultipleLineMacro(Expr) \
    do{                           \
        DoStuffRightHere(Expr);   \
        DoMoreStuff;              \
    }while(0)                     \

void main()
{
    if(1) // Now the macro expands to valid C/C++.
        do {
	    DoStuffRightHere(Input);
	    DoMoreStuff;
	}while(0);
 else
        SomeOtherOperation;

    return 0;
}
{% endhighlight %}
Have a good day!