
# Practical Call Graph Generation

## PyCG:

-   A practical static analyzer for Python call graph generation.
-   Employs a context-insensitive inter-procedural analysis.
-   Designed for efficient call graph generation.
-   Handles complex Python features:
    -   Dynamic type system
    -   Module imports
    -   Class inheritance
    -   Higher-order functions
-   Utilizes an iterative analysis approach to achieve equilibrium in the analysis state.
###  Analysis

-   **Fixpoint Iteration:** PyCG employs a fixpoint iteration algorithm with an evolving analysis state that is updated upon encountering specific Python constructs.
-   **Analysis State:** The analysis state is defined by three components:
    -   **Assignment Graph:** A directed graph representing identifier relationships and points-to relationships, crucial for higher-order function support.
    -   **Scope Tree:** A tree structure identifying the namespace of each identifier, considering Python's nested namespace inheritance.
    -   **Class Hierarchy:** A forest of trees capturing class inheritance chains for accurate method resolution.

###  Implementation

-   **Python-Based:** PyCG is implemented in Python, taking a list of Python modules as input.
-   **AST & Symbol Table Generation:** Leverages Python's `ast` and `symtable` packages to generate abstract syntax trees (ASTs) and symbol tables for analysis.
-   **Automatic Import Discovery:** Uses Python's `importlib` module to automatically discover and analyze imported package modules.
-   **Custom Loader:** Defines a custom loader to log source file locations of imported modules instead of loading their code.

### JSON Output

-   **Structured Output:** PyCG produces JSON-formatted output describing the call graph.
-   **Key Components:** Includes information about the package, version, timestamp, forge, generator, dependencies, modules, class hierarchy, and call graph itself (internal and external calls).

### Limitations

-   **Ignoring Loops & Conditionals:** Prioritizes efficiency by ignoring loops and conditional statements.
-   **Dynamic Execution Exclusion:** Ignores dynamic execution schemes like `eval` and `compile`.
-   **Lack of Type Information:** Does not store type information, limiting the resolution of calls to built-in types.
-   **Limited to Provided Source Code:** Call graph generation is restricted to the provided source code, with approximations for external calls.

## Stitching Process

-   **Goal:**  Combine individual revision call graphs into a single resolved call graph.
-   **Benefits:**  Facilitates research on the Python ecosystem's evolution over time.
-   **Pipeline:**  Packages from PyPI -> PyCG call graph generation -> Stitching -> Resolved call graph.

### Algorithm 1: Python Stitching Process

1.  **Retrieve Calls:**  Get internal and external calls from each revision call graph.
2.  **Add URIs:**  Retrieve URIs from the revision call graph and add them to the resolved call graph.
3.  **Inspect External Destinations:**
    -   Find the revision call graph using the dependency set.
    -   If no dependency exists, use the latest version of the external package.
4.  **Follow Algorithm 2:**  (See below)

### Algorithm 2: External Call Resolution

1.  **Retrieve Entity Name:**  Get the entity-name part of the URI.
2.  **Split and Check:**  Split the entity name by dots and check for all possible combinations of namespace and entity-part in the external revision call graph.
3.  **Return URI:**  Return the associated URI upon finding a match.

### Limitations

-   **Multiple Matches:**  The approach exits after the first match, potentially missing the correct URI if multiple valid combinations exist.
-   **Ambiguity:**  There's no reliable way to determine the correct URI among multiple matches.

## Fine-Grained Project Dependency Graph (FPDG) and Bloated Dependency Code

-   **FPDG:**
    
    -   Captures method-level relationships across project dependencies.
    -   Represented as FPDG = (V, E).
    -   Nodes (V) are methods identified by name and namespace (e.g., mymodule.A.m1).
    -   Edges (E) represent caller-callee relationships.
