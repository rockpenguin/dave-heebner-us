---
title: "Python Third Party Module Exceptions / Errors"
date: 2017-05-30
categories: 
  - "recipes"
  - "techsupport"
---

So after a couple days of searching, I still couldn't find a way to catch a particular exception from the Apache [libcloud](https://libcloud.apache.org) module. Chalk it up to my inexperience with Python, but why couldn't I find anything on the Interwebs in regards to 3rd party module exceptions? Searches were coming up with nuttin' useful.<!--more-->

So, here is the code I was testing:

```
key_pair = do_cxn.create_key_pair('my_keypair', key_data)
```

I knew it was going to fail because there was an equivalent key in the Digital Ocean account and here is the error I received:

```
libcloud.common.exceptions.BaseHTTPError: SSH Key is already in use on your account (code: 422)
```

So I wanted to wrap this up in a try/except stanza, but no matter what I put as the exception it wouldn't work.

I finally found a Stackoverflow answer that was unrelated, but it worked. The answer was to just use "Exception", e.g.:

```
try:
    key_pair = do_cxn.create_key_pair('my_keypair', key_data)
except Exception as e:
    # looks like the key already exists
    print(" Mr. T says, 'The key already exists, fool!' -> %s" % e)
```

But that feels to me almost as wrong as just using an empty **except:** line, like it's going to catch the desired exception (key already exists) as well as other exceptions.

```
try:
    key_pair = do_cxn.create_key_pair('my_keypair', key_data)
except:
    # looks like the key already exists
    print('The key already exists, fool! -> %s' % e)
```

Basically I want libcloud to create the key if it doesn't exist. Hmmmm.

### Update

Looks like I just needed to import the right class from the base module:

```
# exception handling
from libcloud.common.exceptions import BaseHTTPError
```

Now is it wrong to do **import \*** instead? I dunno. But here is the final try/except:

```
try:
    key_pair = conn.create_key_pair('my_keypair', key_data)
except BaseHTTPError as e:
    print('The key already exists, fool! -> %s' % e)
```

Yay!
