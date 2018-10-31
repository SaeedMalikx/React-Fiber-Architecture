

Just another one of my **Self-Study Projects** . I don't tend to make such projects public but this might be of help for someone looking to understand Fiber and React Under the hood. Do remember as of writing this, this is my first time looking at the code for Fiber.

Often times reading a large amount of code can be very intimidating for someone new to programming, so it's best to read each line and document them if needed so you don't go hopping around files searching for that lost variable or get lost on the Import tags. There is no set rules or anything here, just documenting/laying it out the best I can. Any part of this doc might change as my understanding of the whole system grows. 

In order to properly use this, clone it and uncheck all the files and then go through the files one by one and adjust the pseudocode to your liking. Remember it took them 2+ years to make Fiber so be patient as it might take you a while. 

**As for obvious errors and such:**

> “There are no mistakes, only happy accidents.” Bob Ross



**Lot of this comes directly from Fiber codebase itself, a lot of it will be just code reorganized for easier understanding. I Turn a lot of functions into Pseudocode for easier reference and understanding**

Here is the link for the official codebase [REACT](https://github.com/facebook/react/tree/master/packages/react-reconciler)

### Files So Far


- [x] ReactFiber.js > Defining Functions For Creating Fibers
- [x] ReactCurrentFiber.js > Functions for Describing Fibers
- [ ] ReactDebugFiberPerf > probably for debugging DEV
- [x] ReactChildFiber > The Bulk of The Child Reconciliation
- [x] ReactClassComponent.js > Class mounting, class instances, class reconciliation
- [x] ReactContext.js > Probably context stuff
- [x] ReactFiberHostContext > Host Stuff
- [ ] ReactFiberHydrationContext.js > Must Understand the Fast and Furious DOM
- [x] ReactFiberPendingPriority.js > Priority, marks levels for various stuff
- [x] ReactFiberRoot.js > For creating Fiber Roots
- [x] ReactFiberExpirationTime.js > How expiration time is handled
- [ ] ReactProfilerTimer.js > For Recording/Tracking Time React Profiler
- [ ] ReactFiberTreeReflection.js > Finding Where the host fibers are...nature 
- [x] ReactFiberStack.js > Concerns how the "stack" is shifted
- [ ] ReactFiberScheduler > THIS IS SOME GUCCI CODE
- [x] ReactUpdateQueue.js > Scheduling The Updates(or I think so)
- [x] ReactFiberBeginWork.js > This is for begining the work
- [x] ReactCompleteWork.js > CompleteWork
- [x] ReactUnwindWork.js > Unwind
- [ ] ReactCommitWork.js > Committing Work
- [x] ReactFiberReconciler.js >  The Main File
- [ ] REACT-DOM.JS > Where the above files end making sense
- [ ] Scheduler.js > located in a different package > concerns RequestAnimationFrame

-----------------

### Note on Algorithms Used

I thought there would be many complicated algorithms used since it took 2+ years to build. 
These Are All the ones I noticed and are quite simple

```

SinglyLinked Lists
Simple Stacks (push) (pop) are used based on a index/cursor.
The algorithms used for reconciling children are equivalent of going through a JSON file. 
Simply mapping it out, maybe run some while loops on the childrens until all are reconciled. 
Building a clone  then see what changed and perform replacements.
Navigating through the stack in reverse or forward direction

```


----------------


## What Is A Fiber and How Are They Created?

> A Fiber is work on a Component that needs to be done or was done. There can be more than one per component.

It is essentially an object with set properties used to identify what kind of a (component) it is and what works needs to be done. Assign prioirity levels for efficient processing. Since it uses flow, it is also exported as a type. 

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

### How Assigning Priority Works

ReactFiberPendingPriority.js

Most of these functions just if there is work to be done and compare it to expirationTime and Mark different priroity.
Looks complicated but once you read it a few times, it's quite simple. The abbreviations should help in remebering it cause putting the whole word just confuses the brain for like no reason. 

```
// abbreviations

expirationTime = 		ET
erroredExpirationTime =		eET		
earliestPendingTime = 		ePT
latestPendingTime = 		lPT
earliestRemainingTime = 	eRT
earliestSuspendedTime = 	eST
latestSuspendedTime = 		lST
pingedTime =			Pi
latestPingedTime =		lPi
renderExpirationTime = 		rET

markPendingPriorityLevel(root, expirationTime)
	// Update the latest and earliest pending times
	
	const ePT is root's ePT
	if NoWork then roots ePT is the ET
		else if ePT > ET then root's ePT is ET
			else 
				const lPT is root's lPT
				if lPT < ET then root's lPT is ET
					
	findNextExpirationTimeToWorkOn(expiration, root)

// This functions main purpose is to findNextExpirationToWorkOn 
// but if certain conditions on the root are present then markPendingPriorityLevel

markCommittedPriorityLevels(root, earliestRemainingTime)
	
	if eRT = NoWork then clear root's ePT, lPT, eST, lST, lPi 
		and findNextExpirationTimeToWorkOn(NoWork, root)

	const lPT is root's lPT
	if lPT !== NoWork
		if lPT < eRT then root's has no work
		else 
			const ePT is root's ePT
			if ePT < eRT then root's ePT = lPT

	
	const eST is root's eST
	if eST has NoWork then 
		markPendingPriorityLevel
		findNextExpirationTimeToWorkOn
	
	const lST is root's lST
	if eRT > lST then root's eST, lST, lPi = NoWork
		markPendingPriorityLevel
		findNextExpirationTimeToWorkOn
	
	if eRT < eST 
		markPendingPriorityLevel
		findNextExpirationTimeToWorkOn

	
	findNextExpirationTimeToWorkOn

hasLowerPriorityWork(root, erroredExpirationTime)
	assigns const lPT, lST, lPI to Root's counterparts 
		returns a true by checking if any of the above const have NoWork 
			and any of time are greater then the eET
	// thus assigning it low priroity value

isPriorityLevelSuspended(root, expirationTime)
	const eST, lST = root's eST, lST
	return true if eST != NoWork and ET >= eST and ET <=lST

markSuspendedPriorityLevel(root, suspendedTime)
	clearPing(root, suspendedTime) //this function checks if the root was pinged during render phase
	
	// This is a same function as markCommittedPriorityLevels but with ST and without marking priority levels

	findNextExpirationTimeToWorkOn(suspendedTime, root)

markPingedPriorityLevel(root, pingedTime)
	const lPi is root's lPi
	if lPi has NoWork or lPi < pingedTime then root's lPi is Pi
	findNextExpirationTimeToWorkOn

findEarliestOutstandingPriorityLevel(root, renderExpirationTime)
	let eET = rET
	const ePT, eST = root's ePT, eST
	if NoWork and eST < eET then eET is ePT
	return earliestExpirationTime;

didExpireAtExpiration(root, currentTime)
	root's nextExpirationTimeToWorkOn = currenTime


findNextExpirationTimeToWorkOn(completedExpirationTime, root)
	const eST, lST, ePT,lPi = root's counterparts

	// Work on the earliest pending time. Failing that, work on the latest pinged time
  	
	if nextExpirationTimeToWorkOn = NoWork then it = lST
	let ET = nextExpirationTimeToWorkOn
		if NoWork then Et is eST

	root.nextExpirationTimeToWorkOn = nextExpirationTimeToWorkOn;
  	root.expirationTime = expirationTime;

```
----------------

### How The Update Queue Works

**ReactUpdateQueue**

This is a very interesting file because it creates not one but 2 queues. There is a main function **enqueueUpdate** which is responsible for creating those two queues. It has a very nice system, it checks if queue1 is present based on the Fiber's updateQueue. If the queue1 is null then it creates the queue1. The interesting part is that it creates a clone of the same queue obased on the fiber.alternate updateQueue. There is a function for creating the queue **createUpdateQueue**. There are multiple if within the enqueueUpdate such as if both queues are null then create both and if either is null then create clone of the using cloneUpdateQueue. It gets even more interesting because there is a function called **appendUpdateToQueue** which appends the updates to both queues. During processing of the update queue, there is a inner function which checks if the WIP queue is a Clone.

The UpdateQueue, also exported as a type. Intial values listed

```
    baseState,
    firstUpdate: null,
    lastUpdate: null,
    firstCapturedUpdate: null,
    lastCapturedUpdate: null,
    firstEffect: null,
    lastEffect: null,
    firstCapturedEffect: null,
    lastCapturedEffect: null,

```

The update type 

```
  expirationTime: ExpirationTime,

// Tags below
  tag: 0 | 1 | 2 | 3,
  payload: any,
  callback: (() => mixed) | null,

  next: Update<State> | null,
  nextEffect: Update<State> | null,

```


The exported constants for reference

```

 UpdateState = 0;
 ReplaceState = 1;
 ForceUpdate = 2;
CaptureUpdate = 3;


```
One of the coolest functions is **getStateFromUpdate**, which works a lot like Redux.

**getStateFromUpdate** 

```

parameters(workInProgress, queue, update, prevState, nextProps, instance)
	switch case depending on the update.tag
		ReplaceState 
			const payload = update.payload
			if the payload is a 'function' then return a payload.call(instance, prevState, nextProps)
			return payload
		CaptureUpdate
			WIP effectTag
		UpdateState
			const payload
			let partialState
			if payload is a 'function' the partialState = payload.call as above
			 else partialState is payload
			if partialState is null or undefined then return the prevState
			
			return Object.assign({}, prevState, partialState)
			
	// I wonder if it's possible to put a global state here and natively implement Redux
		
		ForceUpdate > hasForceUpdate > return prevState
	return prevState
```

**commitUpdateQueue**

```
parameters(finishedWork, finishedQueue, instance, renderExpirationTime)
	check is the finishedQueue.firstCapturedUpdate isn't null
		if finQeue's lastUpdate isn't null then lastUpdate.next is the firstCapturedUpdate 
			and the lastUpdate is lastCapturedUpdate
		firstCapturedUpdate > lastCapturedUpdate > null
	commitUpdateEffects then set 1stEffect > LastEffect > null
	commitUpdateEffects then set 1stCapturedEffect > lastCapturedEffect > null

commitUpdateEffects(effect, instance) 
	while callback hell
```


-----------------------

### How Context Is Handled

**ReactFiberContext.js**

Most of these functions from the looks of it just seem to be concerned with getting the context from the Fibers. 

There seems to be a general pattern emerging. 
Which is get something from a stack, store/cache, reconcile and then put it back in the stack. 

So there is a function for getting the context from the current WIP fiber and acting on it. 

**getUnmaskedContext** and **getMaskedContext** return the context.

**cacheContext** is reponsible for storing the context and assigning it to the WIP.stateNode

**hasContextChanged** returns a boolean and checks if work was performed while it's in the Stack

The following 4 functions manage the Push Pop actions on the stackCursor

```

popContext
popTopLevelContextObject
pushTopLevelContextObject > double push to contextStack and didWIP
pushContextProvider > double push using previousContext

```

**processChildContext**

```

parameters(fiber, type, parentContext)
	const instance = fiber.stateNode;
  	const childContextTypes = type.childContextTypes;
	let childContext
	get the instance of the child context using the stop/start PhaseTimers
	return {...parentContext, ...childContext};
```

There is a function **invalidateContextProvider** that checks if change occured, then processChildContext and perform a the replacement on the stack using pop > pop and then push > push. If no change then a simple pop push.

**findCurrentUnmaskedContext**

```

parameters(fiber)
	// assigns a node variable and checks the tag
	// lazy component uses getResultFromResolvedThenable comes from shared file ReactLazyComponent
	do 
		switch depending on tag
			HostRoot > return stateNode.context
			ClassComponent > using isContextProvider > return MemoizedMergedChildContext
			ClassComponentLazy > same as ClassComponent but Lazy
		

```

Pretty much this files is much like other files and it performs a switch with WIP fibers in the stack and change/replace them as needed. 

**ReactFiberHostContext.js**


This files is very similar but the main difference I'm seeing is that it creates a container for the Host root. 

Note: Saw this in Reconciler.js

```

Initiliaze the cursors for the Stack

requiredContext(c, Value)
	return c

getRootHostContainer > gets the requiredContext for the rootInstance

pushHostContainer(fiber, nextRootInstance)
	// Push current root instance onto the stack;
	 push rootCursor, > push contextCursor > push NO_CONTEXT
	get HostContext > pop context > push nextRootContext

popHostContainer > pop cursors x 3

getHostContext > using requiredContext get current cursor context

pushHostContext(fiber) > double push on to the stack, the current and next

// Do not pop unless this Fiber provided the current context.
// pushHostContext() only pushes Fibers that provide unique contexts.

popHostContext(fiber) > double pop on the stack and fiber

```
-----------
### How The Work is Handled.

**READ BELOW ONLY IF YOU HAVE READ THE ABOVE, as these work files will easily make sense if the inner parts are understood.**

This file mostly as stated by the name begins the work on various parts. 

```

reconcileChildren(current, workInProgress, nextChildren, renderExpirationTime)
	if there is no work being done on the current 
		mountChildFibers(workInProgress, null, nextChildren, renderExpirationTime)
	else reconcileChildFibers

forceUnmountCurrentAndReconcile(current, workInProgress, nextChildren, renderExpirationTime)
	on the workInProgress.child run reconcileChildFiber x2

updateForwardRef(current, workInProgress, type, nextProps, renderExpirationTime)
	if it doesn't have a hasLegacyContextChanged and memoizedProps=nextProps 
		if ref = current then bailOut

	reconcileChildren
	memoizProps and return the WIP.child

updatePureComponent(current, workInProgress, Component, nextProps, updateExpirationTime, renderExpirationTime)
	 an if bailoutconditions
	
	prepareToReadContext > reconcileChildren > memoizProps > return WIP.child

updateFragment(current, workInProgress, renderExpirationTime)
	same as pure but remember that fragments and the lack of context

updateMode(current, workInProgress, renderExpirationTime)
	same but for pendingProps.Children

updateFunctionComponent(current, workInProgress, Component, nextProps, renderExpirationTime)
	same as but getUnmaskedContext and getMaskedContext before reconciling

updateClassComponent(current, workInProgress, Component, nextProps, renderExpirationTime)
	
	prepareToreadContext
	if the current WIP.stateNode is null(not being worked on)
		then constructClassInstance and mountClassInstance
		else resumeMountClassInstance
	else updateClassInstance since it's already being worked on
	
	return finishClassComponent

finishClassComponent(current, workInProgress, Component, shouldUpdate, hasContext, renderExpirationTime)
	markRef for the current WIP
	an if bail condition
	
	let nextChildren be null if there is no nextChildren, otherwise render the nextChild

	if the current isn't null then forceUnmountCurrentandReconcile otherwise reconcileChildren

	memoizeState and Props > return WIP.child

pushHostRootContext(workInProgress)
	if it has a pendingContext then pushTopLevelContextObject with a true value, otherwise if the root has only context then set it to false
	pushHostContainer

updateHostRoot(current, workInProgress, renderExpirationTime)
	pushHostRootContext
	assign variables to WIP.props/state and processUpdateQueue
	if nextChildren = prevChildren then restHydrationState and bail out
	otherwise hydrate it > mountChildFibers
	and ofcourse reconcileChildren > return WIP.child

updateHostComponent(current, workInProgress, renderExpirationTime)
	pushHostContext 
	assign WIP.type/pendingProps/memoizedProps
	markRef > Deprioritize the subtree if needed > reconcile > memoizeProps > return WIP.child

updateHostText > just updater for above host functions

resolveDefaultProps(Component, baseProps)
	using object.assign and a for let in loop resolve the props taken from ReactElement
	return baseProps

mountIndeterminateComponent(current, workInProgress, Component, updateExpirationTime, renderExpirationTime)
	const props = workInProgress.pendingProps
	if it's 'object' and !==null, and 'function'
		cancelWorkTimer > readLazyComponentType > startWorkTimer
	switch (resolvedTag) 
		FunctionComponentLazy > updateFunctionComponent
		ClassComponentLazy > updateClassComponent
		ForwardRefLazy > updateForwardRef
		PureComponentlazy > updatePureComponent
	return WIP.props > child

	assign variables and get unmasked and masked contexts > prepareToReadContext
	if value is an 'object' and !null and value.render(function and undefined)
		then get DerivedStateFromProps > applyDerivedStateFromProps if needed 
		adoptClassInstance > mountClassInstance > return finishClassComponent
	otherwise reconcile > memoizProps > return WIP.child

updateSuspenseComponent(current, workInProgress, renderExpirationTime)
	check if it's currently being worked on the component has timed out 
	if it's being worked on the forceUnmountCurrentAndReconcile else reconcileChildren
	WIP.memoized Props/State to the nextProps and DidTimeout > return WIP.child
		
updatePortalComponent(current, workInProgress, renderExpirationTime)
	pushHostContainer
	if it isn't being worked then assign and reconcileChildFibers, else is same and > memoize > return

updateContextProvider
	same as above function but newProps/oldProps = memoized
	pushProvider(workInProgress, newValue)

	if oldProps are present, bail out otherwise > propagateContextChange
	reconcile

updateContextConsumer
	prepareToReadContext > assign new values > reconcile > return WIP.child

// beginWork just uses the above function depending on the WIP.tag 
// Keep the expiration Time in mind as well	
// remember that purecomponents/lazy components have thenable functions, and just follows the same methods for pretty much all the cases.

```
	
	
### Complete Work

This file literally just completes what was started by BeginWork. The beginwork sets the pieces in motion and then CompleteWork acts on the pieces. 

**markUpdate** and **markRef** just tag the fibers. 

**appendAllChildren** runs a while loop on the childrens until all the siblings are accounted for. There is an inner function called **appendInitialChild** which sets the loop in motion. 

```

if the work is mutable then 
	updateHostContainer 
	updateHostComponent > get the context > prepare the update > if it has a payload > markUpdate
	updateHostText > markUpdate
elif persists 
	appendAllChildrenToContainer(containerChildSet, workInProgress) > run a while loop on child and siblings
	updateHostContainer > create the container > append > mark > finalizeContainerChildren with the newChildSet
	updateHostComponent > switch old props with memoized and create a new instance > get the host context > prepareUpdate > clone > finalizeInitialChildren > markUpdate	
	updateHostText > if old !== newText then get host container, context > create an instance > markUpdate
else literally do nothing

// The Main Function

completeWork(current, workInProgress, renderExpirationTime)
	switch depending on Tag
		all the functions are same for the Work Tags with little variation
		pop From Container
		Update The Container 

// Remember not all functions have these container...pure/fragments

```

There is no need to go over the entire **ReactFiberUnwindWork** when all it's doing is just setting the values(container/whatever was acted upon in beginwork) to null by popping them from where they were put in and gets ready for the next rinse and repeat
There is also functions for throwing error for various conditions based on the catch error boundary which is part of the new api in React
I will explain this in detail when I get the chance but if you have made it this far then you should automatically know what should happen here.


------------
CommitWork
------------


**ReactFiberReconciler.js**

This file is in charge of creating the container, and scheduling the whole Fiber Update process. 

```

// 0 is PROD, 1 is DEV For BundleType

getContextForSubtree(parentComponent)
	if null then return emptyContextObject

	Get the map of the fiber instance, and findCurrentUnmaskedContext(fiber)
	if the fibers tag is ClassComponent then processChildContext, if it's lazy do the whole thenable and process context

	return parentContext

scheduleRootUpdate(current, element, expirationTime, callback)
	create > enqueue > scheduleWork > return expirationTime
	

updateContainerAtExpirationTime(element, container, parentComponent, expirationTime, callback)
	get the current container, and contextForSubTree
	if context is null then return context else set it to pending
	return schedule RootUpdate

findHostInstance(component)
	get the hostFiber using findCurrentHostFiber and return it's stateNode

createContainer(containerInfo, isConcurrent, hydrate) > return the created Fiber Root

updateContainer(element, container, parentComponent, callback)
	on the current container > computeExpirationForFiber and return updateContainerAtExpirationTime

getPublicRootInstance(container)
	get the current container and depending on it's HostComponent get it's public Instance, which is the child.stateNode


```

it's creating a container for the current component/whatever(get Fibers and all for the root)


and then scheduling an update at Root, 

at this stage setState should trigger, which ends up scheduling the work assigning expirationTimes. 

Scheduling The Work will start the process of 

```
Begin > CompleteWork > UnWind > Commit

Where it does work such as Reconciling Child Fibers, context, and etc. 

```

Update the component/whatever, rinse and repeat



---------------

**Scheduler.js**

Check Out This Repo

https://github.com/spanicker/main-thread-scheduling

----------------
## REACT-DOM AND HOW IT WORKS

---------------------

### Something You May Have Realized. If You Actually Went Through The Files..... 




