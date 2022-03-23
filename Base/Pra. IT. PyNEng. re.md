# Модуль re
id: 202203111258
__
## Match object

Example:

```
var = 'some txt'

match = re.search(r'txt1 (\rex1) txt2 (\rex2) ... txtN (\rexN)', var)

<_sre.SRE_Match object; span=(##, ###), match='txt1 val1 txt2 val2 ...>'
```

```
match.group() # or match.group(0)
'txt1 val1 txt2 val2 ... txtN valN'
```

```
match.group(1)
'val1'

match.group(2)
'val2'

match.group(N)
'valN'

```

```
match.group(1, 2, N)
('val1', 'val2', 'valN')
```

If a group number is larger than number of existing groups, there is an error:

```
match.group(5)
IndexError                      Traceback (most recent call last)
<ipython-input-18-9df93fa7b44b> in <module>()
----> 1 match.group(5)

IndexError: no such group
```

Group may not get anything, then it will be matched with an empty string:

```
match.group(#)
''
```

If group describes a part of template and there are more than one match, method displays the last match:

```
var = 'txt1 ...1....2....N txtN'

match = re.search(r'txt1 (\rex1{4})\.', var)

match.group(1)
'...N'
```

named groups used in expression

```
var = 'txt1 val1 txt2 val2 txt2 ... valN txt3'

match = re.search(r'txt1 (?P<name1>\rex1)'
                  r'txt2 (?P<name2>\rex2)'
                  r'txtN (?P<name3>\rexN)',
                  var)

match.group('name1')
'val1'

match.group('nameN')
'valN'
```

Groups are also available via number:

```
match.group(1)
'val1'

match.group(N)
'val2'
```

Method `groups` returns a tuple with strings

```
match.groups()
('val1', 'val2',..., 'valN')
```

Method `groups` has an optional parameter - default. 
If there is nothing will get into the group. group is optional: `(\rex)?`

```
match = re.search(r'(\rex1) +(\rexN)?', var)

match.groups()
('val1', None)
```

```
match.groups(default='No match')
('val1', 'No match')
```

### groupdict

Method `groupdict` returns a dictionary group names : corresponding lines:
```
match.groupdict()
{'int1': 'Gi0/5', 'int2': 'Gi0/15', 'mac': 'f03a.b216.7ad7', 'vlan': '10'}
```

### start, end, span
`start`, `end`, `span` methods return indexes of the beginning and end of the match of regex.
```
match.start()
##

match.end()
##

match.span()
(##, ##)

line[match.start():match.end()]
'val1     val2    valN'
```

```
match.{start/end/span}(N) # or (nameN)
## (, ##)
```

## Search function

Function `search`:

-   is used to find a substring that matches a template
-   returns Match object if a substring is found
-   returns `None` if no substring was found

Function `search` is suitable when you need to find only one match in a string, for example when a regex describes the entire string or part of a string.

Consider an example of using `search` function to parse a log file. File log.txt contains log messages indicating that the same MAC is too often re-learned on one or another interface. One of the reasons for these messages is loop in network.

Contents of log.txt file:
```
%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in vlan 10 is flapping between port Gi0/16 and port Gi0/24
%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in vlan 10 is flapping between port Gi0/16 and port Gi0/24
%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in vlan 10 is flapping between port Gi0/24 and port Gi0/19
%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in vlan 10 is flapping between port Gi0/24 and port Gi0/16
```

MAC address can jump between several ports. In this case it is very important to know from which ports MAC comes.

Try to figure out which ports and which VLAN was the problem. Check regex with one line from log file:
```
import re

log = '%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in vlan 10 is flapping between port Gi0/16 and port Gi0/24'

match = re.search(r'Host \S+ '
                   r'in vlan (\d+) '
                   r'is flapping between port '
                   r'(\S+) and port (\S+)', log)
```

Regex is divided into parts for ease of reading. It has three groups:

