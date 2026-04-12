A Sandbox scrpt for thirdparty binaries
#######################################

:date: 2026-04-12 12:30
:slug: npm_sandbox_script
:tags: sandbox, npm, paranoid
:author: copyninja
:summary: Blog briefly explains a script created to sandbox thirdparty apps
          created using npm

Recently there have been multiple supply chain attacks on applications written
using npm and even python. Things are being developed very rapidly and many CLI
tools for newer GenAI things like Gemini-CLI are more and more written using
npm. Essentially what I'm trying to say is given pace of things changing there
is noway to escaping using newer tools.

I was always a person who sticks mostly to the tools which are packaged by
distributions. Main reason being they are already validated by the maintainers
before uploading to Debian archive like license validation or code validation
and all dependencies will also be coming from package archives i.e. everything
is verified and uploaded to distribution. But given how things are changing its
really a challenge to pacakge everything for distribution and especially if
something is written using npm ecosystem it takes a lot of packager time to
package entire dependency chain which is not a easy job at all. Due to this I
had no choice but to use 3rd party npm apps like Gemini CLI. But given recent
supply chain attacks which affected axios and even LiteLLM it became scary to
run anything on your personal system. I was discussing this with a colleague of
mine and he showed me what he uses, a flatpak style sandbox script even for
launching Google Chrome. It effectively prevents app from directly acessing the
systembus and his home directory or sensitive folders. It provides only
sufficient permissions and paths that is needed to correctly run the
application.

By providing this script I used OpenCode with Qwen 3.6 Fast free model which was
available at the time to write simiar script which I can use to run tools like
Gemini CLI and even GUI apps like Firefox. The script is on my github you can
find it here `safe-run-binary
<https://github.com/copyninja/dotfiles/blob/master/bin/safe-run-binary>`_ 

