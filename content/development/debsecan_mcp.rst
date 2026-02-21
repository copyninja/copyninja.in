Learning Notes: Debsecan MCP Server
###################################

:date: 2026-02-21 17:13 +0530
:slug: debsecan-mcp
:tags: generative_ai, debsecan, mcp, llm, debian
:author: copyninja
:summary: This post describes a new learning project: an MCP server inspired by
          debsecan.

Since Generative AI is currently the most popular topic, I wanted to get my
hands dirty and learn something new. I was learning about the Model Context
Protocol at the time and wanted to apply it to build something simple.

Idea
====

On Debian systems, we use `debsecan` to find vulnerabilities. However, the tool
currently provides a simple list of vulnerabilities and packages with no
indication of the system's security posture—meaning no criticality information
is exposed and no executive summary is provided regarding what needs to be
fixed. Of course, one can simply run the following to install existing fixes and
be done with it:

.. code-block:: sh

    apt-get install $(debsecan --suite sid --format packages --only-fixed)

But this is not how things work in corporate environments; you need to provide a
report showing the system's previous state and the actions taken to bring it to
a safe state. It is all about metrics and reports.

My goal was to use `debsecan` to generate a list of vulnerabilities, find more
detailed information on them, and prioritize them as critical, high, medium, or
low. By providing this information to an AI, I could ask it to generate an
executive summary report detailing what needs to be addressed immediately and
the overall security posture of the system.

Initial Take
============

My initial thought was to use an existing LLM, either self-hosted or a
cloud-based LLM like Gemini (which provides an API with generous limits via AI
Studio). I designed functions to output the list of vulnerabilities on the
system and provide detailed information on each. The idea was to use these as
"tools" for the LLM.

Learnings
---------

1. I learned about open-source LLMs using Ollama, which allows you to download
   and use models on your laptop.
2. I used Llama 3.1, Llama 3.2, and Granite 4 on my laptop without a GPU. I
   managed to run my experiments, even though they were time-consuming and
   occasionally caused my laptop to crash.
3. I learned about Pydantic and how to use it to parse custom JSON schemas with
   minimal effort.
4. I learned about osv.dev, an open-source initiative by Google that aggregates
   vulnerability information from various sources and provides data in a
   well-documented JSON schema format.
5. I learned about the EPSS (Exploit Prediction Scoring System) and how it is
   used alongside static CVSS scoring to detect truly critical vulnerabilities.
   The EPSS score provides an idea of the probability of a vulnerability being
   exploited in the wild based on actual real-world attacks.

These experiments led to a collection of `notebooks
<https://github.com/copyninja/notebooks/tree/main/langchain>`_. One key takeaway
was that when defining tools, I cannot simply output massive amounts of text
because it consumes tokens and increases costs for paid models (though it is
fine for local models using your own hardware and energy). Self-hosted models
require significant prompting to produce proper output, which helped me
understand the real-world application of prompt engineering.

Change of Plans
===============

Despite extensive experimentation, I felt I was nowhere close to a full
implementation. While using a Gemini learning tool to study MCP, it suddenly
occurred to me: why not write the entire thing as an MCP server? This would save
me from implementing the agent side and allow me to hook it into any
IDE-based LLM.

Design
------

This MCP server is primarily a mix of a "tool" (which executes on the server
machine to identify installed packages and their vulnerabilities) and a
"resource" (which exposes read-only information for a specific CVE ID).

The MCP exposes two tools:

1. List Vulnerabilities: This tool identifies vulnerabilities in the packages
   installed on the system, categorizes them using CVE and EPSS scores, and
   provides a dictionary of critical, high, medium, and low vulnerabilities.
2. Research Vulnerabilities: Based on the user prompt, the LLM can identify
   relevant vulnerabilities and pass them to this function to retrieve details
   such as whether a fix is available, the fixed version, and criticality.

Vibe Coding
-----------

"Vibe coding" is the latest trend, with many claiming that software engineering
jobs are a thing of the past. Without going into too much detail, I decided to
give it a try. While this is not my first "vibe coded" project (I have done this
previously at work using corporate tools), it is my first attempt to vibe code a
hobby/learning project.

I chose Antigravity because it seemed to be the only editor providing a
sufficient amount of free tokens. For every vibe coding project, I spend time
thinking about the barebones skeleton: the modules, function return values, and
data structures. This allows me to maintain control over the LLM-generated code
so it doesn't become overly complicated or incomprehensible. 