-   `(\d+)` - describes VLAN number
-   `(\S+) and port (\S+)` - describes port numbers

As a result, the following parts of line fell into the groups:

```
match.groups()
('10', 'Gi0/16', 'Gi0/24')
```

In the resulting script, log.txt is processed line by line and port information is collected from each line. Since ports can be duplicated we add them immediately to the set in order to get a compilation of unique interfaces (parse_log_search.py file):

```
import re

regex = ('Host \S+ '
         'in vlan (\d+) '
         'is flapping between port '
         '(\S+) and port (\S+)')

ports = set()

with open('log.txt') as f:
    for line in f:
        match = re.search(regex, line)
        if match:
            vlan = match.group(1)
            ports.add(match.group(2))
            ports.add(match.group(3))

print('Loop between ports {} in VLAN {}'.format(', '.join(ports), vlan))
```

The result of script execution:
```
python parse_log_search.py
Loop between ports Gi0/19, Gi0/24, Gi0/16 в VLAN 10
```

## Processing of ‘show cdp neighbors detail’ output

Try to get device parameters from ‘sh cdp neighbors detail’ output.
Example of output for one neighbor:

```SW1#show cdp neighbors detail
...
Device ID: SW2
Entry address(es):
  IP address: 10.1.1.2
Platform: cisco WS-C2960-8TC-L,  Capabilities: Switch IGMP
Interface: GigabitEthernet1/0/16,  Port ID (outgoing port): GigabitEthernet0/1
...
Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 12.2(55)SE9, RELEASE SOFTWARE (fc1)
...
  IP address: 10.1.1.2
```
The goal is to get such fields:

-   neighbor name (Device ID: SW2)
-   IP address of neighbor (IP address: 10.1.1.2)
-   neighbor platform (Platform: cisco WS-C2960-8TC-L)
-   IOS version (Cisco IOS Software, C2960 Software (C2960-LANBASEK9-M), Version 12.2(55)SE9, RELEASE SOFTWARE (fc1))

And for convenience you need to get data in the form of a dictionary. Example of the resulting dictionary for SW2 switch:
```
{'SW2': {'ip': '10.1.1.2',
         'platform': 'cisco WS-C2960-8TC-L',
         'ios': 'C2960 Software (C2960-LANBASEK9-M), Version 12.2(55)SE9'}}
```
Example is checked on file sh_cdp_neighbors_sw1.txt.

The first solution (parse_sh_cdp_neighbors_detail_ver1.py file):
```
import re
from pprint import pprint

def parse_cdp(filename):
    result = {}

    with open(filename) as f:
        for line in f:
            if line.startswith('Device ID'):
                neighbor = re.search('Device ID: (\S+)', line).group(1)
                result[neighbor] = {}
            elif line.startswith('  IP address'):
                ip = re.search('IP address: (\S+)', line).group(1)
                result[neighbor]['ip'] = ip
            elif line.startswith('Platform'):
                platform = re.search('Platform: (\S+ \S+),', line).group(1)
                result[neighbor]['platform'] = platform
            elif line.startswith('Cisco IOS Software'):
                ios = re.search('Cisco IOS Software, (.+), RELEASE',
                                line).group(1)
                result[neighbor]['ios'] = ios

    return result

pprint(parse_cdp('sh_cdp_neighbors_sw1.txt'))
```
The desired strings are selected using startswith() string method. And in a string, a regex takes required part of the string. It all ends up in a dictionary.
The result is:
```
python parse_sh_cdp_neighbors_detail_ver1.py
{'R1': {'ios': '3800 Software (C3825-ADVENTERPRISEK9-M), Version 12.4(24)T1',
        'ip': '10.1.1.1',
        'platform': 'Cisco 3825'},
 'R2': {'ios': '2900 Software (C3825-ADVENTERPRISEK9-M), Version 15.2(2)T1',
        'ip': '10.2.2.2',
        'platform': 'Cisco 2911'},
 'SW2': {'ios': 'C2960 Software (C2960-LANBASEK9-M), Version 12.2(55)SE9',
         'ip': '10.1.1.2',
         'platform': 'cisco WS-C2960-8TC-L'}}

```
It worked out well, but it can be done in a more compact way.
The second version of solution (parse_sh_cdp_neighbors_detail_ver2.py file):
```
import re
from pprint import pprint

def parse_cdp(filename):
    regex = ('Device ID: (?P<device>\S+)'
             '|IP address: (?P<ip>\S+)'
             '|Platform: (?P<platform>\S+ \S+),'
             '|Cisco IOS Software, (?P<ios>.+), RELEASE')

    result = {}

    with open(filename) as f:
        for line in f:
            match = re.search(regex, line)
            if match:
                if match.lastgroup == 'device':
                    device = match.group(match.lastgroup)
                    result[device] = {}
                else:
                    result[device][match.lastgroup] = match.group(
                        match.lastgroup)

    return result

pprint(parse_cdp('sh_cdp_neighbors_sw1.txt'))

```
Explanations for the second option:

