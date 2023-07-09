# fFlexiJson

A flexible JSON parser that bends over backwards to try and make you happy.
Due to the flexibility introduced with FlexiJSON, we broke the pattern of familiarity with the built-in `json` module.

Our primary goal is to support:
1. Line numbers
2. Non-explosive detection of duplicate keys (no exceptions raised, but you can evaluate after you've parsed)
3. Comments (lol, why don't we just use JSON5....)

The decision was made due to a need for a diverse set of data models, based on your needs.
So, we opted for an OO experience.

> Quick note on scope: If you want ultra-flexible, look into JSON5. One day we might add full JSON5 support, but not today.

## Quick Start

```python
from pathlib import Path
import flexijson

json_file = Path("path/to/your/file.json")
json_text = '{"spam": "eggs"}'

# Load from a file and get the JSON
flexi = flexijson.from_file(json_file)
print(flexi.json) # Would look exactly like the output of json.load(json_file)


flexi = flexijson.from_text(json_text)
print(flexi.json)
```
### Line Support
```python
from pathlib import Path
import flexijson

json_file = Path("path/to/your/file.json")

# Load from a file, but you want comments
flexi = flexijson.from_file(json_file, lines=True)
print(flexi.json) # will not show lines

print(flexi.lines)

```

Assuming your JSON looks like this:

```json
{
    "spam": "I love to spam you",
    "eggs": [
        "hard",
        "over-easy"
    ]
}
```

Then output of `flexi.lines` would be:

```python
# I know this is wrong, but I'm 4BD.
{
    "0/": "{"
    "0/spam": "I love to spam you",
    "0/eggs": "[",
    "0/eggs/0": "hard",
    "0/eggs/1": "over-easy",
    "0/eggs/2": "]",
    "1/": "}"
}

```


### Comments Support

```python
from pathlib import Path
import flexijson

json_file = Path("path/to/your/file.json")

# Load from a file, but you want comments
flexi = flexijson.from_file(json_file, comments=True)
print(flexi.json) # will not show comments

# will include comments as part of the closest value
# if a comment exists outside of an object, will create a custom _comment entry
print(flexi.jsonc)

```

Example input
```jsonc
// Some top-level comment
{
    "spam": "/*this isn't a comment - it's a string*/" /*This is a comment*/,
    "eggs": [3, 5, /*6*/, 7], // This is a valid comment
    // This comment came out of nowhere!
}
```

Example data model from `felixjson.from_file(json_file, comments=True).jsonc`

> This is pretty confusing pretty fast. But it's hard to make it not confusing unless you just use raw strings
```python
{
    "_comment1": "// Some top-level comment",
    "_key1": {
        "spam": {
            "_value1": "/*this isn't a comment - it's a string*/",
            "_comment1": "/*This is a comment*/"
        }
    },
    "_key2": {
        "eggs": {
            "_value1": [3, 5],
            "_comment1": "/*6*/",
            "_value2": [7]
        },
        "_comment1": "// This is a valid comment"
    },
    "_comment2": "// This comment came out of nowhere!"
}

```