

Just another one of my **Self-Study Projects** . I don't tend to make such projects public but this might be of help for someone looking to understand Fiber and React Under the hood. Do remember as of writing this, this is my first time looking at the code for Fiber.

Often times reading a large amount of code can be very intimidating for someone new to programming. So it's best to read each line and document them if needed so you don't go hoping around files searching for that lost variable or get lost on the Import tags. There is no set rules or anything here, just documenting/laying it out the best I can. Any part of this doc might change as my understanding of the whole system grows.  

**As for obvious errors and such:**

> “There are no mistakes, only happy accidents.” Bob Ross


**Lot of this comes directly from Fiber codebase itself, a lot of it will be just code reorganized for easier understanding** 
**I Turn a lot of functions into Pseudocode for easier reference and understanding**

Here is the link for the official codebase [REACT](https://github.com/facebook/react/tree/master/packages/react-reconciler)

### Files So Far


- [x] ReactFiber.js > Defining Functions For Creating Fibers
- [x] ReactCurrentFiber.js > Functions for Describing Fibers
- [ ] ReactDebugFiberPerf > probably for debugging DEV
- [ ] ReactChildFiber > The Bulk of The Child Reconciliation
- [ ] ReactClassComponent.js > Class mounting, class instances....
- [ ] ReactContext.js > Probably context stuff
- [ ] ReactFiberHostContext > Host Stuff
- [ ] ReactFiberHydrationContext.js > Must Understand the Fast and Furious DOM
- [ ] ReactFiberPendingPriority.js > Priority, marks levels for various stuff
- [x] ReactFiberRoot.js > For creating Fiber Roots
- [x] ReactFiberExpirationTime.js > How expiration timer is handled
- [ ] ReactProfilerTimer.js > For Recording/Tracking Time(expiration, commits) for React Profile
- [ ] ReactFiberTreeReflection.js > Finding Where the host fibers are...nature 
- [x] ReactFiberStack.js > Concerns how the "stack" is shifted
- [ ] ReactFiberScheduler > THIS IS SOME GUCCI CODE
- [ ] ReactUpdateQueue.js > Scheduling The Updates(or I think so)
- [ ] ReactFiberBeginWork.js > This is for begining the work
- [ ] ReactCommitWork.js > Committing Work
- [ ] ReactCompleteWork.js > CompleteWork
- [ ] ReactUnwindWork.js > Gotta unwind the work once its completed
- [ ] ReactFiberReconciler.js >  The Main File
- [ ] REACT-DOM.JS > Where the above files end making sense
- [ ] Scheduler.js > located in a different package > concerns RequestAnimationFrame

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
createHostRootFiber 
```

Function **assignFiberPropertiesInDev** : Used for stashing WIP properties to replay failed work in DEV.

----------------

### What Are Root Fibers?

**ReactFiberRoot.js**

The Host Fiber and it has the following properties and is created by **createFiberRoot**

```
current: The currently active root fiber. This is the mutable root of the tree.

containerInfo: Any additional information from the host associated with this root.

pendingChildren:

earliestSuspendedTime: earliest priority levels that are suspended from committing.
latestSuspendedTime: latest priority levels that are suspended from committing.

earliestPendingTime: earliest priority levels that are not known to be suspended.
latestPendingTime: latest priority levels that are not known to be suspended.

latestPingedTime: The latest priority level that was pinged by a resolved promise and can be retried.

didError: if error thrown
pendingCommitExpirationTime:

finishedWork: A finished work-in-progress HostRoot that's ready to be committed.

timeoutHandle: Timeout handle returned by setTimeout. Used to cancel a pending timeout

context: Top context object, used by renderSubtreeIntoContainer
pendingContext:

+hydrate: Determines if we should attempt to hydrate on the initial mount

nextExpirationTimeToWorkOn: Remaining expiration time on this root.

expirationTime:

firstBatch:
```

----------------

### Expiration Time

**ReactFiberExpirationTime.js**

Contains functions concerning  ExpirationTime and when low priority and high priority Batches. High priority Batches have a longer expiration time than low priority while in DEV mode.

```
1 unit of expiration time represents 10ms

msToExpirationTime > returns ms/10 + 2
expirationTimeToMS > return (expirationTime - 2) * 10

```

High and Low Priority Times in ms

```
LOW_PRIORITY_EXPIRATION = 5000
LOW_PRIORITY_BATCH_SIZE = 250
HIGH_PRIORITY_EXPIRATION = __DEV__ ? 500 : 150
HIGH_PRIORITY_BATCH_SIZE = 100
```

High Priority Uses the Function **computeInteractiveExpiration** with the currentTime. Low priority uses **computeAsyncExpiration**, likely used for offscreen renders. Both use the function **computeExpirationBucket** (currentTime, expirationInMs, bucketSizeMs).

----------------

### Methods For Detecting Fiber In A Stack

**ReactCurrentFiber.js**

NOTE: Recheck when this is used

The main function for describing what the fiber type is **describeFiber**
```
parameters(Fiber)
  switch (depending on the tag)
    IndeterminateComponent
    FunctionComponent
    FunctionComponentLazy
    ClassComponent
    ClassComponentLazy
    HostComponent
    Mode
      name = getComponentName //Function returning a string based on Types(CAP_NAMES)-paramters is type
      if
        ownerName = getComponentName
      return describeComponentFrame(name, source, ownerName)
```

So **describeFiber** is used in a exported function **getStackByFiberInDevAndProd**

```
do 
  info += describeFiber(node);
  while workInProgress
  return info
```
2 exports being the current Fiber and LifeCyclePhase

Maybe add DEV stuff if they are needed

------------


### How The Stack Works

**ReactFiberStack.js**

NOTE: Updates are not sorted by priority, but by insertion; new updates are always appended to the end of the list.

The stack itself seems to be a push pop based on the index(initial value of -1), either increment the index when pushing or decrement when popping.
```
const valueStack []
let fiberStack [] | null

let index = -1

createCursor for assigning where to start looking in the stack

pop(cursor, fiber) 
 if index < 0 return ''
 current cursor = valueStack[index]
 assign null to valueStack since it's removed from stack
 index--

push(cursor, value, fiber)
  index++
  valueStack[index] is the current cursor
  current cursors is assigned the new value

```

----------------

### How The ClassComponent Works(Updated and such)

**ReactFiberClassComponent.js**

this is incharge of handling how the react components are updated:

the outer function incharge of sending props to the internal state is **applyDerivedStateFromProps**. Which handles
Merging  of the partial state and the previous state. Where memoized state is compared with the previous and using methods like Redux object.assign. 
It assigns a new state. There is a if case present that if it's not in a updatequeue then basestate(initial state) = memiozed state. 


The structure of this is a lot like using Redux(payloads, assigning)

There is a main classComponentUpdater which is contains various functions:

```

isMounted 

enqueueSetState(inst, payload, callback)
	get various values on current time and expirationtimes for the given Fiber
	createUpdate
	enqueueUpdate > takes in Fiber and the update which was just created
	scheduleWork > schedule it for work (fiber and the expirationTime)
enqueueReplaceState
	Same as above function but for replacing state
enqueueForceUpdate
	Same as above but for forced updates
```

Function for checking is the component should update **checkShouldComponentUpdate**

```
parameters(workInProgress, ctor, oldprops, newProps, oldState, newState, nextContext)
 const instance is the workinProgress
	if it's function
		startPhaseTimer
		shouldUpdate > also the returned value
		stopPhaseTimer
	if it's a pure component
		return if they !shallowEqual
```
function **adoptClassInstance** 

```

// The instance needs access to the fiber so that it can schedule updates

instance.updater = classComponentUpdater

```

The main function for constructing the current class instance **constructClassInstance**

```
parameters(workInProgress, ctor, props, renderExpirationTime)
	let isLegacyContextConsumer =  false
	let unmaskedContext = emptyContextObject
	let context = null
	contextType = ctor.contextType
		if contextType is an object and true
			context = contextType.unstable_read()
		else 
			unmaskedContext = getUnmaskedContext(workInProgress, ctor, true)
			const contextTypes = ctor.contextTypes
			set isLegacyContextConsumer to (true?) depending on the contextTypes being present
			context is getMaskedContext if isLegacyContextConsumer is true
	
	const instance = new ctor(props, context)
	the new state is equal to the memoizedState if the state of the current instance is not null		
	adoptClassInstance(workInProgress, instance)
	if isLegacyContextConsumer is true then cacheContext(workInProgress, unmaskedContext, context)
	
	return instance

```

Various calls on the Component

```

callComponentWillMount(workInProgress, instance)
	startPhaseTimer
	old is instance state
	depending on the typeof instance call either componentWillMount or UNSAFE_componentWillMount
	stopPhaseTimer
	
	if oldstate !== instance.state 

callComponentWillRecieveProps(workInProgress, instance, newProps, nextContext)
	startPhaseTimer
	old is instance state
	depending on the typeof instance call either componentWillReceiveProps or UNSAFE_componentWillReceiveProps
	stopPhaseTimer
	
	if instance.state !== oldState then enqueueReplaceState(instance, instance.state, null)

```

Once the class instance is constructed then it needs to be mounted with **mountClassInstance**

```

parameters(workInProgress, ctor, newProps, renderExpirationTime)
	const instance = workInProgress.stateNode;
  	instance.props = newProps;
  	instance.state = workInProgress.memoizedState;
  	instance.refs = emptyRefsObject;

	contextType = ctor.contextType
	if object and not null then instance.context does a unstable_read()
	else 
		const unmaskedContext = getUnmaskedContext(workInProgress, ctor, true);
    		instance.context = getMaskedContext(workInProgress, unmaskedContext);
	
	let updateQueue = workInProgress.updateQueue 
		if not null then processUpdateQueue
			and the instance state will equal memoizedState
	if ctor.getDerivedSTateFromProps is true(function) then applyDerivedStateFromProps
		and instance state is the memoizedState 
```
still need to understand these

**resumeMountClassInstance**

**updateClassInstance**

--------------------------

### How Child Fibers Are Reconciled

**ReactChildFiber.js**

There is one main function **ChildReconciler** which does most of the work. There are multiple inner functions that make up this entire function. 

```

parameters(shouldTrackSideEffects)
	
	deleteChild(returnFiber, childToDelete)
		// Deletions are added in reversed order
		Uses effectTag to determine what child to delete
		if there is no nextEffect then null otherwise 
		
	deleteRemainingChildren(returnFiber, currentFirstChild)
		using the currentFirstChild, 
			it uses a while cascades unto next child until all are deleted uding deleteChild
	
	mapRemainingChildren(returnFiber, currentFirstChild)
		// Add the remaining children to a temporary map
		let existing be the current first
		while existing is not null 
			if key present > existingChildren.set(existingChild.key(if no key then index), existingChild)
			existingChild = sibling
		return existingChildren	
	
	useFiber(fiber, pendingProps, expirationTime)
		creates a fiber clone using createWorkInProgress and sets the index to 0 and sibling to null
		
	placeChild(newFiber, lastPlacedIndex, newIndex)
		const current = alternate of the new fiber
		if current !null oldindex is current
			if < lastPlacedIndex newFiber.effectTag is the new Placement
			else return oldIndex
		// This is an insertion.
      		newFiber.effectTag = Placement;
      		return lastPlacedIndex;
				
	placeSingleChild(newFiber) 
	
	updateTextNode(returnFiber, current, textContent, expirationTime)
		if current null then return a new fiber using 
			createFiberFromText
		else update the existing using 
			useFiber
	
	updateElement(returnFiber, current, element, expirationTime)
		if current !== null and has a type then move it based on index
			useFiber
			coerceRef
			return existing fiber with a updated .ref .return
		else insert and create a fiber using 
			createFiberFromElement 
			coerceRef
			return the createdFiber
			
	updatePortal(returnFiber, current, portal, expirationTime)
		if null tag != HostPortal(worktag)
			insert a new createdfiber using createFiberFromPortal and return it
		else update it using useFiber and return it
		
	updateFragment(returnFiber, current, fragment, expirationTime, key)
		if null no tag != fragment
			insert createFiberFromFragment and return it
		else update useFiber
	
	createChild(returnFiber, newChild, expirationTime)
		// Text nodes don't have keys.
		if typeOf 'string' or 'number' return using createFiberFromText
		if typeOf 'object and !==null
			switch depending on newChild.$$typeof
				REACT_ELEMENT_TYPE > createFiberFromElement > coerceRef > return
				REACT_PORTAL_TYPE > createFiberFromPortal > coerceRef > return
				
			if newChild is an array or has iteration > createFiberFromFragment
				else error throwOnInvalidObjectType
		otherwise return null
	
	updateSlot(returnFiber, OldFiber, newChild, expirationTime)
		// Update the fiber if the keys match, otherwise return null and remember text nodes don't have keys
			
			if typeof newChild is 'string' or 'number' > return updateTextNode
				if key !== null > null
			if typeof newChild is 'object' and !== null
				switch depending on newChild.$$typeof
					REACT_ELEMENT_TYPE > updateElement
						if type is REACT_FRAGMENT_TYPE > return updateFragment
					REACT_PORTAL_TYPE > if newchild.key matches > updatePortal 	
				
				if newChild is an array or has iteration > updateFragment
			throw if none of the cases throwOnInvalidObjectType
	
	// The only difference is the newIdx that was created when mapping new children using mapRemainingChildren
	updateFromMap(existingChildren, returnFiber, newIdx, newChild, expirationTime)
		if typeof newChild is 'string' or 'number' > updateTextNode
		 // const matchedFiber = existingChildren.get(newIdx)
		
		if typeof newChild is 'object' and !== null
			switch depending on newChild.$$typeof
				REACT_ELEMENT_TYPE > updateElement
						if type is REACT_FRAGMENT_TYPE > return updateFragment
				REACT_PORTAL_TYPE > if newchild.key matches > updatePortal

	// Not Sure if this is accurate, FIX LATER
	reconcileChildrenArray(returnFiber, currentFirstChild, newChildren, expirationTime)
		using a For Loop this function will go through each children and their children
			it will compare with created map Idx? and alse keep track using a new Idx 
				updateSlot and if there is newFiber break the loop
				placeChild in the correct index
				
		if the newIdx = newChildren.length then deleteRemainingChildren
		if oldFiber is null > createChild > placeChild
		
		// Add all children to a key map for quick lookups.
    		const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
		
		use another For Loop and Keep scanning and use the map to restore deleted items as moves.
		
		finally return resultingFirstChild
	
	reconcileChildrenIterator(returnFiber, currentFirstChild, newChildrenIterable, expirationTime
		as per comments 
		// This is the same implementation as reconcileChildrenArray(),
    		// but using the iterator instead.
	
	reconcileSingleTextNode(returnFiber, currentFirstChild, textContent, expirationTime)
		if currentFirstChild != null and tag matches HostText
			use deleteRemainingChildren > useFiber > return
		deleteRemainingChildren
		createFiberFromText > return it
	
	reconcileSingleElement(returnFiber, currentFirstChild, element, expirationTime)
			while currentFirstChild !== null 
				deleteRemainingChildren > useFiber > coerceRef else deleteChild
				return > child.sibling 
				// repeat until no siblings
			if element.type is REACT_FRAGMENT_TYPE > return createFiberFromFragment
			else createFiberFromElement > coerceRef > return
	
	reconcileSinglePortal(returnFiber, currentFirstChild, portal, expirationTime)
		same as above SingleElement but Portals
	
	reconcileChildFibers(returnFiber, currentFirstChild, newChild, expirationTime)
		typeof newChild === 'object' and !== null;
			REACT_ELEMENT_TYPE > placeSingleChild(reconcileSingleElement)
			REACT_PORTAL_TYPE > placeSingleChild(reconcileSinglePortal)
		if typeof newChild is 'string' or 'number' > placeSingleChild(reconcileSingleTextNode)
		if newChild isArray > reconcileChildrenArray
		if newChild isiterable > reconcileChildrenIterator
		
		
		deleteReamainingChildren
	
	finally it returns reconcileChildFibers

cloneChildFibers(current, workInProgress)
	creates a newchild by using createWorkInProgress(currentChild, currentChild.pendingProps, currentChild.expirationTime)
	workInProgress.child = newChild
	
	Run a while loop on the children and go to next chid, clone it until none
	
```
Even though this function is MASSIVE, all it really does is implement Georgia Bush's NO CHILD LEFT BEHIND POLICY

----------------
### Fiber Files that are exported to the React-DOM.js (client side)

-All are imported via inline.dom file * as DOMRenderer

-2 Types(flow)(FiberRoot, Batch) from ReactFiberRoot

-React Dom itself is exported from ReactDOMFB.js 

-The rest of DOM files are just various HTML VDOM

----------------

**Scheduler.js**

Holy crap it's using requestAnimationFrame

----------------
## REACT-DOM AND HOW IT WORKS

Rough Understanding of how this all works based on Baking Fiber COOKIES

Straight up preparing a batch of chocolate chip cookies and peanut butter chip cookies. 

Putting it in the oven 

Modifying the chocolate chips cause setState has decided the chips are not good enough but it also doesn't like the peanut butter cookies. 

Ok lets update the chocolate chips cause everyone will see them first cause chocolate chips cookies are high priority and since we put the peanut butter cookies all the way in the back since they suck at Dota and always stuck in low priority, we can change the peanut butter chips right before everyone notices them. 

Commit the chips change and

turns out we used the wrong tray(class components) so now somehow the cookies are floating in magic air and tray needs to be changed.

TING............Delicious Cookies