-   in regex, all lines written via `|` sign (or)
-   if a match is found, `lastgroup` method is checked
-   `lastgroup` method returns name of the last named group in regex for which a match has been found
-   if a match was found for `device` group, the value that fells into the group is written to `device` variable
-   otherwise the mapping of `'group name': 'corresponding value'` is written to dictionary

Result will be the same

# Match function

Function `match`:

-   is used to search at the beginning of string that corresponds to regex
-   returns Match object if match is found
-   returns `None` if no match was found

`Match` function differs from `search` in that `match` always looks for a match at the beginning of the line. For example, if you repeat the example that was used for `search` function, but with `match`:

In [2]: import re

In [3]: log = '%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in vlan 10 is flapping between port Gi0/16 and port Gi0/24'

In [4]: match = re.match(r'Host \S+ '
   ...:                  r'in vlan (\d+) '
   ...:                  r'is flapping between port '
   ...:                  r'(\S+) and port (\S+)', log)
   ...:

The result will be None:

In [6]: print(match)
None

This is because `match` searches for Host word at the beginning of the line. But this message is in the middle.

In this case it is easy to fix expression so that match() function finds match:

In [4]: match = re.match(r'\S+: Host \S+ '
   ...:                  r'in vlan (\d+) '
   ...:                  r'is flapping between port '
   ...:                  r'(\S+) and port (\S+)', log)
   ...:

Expression `\S+:` was added before _Host_ word. Now match will be found:

In [11]: print(match)
<_sre.SRE_Match object; span=(0, 104), match='%SW_MATM-4-MACFLAP_NOTIF: Host 01e2.4c18.0156 in >

In [12]: match.groups()
Out[12]: ('10', 'Gi0/16', 'Gi0/24')

Example is similar to one used in `search` function with minor changes (parse_log_match match.py file):

import re

regex = (r'\S+: Host \S+ '
         r'in vlan (\d+) '
         r'is flapping between port '
         r'(\S+) and port (\S+)')

ports = set()

with open('log.txt') as f:
    for line in f:
        match = 

re.match(regex, line)
        if match:
            vlan = match.group(1)
            ports.add(match.group(2))
            ports.add(match.group(3))

print('Loop between ports {} in VLAN {}'.format(', '.join(ports), vlan))

The result is:

$ python parse_log_match.py
Loop between ports Gi0/19, Gi0/24, Gi0/16 in VLAN 10

__

***Status***
#pro #play

***Type*** 
#pro #it #job #edu #resume

***Prior***
#pra #pro #event #job #main #critical #middle

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
__

***Goal***
Knowledge, Habbits, Effectivity, Productivity

***Result***
To be able to solve all tasks in course and to use in job practice
__
 
 ***Mnemonika***
__

### Keys: 
[[IT]] [[Python]] [[PyNEng]] [[re]] [[code reapeat]]