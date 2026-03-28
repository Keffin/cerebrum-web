---
title: "Python tricks with MF DOOM"
date: "2026-03-28"
summary: "Data classes featuring MF DOOM."
description: "Quality of life improvements using data classes"
toc: false
autonumber: false
math: true
tags: ["python", "records", "dataclass", "MF DOOM"]
showTags: true
hideBackToTop: true
hidePagination: true
---

Python is a great language. Simple and fun to read.

But I can sometimes come across code bases where certain quality of life improvements are missing.
Coming from a background of mostly writing Java and C# code, I can sometimes like to get inspiration from those languages when writing Python code.

So here are some features in the language that I personally like to use to improve my Python code.

## Data classes

As of [PEP 557](https://peps.python.org/pep-0557/), data classes have been a great addition to the standard library.
I like to compare data classes in Python to records in Java. A clean and simple way to represent and structure data.

### Frozen data classes

The data class library introduces a lot of other nice features as well.
One I'm particularly keen on is emulating immutability, passing `Frozen=True` to a data class.
When a data class is frozen, attempts to modify its attributes will raise a FrozenInstanceError.

The mutable methods in question are `__setattr__()` and `__delattr__()` which are added to data classes.

```python
@dataclass(frozen=True)
class Doom:
    name: str

def main() -> None:
    mf = Doom("DOOM")
    mf.name = "King Geedorah" # Bad!

# dataclasses.FrozenInstanceError: cannot assign to field 'name'
```

Making it frozen, we prevent any accidental or intentional changes to a class that we don't want to mutate.

### Frozen is not read-only

Frozen data classes are still vulnerable to some extent of mutability. The immutability only applies to the attributes themselves, not the objects they reference.

The content of mutable objects stored in a data class is still able to be changed. E.g a list attribute. A frozen data class with a list attribute can still be mutated later on.

```python
@dataclass(frozen=True)
class Doom:
    name: str
    aliases: list[str]

def main() -> None:
    mf = Doom(name="DOOM", aliases=["King Geedorah"])
    mf.aliases.append("Viktor Vaughn") # Completely fine
```

This doesn’t mutate the data class itself, but rather the list it holds, since lists are mutable.

#### Explicit data class mutation

In some cases, such as inside `__post_init__`, you may still want to modify a frozen data class. This can be done using `object.__setattr__()`.

```python
@dataclass(frozen=True)
class Doom:
    name: str

    def __post_init__(self):
        all_caps = self.name.upper()
        object.__setattr__(self, "name", all_caps)

def main() -> None:
    mf = Doom(name="doom")
    print(mf) # MF(name='DOOM')
```

### Computed properties

Python provides a nice way of creating methods that behave like attributes. This is good in larger projects to prevent us from breaking backwards compatibility.
We can, instead of creating a method call, have something like a regular attribute access without breaking the class objects public API.

An FYI, this feature is not limited to data classes only.

```python

@dataclass(frozen=True)
class Doom:
    name: str

    @property
    def reminder(self) -> str:
        return self.name.upper()


def main() -> None:
    mf = Doom(name="doom")
    print(mf.reminder) # DOOM
```

### Hiding output

In some cases, we want to hide output from a class's `repr`, in these scenarios, we can use the `field(repr=False)`.
This is especially useful when working with e.g. dataframes or arrays that could clutter logs.

```python
@dataclass
class Doom:
    name: str
    lyrics: str = field(repr=False)


def main() -> None:
    mf = Doom(
        name="doom",
        lyrics="""
                I get no kick from champagne
                Mere alcohol doesn't thrill me at all
                So tell me, why shouldn't it be true?
                I get a kick out of brew
                """,
    )

    print(mf) # Doom(name='doom')
```
