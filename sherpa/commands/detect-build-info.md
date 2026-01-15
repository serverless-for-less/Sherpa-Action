---
description: Detect a projects build info.
---

# Detect Build Info

Detect the language, build commands, framework, and other info necessary to build a CI/CD pipeline for the project.

1. Install `npm i -g @netlify/build-info` if its not available.
2. Then detect the json build info by running `build-info path/to/site --rootDir /project/root/dir`. Use these user provided paths: "$ARGUMENTS". If no paths are provided, assume you are in the correct path.
3. If you cannot determine the language, build commands (npm i, npm run build, etc) with #2, read the codebase and make your best guess.
4. ALWAYS read the code base to determine the appropriate package manager to use (npm, pnpm, yarn, bun, deno, etc)
5. Write the output to the file `.sherpa.sh/build-info.md`. Ensure .sherpa.sh is in the root of the repo next to .git folder. Never write anything explicitly referencing netlify into the config. Always include the framework, build commands and the folder where the build command must be run from.
6. Summarize to the user how this project will get build with the `/Sherpa/build` command. 
7. Continue with the original command, if there is one.