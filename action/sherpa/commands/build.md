---
description: Builds your current project based on the the build-info file
---

If (`.sherpa.sh/build-info.md` doesn't exist) { run the plugin command `/Sherpa/detect-build-info` }
Else { 
    Use the info in `.sherpa.sh/build-info.md` and the lastest file in `.sherpa.sh/executionlog` to build the project. 
    Examine the build commands in the executionlog. Decide if you should just repeat them. Usually, you want to repeat them.
}