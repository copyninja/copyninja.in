Learning Notes: Debsecan MCP Server
###################################

:date: 2026-02-14 17:17 +0530
:slug: debsecan-mcp
:tags: generative_ai, debsecan, mcp, llm
:author: copyninja
:status: draft
:summary: Post describes a new learning project, MCP server inspired from
          debsecan

Since Generative AI is currently the most happening topic, even I wanted to get
my hands dirty and learn something new. I was learning about Model Context
Protocol at that time and wanted to apply it to prepare something simple.

Idea
====

We use debsecan on Debian system for finding the vulnerabilities on the system
but currently tool simply gives a list of vulnerabilities and packages with no
indication of security posture of the system, i.e. no criticality information
etc is exposed and no executive summary on what needs to be fixed etc. Ofcourse
we can simply run following to install existing fix and be done with it.

.. code-block:: sh

    apt-get install $(debsecan --suite sid --format packages --only-fixed)

But this is not how things work in corporates, you need to show a report on how
was your system earlier and what work you did to bring to current safe state.
Its all about metrics and reports.

So what I wanted to do here was use `debsecan` to get list of vulnerabilities in
the system and then find more information on vlunerabilities and prioritise them
as critical, high, medium and low provide this information to AI and ask it to
spit out an executive summary report with what needs to be address at earliest
and how is security posture of the system.

Initial Take
============

My initial thought was to use existing LLM either self hosted or the cloud LLM
like Gemini (As it provide API with generous limits using AI Studio). I designed
the functions to spit out the list of vulnerabilities on system as well as give
detailed information on the vulnerabilities. Idea was to use them as `tools` for
the LLM to use.

Learnings
---------
1. Learnt about Opensource LLM using ollama which you can download and use on
   your laptops.
2. Used llama3.1 llama3.2 and granite4 on my laptops without GPU. Managed to run
   my experiemtns even though it took bit too much time and even crashed my
   laptop in doing it.
3. Learnt about pydantic and to use it to parse custom  json schema without much
   work!.
4. Learnt about osv.dev and open source initiative by Google to get all
   vulnerability information from various sources and provide data in a well
   documented JSON schema format.
5. Learnt about EPSS (Exploit Prediction  Scoring System) scoring of
   vulnerabilities and how its used to detect really critical vulnerabilities
   along with static CVSS scoring. EPSS score gives idea of probability of
   exploit in wild from actual attacks in real world.

All these experiments lead to these bunch of `notebooks
<https://github.com/copyninja/notebooks/tree/main/langchain>`_. One learning was
when defining tools I can't just runs tons of text because it costs token and
not good for paid models, of course its fine with locally running model which
will use your laptop and energy. The self hosted model requires pretty good
amount of prompting to get a proper output, here I understood prompt engineering
class and its real application.

Change of Plans
===============

Even after so much experimenting I was feeling I was nowhere close to the full
implementation and at the same time I started using Gemini Learning tool to
learn about MCP and then it suddenly occured to me why not write this entire
thing as MCP server that saves me implementation of agent side and can hook it
to any IDE based LLMs.

Design
------
This MCP server will be mostly a mix of `tool` which executes on server machine
to identify list of packages installed and vulnerabilities in it and also a
`resource` kind MCP because it exposes read only information on CVE ID passed to
it. There will be 2 tools exposed by this MCP.

1. List Vulnerabilities: Responsible for figuring of vulnerabilities in the
   installed packages on system and categorise them using CVE as well as EPSS
   score and provide a dictionary of critical, high, mid and low
   vulnerabilities.
2. Research Vulnerabilities: Based on user prompt LLM can decide relevant
   vulnerabilities and pass it to this function to get more details like fix
   available or not, if available version fixed in, criticality etc.

Vibe Coding
-----------

So this is the new kid in town and every one going gaga about vibe coding and
saying software engineering job is done for. Without going to much in detail, I
thought I will give it a try. Though this is not my first vibe coded project I
had done it before all this hype on social media but for work and using tools
provided to me by work. This is my first attempt to vibe code a hobby/learning
project. I choose antigravity because that is only editor giving good enough
free usage tokens?. If there are others please do let me know in comments.

To all vibe coding project I spend sometime  in thinking of barebone skeleton of
utility the modules, functions return values and any data structures. This
allows me to have control over generate code by LLM and it won't go loose an do
complicated stuff which later I can't understand. So as a first step I wrote
down my initial design in `a requirement.md
<https://github.com/copyninja/debsecan-mcp/blob/main/docs/requirement.md>`_.

