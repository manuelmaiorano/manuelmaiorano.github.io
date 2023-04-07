---
title: "PR recommendation tool"
date: 2023-1-16
categories: Python Tools
---
This project was the final assignment of a Software Engeneering course at my university. I got to learn a lot of new things that before i didn't know or that i undervalued, like testing, version control, project management, automation. In particular on this last point i was made aware of the importance of a script that automatically does everything that is necessary (testing, integration, deployement, documentaton updates) with a single mouse click.

This tool given a pull request on github automatically recommends a list of possible reviewers. The way it does it is based on simple information retriveal algorithms. It downloads a certain number of previous pull requests and runs a tokenizer with a regular expression that matches import statements in various programming languages. It then generates an inverted index data structure for fast retriveal of related document based on the relevant tokens found so far. The new pull request is processed in the same way, with imported libraries, packages, modules extracted by the tokenizer that go through the inverted index to get similar pull requests. Then the reviewer associated with those requests gets a score (the cosine similarity between the two pull request). The final output is a list of the suggested reviewer sorted from highest score to lowest.

The tool is deployed on PyPI with a github action; similarly a documentation is automatically generated at every push to the main branch using Sphinx.

Here's the way you use it as a command line tool:

```bash
usage: review_recommender [-h] owner repo num token

Given pull request, rank revisors

positional arguments:
  owner       the owner of the repository
  repo        the name of the repository
  num         the number of the pull request
  token       the github access token

optional arguments:
  -h, --help  show this help message and exit
```

and here's a sample output (i tried it on the log4j repository):

```bash
Reviewer         | Score      
-----------------------------
vy               | 24.66 %
rgoers           | 19.96 %
DongjianPeng     | 15.44 %
dafengsu7        | 9.98 %
schlosna         | 9.98 %
jvz              | 9.98 %
marcwrobel       | 9.98 %

```