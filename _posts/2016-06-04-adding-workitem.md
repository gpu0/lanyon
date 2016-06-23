---
layout: post
title: Adding Workitem inside IR
---

After adding address space to arguments, the values in the address location need to be loaded to vector GPRs (General Purpose Registers). The way we load values is per workitem, that is each load operation happens per workitem. In order to use per-workitem load, we need ID of calling work item. Here, we try to get the workitem inside IR. AMDGPU backend has an intrinsic to access the workitem `llvm.amdgcn.workitem.x()` it returns a value of `i32`.

### Getting WorkItem

We are going to write similarly the way we created argumented function.

We create a vector of arguments passed to the function `llvm.amdgcn.workitem.x`. As the number of arguments are zero, we create an empty vector. As the function returns `i32`, we create `FunctionType` with that signature.

{% highlight cpp %}
std::vector<llvm::Type*> workItemArgs;
llvm::FunctionType *workItemType = 
	llvm::FunctionType::get(llvm::Type::getInt32Ty(context), workItemArgs, false);
{% endhighlight %}

We create a function with name `llvm.amdgcn.workitem.id.x` using
{% highlight cpp %}
llvm::Function::Create(workItemType, llvm::Function::ExternalLinkage, "llvm.amdgcn.workitem.id.x", module);
{% endhighlight %}

Once we made that function, we shall pass an argument and get return value. As we are not passing any argument, we use an empty argument value vector. We get the workitem id function from LLVM module. The return variable name is passed to a variable `tid` which is passed to `CreateCall`. The output of the function is `llvm::Value` pointer which stores the name and type of the return value which is used to access different elements of memory.

{% highlight cpp %}
std::vector<llvm::Value*> workItemArgsValue;
llvm::Function *workItem = module->getFunction("llvm.amdgcn.workitem.id.x");
llvm::Value *workItemPtr = builder.CreateCall(workItem, workItemArgsValue, "tid");
{% endhighlight %}
