To handle cases where some JSON objects lack a "stacktrace" field (causing `jq` to error on missing keys), make the transformation conditional using `has("key")` to check for its existence. This ensures the command processes the JSON gracefully without failing.

### Updated Command
Pipe your JSON stdout into this:
```
jq 'if has("stacktrace") then .stacktrace |= split("\n") else . end'
```

- **How It Works**:
  - `has("stacktrace")`: Returns `true` if the field exists, `false` otherwise.
  - If true, apply the split (converting the string to an array of lines for formatting).
  - If false, output the JSON unchanged (via `else . end`).
  - This preserves the overall pretty-printing like `jq .` while only formatting "stacktrace" when present.

### Example
Input JSON (mixed objects):
```json
[
  {"error": "Fail A", "stacktrace": "file1.py:10\nfile2.py:20"},
  {"error": "Fail B"}  // No stacktrace
]
```

Output:
```json
[
  {
    "error": "Fail A",
    "stacktrace": [
      "file1.py:10",
      "file2.py:20"
    ]
  },
  {
    "error": "Fail B"
  }
]
```

If all objects lack "stacktrace", it just pretty-prints without changes or errors.

### Tips
- For raw stacktrace output (with newlines) only when present: `jq -r 'if has("stacktrace") then .stacktrace else empty end'`.
- If the field name varies (e.g., "error_stack"), replace "stacktrace" accordingly.
- Test with your data: `your_command | jq 'if has("stacktrace") then .stacktrace |= split("\n") else . end'`.
