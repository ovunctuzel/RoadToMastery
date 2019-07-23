
<p align="center">
  <a href="index.html">Programming</a> |
  <a href="unreal.html">Unreal</a> |
  <a href="misc.html">Misc</a> |
  <a href="soon.html">Coming Soon</a>
</p>

* * *
### Unreal
* [Replication Tips](#replication-tips)
* [Delegates](#delegates)
* [Reflection](#reflection)
* [Avoid Optimizations](#avoid-optimizations)
* * *

## Unreal
### Replication Tips
We can replicate variables using ReplicatedUsing or Replicated tags. bReplicates must be set to true in the constructor.
```
	UPROPERTY(Replicated)
	bool RepVar = false;
 
	UPROPERTY(ReplicatedUsing = OnRep_ReadyForIMUSolveInit)
	bool RepVar2 = false;
```
Whenever the value of RepVar changes, the change propagates to clients. Whenever the value on RepVar2 changes, the change propagates to clients and OnRep_ReadyForIMUSolveInit function is called. 

```
void USomeComponent::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);
	DOREPLIFETIME(USomeComponent, RepVar);
	DOREPLIFETIME(USomeComponent, RepVar2);
}
```
The above function must be overriden for proper replication of variables.

### Delegates
**From Unreal Wiki (Rama):** A delegate is basically an event that you can define and call and respond to. Every time the event is fired off, anyone who is listening for this event will receive it and be able to take appropriate action. In the case of multicast delegates, any number of entities within your code base can respond to the same event and receive the inputs and use them. In the case of dynamic delegates, the delegate can be saved/loaded within a Blueprint graph (they're called Events/Event Dispatcher in BP).

#### Example
Declare the delegate in header file.
``` DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnChoiceReceived, int, id); ```

We can expose the delegate to blueprints like this:
``` 
UPROPERTY(BlueprintAssignable)
FOnChoiceReceived OnChoiceReceived;
```
The blueprint can then bind to this delegate using any function with the same signature.
Whenever we call something like ``` OnChoiceReceived.Broadcast(152) ``` all subscribers to OnChoiceReceived will be notified.

In C++ we can bind to a delegate like this:
```
OnChoiceReceived.AddDynamic(this, &USomeClass::RespondToChoice);
```
Use ``` AddUObject ``` for non-dynamic delegates, and use ``` BindUObject ``` for non-multicast delegates.

### Reflection
Details coming soon...

Reflection is the ability of a program to examine itself at runtime. This is hugely useful and is a foundational technology of the Unreal engine, powering many systems such as detail panels in the editor, serialization, garbage collection, network replication, and Blueprint/C++ communication. However, C++ doesn’t natively support any form of reflection, so Unreal has its own system to harvest, query, and manipulate information about C++ classes, structs, functions, member variables, and enumerations.

https://www.unrealengine.com/en-US/blog/unreal-property-system-reflection?sessionInvalidated=true

Two ways of getting a property on a class:
```
UMulticastDelegateProperty* delProp = FindField<UMulticastDelegateProperty>(convo->GetClass(), FName("TestDelegate"));
if (delProp != NULL)
{
	FMulticastScriptDelegate del = delProp->GetPropertyValue_InContainer(convo);
	del.ProcessMulticastDelegate<AVoidConversation>(nullptr);
}
```
```
for (TFieldIterator<UProperty> PropIt(convo->GetClass()); PropIt; ++PropIt)
{
	UProperty* Property = *PropIt;

	if (Property->GetName().Equals("TestDelegate"))
	{
		UMulticastDelegateProperty* deleg = Cast<UMulticastDelegateProperty>(Property);
		FMulticastScriptDelegate* a = deleg->ContainerPtrToValuePtr<FMulticastScriptDelegate>(convo);
		a->ProcessMulticastDelegate<AVoidConversation>(nullptr);
	}
}
```
Both approaches do the same thing. If we know the type of the property, we can cast it to a subclass such as UFloatProperty, or UDelegate property. Then we can get its value using ContainerPtrToValuePtr (returns pointer), or GetPropertyValue_InContainer (returns value). In the above example, the delegates are invoked using ProcessMulticastDelegate. Normally Broadcast is used instead of ProcessMulticastDelegate, but since blueprints automatically generate delegate types, the base class had to be used. 

### Avoid Optimizations
C++ style: Wrap a function between these keywords to avoid optimizations for a block of code:

```
#pragma optimize("", off)
// Some Code
#pragma optimize("", on)
```

Unreal: Using "DebugGame Editor" instead of "Development Editor" also compiles debugging symbols.
