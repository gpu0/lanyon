---
layout: post
title: Get Element Pointer from Memory Space
---

Once we get the id for current workitem, we need to obtain the relative address of the memory for loading the value inside the vGPRs. We use GEP instructions in LLVM to get the relative memory for each workitem. The GEP instruction returns address requested by each workitem, takes vector of relative addressing values (refer to GEP LLVM docs for more info on addressing) and argument names. As we have two memory location, we get two relative address for each workitem. Hence we use a return GEP vector.

{% highlight cpp %}
std::vector<llvm::Value*> argsGEP(1, workItemPtr);
std::vector<llvm::Value*> retnGEP;
for(uint32_t i = 0; i < argNames.size(); i++){
	retnGEP.push_back(builder.CreateGEP(NamedValues[i], argsGEP, argNames[i] + "_ptr"));
}
{% endhighlight %}
