# dictobject
A python dictionary that is accessible (like an object) through attributes and allows default value and warnings
Adds to the python dict by:
- Allowing access through the dict.key notation
- New items may be set by dict.key = value notation
- Allowing a default value to be set, which is returned if the key is not in the dictionary
- Optionally warn if the key is not found and default value is being returned
- Recursively change any nested dicts to DictObjects and back again
- Deepcopy implemented

## Installation
```
git clone https://github.com/pdrharris/dictobject.git
cd dictobject
python setup.py install
```

## Import
```python
from dictobject import DictObject
```
## Initialise
```python
dob = DictObject({'foo': 1, 'bar': 2})
```

## Parameters:
- input_dict(dict): Optional. If not set, an empty dictionary is initialised.
- default_to: Optional. If set, this is the value that is returned if a key is not in the dictionary.
- warn_key_not_found (bool): Optional prints a warning if a key is not found and default_to is returned. Defaults to False.

# Example usage
```python
dob = DictObject({'foo': 1, 'bar': 2})
# default_to is returned if the key is not found
# Probably more normal would be to set default_to to None

>>> dob.foo
1
>>> dob['bar']
2

>>> dob.baz = 3 # Set new value with dot notation
>>> dob.baz
3

```
### Return default value if key not found
```python
>>> dob = DictObject({'foo': 1, 'bar': 2}, default_to='Key Not Set')
>>> dob.baz
'Key Not Set'

>>> dob.baz = 3
>>> dob.baz
3

# set default_to after instantiation
>>> dob = DictObject({'foo': 1, 'bar': 2})
>>> dob.bootle
# Raises KeyError

>>> dob.set_default_to['Key not set']
>>> dob.bootle
'Key not set'
```
### Nesting
(See later for converting nested dictionaries)

```python
dob.baz = DictObject({'a': 3, 'b': 4})
dob
# Note: Display indenting added for clarity
DictObject({
    'foo': 1,
    'bar': 2,
    'baz': DictObject({
        'a': 3,
        'b': 4
        })
    })

>>> dob.baz.a
3

>>> dob.baz.c = 5
>>> dob
DictObject({
    'foo': 1,
    'bar': 2,
    'baz': DictObject({
        'a': 3,
        'b': 4,
        'c': 5
        })
    })
```
### Optional: warn if key not found but do not stop flow
Warn if a key is not found and default_to is being returned.
Particularly useful if default_to is set to _None_
```python
>>> dob = DictObject({'foo': 1, 'bar': 2}, default_to=None, warn_key_not_found=True)
>>> dob.not_a_key
UserWarning: Key not_a_key not found in DictTuple. Returning None
# Returns None

# Set turn warnings on or off after instantiation
>>> dob.warn_key_not_found() # turns warnings on
>>> dob.warn_key_not_found(False) # turns warnings off
```
### Converting to / from a dict

Convert any nested dictionaries to DictObject
```python

>>> DictObject({'foo': 1, 'bar': 2, 'baz': {'c': 3, 'd': 4}, convert_nested=True)
DictObject({'foo': 1, 'bar': 2, 'baz': DictObject{'c': 3, 'd': 4}})

>>> dob.zonk = {'a': 'zonk', 'b': 'zonky'}
>>> dob
DictObject({'foo': 1, 'bar': 2, 'zonk': {'a': 'zonk', 'b': 'zonky'}})

# Convert nested objects after initialisation
>>> dob = DictObject({{'foo': 1, 'bar': 2, 'zonk': {'a': 'zonk', 'b': 'zonky' 'c': {'d': 3, 'e': 4}}})
dob
DictObject({
  'foo': 1,
  'bar': 2,
  'zonk': {
      'a': 'zonk',
      'b': 'zonky' 'c': {
          'd': 3,
          'e': 4
            }
      }
  })
>>> dob.convert_dicts(recursive=True)
>>> dob
DictObject({
  'foo': 1,
  'bar': 2,
  'zonk': DictObject{
      'a': 'zonk',
      'b': 'zonky',
      'c': DictObject{
          'd': 3,
          'e': 4}
      }
  })

# Access nested members
>>> dob.zonk.c.d
3

# Convert back to a dict, including any nested objects
>>> dob.todict() # Defaults to recursive
{'foo': 1, 'bar': 2, 'zonk': {'a': 'zonk', 'b': 'zonky' 'c': {'d': 3, 'e': 4}}}

>>> dob.todict(False) # Nested DictObjects not converted to dict
{
  'foo': 1,
  'bar': 2,
  'zonk': DictObject{
      'a': 'zonk',
      'b': 'zonky',
      'c': DictObject{
          'd': 3,
          'e': 4}
      }
  }
```
# All standard python dict functions work
```python
>>> dob = DictObject({'foo': 1, 'bar': 2}, default_to='Not Set')
>>> dob.update({'bar': 'Two', 'baz': 3})
>>> dob
DictObject{'foo': 1, 'bar': 'Two', 'baz': 3}
>>> dob['bar'] = 'II'
>>> dob
DictObject{'foo': 1, 'bar': 'II', 'baz': 3}

>>> dob.keys()
dict_keys(['foo', 'bar', 'baz'])

>>> for key in dob:
        print(f'{key} = {dob[key]}')
foo = 1
bar = II
baz = 3

>>> 'bazz' in dob
False

>>> dob = DictObject({'foo': 1, 'bar': 2}) # default_to is not set
>>> dob.baz
# Raises KeyError

>>> dob['baz']
# Raises KeyError

>>> dob.get('baz', 3)
3
```
### Copying

#### Deepcopy
Deep copies create new copies of any nested objects.
```python
>>> from copy import deepcopy
>>> dob = DictObject({'foo': 1, 'bar': 2}, warn_key_not_found=True, default_to=None)
>>> dob.baz = {'a': 1, 'b': {'c': 3, 'd': 4}}
>>> dob.convert_dicts()
>>> dob
 DictObject{'foo': 1, 'bar': 2, 'baz': DictObject{'a': 1, 'b': {'c': 3, 'd': 4}}}
>>> dob2 = deepcopy(dob)
>>> dob.baz.a = 3
>>> dob
DictObject{'foo': 1, 'bar': 2, 'baz': DictObject{'a': 3, 'b': {'c': 3, 'd': 4}}}
dob2
DictObject{'foo': 1, 'bar': 2, 'baz': DictObject{'a': 1, 'b': {'c': 3, 'd': 4}}}
```
#### Shallow copy
In the example above,
if dob2 is created with using
```python
>>> dob2 = dob.copy()
```python
setting dob.baz.a = 3 would also change dob2.baz.a to 3
