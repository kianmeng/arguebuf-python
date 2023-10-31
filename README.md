# Arguebuf Python

Arguebuf is a format for serializing argument graphs and specified using Protobuf.
The complete specification and documentation is available at the [Buf Schema Registry](https://buf.build/recap/arg-services/docs/main:arg_services.graph.v1).
While Protobuf automatically generates native code for all major programming languages (including Python), we created a custom implementation that provides some additional benefits, including:

- The ability to import existing formats like [AIF](http://www.argumentinterchange.org), [SADFace](https://github.com/Open-Argumentation/SADFace), and a few others.
- Export of Arguebuf graphs to AIF, [NetworkX](https://networkx.org), and [Graphviz](https://graphviz.org).
- Integration with the popular NLP library [spaCy](http://spacy.io).
- Various helper methods to programmatically manipulate/create argument graphs.
- More pythonic interfaces than the regular code generated by `protoc`.

You can easily install the library from [PyPI](https://pypi.org/project/arguebuf/) using pip. The documentation is hosted on [ReadTheDocs](https://arguebuf.readthedocs.io/en/latest/)

## Command Line Interface (CLI)

We also offer some tools to simplify dealing with structured argument graphs.
Among others, it is possible to convert graphs between different formats, translate them, and render images using graphviz.
To use it, install the `cli` extras when installing the package.
When using `pip`, this can be accomplished with

`pip install arguebuf[cli]`

Afterwards, you can execute it by calling `arguebuf`, for example:

`arguebuf --help`

Alternatively, you can use the Docker image available at `ghcr.io/recap-utr/arguebuf-python`.
To use it, mount a folder to the container and pass the options as the command.

`docker run --rm -v $(pwd)/data:/data ghcr.io/recap-utr/arguebuf-python:latest --rm`

If you use the `nix` package manager, you can run it as follows:

`nix run github:recap-utr/arguebuf-python -- --help`

## Theoretical Foundations

An argument graph is way to represent _structured_ argumentation.
What sets it apart from _unstructured_ argumentation (e.g., in newspaper articles) is that only the essential/argumentative parts of a text art part of this representation.
These _units_ of argumentation are also called ADUs (argumentative discourse units).
The length of ADUs can differ dramatically (depending on various factors like the context), meaning they might contain only a few words, a sentence, or even a whole paragraph.
The structure of an argument is then represented through _relations_ between these units.
For this purpose, they can be further subdivided into _claims_ and _premises_:
A claim is a statement that is supported or attacked by one or multiple premises.
At the same time, a claim may also function as a premise for another claim, making it possible to construct even complex _argument graphs_.
One of the claims is called _major claim_ and represents the overall claim of the whole argument.
In many cases (but not all), this major claim is located right at the top of the graph
Here is a rather simple example.

![Exemplary argument graph](./assets/programmatic.png)

Claims, premises and the major claim are represented as _atom nodes_ while relations between them are represented by _scheme nodes_.
The set of nodes $V = A \cup S$ is composed of the set of atom nodes $A$ and the set of scheme nodes $S$.
The supporting or attacking relations are encoded in a set of edges $E \subseteq V \times V \setminus A \times A$.
Edges can be drawn between any type of nodes except for two atom nodes.
For instance, it is possible to connect two scheme nodes to support or attack the _inference_ between two ADUs.
Based on this, we define an argument graph $G$ as the triple $G = ( V , E , M )$.

## User Guide

When importing this library, we recommend using an abbreviation such as `ag` (for _argument graph_).

```python
import arguebuf as ag
```

In the following, we will introduce the most important features of `arguebuf`.
For more details including examples, check out our API documentation.

### Importing an Existing Graph

We support multiple established formats to represent argument graphs: `AIF`, `OVA`, `SADFace`, `ArgDown`, `BRAT`, and `Kialo`.
Given an input file, the library can automatically determine the correct format and convert it to a representation in the `arguebuf` format.
One can either pass a string pointing to the file or a `pathlib.Path` object.

```python
graph = ag.load.file("graph.json")
```

It is also possible to load multiple graphs within a folder.
Here, you need to pass the folder along with a [glob pattern](https://docs.python.org/3/library/fnmatch.html#module-fnmatch) for selecting the argument graphs.
This also enables to recursively load all argument graphs from a common parent directory.

```python
graphs = ag.load.folder("./data", "**/*.json")
```

Since atom nodes contain textual information that may need to be analyzed using NLP techniques, we support passing a custom `nlp` function to these loader methods.
This also makes it really easy to integrate the popular [`spacy` library](http://spacy.io) with `arguebuf` as all texts of atom nodes are automatically converted to a spacy `Doc`.

```python
import spacy
nlp = spacy.load("en_core_web_lg")
graph = ag.load.file("graph.json", nlp=nlp)
```

### Programmatically Create a New Graph

Instead of importing an existing graph, you can also create a new one using an object-oriented API using our library.
To illustrate this, we generate a graph with two premises that are connected to a major claim.
**Please note:** In case edges with nodes not yet contained in the graph are added, the respective nodes are added automatically.

```python
graph = ag.Graph()

premise1 = ag.AtomNode("Text of premise 1")
premise2 = ag.AtomNode("Text of premise 2")
claim = ag.AtomNode("Text of claim")

scheme1 = ag.SchemeNode(ag.Support.DEFAULT)
scheme2 = ag.SchemeNode(ag.Attack.DEFAULT)

graph.add_edge(ag.Edge(premise1, scheme1))
graph.add_edge(ag.Edge(scheme1, claim))
graph.add_edge(ag.Edge(premise2, scheme2))
graph.add_edge(ag.Edge(scheme2, claim))

graph.major_claim = claim
gv_graph = ag.dump.graphviz(graph)
ag.render.graphviz(gv_graph, "./assets/programmatic.png")
```

With this code, we get the following output

![Output of programmatic graph creation](./assets/programmatic.png)

### Exporting Argument Graphs

We support different output formats and integration with other libraries to ease the use of argument graphs.
Have a look at the following code snippet to get an overview of the possibilities

```python
# Export to graphviz DOT format
dot = ag.dump.graphviz(graph)

# Export an image of this dot source to a file
ag.render.graphviz(dot, "./graph.pdf")

# Convert to NetworkX graph
nx = ag.dump.networkx(graph)

# Save the graph as Arguebuf
ag.dump.file(graph, "./graph.json")

# Save the graph as AIF
ag.dump.file(graph, "./graph.json", ag.GraphFormat.AIF)
```

## Development

To pull the testing data, make sure to install [DVC](https://dvc.org/doc/install) and run `dvc pull` in the root directory of the repository.
