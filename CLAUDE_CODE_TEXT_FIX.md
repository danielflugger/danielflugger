# Claude Code — Text Fix + Review Notes
# Run from: ~/Documents/danielflugger

## TASK — One text change in ai-studio-guide.html

Find this exact sentence in ai-studio-guide.html:

"This is not hypothetical — it is the exact architecture described in the product plan and the exact project you will execute on danielflugger.com."

Replace with:

"This is a minimal instructions stack for a new project on a simple cms project with gated content."

One change, one file. After making it:
git add ai-studio-guide.html
git commit -m "fix: update walkthrough project description"
git push origin main && firebase deploy
