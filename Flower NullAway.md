Flower NullAway feature is intended to prevent `NullPointerException`s by checking at initialization time that all input parameters are either using initialized value or marked as nullable.
In Flower the following types of input parameters exist: `@In`, `@InFromFlow`, `@InOut`, `@InRet`, `@InRetOrException`.

The following are considered value initializers:
- `final` Flow fields - initialized in constructor;
- `@InOut`, `@Out` - field initialized by function output;
- `@StepFunction(returnTo=...)` or `@StepCall(returnTo=...)` - field initialized by return value of step function.

The following are NOT considered value initializers:
- `@InOut(out=OPTIONAL)`, `@Out(out=OPTIONAL)` - output is optional;
- `@Nullable @StepFunction(returnTo=...)` or `@Nullable @StepCall(returnTo=...)` - function can return null.

Note that once a field is initialized, it can't be nullified. It means that if `@InOut(out=OPTIONAL)`, `@Out(out=OPTIONAL)` do not set return value or return `null`, the corresponding Flow Field is not altered. The same is true for a `@Nullable` function with `returnTo` that returns `null`.

The following parameters are Non-Nullable and must be set from initialized sources:
- `@In` `Type` `param` - field must be initialized;
- `@InOut` `InOutPrm<Type>` `param` - field must be initialized;
- `@InRet` `Type` `param` - step function's return value can NOT be @Nullable.

Please note that `@InFromFlow` must always be accompanied by `@Nullable`, otherwise an initialization Exception will be raised.

The following parameters are `Nullable` and don't have to be set from initialized sources:
- `@Nullable @In` `Type` `param` - field doesn't have to be initialized;
- `@Nullable @InFromFlow` `Type` `param` - in this case Flow field doesn't have to be initialized or even exist;
- `@InOut` `NullableInOutPrm<Type>` `param` - field doesn't have to be initialized;
- `@Nullable @InRet` `Type` `param` - step function's return value can be `@Nullable`;
- `@InRetOrException` -

The parameter used with `@InRetOrException` is defined by the following interface:
```
public interface ReturnValueOrException<T> {  
  Optional<T> returnValue();
  Optional<Throwable> exception();
}
```
Since `returnValue` is `Optional`, it naturally supports nulls returned from `@Nullable` StepFunctions, assuming empty value. In case when StepFunction returned `null`, both `returnValue()` and `exception()` are empty.


An initialized field is a field that's initialized on ALL execution paths leading from the FirstStep to the function.
Similar approach is employed with EventProfiles, the order of execution assumed as the following:
```
  BEFORE_FLOW,
  BEFORE_STEP,
  BEFORE_STEP_ITERATION,
  BEFORE_EXEC,
  AFTER_EXEC,
  BEFORE_TRANSIT,
  AFTER_TRANSIT,
  AFTER_STEP_ITERATION,
  AFTER_STEP,
  AFTER_FLOW
```
A special case with Event Profiles is `FLOW_EXCEPTION`. In this case the corresponding function can use fields initialized in `BEFORE_FLOW`, `BEFORE_STEP`, `BEFORE_STEP_ITERATION`, `BEFORE_EXEC`, but due to specifics of this exception we don't consider any fields initialized by the function itself as initialized in any other functions.

Event functions may fail without failing the flow. If EventFunction is not SYNCHRONIZED_BREAKING concurrency

If a parameter is uninitialized and is not explicitly marked as nullable, Flower NullAway feature raises initialization exception.
Current implementation for EventFunctions' `@InFromFlow` mandates the parameter to be annotated as `@Nullable`. The idea is to handle the following situations:
1) Flow state field is not initialized;
2) Flow doesn't have a corresponding field.
   In both case a null/default value will be passed as a parameter.






---
Disclaimer:
The above leads to a not so obvious scenario when both `returnValue()` and `exception()` are empty, which happens if there was no Exception raised in StepFunction, and it returned `null`.
The best "rule of thumb" in this case is to determine success or failure of a corresponding StepFunction by testing that `exception()` is not empty, because `returnValue()` being empty is ambiguous.

I'm leaving this as is for now, but probably some ideas will emerge in the future on how to make this structure less ambiguous, while being significantly more elegant than something like:
```
----,,,THIS IS BAD,,,-----
public interface NullableReturnValueOrException<T> {  
  @Nullable Optional<T> returnValue();
  Optional<Throwable> exception();
}
----^^^THIS IS BAD^^^-----
```
The problem statement here is that the structure should be concise and elegant, while capable of unambiguously expressing 3 distinct states:
1. `exception` is present, `returnValue` is not present - Exception was raised;
2. `exception` is not present, `returnValue` is present - successful return from a non-Nullable function;
3. `exception` is not present, `returnValue` is not present (null) - successful return of null value from a Nullable function or void function.
   At the same time, the structure should be compatible with regular NullAway, and expressible in Nullable and non-Nullable way, similarly to `InOutPrm` and `NullableInOutPrm`, to be further supported in Flower NullAway.

Before such solution is found, we will support the existing structure as always Nullable, despite small shortcomings in such approach.