As a first step, I wrote down my initial design in a requirements document. In
that document, I explicitly called for using `debsecan
<https://github.com/copyninja/debsecan-mcp/blob/main/docs/requirement.md>`_ as
the model for various components. Additionally, I asked the AI to reference my
specific `code for the EPSS logic
<https://github.com/copyninja/notebooks/blob/main/langchain/secscan-common.ipynb>`_.
The reasons were:

1. `debsecan` already solves the core problem; I am simply rebuilding it.
   `debsecan` uses a single file generated by the Debian Security team
   containing all necessary information, which prevents us from needing multiple
   external sources.
2. This provides the flexibility to categorize vulnerabilities within the
   listing tool itself since all required information is readily available,
   unlike my original notebook-based design.

I initially used Gemini 3 Flash as the model because I was concerned about
exceeding my free limits.

Hiccups
-------

Although it initially seemed successful, I soon noticed discrepancies between
the local `debsecan` outputs and the outputs generated by the tools. I asked the
AI to fix this, but after two attempts, it still could not match the outputs. I
realized it was writing its own version-comparison logic and failing
significantly.

Finally, I instructed it to depend entirely on the *python-apt* module for
version comparison; since it is not on PyPI, I asked it to pull directly from
the Git source. This solved some issues, but the problem persisted. By then, my
weekly quota was exhausted, and I stopped debugging.

A week later, I resumed debugging with the Claude 3.5 Sonnet model. Within 20-25
minutes, it found the fix, which involved `four lines of changes
<https://github.com/copyninja/debsecan-mcp/commit/4fdf5a2ab139f3c0c335d24973892ddfaf2b08e0#diff-9d3ed702945a5c91f0ed3e54404324fecfca1fbd7ae5fb44508081c6040e9276>`_
in the parsing logic. However, I ran out of limits again before I could proceed
further.

In the requirements, I specified that the *list vulnerabilities* tool should
only provide a dictionary of CVE IDs divided by severity. However, the AI
instead provided full text for all vulnerability details, resulting in excessive
data—including negligible vulnerabilities—being sent to the LLM. Consequently,
it never called the *research vulnerabilities* tool. Since I had run out of
limits, I manually fixed this in a `follow-up commit
<https://github.com/copyninja/debsecan-mcp/commit/32f291d5ec4966b1349d39cfcb5d154e64ad844d>`_.

How to Use
==========

I have published the current work in the `debsecan-mcp
<https://github.com/copyninja/debsecan-mcp>`_ repository. I have used the same
license as the original *debsecan*. I am not entirely sure how to interpret
licenses for vibe-coded projects, but here we are.

To use this, you need to install the tool in a virtual environment and configure
your IDE to use the MCP. Here is how I set it up for Visual Studio Code:

1. Follow the `guide from the VS Code documentation
   <https://code.visualstudio.com/docs/copilot/customization/mcp-servers#_add-an-mcp-server>`_
   regarding adding an MCP server.
2. My global `mcp.json` looks like this:

.. code-block:: json

  {
    "servers": {
        "debsecan-mcp": {
            "command": "uv",
                "args": [
                    "--directory",
                    "/home/vasudev/Documents/personal/FOSS/debsecan-mcp/debsecan-mcp",
                    "run",
                    "debsecan-mcp"
                ]       
            }
    },
    "inputs": []
  }

3. I am running it directly from my local codebase using a virtualenv created
   with *uv*. You may need to tweak the path based on your installation.
4. To use the MCP server in the Copilot chat window, reference it using
   `#debsecan-mcp`. The LLM will then use the server for the query.
5. Use a prompt like: *"Give an executive summary of the system security status
   and immediate actions to be taken."*
6. You can observe the LLM using `list_vulnerabilities` followed by
   `research_cves`. Because the first tool only provides CVE IDs based on
   severity, the LLM is smart enough to research only high and critical
   vulnerabilities, thereby saving tokens.

What's Next?
============

This MCP is not yet perfect and has the following issues:

1. The `list_vulnerabilities` dictionary contains duplicate CVE IDs because the
   code used a list instead of a set. While the LLM is smart enough to
   deduplicate these, it still costs extra tokens.
2. Because I initially modeled this on `debsecan`, it uses a raw method for
   parsing `/var/lib/dpkg/status` instead of `python-apt`. I am considering
   switching to `python-apt` to reduce maintenance overhead.
3. Interestingly, the AI did not add a single unit test, which is disappointing.
   I will add these once my limits are restored.
4. I need to create a cleaner README with usage instructions.
5. I need to determine if the MCP can be used via HTTP as well as stdio.

Conclusion
===========

Vibe coding is interesting, but things can get out of hand if not managed
properly. Even with a good process, code must be reviewed and tested; you cannot
blindly trust an AI to handle everything. Even if it adds tests, you must
validate them, or you are doomed!
