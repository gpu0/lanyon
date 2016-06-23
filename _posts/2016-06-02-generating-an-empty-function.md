---
layout: post
title: Generating an empty function using LLVM
---

For generating LLVM IR, the Kaledioscope example presents a good start. But, there is a steep learning curve once you start using LLVM functions and methods. The idea of this post is to not build a compiler but to generate IR straight from LLVM APIs. Writing a frontend seemed more easy with knowing LLVM upfront. This helps to design code before hand rather than trying to change during learning LLVM.

This post focus on writing a simple LLVM code generator which does not take any arguments and does not return. Lets start with headers.

### Headers
Through out the blog posts the following headers are used and the description of the headers can be found in LLVM documentation.

{% highlight cpp %}

#include "llvm/ADT/ArrayRef.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Constants.h"
#include <vector>
#include <string>
#include <stdio.h>
#include <iostream>

{% endhighlight %}


### LLVM Code

To make LLVM to generate code, we need to initialize 3 variables.

{% highlight cpp %}

llvm::LLVMContext context;
llvm::Module *module = new llvm::Module("amdgpu", context);
llvm::IRBuilder<> builder(context); 

{% endhighlight %}

Then, we have to setup our arguments. As we dont plan on passing arguments yet, we will create a empty c++ vector of type `llvm::Type*`. This makes `builder` to generate a `0` argument function.

{% highlight cpp %}

std::vector<llvm::Type*> argPointers;

{% endhighlight %}

Next, we have to create a function-type `llvm::FunctionType`. The `FunctionType` is generated using `get` method from its class. It takes 3 arguments.

1. Return Type
2. Arguments Type
3. Whether the function take variable arguments

This depicts the structure of the function we want to create.

{% highlight cpp %}
llvm::FunctionType *funcType = 
	llvm::FunctionType::get(builder.getVoidTy(), argPointers, false);

{% endhighlight %}

Once the required type of function is created, we create the actual function. We create function of type `llvm::Function`. It is created using `Create` method from `llvm::Function` class. It takes 4 arguments.

1. FunctionType
2. Type of linkage
3. Name of the function
4. LLVM Module

{% highlight cpp %}
llvm::Function *mainFunc = 
	llvm::Function::Create(funcType, llvm::Function::ExternalLinkage, "vector_copy_kernel", module);
{% endhighlight %}

Then, we have to create the contents of the function. The body of the function is built using `llvm::BasicBlock`. It is created the same way `Function` and `FunctionType` are created. It takes 3 arguments

1. LLVM Context
2. Name of entry point
3. The function it belongs to
And, we set it as the starting point to the function.

{% highlight cpp %}
llvm::BasicBlock *entry = llvm::BasicBlock::Create(context, "", mainFunc);
builder.SetInsertPoint(entry);
{% endhighlight %}

Finally, setting up the return statement. 
{% highlight cpp %}
builder.CreateRetVoid();
{% endhighlight %}

The generated function can be displayed through
{% highlight cpp %}
module->dump();
{% endhighlight %}

### Building the generator

{% highlight shell %}
$ g++ -g -O3 t01.cpp `llvm-config --cxxflags --ldflags --system-libs --libs core` -o t01
{% endhighlight %}
