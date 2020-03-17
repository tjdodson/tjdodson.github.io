---
layout: post
title: Dotenv Files for Simpler and Safer Script Development
date: 2019-10-31 00:00:0 -0500
description: # Add post description (optional)
img: user_authentication.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Environment Variables, dotenv, Python, Elasticsearch]
---

![How Hacking Works]({{site.baseurl}}/assets/img/how_hacking_works.png){: .center-image }

We have all been there. We are configuring authorization credentials or API tokens to make a script process work. In my day to day the most common occurrence of this is when I am pulling or pushing data to Elasticsearch. I will often implement a getpass or input line to allow me to type these things and prevent from hard-coding my username and password within the script. Now with longer passwords this isn't ideal for running down a proof of concept, and inevitably I get lazy. Before I know it my credentials are there saved as a simple string.

:pensive:

I am ashamed, but I move on. I'll just delete that line before updating the repo...

:scream:

And, there is my password for anyone with access to the enterprise GitHub to see (or public repo if just using your personal GitHub). Adding sensitive information to code takes a second, and is just as easy to forget when committing, and GitHub repos remember EVERYTHING!!! Now, I can write another post on how to rectify this mistake when it occurs. I am currently in the process of going through and identifying repositories that are currently compromised or probably were at some point, but I want to make this issue a thing of the past, while still being as user friendly and unconfusing as possible. Everything I am about to show you can be handled in the command line, but that tends to be daunting.

So, how can we fix this?

===============================================================================

```
        _______ .__   __. ____    ____
       |   ____||  \ |  | \   \  /   /
       |  |__   |   \|  |  \   \/   /
       |   __|  |  . `  |   \      /
    __ |  |____ |  |\   |    \    /
   (__)|_______||__| \__|     \__/
```

I started using environment variables when building Django and Flask applications, and I knew this could be the way forward to the simple script management workflow I had always wished to obtain.

[From the docs](https://pypi.org/project/python-dotenv/):

python-dotenv

Reads the key-value pair from `.env` file and adds them to environment
variable. It is great for managing app settings during development and
in production using [12-factor](http://12factor.net/) principles.

> Do one thing, do it well!

## Usages

The easiest and most common usage consists on calling `load_dotenv` when
the application starts, which will load environment variables from a
file named `.env` in the current directory or any of its parents or from
the path specificied; after that, you can just call the
environment-related method you need as provided by `os.getenv`.

`.env` looks like this:

```shell
# a comment that will be ignored.
REDIS_ADDRESS=localhost:6379
MEANING_OF_LIFE=42
MULTILINE_VAR="hello\nworld"
```

You can optionally prefix each line with the word `export`, which is totally ignored by this library, but might allow you to [`source`](https://bash.cyberciti.biz/guide/Source_command) the file in bash.

```
export S3_BUCKET=YOURS3BUCKET
export SECRET_KEY=YOURSECRETKEYGOESHERE
```

`.env` can interpolate variables using POSIX variable expansion,
variables are replaced from the environment first or from other values
in the `.env` file if the variable is not present in the environment.
(**Note**: Default Value Expansion is not supported as of yet, see
[\#30](https://github.com/theskumar/python-dotenv/pull/30#issuecomment-244036604).)

```shell
CONFIG_PATH=${HOME}/.config/foo
DOMAIN=example.org
EMAIL=admin@${DOMAIN}
```

===============================================================================

## Simple Use Case
You can read the rest of the [docs](https://pypi.org/project/python-dotenv/), but I would like to show you a simple way to create an Elasticsearch client connection without being caught with your pants down.

Typically, you will create the `.env` within your project directory, and call it with:
#### Example Flask Directory Structure
```bash
  └── flask_project/
        ├── __init__.py
        ├── models/
              ├── __init__.py
              ├── base.py
              ├── users.py
              ├── posts.py
              ├── ...
        ├── routes/
              ├── __init__.py
              ├── home.py
              ├── account.py
              ├── dashboard.py
              ├── ...
        ├── templates/
              ├── base.html
              ├── post.html
              ├── ...
        ├── .env
        └── app.py
```

#### Example Values in `.env`
  ```shell
  uname = 't0d00bh'
  upass = 'bronies_4_ever!'
  ```

#### Loading Environment Variables from `.env` within `app.py`
  ```python
  import os
  from dotenv import load_dotenv
  load_dotenv()

  uname = os.getenv("uname") # Returns the string 't0d00bh'
  upass = os.getenv("upass") # Returns the string 'bronies_4_ever!'
  ```

Now you can use the uname and upass variable to pass credentials for various connections you may want to use in your apps/scripts. However, you still have to remember to add `.env` to your `.gitignore` file.

## Making .env Modular
In addition to potentially forgetting to add `.env` to your `.gitignore` file, the problem with the above approach is that anytime credentials change (or you move the machine running the Elasticsearch instance), you will have to go into all of your projects and edit the `.env` file to reflect those changes.

So my solution as of now is very simple, one `.env` file at a common location to be loaded upon import for all scripts. In python you can see all of your machines environment variables by running:
```python
import os
os.environ
```

Or if you want to print it in a slightly nicer format:
```python
import os
import pprint
pprint.pprint(dict(os.environ))
```

Peruse that and you will most likely see a key similar to ['HOME'](https://en.wikipedia.org/wiki/Home_directory#Default_home_directory_per_operating_system). Per wikipedia:
>A [home directory](https://en.wikipedia.org/wiki/Home_directory) is a file system directory on a multi-user operating system containing files for a given user of the system.

#### Default Home Directory Per Operating System
```shell
|           Operating system            |          Path           |
| ------------------------------------- | ----------------------- |
| Microsoft Windows Vista, 7, 8 and 10  | <root>\Users\<username> |
| Unix-based                            | <root>/home/<username>  |
| Linux / BSD (FHS)                     | /home/<username>        |
| macOS                                 | /Users/<username>       |
```

A common location provides an opportunity to create an OS agnostic import load of environment variables that only persists as long as the script is running. This means you can refer to your sensitive information using only generically named variables and put your worries behind of sharing sensitive information with any [Tom, Dick, or Harry](https://en.wikipedia.org/wiki/Tom,_Dick_and_Harry).

## Setting Up Elasticsearch Client

#### Example Values in `.env`
  ```shell
  esurl = 'https://hazr1num3letters.cloud.wal-mart.com:9200'
  uname = 't0d00bh'
  upass = 'bronies_4_ever!'
  certs = '/path/to/es_certs.pem'
  ```

#### Example Python Script
  ```python
  import os
  from dotenv import load_dotenv
  from elasticsearch import Elasticsearch

  load_dotenv(dotenv_path=os.path.join(os.getenv("HOME"),'.env'))

  esurl = os.getenv("esurl")
  uname = os.getenv("uname")
  upass = os.getenv("upass")
  certs = os.getenv("certs")

  client = Elasticsearch(esurl, http_auth=(uname, upass),
  use_ssl=True, ca_certs=certs, verify_certs=True)
  ```

![Tada!]({{site.baseurl}}/assets/img/tada.gif){: .center-image }
