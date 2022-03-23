# Модуль re
id: 202203111258
__

## Match object

Example of Match object:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (\S+) in vlan (\d+) .* port (\S+) and port (\S+)', log)

<_sre.SRE_Match object; span=(47, 124), match='Host f03a.b216.7ad7 in vlan 10 is flapping betwee>'
```

The 3rd line output simply displays information about object. Therefore, it is not necessary to rely on what is displayed in match part as displayed line is cut by a fixed number of characters.

## group

Method `group` returns a substring that matches an expression or an expression in a group.

If method is called without arguments, the whole substring is displayed:

```
match.group()
'Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'
```

The same result returns group 0:

```
match.group(0)
'Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'
```

Other numbers show only the contents of relevant group:

```
match.group(1)
'f03a.b216.7ad7'

match.group(2)
'10'

match.group(3)
'Gi0/5'

match.group(4)
'Gi0/15'
```

If you call a `group` method with a group number that is larger than number of existing groups, there is an error:

```
match.group(5)
IndexError                      Traceback (most recent call last)
<ipython-input-18-9df93fa7b44b> in <module>()
----> 1 match.group(5)

IndexError: no such group
```

If you call a method with multiple group numbers, the result is a tuple with strings that correspond to matches:

```
match.group(1, 2, 3)
('f03a.b216.7ad7', '10', 'Gi0/5')
```

Group may not get anything, then it will be matched with an empty string:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (\S+) in vlan (\D*)', log)

match.group(2)
''
```

If group describes a part of template and there are more than one match, method displays the last match:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (\w{4}.)+', log)

match.group(1)
'b216.'
```

This is because expression in parentheses describes four letters or numbers, dot and then there is a plus. The first and the second part of MAC address matched to expression in parentheses. But only the last expression is remembered and returned.

If named groups are used in expression, group name can be passed to `group` method and the corresponding substring can be obtained:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (?P<mac>\S+) '
                   r'in vlan (?P<vlan>\d+) .* '
                   r'port (?P<int1>\S+) '
                   r'and port (?P<int2>\S+)',
                   log)

match.group('mac')
'f03a.b216.7ad7'

match.group('int2')
'Gi0/15'
```

Groups are also available via number:

```
match.group(3)
'Gi0/5'

match.group(4)
'Gi0/15'
```

Method `group` returns a tuple with strings in which the elements are those substrings that fall into respective groups:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (\S+) '
                   r'in vlan (\d+) .* '
                   r'port (\S+) '
                   r'and port (\S+)',
                   log)


match.groups()
('f03a.b216.7ad7', '10', 'Gi0/5', 'Gi0/15')
```

Method `groups` has an optional parameter - default. It returned when anything that comes into group is optional.

For example, with this line the match will be in both the first group and the second:

```
line = '100     aab1.a1a1.a5d3    FastEthernet0/1'

match = re.search(r'(\d+) +(\w+)?', line)

match.groups()
('100', 'aab1')
```

If there is nothing in the line after space, nothing will get into the group. But the match will be because it is stated in regex that the group is optional:

```
line = '100     '

match = re.search(r'(\d+) +(\w+)?', line)

match.groups()
('100', None)
```

Accordingly, for the second group the value is None.

If `group` method is given a default value, it will be returned instead of None:

```
line = '100     '

match = re.search(r'(\d+) +(\w+)?', line)

match.groups(default=0)
('100', 0)

match.groups(default='No match')
('100', 'No match')
```

## groupdict

Method `groupdict` returns a dictionary in which keys are group names and values are corresponding lines:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (?P<mac>\S+) '
                   r'in vlan (?P<vlan>\d+) .* '
                   r'port (?P<int1>\S+) '
                   r'and port (?P<int2>\S+)',
                   log)


match.groupdict()
{'int1': 'Gi0/5', 'int2': 'Gi0/15', 'mac': 'f03a.b216.7ad7', 'vlan': '10'}
```

## start, end

`start` and `end` methods return indexes of the beginning and end of the match of regex.

If methods are called without arguments, they return indexes for whole match:

