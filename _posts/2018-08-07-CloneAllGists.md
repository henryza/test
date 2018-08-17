---
layout: post
category: github
title: How to download all your gists
tagline: by Henry
tags:
  - github
published: true
---

# How to downlaod all your gists
Gists are handy, but can be burdensome to look after.
This simple [script/exe](https://github.com/henryza/DownloadAllGists) makes a clone of all your gists into the
local running folder.

## Python Script to download Gists
``` python
#!/usr/bin/python
# -*- coding: <utf-8> -*-
from __future__ import print_function
import requests
from subprocess import call
import argparse

class getGists(object):

	def main(self, username):
		r = requests.get('https://api.github.com/users/{}/gists'.format(username))
		for i in r.json():
			call(['git', 'clone', i['git_pull_url']])
			description_file = './{0}/description.txt'.format(i['id'])
			with open(description_file, 'w') as f:
				f.write('{0}\n'.format(i['description']))

if __name__ == "__main__":
	argParser = argparse.ArgumentParser()
	argParser.add_argument("-u","--username", dest="username", default=False, help="Git Usernameh")
	args = argParser.parse_args()
	m = getGists()
	m.main(args.username)
```
