---
layout: post
title: Manipulating memory
---
Once we are able to get the memory relative to each work item, we can operate on it using `load`,`store` and `math` instructions. Here, we try to do a simple vector copy where argument one `a` is copied to argument two `b`.

We create a load instruction which returns the loaded variable. The store instruction create a store instruction to store the value inside memory location of second vector.
{% highlight cpp %}
llvm::Value *load = builder.CreateLoad(retnGEP[0], "val");
llvm::Value *store = builder.CreateStore(load, GEP[1]);
{% endhighlight %}

### Output
The final code looks like this

```
; ModuleID = 'top'
source_filename = "top"

define void @vector_copy_kernel(i32 addrspace(1)* %a, i32 addrspace(1)* %b) {
  %tid = call i32 @llvm.amdgcn.workitem.id.x()
  %a_ptr = getelementptr i32, i32 addrspace(1)* %a, i32 %tid
  %b_ptr = getelementptr i32, i32 addrspace(1)* %b, i32 %tid
  %l1 = load i32, i32 addrspace(1)* %a_ptr
  store i32 %l1, i32 addrspace(1)* %b_ptr
  ret void
}

; Function Attrs: nounwind readnone
declare i32 @llvm.amdgcn.workitem.id.x() #0

attributes #0 = { nounwind readnone }

```