```
line = '  10     aab1.a1a1.a5d3    FastEthernet0/1  '

match = re.search(r'(\d+) +([0-9a-f.]+) +(\S+)', line)

match.start()
2

match.end()
42

line[match.start():match.end()]
'10     aab1.a1a1.a5d3    FastEthernet0/1'
```

You can pass number or name of the group to methods. Then they return indexes for this group:

```match.start(2)
9

match.end(2)
23

line[match.start(2):match.end(2)]
'aab1.a1a1.a5d3'
```

Similarly for named groups:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (?P<mac>\S+) '
                   r'in vlan (?P<vlan>\d+) .* '
                   r'port (?P<int1>\S+) '
                   r'and port (?P<int2>\S+)',
                   log)


match.start('mac')
52

match.end('mac')
66
```

## span
Method `span` returns a tuple with an index of the beginning and end of substring. It works in a similar way to `start` and `end` methods, but returns a pair of numbers.

Without arguments `span` returns indexes for whole match:

```
line = '  10     aab1.a1a1.a5d3    FastEthernet0/1  '

match = re.search(r'(\d+) +([0-9a-f.]+) +(\S+)', line)

match.span()
(2, 42)
```

But you can also pass number of the group:

```
line = '  10     aab1.a1a1.a5d3    FastEthernet0/1  '

match = re.search(r'(\d+) +([0-9a-f.]+) +(\S+)', line)

match.span(2)
(9, 23)
```

Similarly for named groups:

```
log = 'Jun  3 14:39:05.941: %SW_MATM-4-MACFLAP_NOTIF: Host f03a.b216.7ad7 in vlan 10 is flapping between port Gi0/5 and port Gi0/15'

match = re.search(r'Host (?P<mac>\S+) '
                   r'in vlan (?P<vlan>\d+) .* '
                   r'port (?P<int1>\S+) '
                   r'and port (?P<int2>\S+)',
                   log)


match.span('mac')
(52, 66)

match.span('vlan')
(75, 77)
```

__

***Status***
#pro #play

***Type*** 
#pro #it #job #edu

***Prior***
#pra

***Tags***
#coding #python 
__

***Source***
[[https://pyneng.readthedocs.io/en/latest/book/15_module_re/]]
[[https://pyneng.readthedocs.io/ru/latest/book/15_module_re/]]
[[Pro/IT/PyNEng]]

## Description
Python uses `re` module to work with regular expressions.

Core functions of `re` module:
-   `match` - searches a sequence at the beginning of the line
-   `search` - searches for first match with template
-   `findall` - searches for all matches with template. Returns the resulting strings as a list
-   `finditer` - searches for any matches with template. Returns an iterator
-   `compile` - compiles regex. You can then apply all of listed functions to this object
-   `fullmatch` - the entire line must conform to regex described
In addition to functions that search matches, module has the following functions:
-   `re.sub` - for replacement in strings
-   `re.split` - to split string into parts
-   [Match object](https://pyneng.readthedocs.io/en/latest/book/15_module_re/match_object.html)
-   [Search function](https://pyneng.readthedocs.io/en/latest/book/15_module_re/search.html)
-   [Match function](https://pyneng.readthedocs.io/en/latest/book/15_module_re/match.html)
-   [Finditer function](https://pyneng.readthedocs.io/en/latest/book/15_module_re/finditer.html)
-   [Findall function](https://pyneng.readthedocs.io/en/latest/book/15_module_re/findall.html)
-   [Compile function](https://pyneng.readthedocs.io/en/latest/book/15_module_re/compile.html)
-   [Flags](https://pyneng.readthedocs.io/en/latest/book/15_module_re/flags.html)
-   [Function re.split](https://pyneng.readthedocs.io/en/latest/book/15_module_re/split.html)
-   [Function re.sub](https://pyneng.readthedocs.io/en/latest/book/15_module_re/sub.html)
-   [Further reading](https://pyneng.readthedocs.io/en/latest/book/15_module_re/further_reading.html)
-   [Tasks](https://pyneng.readthedocs.io/en/latest/exercises/15_exercises.html)
__

***Goal***
Knowledge, Habbits, Effectivity, Productivity

***Result***
To be able to solve all tasks
__
 
 ***Mnemonika***
__

### Keys: 
[[IT]] [[Python]] [[PyNEng]] [[re]] [[code reapeat]]