-   **Bloated Dependency Code:**
    
    -   Focuses on unused code from third-party libraries (direct or transitive dependencies).
    -   Project Representation: (n, F, D)
        -   n: Project name
        -   F: Set of source files (filename, methods)
        -   D: Set of direct dependencies
    -   Bloated Method: A dependency method not reachable by any method in the current project.
    -   Bloated File: A dependency file where all methods are bloated.
    -   Bloated Dependency: A dependency where all files are bloated.
-   **Relation to Python Import Statements:**
    
    -   Two import styles (`import x`,  `from x import a, b`) can lead to underutilized dependency code.
    -   Unloaded dependency code: Never imported or loaded.
    -   Unused imports: Imported but unused.
    -   Formal definitions treat both instances as indistinguishable, focusing on unused code.

Key Points:

-   FPDG provides a detailed view of inter-project dependencies at the method level.
-   Bloated dependency code is defined at different granularities (method, file, dependency).
-   The concept of bloated dependency code aligns with previous empirical studies.
-   Python import styles can contribute to unused dependency code, which the definitions capture.


## Demo Execution 

Select the project: aiomonitor

**aiomonitor**  is a module that adds monitor and cli capabilities for  [asyncio](http://docs.python.org/3/library/asyncio.html)  applications. Idea and code were borrowed from  [curio](https://github.com/dabeaz/curio)  project. Task monitor that runs concurrently to the  [asyncio](http://docs.python.org/3/library/asyncio.html)  loop (or fast drop-in replacement  [uvloop](https://github.com/MagicStack/uvloop)) in a separate thread as result monitor will work even if the event loop is blocked for some reason.

This library provides a python console using  [aioconsole](https://github.com/vxgmichel/aioconsole)  module. It is possible to execute asynchronous commands inside your running application. Extensible with you own commands, in the style of the standard library's  [cmd](http://docs.python.org/3/library/cmd.html)module

#### Example 

1. Find the project's dependencies
Resolving Project Dependencies:

-   **Configuration Files:** Look for standardized configuration files (setup.py, pyproject.toml, requirements.txt) to determine Python dependencies.
-   **Pip Installation:** If at least one configuration file is found, use `pip` to install dependencies in a fresh, isolated virtual environment created with `virtualenv`.
-   **Success Rate:** Successfully resolved dependencies and obtained corresponding releases for 1,644 out of 2,215 projects.
-   **Failure Reasons:**
    -   Missing configuration files for dependency resolution.
    -   Unexpected errors during installation due to dependency conflicts or missing dependencies.
-   **Exclusion:** Projects without resolved dependencies are excluded from analysis.
-   **Source Code Download:** Download source code of resolved releases using `pip download`.
Download the dependencies for the subset of projects found in ~/data/subset/sample_projects.csv
```
sh scripts/dependency_resolution/run_dep_resolution.sh $GH_TOKEN data/subset subset
```
2. Produce partial call graphs
* Produce the partial call graphs for each project and its dependencies.
* Store the produced partial call graphs, as well as the source code of each dependency within the data/subset/ directory, maintaining the same directory structure with the one described on the previous sub-section.
* Finally, it will generate a file named data/subset/project_dependencies_final_subset.json, containing the final dataset of projects and dependencies after performing this step.

```
sh scripts/partial_cg_generation/run_partial_cg_generation.sh $GH_TOKEN \
data/subset data/subset/project_dependencies_subset.json \
data/subset/project_dependencies_final_subset.json 
```
Observe Partial Call Graphs
```
find data/subset/partial_callgraphs/   -type f -name 'cg.json'
```

Stitching of Call Graphs

-   **Goal:** Combine partial call graphs into a Fine-Grained Project Dependency Graph (FPDG).
-   **Key Idea:** Merge call graphs by connecting external nodes (methods) to their counterparts in dependency call graphs.
-   **Stitching Procedure (Algorithm 1):**
    1.  Add orphan nodes from each partial call graph to the FPDG.
    2.  For each edge (s, t) in a call graph, resolve external node t using the `resolveExternalNode` method.
    3.  Create a new edge in the FPDG with source s and resolved method t.
    4.  Repeat for all call graphs.
    5.  Return the resulting FPDG.
-   **Resolving External Nodes (Algorithm 2):**
    1.  Retrieve the dependency and call graph associated with the external call.
    2.  If the node is found in the dependency call graph, return it.
    3.  Otherwise, use a dynamic approach:
        -   Install the dependency.
        -   Use Python's metaprogramming to find the object corresponding to the target method.
        -   Use Python's `inspect` module to get the module name and fully qualified name of the method.
        -   Combine the module name and method name to resolve the node.
-   **Discussion:**
    -   Dynamic resolution ensures precision and avoids false positives.
    -   Limitations: Fails for callee methods defined in non-Python files, requiring cross-language analysis.
    -   Statistics: ~3% of unique external calls remain unresolved due to non-Python callee functions.
-   **Implementation Note:** Algorithm 2 runs in a separate process to avoid namespace pollution.

Reachability analysis
-   **Goal:** Compute reachable methods in each project dependency using Breadth-First Search (BFS).
-   **Output:** Identify bloated methods, files, and dependencies based on the reachability analysis results and definitions.
```
python scripts/stitched_cg_generation/stitch.py --source data/subset/partial_callgraphs \
  --json data/subset/project_dependencies_final_subset.json
```

Observe Stitched Call Graphs for each project
```
find data/subset/stitched_callgraphs/   -type f -name 'cg.json' 
```
Security Analysis
```
 sh scripts/security_analysis/run_security_analysis.sh  $GH_TOKEN  \
  data/security data/project_dependencies_final.json
```

The script performs the following steps:

- It will first download the repository of the advisory database, which contains the known PyPI vulnerabilities
- It will parse the repository to identify vulnerabilities affecting PyPI releases
- It will then find all the vulnerable releases affecting our dataset, and it will produce a file data/security/project_vulnerabilities.json, which has the following format:

```
{
  "widdowquinn/pyani": [
    {
      "Pillow:10.0.0": [
        "GHSA-3f63-hfp8-52jq",
        "GHSA-j7hp-h8jx-5ppr"
      ]
    },
    {
      "fonttools:4.40.0": [
        "GHSA-6673-4983-2vx5"
      ]
    }
  ],
  "pelican-plugins/image-process": [
    {
      "Pillow:10.0.0": [
        "GHSA-3f63-hfp8-52jq",
        "GHSA-j7hp-h8jx-5ppr"
      ]
    },
    {
      "jinja2:3.1.2": [
        "GHSA-h5c8-rqwp-cp95"
      ]
    }
  ],
// More projects...
}
```
```
python scripts/stitched_cg_generation/security_reachability_analysis.py \
  --host data/stitched_callgraphs \
  --project_vulns  data/security/project_vulnerabilities.json \
  --vuln2func data/security/vulnerability2function.json
```
This step includes retrieving the stitched call graph of each project and checking whether each vulnerability affecting the dataset resides in bloated code sections. To do this you need the already existing file 
```data/security/vulnerability2function.json``` which contains the manually created mapping of each vulnerability encountered in our dataset with the actual vulnerable function (See second paragraph of Section 2.4 of the paper).

Quoting the paper: ""In nearly three quarters of the cases (606/816), a project depends on a vulnerable package
that it does not actually use. Therefore, package-level debloating alone, although less granular,
could still effectively eliminate the number of dependencies to vulnerable code."

## Demo on running PyCG on an internal project 

1. Set up a virtual environment 
```
python3 -m venv  myvenv
source myenv/bin/activate
```
2. Install Pycg

```
#!/bin/bash

CURRENT_DIR=$(pwd)
echo Cloning PyCG repository...
git clone https://github.com/gdrosos/PyCG.git
cd PyCG
pip3 install .
PATH="$HOME/.local/bin:$PATH"
cd ..
rm -rf PyCG
```

4. Create call graph 

```
pycg  --package mypackage  --version 1 --forge PyPI --timestamp 0 --output mypackage $(find mypackage -type f -name "*.py")
```
