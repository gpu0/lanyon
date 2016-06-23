---
layout: post
title: Generating AMDGPU IR arguments
---
Being able to generate LLVM IR, now this post adds arguments to the function created.
Before creating arguments, the type of memory location the argument value is present need to be found.
The kernels take device memory pointer as an argument. GPUs can address different memory spaces,

- `0` for Generic memory space
- `1` for Device memory space. As in GDDR or HBM
- `3` for Local Data Share memory space.
- `4` for Constant memory space.

Most of the kernels point to `Device Memory` space. Hence we use the id 1.

### Creating arguments
We need two things before creating the arguments.
1. Type and Location of the pointer.
2. Name of the argument variable.

{% highlight cpp %}
std::vector<std::string> argNames = {"a", "b"};
std::vector<llvm::Type*> argPointers(argNames.size());
for(uint32_t i=0;i<argNames.size();i++){
  argPointers[i] = llvm::PointerType::get(builder.getInt32Ty(), 1);
}
llvm::FunctionType *funcType =
  llvm::FunctionType::get(builder.getVoidTy(), argPointers, false);
{% endhighlight %}
The API `llvm::PointerType::get` takes the second argument, the address space value.
We have attached type and location argument to `FuntionType` but not name.

{% highlight cpp %}
std::vector<llvm::Value*> NamedValues;
unsigned Idx=0;
for(auto &Arg: mainFunc->args()){
  Arg.setName(argNames[Idx++]);
  NamedValues.push_back(&Arg);
}
{% endhighlight %}

The above code attachs the name of the argument to type of the argument. 
