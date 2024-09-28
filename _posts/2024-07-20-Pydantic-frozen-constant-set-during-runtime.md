---
title: "Pydantic: Model with a constant attribute / frozen value after its first set"
date: 2024-07-20
tags: python
---

When working with Pydantic models, you sometimes need certain fields to remain constant once they're set. This is particularly useful when loading data incrementally, where some attributes determine how an object should be processed and cannot be changed afterward.

## The Problem

Consider a scenario where you're loading data into models with different types of attributes:
- Some are known immediately during initialization while others are loaded dynamically later.
- Some are calculated based on other fields (once filled in).
- Certain values must remain constant once set to ensure data integrity.

The challenge is preventing modification of these critical fields while still allowing initial assignment. 

## Basic Setup

The following imports and types are used throughout the examples:

```python
from enum import Enum
from typing import Optional, ClassVar, Annotated
from pydantic import BaseModel, Field, ConfigDict

class AppType(str, Enum):
    APP_GROUP = "APP_GROUP"
    APP = "APP"
```

## Solution 1: Using ClassVar for Known Constants

When you know the constant value from the beginning, use `ClassVar` to define it as a class-level constant:

```python
class ClientType(BaseModel):
    attribute: Optional[str] = None
    object_type: ClassVar[str] = "CLIENT_TYPE"
```

Attempting to modify this value raises an `AttributeError`:
```
AttributeError: 'object_type' is a ClassVar of 'ClientType' and cannot be set on an instance.
```

## Solution 2: Using Frozen Fields

For fields that should become immutable after being set, use `Field` with `frozen=True`. This works with default values:

```python
class Policy(BaseModel):
    object_type: Optional[str] = Field("DEFAULT_VALUE", frozen=True)
    name: Optional[str] = None
```
An alternative syntax with the same behavior:
```python
class Policy(BaseModel):
    object_type: Annotated[Optional[str], Field(frozen=True)] = "DEFAULT_VALUE"
    name: Optional[str] = None
```

So now you can either init an instance with the default value or assign a desired one at init:

```python
policyA = Policy(name="Test Policy A")
print(policyA)

policyB = Policy(name="Test Policy B", object_type="NEW")
print(policyB)
```

To get output

```
object_type='DEFAULT_VALUE' name='Test Policy A'
object_type='NEW' name='Test Policy B'
```

Attempting to modify a frozen field raises a `ValidationError`:
```
pydantic_core._pydantic_core.ValidationError: 1 validation error for Policy
object_type
  Field is frozen [type=frozen_field, input_value='CHANGED', input_type=str]
    For further information visit https://errors.pydantic.dev/2.8/v/frozen_field
```

`ClassVar` raises an `AttributeError` on any instance assignment attempt, while `frozen=True` raises a `ValidationError`. Neither requires `validate_assignment=True` to enforce immutability — that setting controls whether *validators* run on assignment, which is a separate concern.

## Solution 3: Value Not Known at Initialization

If the value is not known at initialization time, Pydantic v2 does not offer a built-in way to freeze a field after its first assignment. You need to implement this yourself. One simple approach:

```python
class Policy(BaseModel):
    object_type: Optional[str] = None
    name: Optional[str] = None

    def __setattr__(self, name: str, value):
        if name == 'object_type' and self.__dict__.get(name) is not None:
            raise AttributeError(f"Field '{name}' cannot be modified after being set")
        super().__setattr__(name, value)


policy = Policy(name="Test Policy")
print(policy)

# initialize the object_type attribute a bit later after object creation

policy.object_type = "set a bit later after creation"
print(policy)
```
If you try to change the attribute again, you get the expected error:

```
AttributeError: Field 'object_type' cannot be modified after being set
```

Depending on your requirements, you may need a more sophisticated approach — for example, a reusable Python [descriptor](https://docs.python.org/3/howto/descriptor.html) or the `attrs` library's `on_setattr` hooks. This is meant as a starting point.