If you go through the requirement/design document you will notice that I called
out explicitly to use debsecan as model for various things. Additionally some
portion of things I asked it to reference my code especially the EPSS part.
Reasons are as follows.

1. debsecan already solves the problem, I'm just doing it again and debsecan
   does it using single file generated by Debian Security team which contains
   all information needed. So this saves us from going to different sources to
   fetch more information.
2. This also gives me flexibility to do the categorisation of vulnerabilities in
   listing tool itself as I have all required information readily available,
   unlike my original design (notebooks).

I initially used *Gemini 3 Flash* as model as I was worried about running out of
my free limits.

Hiccups
-------

Though initially it looked like I nailed it and got it right I soon noticed the
deviation between the debsecan local outputs and output generated from the
toolings are different. I asked it to fix this and after 2 runs it looked like
it still was not able to match the outputs. At this point I noticed that it was
actually writing its own version comparing logic and failing badly at it.
Finally I asked it to completely depend on *python-apt* modules for version
comparing and since its not on pypi asked it to pull it directly from git
source. This did solve a bit of issue but the problem was not completely gone
away. By this time that week's quota got over and I stopped debugging further.

After one week I again tried the debugging but this time with Claude Sonnet 4.5
model and it took it about 20-25 minutes but finally found the fix which was
some `4 line changes in its parsing logic
<https://github.com/copyninja/debsecan-mcp/commit/4fdf5a2ab139f3c0c335d24973892ddfaf2b08e0#diff-9d3ed702945a5c91f0ed3e54404324fecfca1fbd7ae5fb44508081c6>`_.
By the time I wanted to proceed again I was out of limits again for one more
week.

In requirement/design I had mentioned *list vulnerabilities* tool to only
provide dictionary of CVE IDs divided based on severity. But the AI went ahead
and started giving text of all vulnerability details as per severity, end result
too much data given to AI including non-neglible vulnerabilities. It never
called *research vulnerabilities* tool at all. Since I already had run out of
limits I manually fixed this part in `this commit
<https://github.com/copyninja/debsecan-mcp/commit/32f291d5ec4966b1349d39cfcb5d154e64ad844d>`_. 

How to Use
==========

I've published the work done till now at `debsecan-mcp repo
<https://github.com/copyninja/debsecan-mcp>`_. I've made the license same as
original *debsecan*. Now I don't know how do you interpret vibe coded project
license but anyway. To use this you will need to install this tool in a virtual
env and configure your ide to use the MCP. This is how I did it for visual
studio code.

1. Used this `guide from VS Code documentation
   <https://code.visualstudio.com/docs/copilot/customization/mcp-servers#_add-an-mcp-server>`_.
2. My global mcp.json looks like this

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

3. Note that I'm just using it directly from my local codebase which also has
   virtualenv created using *uv*. You might need to tweak it based on your
   virtualenv path where you installed the tool.
4. To use the MCP server in your co-pilot chat window, just reference it using
   #debsecan-mcp and selected LLM will use the MCP server for the query.
5. Use prompt like *"Give a executive summary of system security status and
   immidiate action to be taken"*
6. You can see LLM using *list_vulnerabilities* followed by *research_cves* and
   you can also inspect output of these tools. Since I'm only giving CVE ID
   based on severity in first tool LLM will be smart enough to use only high and
   critical vulnerabilities in *research_cves* there by making some token save.

What Next?
==========

This MCP is not yet perfect it has following issues based on my initial review.

1. List Vulnerabilities dictionary return has duplicate CVE ID because of a
   issue in code where it should have used set instead of list to get only
   unique vulnerabilities. Though not major and LLM is smart enough to
   deduplicate it will still costs some extra tokens based on amount of
   vulnerabilities on system.
2. Since I originally asked it to model entire thing based on debsecan, it uses
   raw method of parsing */var/lib/dpkg/status* instead of using *python-apt*. I
   don't know why debsecan chose this way but I'm considering switching to
   *python-apt* and reduce some maintenance overhead.
3. Interestingly the AI did not add a single unit test! I'm disappointed. So
   once my limits are restored I will consider adding unit tests.
4. A proper cleaner README on how to use it.
5. Not yet sure about if MCP can be used only via stdio or http as well need to
   figure those part out.

Conclussion
===========

So my take vibe coding is interesting but things will go out of hand if not done
properly. Even after doing things properly code needs to be reviewed and tested
you can't blindly trust AI to take care of everything. Even though it might add
test you still need to validate if tests are correct or not else you are
doomed!. 
