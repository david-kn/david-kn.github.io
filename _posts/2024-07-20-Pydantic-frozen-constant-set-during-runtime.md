---
title: "Pydantic: Model with a constant attribute / frozen value after its first set"
date: 2024-07-20
tags: python
---

**Issue description:**
I was trying to define a model with a frozen (constant) value in Pydantic once. To provide a bit more real-world context: I was loading larger data into models where some attributes are known right away, some are loaded later on the fly, some are calculated based on others and some of those data becomes constant when set (as they for example determine the what way the individial object should be handled in the next steps and it can't be altered). The tricky part starts when some value is not known during initialization...

**Desired state**:
To keep a constant value within an object and do not allow any further change of such value. Any attempt to modify such an attribute should raise an error / exception.

**Solution:**

I found several ways how to implement this behaviour although none of these was the one I would really expect:

Altogether with a necessary `validate_assignment=True` configuration on a model following model works and prevents from attribute being modified after being once set. So here comes the basic setup upon which we will deny further modification:

```
class AppType(str, Enum):
    APP_GROUP = "APP_GROUP"
    APP = "APP"

class AppCondition(BaseModel):
    model_config = ConfigDict(validate_assignment=True)

    name: str
    object_type: Optional[AppType] = None
```


### Knowing constant value from the very beginning
## ClassVar as a class constant
1/ Use a `ClassVar` that defines a value as the class value and should not be altered within an instance. Works great when you know the value from the beginning and you can set it during `__init__`.
```
class ClientTypeValues(NameStruct):
    attribute: Optional[str] = None
    object_type: ClassVar[str] = "CLIENT_TYPE"
```

If trying to modify such value, following error is raised: _AttributeError: 'object_type' is a ClassVar of `ClientTypeValues` and cannot be set on an instance. If you want to set a value on the class, use `ClientTypeValues.object_type = value`_

-----------
## Use _frozen_ immutable attrubute configuration
2/ Use a `Field` attribute `frozen=True` that together with setting a default value doesn't allow any further changes. There are two ways how to define such an field attribute - either via `Annotated` or without.
```
class Policy(BaseModel):
    object_type: Optional[str] = Field("DEFAULT_VALUE", frozen=True)
    name: Optional[str] = None
```
or alternative syntax with the same behavior
```
class Policy(BaseModel):
    object_type: Annotated[str, Field(frozen=True)] = "DEFAULT_VALUE"
    name: Optional[str] = None
```


If an attempt to change the value is made, a following exception is raised:
```
pydantic_core._pydantic_core.ValidationError: 1 validation error for Policy
object_type
  Field is frozen [type=frozen_field, input_value='DEFAULT_VALUE', input_type=str]
    For further information visit https://errors.pydantic.dev/2.8/v/frozen_field
```
