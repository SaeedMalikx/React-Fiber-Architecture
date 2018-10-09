

Just another one of my **Self-Study Projects** . I don't tend to make such projects public but this might be of help for someone looking to understand Fiber and React Under the hood. Do remember as of writing this, this is my first time looking at the code for Fiber.

Often times reading a large amount of code can be very intimidating for someone new to programming. So it's best to read each line and document them if needed so you don't go hoping around files searching for that lost variable or get lost on the Import tags. There is no set rules or anything here, just documenting/laying it out the best I can. Any part of this doc might change as my understanding of the whole system grows.  

**As for obvious errors and such:**

> “There are no mistakes, only happy accidents.” Bob Ross


**Lot of this comes directly from Fiber codebase itself, a lot of it will be just code reorganized for easier understanding** 
**I Turn a lot of functions into Pseudocode for easier reference and understanding**

Here is the link for the official codebase [REACT](https://github.com/facebook/react/tree/master/packages/react-reconciler)

### Files So Far

**(only important/big ones are listed here, checked are the ones explained so far)**

- [x] ReactFiber.js > Defining Functions For Creating Fibers
- [ ] ReactCurrentFiber.js > ....
- [ ] ReactDebugFiberPerf > probably for debugging DEV
- [ ] ReactChildFiber > The Bulk of The Child Reconciliation
- [ ] ReactClassComponent.js > Class mounting, class instances....
- [ ] ReactContext.js > Probably context stuff
- [ ] ReactFiberHostContext > Host Stuff
- [ ] ReactFiberHydrationContext.js > Must Understand the Fast and Furious DOM
- [ ] ReactFiberPendingPriority.js > Priority, marks levels for various stuff
- [ ] ReactFiberRoot.js > For creating Fiber Roots
- [ ] ReactProfilerTimer.js > For Recording/Tracking Time(expiration, commits)
- [ ] ReactFiberTreeReflection.js > Finding Where the host fibers are...nature 
- [ ] ReactFiberScheduler > THIS IS SOME GUCCI CODE
- [ ] ReactUpdateQueue.js > Scheduling The Updates(or I think so)
- [ ] ReactFiberBeginWork.js > This is for begining the work
- [ ] ReactCommitWork.js > Committing Work
- [ ] ReactCompleteWork.js > CompleteWork
- [ ] ReactUnwindWork.js > Gotta unwind the work once its completed
- [ ] ReactFiberReconciler.js >  The Main File


## What Is A Fiber and How Are They Created?

> A Fiber is work on a Component that needs to be done or was done. There can be more than one per component.

It is essentially an object with set properties used to identify what kind of a (component) it is and what works needs to be done. Assign prioirity levels for efficient processing. since it uses flow, it is also exported as a type. 

Most/if not all is in: **ReactFiber.js**

```

tag: Tag identifying the type of fiber

key: Unique identifier of this child

type: The function/class/module associated with this fiber

stateNode: The local state associated with this fiber

return: The Fiber to return to after finishing processing this one

child: Singly Linked List Tree Structure

sibling: Singly Linked List Tree Structure

index: Singly Linked List Tree Structure

ref: The ref last used to attach this node

pendingProps: Input is the data coming into process this fiber/Arguments/Props

memoizedProps: The props used to create the output

updateQueue: A queue of state updates and callbacks

memoizedState: The state used to create the output

firstContextDependency: A linked-list of contexts that this fiber depends on

mode: (Bitfield) that describes properties about the fiber and its subtree

effectTag: Effect

nextEffect: Singly linked list fast path to the next fiber with side-effects

firstEffect: The first fiber with side-effect within this subtree

lastEffect: The last fiber with side-effect within this subtree

expirationTime: Represents a time in the future by which this work should be completed

childExpirationTime: This is used to quickly determine if a subtree has no pending changes

alternate: A pooled version of a Fiber-Every fiber that gets updated will

actualDuration: Time spent rendering this Fiber and its descendants for the current update

actualStartTime: This marks the time at which the work began(only if Fiber is active in render phase)

selfBaseDuration: Duration of the most recent render time for this Fiber

treeBaseDuration: Sum of base times for all descedents of this Fiber.
```

While in DEV Mode, additional properties for (tracking) fibers
```
_debugID
_debugSource
_debugOwner
_debugIsCurrentlyTiming
```
Function **FiberNode** is incharge of creating the Fiber, which has a constructor function **createFiber** returning Fibernode . Both are relying on the following parameters

```
tag,
pendingProps,
key, 
mode
```
Function **createWorkInProgress** is used to create an alternate fiber.(Pooling). Uses a double buffering pooling technique.
```
3 parameters (current, pendingProps, expirationTime)
making a work in progress fiber
  
  if workInprogress = null
    createFiber (new)
  else
    reset effect tags on the alternate
the rest of work in progress fiber
```

Function(exported) **createFiberFromElement**. Hmm look this is incharge of creating all the fibers based on the React element type. 
```
parameters(element, mode, expirationTime)

  type = element.type
  fiberTag
    if 
      function > shouldConstruct ? ClassComponent : InderterminateComponent
    elif 
      string > HostComponent
    else
      switch: case depending on type
          REACT_FRAGMENT_TYPE > return createFiberFromFragment(pendingProps, mode, expirationTime, key)
          REACT_CONCURRENT_MODE_TYPE > ConcurrentMode
          REACT_STRICT_MODE_TYPE > StrictMode
          REACT_PROFILER_TYPE > createFiberFromProfiler(pendingProps, mode, expirationTime, key)
          REACT_PLACEHOLDER_TYPE > PlaceholderComponent
        default 
            REACT_PROVIDER_TYPE > ContextProvider
            REACT_CONTEXT_TYPE > ContextConsumer
            REACT_FORWARD_REF_TYPE > ForwardRef
            REACT_PURE_TYPE > PureComponent
         invariant 
    createFiber(fiberTag, pendingProps, key, mode)
   return the created fiber
          
```
ReactWorkTags For Quick Reference

```
FunctionComponent = 0;
FunctionComponentLazy = 1;
ClassComponent = 2;
ClassComponentLazy = 3;
IndeterminateComponent = 4; 
HostRoot = 5; 
HostPortal = 6;
HostComponent = 7;
HostText = 8;
Fragment = 9;
Mode = 10;
ContextConsumer = 11;
ContextProvider = 12;
ForwardRef = 13;
ForwardRefLazy = 14;
Profiler = 15;
PlaceholderComponent = 16;
PureComponent = 17;
PureComponentLazy = 18;

also MAX_SIGNED_31_BIT_INT = 1073741823
```

Following Functions create different type of fibers, assign a type and expirationTime. Then using **createFiber** , return a Fiber.

```
createFiberFromFragment
createFiberFromProfiler
createFiberFromText
createFiberFromHostInstanceForDeletion
createFiberFromPortal
```

Function **assignFiberPropertiesInDev** : Used for stashing WIP properties to replay failed work in DEV.

----------------
### Fiber Files that are exported to the React-DOM.js (client side)

-All are imported via inline.dom file * as DOMRenderer
-2 Types(flow)(FiberRoot, Batch) from ReactFiberRoot
-React Dom itself is exported from ReactDOMFB.js where the ReactFiberTreeReflection 
