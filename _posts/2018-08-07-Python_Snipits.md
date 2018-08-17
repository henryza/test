---
layout: post
category: python
title: python Snips
tagline: by Henry
tags:
  - python
  - snips
published: true
---

# Python Snipits

# How to cleat the screen
``` Python
import os
os.system('cls')  # on windows
os.system('clear')  # on linux / os x
```

## Python the date thing
``` powershell
import datetime
dDate = datetime.datetime.now().strftime("%m/%d/%Y %I:%M:%S %p")
```

## Permenently delete files by date
``` Python
#!/usr/bin/python
# -*- coding: <utf-8> -*-
from __future__ import print_function
import os
import time

current_time = time.time()

for file in os.listdir():
    creation_time = os.path.getctime(file)
    if (current_time - creation_time) // (24 * 3600) >= 7:
        os.unlink(file)
        print('{} removed'.format(f))
```

## Clean backslashes
``` Python
def trailingBackSlash(location):
    if location[-1:] != "/" and "/" in location:
        location = location + "/"
    elif location[-1:] != "\\"  "\\" in location:
        location = location + "\\"
```

## Json examples in Python
``` python
#!/usr/bin/python
# -*- coding: <utf-8> -*-
from __future__ import print_function
import Json

dataList = []
dataDic = {}
dataDic["node1"] = "node1"
dataList.append(dataDic)

dataDic = {}
dataList.append(dataDic)
dataDic["node1"] = "node1"
dataList.append(dataDic)

jsonDATA = json.dumps(dataList, encoding="utf-8", indent=4)

node1 = dataList[0]["node1"]

open(filePath, 'w').write(josnDATA)
loadedJson = json.loads(jsonDATA)
```

## Python Json with namespaces
``` Python
#!/usr/bin/python
# -*- coding: <utf-8> -*-
from __future__ import print_function
try:
    from types import SimpleNamespace as Namespace
except ImportError:
    # Python 2.x fallback
    from argparse import Namespace

dataDic = json.loads(data, object_hook=lambda d: Namespace(**d))
dataDic.NodeName #access the item like this
```

## Python example setup logging
``` Python
#!/usr/bin/python
# -*- coding: <utf-8> -*-
from __future__ import print_function
import logging
def setupLogging(self, loglevel="CRITICAL"):
    name = "Zone-ApiPush"
    location = "./Logs/"
    # Get Date for the logfile
    dDate = datetime.datetime.now().strftime("%d-%m-%Y")
    filename = location + name + "_" + dDate + ".txt"
    fmt = "%(asctime)s - %(levelname)s - FUNCTION:%(funcName)s() - LINE:%(lineno)s - %(message)s"
    self.logger = logging.getLogger(__name__)
    fileHandler = logging.StreamHandler()
    fileHandler.setFormatter(logging.Formatter(fmt))
    streamHandler = logging.FileHandler(filename)
    streamHandler.setFormatter(logging.Formatter(fmt))
    self.logger.addHandler(fileHandler)
    self.logger.addHandler(streamHandler)
    self.logger.setLevel(loglevel.upper())
```

## Python draft
``` python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from __future__ import print_functio

Class testclass(object):
  __doc__ = """ """

  def init(self):
    pass

  def logic(self,input)
    pass

if __name__ == "__main__":
  pass
```

## Python argparse example
``` Python
class test(object, applicationpush, applicationpull, update, configlocation, loglevel):
  pass

if __name__ == "__main__":
    pyA = test()
    argParser = argparse.ArgumentParser()
    argParser.add_argument("--push", dest="applicationpush", action="store_true",  default = False,
                           help="To be selected if type applicationpush")
    argParser.add_argument("--pull", dest="applicationpull", action="store_true", default = False,
                           help="To be selected if type applicationpull")
    argParser.add_argument("--api", dest="apipush", action="store_true", default = False,
                           help="To be selected if type apipush")
    argParser.add_argument("--update", dest="update", action="store_true", default = False,
                           help="To be selected if type update")
    argParser.add_argument("-c", "--configlocation", dest="configlocation", default = "./pyAgentConfig.conf",
                           help="Location of the application config")
    argParser.add_argument("-l", "--loglevel", dest="loglevel", default="CRITICAL", help="Level for logging")
    args = argParser.parse_args()

    pyA.main(args.applicationpush, args.applicationpull, args.apipush, args.update, args.configlocation, args.loglevel)
```

## Python build file locations from input string
``` Python
def buildLocation(self, dataFileLocaiton):
        #Make sure paths exist, build if they do not
        location = []
        # CASE path contains /, seperate slashes to build path
        if dataFileLocaiton.find("/") > 0:
            Slash = dataFileLocaiton.find("/")+1
            counter = 0
            while (counter) < dataFileLocaiton.count("/") :
                if counter == 0:
                    location.append(dataFileLocaiton[:Slash])
                else:
                    Slash = dataFileLocaiton.find("/", Slash) + 1
                    location.append(dataFileLocaiton[:Slash])
                counter = counter + 1

        # CASE path contains \\, seperate slashes to build path
        elif dataFileLocaiton.find("\\") > 0:
            Slash = dataFileLocaiton.find("\\") + 1
            counter = 0
            while (counter) < dataFileLocaiton.count("\\"):
                if counter == 0:
                    location.append(dataFileLocaiton[:Slash])
                else:
                    Slash = dataFileLocaiton.find("\\", Slash) + 1
                    location.append(dataFileLocaiton[:Slash])
                counter = counter + 1

        #Loop through locations and create if not existing
        for path in location:
            if not(os.path.exists(path)):
                try:
                    os.mkdir(path)
                except Exception as e:
                    self.logger.exception("Unable to create path {}".format(path)
```

## Python Hashes
``` Python
import hashlib
hashlib.md5("hash this").hexdigest()
```

## Check data
This I usually use when I have multiple inputs
``` Python
def checkValues(self, itemName, itemValue):
  if isinstance(s, str):
        if itemValue != None:
          if itemValue.strip() != "":
            return itemValue
          else:
            print("{} is empty".format(itemName))
        else:
          print("{} is empty".format(itemName))


    elif isinstance(s, unicode):
        s = itemValue.encode("utf-8")
        self.checkValue(itemName, itemValue)

    elif isinstance(s, int):
      pass
    else:
      pass
```

## Ptyhon and Base64
``` python
import base64
s = "this is a string"
base64.b64encode(s)
base64.b64decode(s)
```

## Python encryption
[AES Encryption](https://eli.thegreenplace.net/2010/06/25/aes-encryption-of-files-in-python-with-pycrypto)
[RSA Encryption](https://stackoverflow.com/questions/30056762/rsa-encryption-and-decryption-in-python)
[RSA Helper Class](https://gist.github.com/dennislwy/0194036234445776d48ad2fb594457d4)




## Common used Python Libraries
| Installations             |Use                            |
|---------------------------|-------------------------------|
|pip install pywin32        | Windows api                   |
|pip install pyisntaller    | convers python to exe         |
|pip install requests       | This is a windows web interface|
| pip install pika          | This is a library for Rabbitmq|
| Beautiful Soup            | Web scraping                  |
