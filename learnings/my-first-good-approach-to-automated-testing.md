# my first "good" approach to automated testing

my test automating journey begins when i saw [my dhcp starvation script](https://github.com/AliGhaffarian/dhcp-starvation-using-scapy) having too many unnoticed bugs, which to this day i havn't fixed it completely

## my first test automation experiance

this is a part of my test automating for my dhcp starvation script

```python3
from scapy.all import *
import dhcp #my script

... #making the bad_dhcp_list and not_dhcp variables

for packet in bad_dhcp_list:
    assert dhcp.is_dhcp_offer(packet) == False
    assert dhcp.is_dhcp_ack(packet) == False

assert dhcp.is_dhcp(not_dhcp) == False

```

as you can see theres a lot of possiblities for packets to not be a dhcp offer or dhcp pdu at all, and it's nearly impossible for me to write them all
this introduced the dataset generation to me, which i was trying to automate as much as possible ever since
 

after some time i started to feel the need to automate linux interface configuration
this time i knew if i don't have a good automated testing i can't really use my own software because of unnoticed bugs

going forward with [my linux automation script](https://github.com/AliGhaffarian/reface2/blob/main/reface2/utilities/pyroute2_utilities.py) i wrote this function
let this be our testing target
```python3
set_host_data(ifname, ip, mac, netmask=32, ttl=None, mtu=None)
```

the function takes 6 parameters, thefore the number of possibilities in terms of:
1. whether passed parameters are valid or not
2. if not for what reason

is like this
```python3
function_call_possibilities = ifname_possibilities * ip_possibilities * mac_possibilities * netmask_possibilities * ttl_possibilities * mtu_possibilites
```

if every parameter has two possibilities of being invalid and one possiblity of being valid, we need to write 3^6(729) sets of arguments for the function to be called with and validate the outcome appropriately

if i find a way to introduce all the possiblities of a argument being invalid and have the dataset automated i can get away with writing 3\*6 (18) sets of data and get on with my day

## my first "good" approach

when we test a function by calling we usually have a mindset like this

* **if arguments are correct:**
	i expect this function to do it's job and return with status success
* **if argument are incorrect:**
	i expect this function to exit with the status being a subset of my arguments "reason of invalidation" and (if is expected to be atomic) undo everything it did until it encountered the invalid argument

we can implement this in our tester script
by having generating a list with each entry containing a set of argument for the function call and have the expected return status as a list
much like this
```python3
[
(arg1,arg2,[expected err codes]),
(arg1_1,arg2_2,[expected err codes])
]
```

**for us to automate the generation this dataset we need to:**
1. write an instance of each reason of invalidation for a parameter of a function
2. generate the list of function call data like this
[(one entry of product of product(arg1_list, arg2_list), union of each argument of this entry's expected status)]

### example
#### 1_writing arguments
```python3
params = {
        "ifname" : INVALID_IFNAME_LIST + VALID_IFNAME_LIST,
        "ttl" : INVALID_TTL_LIST + VALID_TTL_LIST,
        "mac" : INVALID_MAC_LIST + VALID_MAC_LIST,
        "ip" : VALID_IPV4_LIST ,
        "ipv4_net_mask" : INVALID_IPV4_NETMASK_LIST + VALID_IPV4_NETMASK_LIST,
        "mtu" : VALID_MTU_LIST + INVALID_MTU_LIST
        }
```
with each entry of each list like this
```python3
(argument, [expected error number])
```
for instance
```python3
INVALID_MAC_LIST.append(("FF:FF:FF:FF:FF:FF", errno.EADDRNOTAVAIL))
```


#### 2_generating dataset for set_host_data()
```python3
import itertools
def generate_pytest_params(*arrays):
    """
    Generates test parameter combinations.

    Takes arrays of 2-entry tuples and returns an array of tuples with combinations
    of the first entry from each input array and a union of the second entries.

    Parameters:
    *arrays : one or more lists of 2-entry tuples.

    Returns:
    List of tuples containing combinations of the first entries and a list of second entries.
    """
    combined = []
    for combination in itertools.product(*arrays):
        first_entries = tuple([t[0] for t in combination])
        second_entries = [t[1] for t in combination]
        second_entries = list(dict.fromkeys(second_entries))
        if 0 in second_entries:
            second_entries.remove(0)
        iter_result = first_entries + ((second_entries,))
        combined.append(iter_result)
    return combined
```

note that we remove any error code 0 of the expected error codes, to remove any chance of mistakenly accepting a succeeded function which was'nt supposed to

### function call time!
for this we pytest has our back with automated function calls which is quite nice

```python3
import pyroute2
import pytest
@pytest.mark.parametrize("ifname, ip, mac, netmask, ttl, mtu, expected_exception_codes", generate_pytest_params(params['ifname'], params['ip'], params['mac'], params['ipv4_net_mask'], params['ttl'], params['mtu']))
def test_set_host_data(ifname, ip, mac, netmask, ttl, mtu, expected_exception_codes):
    logger.debug(f"called with {ifname=}, {ip=}, {mac=}, {ttl=}, {mtu=}, {expected_exception_codes=}")
    
    try:
        pyroute2_utilities.set_host_data(ifname=ifname, mac=mac, ip=ip, netmask=netmask, ttl=ttl, mtu=mtu)
    except Exception as e:
        assert e.args[0] in expected_exception_codes
        return
		#if we don't catch an exception the list of expected error codes must be empty
    assert len(expected_exception_codes) == 0
```

this made my testing automation much easier specially when some of the argument of already tested function overlap with the future functions that need testing

## problems with this approach
### where this doesn't work
for this approach to work we need the function to throw exceptions and not return the error code 
### where this is not enough
and we didn't automate the atomicity of the function or side effect other than the error code
