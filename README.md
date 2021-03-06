# Base info:

Forked from: https://github.com/pawelrychlik/jira-dependency-graph

# Changes:

Added more info about ticket to graph:
* sprint name;
* status;
* estimate;
* assignee.

Added next cmd arguments:

* `--include-only-tickets-linked-by-type` - include only tickes that are linked with specific link type. It can be useful when we plan sprint and want to see graph of sprint tickets with blockers (we are not interested in tickets that were cloned or are related, only blockers);
* `--estimate_field_name` - specifies field name with estimate information;
* `--sprint_field_name` - specifies field name with sprint information;


# How to use:

1. Install docker on your PC: 

Windows: https://hub.docker.com/editions/community/docker-ce-desktop-windows

Mac: https://docs.docker.com/v17.12/docker-for-mac/install/#download-docker-for-mac

2. Example of usage with docker: 
```
docker run --rm shnax0210/jira-dependency-graph python jira-dependency-graph.py --user=jirauser --password=jirapassword --jira=https://jira.myproject.com --include-only-tickets-linked-by-type 'is blocked by' --include-only-tickets-linked-by-type 'blocks' --exclude-link 'is blocked by' --ignore-subtasks --word-wrap --estimate_field_name 'customfield_10004' --sprint_field_name 'customfield_10300' --local TASK-1 TASK-2 TASK-3 TASK-4 | dot -Tpng > result_graph.png
```

To use above example please change next cmd arguments: `--user`, `--password` `--jira` and list of tickets(TASK-1 TASK-2 TASK-3 TASK-4). Result will placed in `result_graph.png` file to currect folder. Also it is most likelly that your project will have another identifiers for estimate and sprint fields (since this fields are custom), so to you also need to change `-estimate_field_name 'customfield_10004' --sprint_field_name 'customfield_10300'`.

# Original readme file:

jira-dependency-graph
=====================

Graph visualizer for dependencies between JIRA tickets. Takes into account subtasks and issue links.

Uses JIRA rest API v2 for fetching information on issues.
Uses [Google Chart API](https://developers.google.com/chart/) for graphical presentation.

Example output
==============

![Example graph](examples/issue_graph_complex.png)

Requirements:
=============
* Python 2.6+
* [requests](http://docs.python-requests.org/en/master/)

Or
* [docker](https://docs.docker.com/install/)

Usage:
======
```bash
$ git clone https://github.com/pawelrychlik/jira-dependency-graph.git
$ virtualenv .virtualenv && source .virtualenv/bin/activate # OPTIONAL
$ cd jira-dependency-graph
$ pip install -r requirements.txt
$ python jira-dependency-graph.py --user=your-jira-username --password=your-jira-password --jira=url-of-your-jira-site issue-key
```

Or if you prefer running in docker:
```bash
$ git clone https://github.com/pawelrychlik/jira-dependency-graph.git
$ cd jira-dependency-graph
$ docker build -t jira .
$ docker run -v $PWD/out:/out jira python jira-dependency-graph.py --user=your-jira-username --password=your-jira-password --jira=url-of-your-jira-site --file=/out/output.png issue-key
```

```
# e.g.:
$ python jira-dependency-graph.py --user=pawelrychlik --password=s3cr3t --jira=https://your-company.jira.com JIRATICKET-718

Fetching JIRATICKET-2451
JIRATICKET-2451 <= is blocked by <= JIRATICKET-3853
JIRATICKET-2451 <= is blocked by <= JIRATICKET-3968
JIRATICKET-2451 <= is blocked by <= JIRATICKET-3126
JIRATICKET-2451 <= is blocked by <= JIRATICKET-2977
Fetching JIRATICKET-3853
JIRATICKET-3853 => blocks => JIRATICKET-2451
JIRATICKET-3853 <= relates to <= JIRATICKET-3968
Fetching JIRATICKET-3968
JIRATICKET-3968 => blocks => JIRATICKET-2451
JIRATICKET-3968 => relates to => JIRATICKET-3853
Fetching JIRATICKET-3126
JIRATICKET-3126 => blocks => JIRATICKET-2451
JIRATICKET-3126 => testing discovered => JIRATICKET-3571
Fetching JIRATICKET-3571
JIRATICKET-3571 <= discovered while testing <= JIRATICKET-3126
Fetching JIRATICKET-2977
JIRATICKET-2977 => blocks => JIRATICKET-2451

Writing to issue_graph.png
```
Result:
![Example result](examples/issue_graph.png)


Advanced Usage:
===============

List of all configuration options with descriptions:

```
python jira-dependency-graph.py --help
```

### Excluding Links

In case you have specific issue links you don't want to see in your graph, you can exclude them:

```bash
$ python jira-dependency-graph.py --user=your-jira-username --password=your-jira-password --jira=url-of-your-jira-site --exclude-link 'is required by' --exclude-link 'duplicates' issue-key
```

The grapher will still walk the link, just exclude the edge. This especially useful for bidirectional links and you only
want to see one of them, e.g. *depends on* and *is required by*.

### Excluding Epics

In case you want to exclude walking into issues of an Epic, you can ignore them:

```bash
$ python jira-dependency-graph.py --user=your-jira-username --password=your-jira-password --jira=url-of-your-jira-site --ignore-epic issue-key
```

### Including Issues

In order to only specify issues with a certain prefix pass in `--issue-include <XXX>` and all tickets will be checked that they match the prefix `XXX`.

### Authentication

It is possible to either use the username/password combination or to login via the browser passing in `--cookie <JSESSIONID>`. This logins via the browser and is useful in scenarios where Kerberos authentication is required.

### Closed Issues

By passing in `--ignore-closed` the system will ignore any ticket that is closed.

### Multiple Issues

Multiple issue-keys can be passed in via space separated format e.g.
```bash
$ python jira-dependency-graph.py --cookie <JSESSIONID> issue-key1 issue-key2
```


Usage without Google Graphviz API:
============
If you have issues with the Google Graphviz API limitations you can use your local graphviz installation like this:

```bash
$ git clone https://github.com/pawelrychlik/jira-dependency-graph.git
$ cd jira-dependency-graph
$ python jira-dependency-graph.py --user=your-jira-username --password=your-jira-password --jira=url-of-your-jira-site --local issue-key | dot -Tpng > issue_graph.png
```

*Note*: Its possible that the graph produced is too wide if you have a number of issues. In this case, it is better to firstly pipe the graph to a 'dot' text file e.g.

```bash
$ python jira-dependency-graph.py --jira=url-of-your-jira-site --local issue-key > graph.dot
```

and then process it using `unflatten`:

```bash
unflatten -f -l 4 -c 16 graph.dot  | dot | gvpack -array_t6 | neato -s -n2 -Tpng -o graph.png
```

For a slightly cleaner layout (that preserves the ranks), or if your system doesn't have `unflatten`, you can use `sed` to insert `rankdir=LR;` into the dot file before processing it:
```bash
sed -i 's/digraph{/digraph{ rankdir=LR;/g' graph.dot | dot -o graph.png -Tpng
```

Notes:
======
Based on: [draw-chart.py](https://developer.atlassian.com/download/attachments/4227078/draw-chart.py) and [Atlassian JIRA development documentation](https://developer.atlassian.com/display/JIRADEV/JIRA+REST+API+Version+2+Tutorial#JIRARESTAPIVersion2Tutorial-Example#1:GraphingImageLinks), which seemingly was no longer compatible with JIRA REST API Version 2.
