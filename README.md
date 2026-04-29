# Pydantic Tutorial

> Data validation in Python — from zero to confident.

<img src="img/pydantic.png" width="77%"/>

---

A hands-on tutorial for Pydantic — the most widely used data validation library for Python.

---

## What you will learn

- Why Python silently accepts wrong data and why that's a problem
- How to define models with `BaseModel` and enforce types automatically
- Type coercion and `ValidationError` — when Pydantic fixes data vs. rejects it
- Complex types — `bool`, `List`, `Dict`
- Optional fields and default values
- Built-in validators — `EmailStr` and `AnyUrl`
- `Field()` constraints — ranges, lengths, patterns
- `Annotated` — reusable type definitions across models
- `@field_validator` — custom validation and transformation logic
- `@model_validator` — rules that involve multiple fields together
- `@computed_field` — fields calculated from other fields
- Nested models — using one model as a field inside another
- Serialization — converting models to `dict` and JSON with `model_dump()`

---

## Test Your Understanding

Before diving in — how much of this can you read?

```python
from typing import Annotated, Optional, List, Dict
from typing_extensions import Self
from pydantic import BaseModel, EmailStr, AnyUrl, Field, field_validator, model_validator, computed_field

# Annotated — constraint only (required)
Age = Annotated[int, Field(ge=0, le=120)]

# Annotated — constraint + default bundled in
Temperature = Annotated[float, Field(ge=35.0, le=42.0, default=37.0, description="Body temperature in °C")]
Visits = Annotated[int, Field(ge=0, default=0)]

# Annotated — Optional bundled in
Allergies = Annotated[Optional[List[str]], Field(default=None)]

class Patient(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    email: EmailStr                # built-in validator
    age: Age                       # Annotated — int 0–120
    married: bool                  # bool
    weight: float                  # kg — used to compute BMI
    height: float                  # metres — used to compute BMI
    contact: Dict[str, str]        # complex type
    url: AnyUrl                    # built-in validator
    temperature: Temperature       # Annotated with default
    visits: Visits                 # Annotated with default
    allergies: Allergies           # Annotated with Optional
    blood_type: str = "Unknown"    # plain default
    notes: Optional[str] = None    # plain Optional

    @field_validator('email')
    @classmethod
    def check_email_domain(cls, value) -> str:
        valid_domains = {'hdfc.com', 'kpmg.com', 'hospital.com'}
        if value.split('@')[-1] not in valid_domains:
            raise ValueError('Email must be from a registered domain')
        return value

    @model_validator(mode='after')
    def check_emergency_contact(self) -> Self:
        if self.age > 60 and 'emergency' not in self.contact:
            raise ValueError('Patients older than 60 must have an emergency contact')
        return self

    @computed_field
    @property
    def bmi(self) -> float:
        return round(self.weight / (self.height ** 2), 2)

patient = Patient(
    name="John Doe",
    email="john@hospital.com",
    age=35,
    married=True,
    weight=80.0,
    height=1.75,
    contact={"phone": "123-456-7890", "emergency": "Jane Doe"},
    url="https://hospital.com/patients/john",
    allergies=["penicillin", "peanuts"],
    # temperature, visits, blood_type not required as they are default
)
```

> Not sure about something in the code above? The notebook has you covered:
> - Just want a quick refresher? Check it out here: [basics.ipynb](https://github.com/kameshcodes/pydantic-tutorial/blob/main/basics.ipynb)
> - New to this? Open it in Colab and play around as you learn: https://colab.research.google.com/github/kameshcodes/pydantic-tutorial/blob/main/basics.ipynb

---

## Prerequisites

- Python 3.8+
- Basic familiarity with Python functions, classes, and type hints

No prior Pydantic experience needed.
