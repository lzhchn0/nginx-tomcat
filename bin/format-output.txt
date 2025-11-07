You’re right to question it — map(gsub("\\\\t"; "\t")) will not work as expected if the \t in your JSON is already unescaped (i.e., it’s a real tab character, not the literal string \t).
Let’s diagnose and fix this properly.

Root Cause: JSON Unescaping Happens Before `jq` Sees It
When JSON is parsed:
"stacktrace": "line1\n\tline2"
→ jq receives:
	•	"line1\n\tline2" → real newline + real tab
So there is no \t string to match with gsub("\\\\t"; ...).
Your gsub("\\\\t"; "\t") is looking for the literal characters \ + t, but they were already converted by JSON parsing.

Correct Solution: Replace Actual Tab Characters
If you want to convert real tabs (\t) into spaces or keep them visible, you must match the tab character, not \t.
Option 1: Replace real tabs with 4 spaces (most common)
jq '
  if has("stacktrace") then
    .stacktrace |= (
      split("\n")
      | map(gsub("\t"; "    "))   # \t = real tab, replace with 4 spaces
    )
  else
    .
  end
'
Use "\t" inside gsub to match the real tab character.

Option 2: Keep real tabs (but ensure they display)
Tabs are preserved in JSON output. If your terminal/editor shows them as → or ···, that’s fine.
jq '
  if has("stacktrace") then
    .stacktrace |= split("\n")   # just split, tabs stay as \t
  else
    .
  end
'
→ Output will have real tabs — no gsub needed.

Option 3: You do have literal `\t` (escaped in string)
Only if your JSON looks like this:
"stacktrace": "line1\\tline2\\nline3"
→ Then use:
map(gsub("\\\\t"; "\t"))   # converts \\t → real tab
But this is rare — most logs emit real \t, not \\t.

How to Debug: See What `jq` Actually Sees
Run this to inspect raw bytes:
jq -r 'if has("stacktrace") then .stacktrace else empty end'
	•	-r = raw output (shows real \n, \t)
	•	If you see indented lines, tabs are already real.
	•	If you see \t, then use gsub("\\\\t"; "\t").

Final Recommended Command (Handles Real Tabs → 4 Spaces)
your_command | jq '
  if has("stacktrace") then
    .stacktrace |= (
      split("\n")
      | map(gsub("\t"; "    "))   # replace real tabs with 4 spaces
    )
  else
    .
  end
'
Example Input:
{"msg": "error", "stacktrace": "File A\n\tat func1\n\tat func2"}
Output:
{
  "msg": "error",
  "stacktrace": [
    "File A",
    "    at func1",
    "    at func2"
  ]
}
Perfect for logs, readable in any editor.

TL;DR
You See in Raw JSON
Use This gsub
Real indentation (tabs)
gsub("\t"; " ")
Literal \t in string
gsub("\\\\t"; "\t")
99% of cases: Use gsub("\t"; " ") — not \\\\t.
Let me know what your actual input looks like (paste a sample line), and I’ll give you the exact jq!
