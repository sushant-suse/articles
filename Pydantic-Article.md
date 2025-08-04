# Pydantic Solves a Problem You Did not Know You Had

<!-- 
Other Article Title Options
1. Pydantic: The Python Library Every Developer Should Know
2. Everything You Need to Know About Pydantic (and Why It Matters)
3. Tired of Buggy Data Models? Meet Pydantic.
4. Pydantic in Action: The Must-Know Tool for Python Developers
-->

Data exchange is a fundamental part of modern applications, especially those that interact with APIs, databases, or external services. In [Python](https://docs.python.org/3/), ensuring data is structured, validated, and easily converted between formats is critical and this is where Pydantic comes into Picture.

[Pydantic](https://docs.pydantic.dev/latest/) is a powerful Python library designed for data validation and settings management. It utilizes Python type hints to define, parse, and enforce strict typing on data [models](https://docs.pydantic.dev/latest/concepts/models/), making it easier to work with structured and semi-structured data.

So, whether you are handling user input, API responses, or configuration files, Pydantic ensures that your application receives data in the correct format. If the data does not match the defined schema, Pydantic raises clear and informative validation errors. This helps to reduce the debugging time and improves code reliability.

### The Role of `BaseModel`

The core of Pydantic is the [`BaseModel`](https://docs.pydantic.dev/latest/api/base_model/) class. Every custom model in Pydantic is built by inheriting this base class. It provides essential functionality, including:

- Field definition and type enforcement
- Built-in validation and error messaging
- Data serialization and deserialization
- Automatic type conversion when possible

By extending `BaseModel`, you gain access to a robust set of tools that ensure your data conforms to the expected types and structures.

Example (Product class inherits `BaseModel` class):

```python
from pydantic import BaseModel

class Product(BaseModel):
    price: int
```

### Automatic Type Conversion

One of the most useful features of Pydantic is its ability to perform type coercion automatically. When **possible**, Pydantic converts input data to match the declared type.

Example:

```python
from pydantic import BaseModel

class Product(BaseModel):
    price: int

Product(price='99')       # Allowed: '99' is converted to 99
Product(price='99a')      # Raises ValidationError: cannot convert to int
```
<!-- Maybe show here the abbreviated traceback of the ValidationError?
    This gives readers a better understanding how this looks like.
 -->


In this example, a string containing numeric characters (`'99'`) is accepted and converted to an integer, while an invalid string (`'99a'`) results in a validation error.

### Real-World Example (Parsing JSON)

Consider a scenario where your application receives JSON data from an external API:

```json
{
  "id": "101",
  "name": "Sam",
  "salary": "12000"
}
```

You can use Pydantic to model and validate this data:

```python
from pydantic import BaseModel

class Employee(BaseModel):
    id: int
    name: str
    salary: float

emp = Employee(**json_data)
print(emp)
# Employee(id=101, name='Sam', salary=12000.0)
```

**Observation**: Even though the `id` and `salary` fields arrive as strings, Pydantic automatically converts them to the appropriate numeric types.
<!-- Maybe add a reference to the "Enforcing Strict Types ..." section?

  I'd suggest to add something like:
  "If you don't want or like the automatic conversion, Pydantic allows
  to enable the strict feature (see section ...)."
 -->

> **Note**: In the above example, the `**` operator is unpacking the `json_data` dictionary into keyword arguments.

## Comparing `typing` and Pydantic
<!-- I think this section is a bit weird. Your comparison is technically
     correct, but IMHO it compares apples with oranges. ;)

  The two modules serve completely different goals (albeit Pydantic
  uses types a lot). The typing module was never about validation nor was
  it a goal.
  The "validation" (if you want to name it that way), is done by an external
  tool.

  If you want to add a comparison, you should compare it with the same
  level of libraries. This would be dataclasses or attrs.
  ArjanCodes has a good video about that:
     "Attrs, Pydantic, or Python classes"
     https://www.youtube.com/watch?v=zN4VCb0LbQI
 -->
[`typing`](https://docs.python.org/3/library/typing.html) is Python's built-in module that provides static type hints. It is useful for development tools and linters like `mypy` or `Pylance`. However, these hints do not enforce or validate data at runtime. This is where Pydantic adds significant value by enforcing type constraints dynamically and validating incoming data during execution.

| Feature | `typing` | Pydantic |
| ------- | -------- | -------- |
| Type enforcement | Only provides static hints | Performs runtime enforcement and validation |
| Validation | No validation | Validates and coerces data automatically |
| Runtime behavior | Passive  | Active |
| Tooling | Checked by `mypy`, `pylance`, etc. | Validates actual input during execution |

Example:

```python
from typing import List

# Using only typing:
def greet(names: List[str]):
    pass
# No runtime validation

# Using Pydantic:
from pydantic import BaseModel

class Model(BaseModel):
    names: List[str]
# Enforces List[str] at runtime
```

In short, if you are using `typing` alone then incorrect data types pass silently unless caught by linters. But, Pydantic adds runtime enforcement, making data integrity more reliable.

## Understanding `Field` Function 

In Pydantic, [`Field`](https://docs.pydantic.dev/latest/api/fields/) is used to configure individual model attributes. It allows you to define default values, add validation constraints, and provide metadata such as titles, descriptions, or example values. 

It is not mandatory but using `Field` helps improve clarity, maintainability, and control over model behavior.

The key use cases for `Field` include:

- Defining default values
- Setting validation constraints (such as minimum or maximum length)
- Adding metadata (like descriptions or usage examples)
- Customizing serialization or documentation behavior

The common parameters used with `Field` include:

- `default`, `title`, `description`, `examples`
- `min_length`, `max_length`, `regex`
- `ge` (greater than or equal to), `le` (less than or equal to)
- `strict`, `frozen`, and others

Example:

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    id: int
    name: str = Field(min_length=3)
    age: int = Field(default=18, ge=0)
```

Here, `name` must be at least three characters long, and `age` must be non-negative, with a default value of 18.

> **Note**: 
> - To mark a field as optional, wrap the type with `Optional[...]`.
> - Use `...` (Ellipsis) to mark a field as required when no default is provided.
<!-- Maybe add a third list item to mention PositiveInteger
 instead of using Field(default=18, ge=0)?
-->

### Enforcing Strict Types with `Field(strict=True)`

By default, Pydantic tries to coerce values into the expected type. This behavior is helpful in many situations. But, in some scenarios, you may need stricter validation for sensitive or critical fields. For such scenarios, use `strict=True` to enforce exact type matching:

Example:

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    age: int = Field(strict=True)

User(age="21")  # Raises validation error: str is not int
```

## Using `Annotated` for Cleaner Type Definitions

`Annotated` type was introduced in the `typing` module to allow additional metadata to be attached to a type hint. Pydantic v2 adopts `Annotated` to define constraints, descriptions, and field-level metadata in a more structured and expressive way.

> Note: To learn about 
<!-- Incomplete note? Or does the following parts belong to the note? -->

Reasons to use Annotated:

- Avoids mixing logic between type declaration and field definition
- Keeps the syntax cleaner and easier to read
- Enhances compatibility with tools like [`FastAPI`](https://fastapi.tiangolo.com/) that generate documentation from model metadata

**Example: Classic `Field` Usage (Without `Annotated`)**

```python
from pydantic import BaseModel, Field

class Patient(BaseModel):
    name: str = Field(
        title="Patient Name",
        description="It contains the name of the patient",
        examples=["Aman", "Suman"]
    )
```

**Example: Modern Usage With `Annotated`**

```python
from typing import Annotated
from pydantic import BaseModel, Field

class Patient(BaseModel):
    name: Annotated[
        str,
        Field(
            title="Patient Name",
            description="It contains the name of the patient",
            examples=["Aman", "Suman"]
        )
    ]
```

**Example: Combining Constraints and Metadata**

This approach ensures clean definitions, enforces constraints, and provides rich metadata for tools and documentation.

```python
from typing import Annotated
from pydantic import BaseModel, Field

class Patient(BaseModel):
    name: Annotated[
        str,
        Field(
            title="Patient Name",
            description="It contains the name of the patient",
            min_length=2,
            examples=["Aman", "Suman"]
        )
    ]
    age: Annotated[
        int,
        Field(
            ge=0,
            description="Patient age (non-negative)"
        )
    ]
```

## What are Validators and `ValidationError` in Pydantic?

Pydantic supports custom data validation through [**validators**](https://docs.pydantic.dev/latest/concepts/validators/). Validators provide fine-grained control over how fields or models are validated, making them useful when the built-in validation logic is insufficient.

Pydantic includes two primary types of validators: [**field validators**](https://docs.pydantic.dev/latest/concepts/validators/#field-validators) and [**model validators**](https://docs.pydantic.dev/latest/concepts/validators/#model-validators).

### Field Validators

Field-level validators allow you to define custom logic for individual fields. These validators are declared using the `@field_validator` decorator. It was introduced in Pydantic v2 in combination with the `@classmethod` decorator.
<!--
  It sounds like the classmethod decorator was also introduced in version 2.
  However, classmethod is a builtin Python decorator. Maybe you want to
  distinguish between the two. Maybe use something like this:

  Validators are declared using the `@field_validator` decorator. Introduced in Pydantic v2, this decorator must be used in combination with Python's standard @classmethod decorator.
-->

Example:

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    username: str

    @field_validator("username")
    @classmethod
    def validate_username(cls, value):
        if " " in value:
            raise ValueError("Username must not contain spaces")
        return value
```

In the above example:

- `field_validator` decorator contains the field name to validate.
- `cls` refers to the model class (`User`).
- `value` is the input provided for the `username` field.

> **Note**: The `@classmethod` is required here because Pydantic v2's `@field_validator` expects a class method (not a static method or instance method).
<!-- It's important that you also mention the correct order. Otherwise
it may not work correctly. -->

You can even apply a single validator to multiple fields. 

Example:

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    username: str
    email: str

    @field_validator("username", "email")
    @classmethod
    def no_spaces_allowed(cls, value, info):
        if " " in value:
            raise ValueError(f"{info.field_name} must not contain spaces")
        return value
```

In the above example, the `no_spaces_allowed()` function is executed once per field, and the `info` parameter provides metadata such as the current field name.

> **Note:** Field validators execute before type coercion by default, which makes them ideal for validating raw input values.

### Model Validators

Model-level validators enable cross-field validation. You can use them when validation depends on the relationship between two or more fields. In Pydantic v2, the `@model_validator` decorator replaces the earlier `@root_validator` from Pydantic v1.
<!-- Hmn, I'm not sure if I understand you correctly.

     Aren't model validators methods to validate the *entire data model
     at once*, rather than validating each field individually?

    IMHO, I think you should emphasize the "validate the entire data model
    at once" more.
 -->

```python
from pydantic import BaseModel, model_validator

class User(BaseModel):
    password: str
    confirm_password: str

    @model_validator(mode="after")
    def passwords_match(self):
        if self.password != self.confirm_password:
            raise ValueError("Passwords do not match")
        return self
```

In the above example, `self` resembles the instance of the model.

### Validator Modes (`before` and `after`)

You can control when a validator runs using the `mode` argument:

- **`mode="before"`** executes before standard Pydantic validation.
- **`mode="after"`** executes after Pydantic validation and type coercion.

| Validator Type  | `before` Mode  | `after` Mode (Default) |
| --------------- | -------------- | ---------------------- |
| Field Validator | Receives raw input            | Receives parsed and type-coerced value  |
| Model Validator | Receives raw input dictionary | Receives fully validated model instance |

In short, use `"before"` when you need to clean or reject invalid data early. Use `"after"` for final checks, consistency validation, or business logic after all fields have been validated.

### Key Differences between Field and Model Validator 

Although `@field_validator("username", "email")` accepts multiple fields, it still processes each field **independently**. This means that the validator runs separately for each field, even if the logic is identical. This approach can lead to redundant processing and is not ideal for validations that depend on multiple fields. In such cases, using a `@model_validator` is more efficient and appropriate, as it processes the entire model at once and allows for cross-field validation in a single pass.

| Aspect | Field Validator | Model Validator |
| ------ | --------------- | --------------- |
| Execution | Per field  | Once per model instance |
| Input | Single field value (+ metadata) | Entire model (either raw or parsed) |
| Use Case | Field-level validation and transformation | Cross-field validation, business logic |

### Handling Validation Errors

Whenever validation fails, Pydantic raises a [`ValidationError`](https://docs.pydantic.dev/latest/concepts/validators/#raising-validation-errors). This exception provides a detailed breakdown of the issue, including the field name, the error message, and the error type.

Example:

```python
from pydantic import BaseModel, ValidationError

class Product(BaseModel):
    price: float

try:
    Product(price="free")
except ValidationError as e:
    print(e)

# Output:
# 1 validation error for Product
# price
#   Input should be a valid number (type=type_error.float)
```

To programmatically inspect errors, you can use `e.errors()` which returns a list of structured error dictionaries.

## How to Dump Model Data in Pydantic?

Pydantic models provide two helpful methods to extract data:

- [**`.model_dump()`**](https://docs.pydantic.dev/latest/concepts/serialization/#modelmodel_dump) returns the model as a standard Python `dict`.
- [**`.model_dump_json()`**](https://docs.pydantic.dev/latest/concepts/serialization/#modelmodel_dump_json) returns the model data as a JSON string.

These methods are commonly used when serializing models for storage, logging, or API responses.

Example:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

user = User(name="Sam", age=25)

print(user.model_dump())
# Output: {'name': 'Sam', 'age': 25}

print(user.model_dump_json())
# Output: {"name":"Sam","age":25}
```
<!--
  Your example is technically correct, but you can't really see the difference
  (apart from the single/double quoting).

  What about this:

  print(type(user.model_dump()))      # would print <class 'dict'>
  print(type(user.model_dump_json())) # would print <class 'str'>
-->

To learn about these methods in detail, check [Serialization](#understanding-serialization-in-pydantic).

## Understanding Computed Fields in Pydantic

In many applications, certain values are not provided directly by the user but are derived from other fields. These values are known as [**computed fields**](https://docs.pydantic.dev/2.0/usage/computed_fields/).

Pydantic v2 provides support for computed fields through the `@computed_field` decorator. This eliminates the need for workarounds, such as using `@property`, and provides better integration with [Pydantic’s serialization](https://docs.pydantic.dev/latest/concepts/serialization/) system.

Example:

```python
from pydantic import BaseModel, computed_field

class Rectangle(BaseModel):
    width: float
    height: float

    @computed_field
    def area(self) -> float:
        return self.width * self.height
```

In the above example:

- `area` is computed dynamically based on `width` and `height`.
- The `@computed_field` decorator registers the method as a virtual field.

> **Note:** Computed fields are not included in serialized outputs by default, preserving a clear boundary between user-provided input and derived values.

### Including Computed Fields in Output

By default, computed fields are excluded from methods such as `.model_dump()` and `.model_dump_json()`. To include them, use the `include_computed=True` argument:

```python
rect = Rectangle(width=10, height=5)
print(rect.model_dump())
# Output: {'width': 10, 'height': 5}

print(rect.model_dump(include_computed=True))
# Output: {'width': 10, 'height': 5, 'area': 50}
```

This behavior is intentional as it ensures that:

- Computed values are not accidentally persisted or sent over APIs.
- Only explicitly requested values are included.
- Expensive calculations are avoided unless needed.

## Pydantic v1 vs. Pydantic v2: What Changed?

Pydantic v2 represents a [major upgrade over v1](https://docs.pydantic.dev/latest/migration/), offering improved performance, enhanced type safety, cleaner validation syntax, and better serialization tools. Although many v1 models will still work with v2, some migration is required due to changes in decorators and method names.

**Key Differences at a Glance**:

| Feature              | Pydantic v1          | Pydantic v2                                 |
| -------------------- | -------------------- | ------------------------------------------- |
| Validation engine    | Pure Python          | Rust-based core (faster)                    |
| Field validators     | `@validator`         | `@field_validator`                          |
| Model validators     | `@root_validator`    | `@model_validator`                          |
| Computed fields      | Via `@property`      | Native via `@computed_field`                |
| Type system          | Standard type hints  | Better support for advanced typing          |
| Strict type handling | Limited              | Enhanced (`StrictStr`, `StrictInt`, etc.)   |
| Serialization        | `.dict()`, `.json()` | `.model_dump()`, `.model_dump_json()`       |
| Environment settings | `BaseSettings`       | Improved support for config and env parsing |
| Error reporting      | Simple               | More structured and user-friendly           |

### Major Improvements in Pydantic v2 

#### Improved Validation Flow

The introduction of `"before"` and `"after"` modes for both field and model validators gives developers precise control over the validation lifecycle. This enables early rejection of bad data or sophisticated cross-field logic as needed.

#### Enhanced Serialization

Pydantic v2’s `.model_dump()` and `.model_dump_json()` provide a more flexible and consistent serialization API, especially when dealing with computed fields or nested models.

## Working with Nested Models in Pydantic

Pydantic supports nested models, enabling developers to represent structured, hierarchical data in a clean and intuitive way. This is particularly useful in applications that deal with complex schemas such as user profiles, blog posts, or nested comment threads (where one model depends on or contains another).

Nested models in Pydantic allow you to:

- Represent relationships like `UserProfile → Address → Country`
- Encapsulate multi-level structured data (for example, `Blog → Comment → Author`)
- Validate recursive structures (for example, trees, chat threads)

Pydantic offers three main types of nested model patterns:

### 1. Standard Nesting (Referencing Other Models)
<!-- This is basically composition, right? -->

The most common scenario involves referencing one Pydantic model inside another using type annotations. This lets you build layered schemas and ensures each component gets validated properly.

```python
class Lesson(BaseModel):
    title: str
    content: str

class Module(BaseModel):
    name: str
    lessons: List[Lesson]  # Nested model: referencing Lesson inside Module
```

This setup allows each `Module` to encapsulate a list of validated `Lesson` instances, promoting reusability and data consistency.

### 2. Self-Referencing Models (Recursive Structures)

Pydantic also supports models that reference themselves. These are especially useful for representing recursive structures such as tree hierarchies or nested folders.

```python
class Node(BaseModel):
    name: str
    children: List['Node']  # Forward reference using a string

Node.model_rebuild()
```

Because the class refers to itself and has not yet been fully constructed at the time of annotation, Pydantic requires `model_rebuild()` to resolve the forward reference. This ensures the `children` field is properly typed for validation and schema generation.

#### 3. Forward Referencing Between Multiple Models
<!-- Is this the correct heading level?
  In the previous section you have three #, but here four.
-->

For mutually dependent models where, for example, an `Employee` references a `Manager`, and the `Manager` holds a list of `Employee` instances; Pydantic supports forward references using string annotations.

```python
class Employee(BaseModel):
    name: str
    manager: 'Manager' = None  # String-based forward reference

class Manager(BaseModel):
    name: str
    team: List[Employee] = []

# Resolve circular references
Employee.model_rebuild()
Manager.model_rebuild()
```

Without `model_rebuild()`, these forward references would remain unresolved and lead to validation errors or incorrect schema generation.

#### Why `model_rebuild()` Matters

When forward references are used (either to the same model or another model that has not been defined yet), `model_rebuild()` is necessary to:

- Re-evaluate type hints that were expressed as strings
- Replace string references with actual class objects
- Finalize model field definitions for accurate parsing and validation

This post-definition step allows Pydantic to maintain correctness in complex model structures.

<!-- Isn't a forth section missing? What about deriving one
     BaseModel from another one?

     For example:

     class Person(BaseModel):
        name: str
        age:  str

     class Worker(Person):
        company : str
        team: str

     Would that be possible? Is it even recommended?
     It introduces tight coupling which is probably not always wanted.
     In most(?) cases, Pydantic(?) recommends composition over
     inheritance, right?
 -->

### Example: Nested `Comment` Model

Example of nested `Comment` model commonly found in blog platforms. It incorporates all three nested model concepts:

- Referencing an `Author` model
- Self-referencing through nested replies
- Forward referencing using strings

```python
from pydantic import BaseModel
from typing import List, Optional

class Author(BaseModel):
    user_id: int
    username: str

class Comment(BaseModel):
    comment_id: int
    content: str
    author: Author  # Standard nesting
    replies: Optional[List['Comment']] = None  # Self-referencing

Comment.model_rebuild()

author1 = Author(user_id=1, username="Sam")

reply1 = Comment(
    comment_id=2,
    content="Replying to your comment!",
    author=author1
)

main_comment = Comment(
    comment_id=1,
    content="This is the main comment",
    author=author1,
    replies=[reply1]
)

print(main_comment.model_dump(indent=2))
```

**Output**:

```json
{
  "comment_id": 1,
  "content": "This is the main comment",
  "author": {
    "user_id": 1,
    "username": "Sam"
  },
  "replies": [
    {
      "comment_id": 2,
      "content": "Replying to your comment!",
      "author": {
        "user_id": 1,
        "username": "Sam"
      },
      "replies": null
    }
  ]
}
```

## Understanding Serialization in Pydantic

[Serialization](https://docs.pydantic.dev/latest/concepts/serialization/) in Pydantic refers to the process of converting a model into a format suitable for storage or transmission. 

This typically means converting a model to a dictionary (`dict`) or a JSON string (`str`). 

Pydantic also handles deserialization, which converts raw input data into structured model instances.

Pydantic supports two key directions in serialization:

### 1. Up Serialization (Deserialization)

Up serialization, also known as deserialization, is the process of converting raw data such as a dictionary or JSON object into a Pydantic model. This enables structured handling of unstructured input data.

**Example:**

```python
data = {"name": "Sam", "joined": "2023-07-24T10:00:00"}
user = User(**data)  # Deserializes raw dict into a User model
```

### 2. Down Serialization

Down serialization refers to converting a Pydantic model into a format like a Python dictionary or a JSON string. This is typically used when the model data needs to be sent over a network, saved to a file, or logged.

### Methods for Serialization

#### `model_dump()`

The `model_dump()` method converts a Pydantic model into a standard Python dictionary It excludes computed fields by default unless `include_computed=True` is specified.It is best suited for internal Python logic, debugging, or when the data remains within the application.

#### `model_dump_json()`

The `model_dump_json()` method returns a JSON-formatted string. It automatically converts non-JSON-native types (such as `datetime` or `Decimal`) into JSON-compatible representations, such as ISO 8601 strings. It is ideal for API communication, file storage, or external logging.

Example with `datetime`:

```python
from pydantic import BaseModel
from datetime import datetime

class User(BaseModel):
    name: str
    joined: datetime

# Up serialization: Create model from raw data
user = User(name="Sam", joined="2023-07-24T10:00:00")

# Down serialization: Convert to dictionary
print(user.model_dump())
# Output: {'name': 'Sam', 'joined': datetime.datetime(2023, 7, 24, 10, 0)}

# Down serialization: Convert to JSON string
print(user.model_dump_json())
# Output: {"name": "Sam", "joined": "2023-07-24T10:00:00"}
```

## FAQs

### How does field aliasing affect serialization in Pydantic?

Pydantic supports aliasing field names using the `alias` parameter in the `Field()` function. This is particularly useful when you need to conform to naming conventions, such as camelCase in API responses.

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    full_name: str = Field(..., alias="fullName")

user = User(fullName="Sam")
print(user.model_dump(by_alias=True))  # {'fullName': 'Sam'}
```

To ensure aliases are included in the serialized output, set `by_alias=True` when calling `model_dump()` or `model_dump_json()`.

### Are `model_dump()` and `model_dump_json()` available in Pydantic v1?

No. These methods are part of **Pydantic v2**. In Pydantic v1, serialization was handled using the `dict()` and `json()` methods.

If you're upgrading from v1 to v2, note the following changes:

- `model.dict()` → `model.model_dump()`
- `model.json()` → `model.model_dump_json()`

### What happens if a field contains a non-serializable object during `model_dump_json()`?

If a field contains a non-JSON-serializable object, such as a custom class or complex data type, `model_dump_json()` will raise a `TypeError`. To handle this, you can either preprocess the data or define custom `json_encoders` in your model's configuration.

```python
class Config:
    json_encoders = {
        CustomType: lambda v: str(v)
    }
```

###  Can I exclude certain fields when using `model_dump()` or `model_dump_json()`?

Yes. Both methods support parameters like `exclude`, `include`, and `exclude_unset`.

```python
model.model_dump(exclude={"password"})
```

These options help you control exactly what gets serialized. It is useful when returning partial responses or removing sensitive data like passwords or tokens.

### Is it possible to serialize nested models using `model_dump()` or `model_dump_json()`?

Yes. Nested models are fully supported. When serialized, they are recursively converted to dictionaries or JSON strings.

```python
class Address(BaseModel):
    city: str

class User(BaseModel):
    name: str
    address: Address

user = User(name="Sushant", address=Address(city="Delhi"))
print(user.model_dump())
# {'name': 'Sushant', 'address': {'city': 'Delhi'}}
```

### How to print compact or pretty-printed JSON output using `model_dump_json()`?

By default, `model_dump_json()` produces compact JSON. If you want pretty-printed (indented) output, you can pass formatting arguments using the `indent` parameter.

```python
user.model_dump_json(indent=2)
```
