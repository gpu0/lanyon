---
layout: post
title: Full Code listing for generating vector copy LLVM IR
---

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

int main(){
  llvm::LLVMContext context;
  llvm::Module *module = new llvm::Module("top", context);
  llvm::IRBuilder<> builder(context); 

  std::vector<std::string> argNames = {"a", "b"};

  std::vector<llvm::Type*> argPointers(argNames.size());
  for(int i=0;i<argNames.size();i++){
    argPointers[i] = llvm::PointerType::get(builder.getInt32Ty(), 1);
  }

  llvm::FunctionType *funcType = 
    llvm::FunctionType::get(builder.getVoidTy(), argPointers, false);
  llvm::Function *mainFunc = 
     llvm::Function::Create(funcType, llvm::Function::ExternalLinkage, "vector_copy", module);

  llvm::BasicBlock *entry = llvm::BasicBlock::Create(context, "", mainFunc);
  builder.SetInsertPoint(entry);

  std::vector<llvm::Value*> NamedValues;
  unsigned Idx=0;
  for(auto &Arg: mainFunc->args()){
    Arg.setName(argNames[Idx++]);
    NamedValues.push_back(&Arg);
  }

  std::vector<llvm::Type*> workItemArgs;

  llvm::FunctionType *workItemType = 
    llvm::FunctionType::get(llvm::Type::getInt32Ty(context), workItemArgs, false);
  llvm::Function::Create(workItemType, llvm::Function::ExternalLinkage, "llvm.amdgcn.workitem.id.x", module);

  llvm::Function *workItem = module->getFunction("llvm.amdgcn.workitem.id.x");
  std::vector<llvm::Value*> workItemArgsValue;

  llvm::Value *workItemPtr = builder.CreateCall(workItem, workItemArgsValue, "tid");

  std::vector<llvm::Value*> argsGEP(1, workItemPtr);
  std::vector<llvm::Value*>GEP;
  for(int i=0;i<2;i++){
    GEP.push_back(builder.CreateGEP(NamedValues[i], argsGEP, argNames[i]+"_ptr"));
  }
  llvm::Value *load1 = builder.CreateLoad(GEP[0], "l1");
  llvm::Value *store = builder.CreateStore(load1, GEP[1]);

  builder.CreateRetVoid();
  module->dump( );
}

{% endhighlight %}
