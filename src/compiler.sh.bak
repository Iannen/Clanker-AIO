#!/usr/bin/env python3

import os
import sys
import base64
from pathlib import Path
from types import SimpleNamespace
import json

SCRIPT_DIR = os.environ.get("SCRIPT_DIR", ".")
PROMPT_DIR = Path(SCRIPT_DIR) / "src/prompt-fragments"

INCLUDE_PATTERN_START = "§§"
INCLUDE_PATTERN_END = "§§"


def main():
    # 1. Enforce that an action flag must be provided
    if len(sys.argv) < 2:
        print(
            "Error: Action flag is mandatory. Usage: compiler.sh <q|w|e|r>",
            file=sys.stderr
        )
        sys.exit(1)
        
    action = sys.argv[1].lower()
    
    # 2. Read the piped JSON context from stdin
    try:
        raw_state = json.loads(sys.stdin.read())
    except Exception:
        print("Error: Invalid context received by compiler.", file=sys.stderr)
        sys.exit(1)

    repo_state = SimpleNamespace(
        initialized=raw_state.get("initialized", False),
        src=raw_state.get("src", "No src directory found")
    )
    
    # 3. Handle behaviors based on the strict action flag
    if action == "q":
        # Compile Behavior
        root = PROMPT_DIR / "operation-instructions.fragment"
        if not root.exists():
            print(f"Error: missing root fragment: {root}", file=sys.stderr)
            sys.exit(1)

        compiled = resolve_file(root, [])
        copy_to_clipboard(compiled)
        
    elif action == "w":
        # Status/Metadata Check Behavior
        print("--- [Compiler Status Check] ---")
        print(f"Repo initialized status: {repo_state.initialized}")
        print(f"Source Directory State: {repo_state.src}")
        
    elif action == "e":
        # Cleaning/Purge Routine
        print("--- [Compiler Clean Command] ---")
        print("No Python-level clean instructions defined yet.")
        
    elif action == "r":
        # Reinitialization Routine
        print("--- [Compiler Reinitialize Command] ---")
        print("No Python-level reinitialization instructions defined yet.")
        
    else:
        print(
            f"Error: Unsupported action flag '{action}'. Expected q, w, e, or r.", 
            file=sys.stderr
        )
        sys.exit(1)


def resolve_file(path: Path, stack: list[Path]) -> str:
    path = path.resolve()

    if path in stack:
        cycle = " -> ".join(str(p.name) for p in stack + [path])
        print(f"Error: include cycle detected:\n{cycle}", file=sys.stderr)
        sys.exit(1)

    if not path.exists():
        print(f"Error: missing fragment: {path}", file=sys.stderr)
        sys.exit(1)

    content = path.read_text(encoding="utf-8")

    return resolve_content(
        content,
        stack + [path]
    )


def resolve_content(content: str, stack: list[Path]) -> str:
    lines = content.splitlines(keepends=True)

    while True:
        include_index = find_include_line(lines)

        if include_index is None:
            break

        include_name = extract_include_name(lines[include_index])
        include_path = PROMPT_DIR / f"{include_name}.fragment"

        replacement = resolve_file(
            include_path,
            stack
        )

        before = lines[:include_index]
        after = lines[include_index + 1:]

        lines = before + [replacement] + after

    return "".join(lines)


def find_include_line(lines):
    for index, line in enumerate(lines):
        if (
            INCLUDE_PATTERN_START in line
            and INCLUDE_PATTERN_END in line.split(
                INCLUDE_PATTERN_START,
                1
            )[1]
        ):
            return index

    return None


def extract_include_name(line):
    start = line.index(INCLUDE_PATTERN_START)
    end = line.index(
        INCLUDE_PATTERN_END,
        start + len(INCLUDE_PATTERN_START)
    )

    return line[
        start + len(INCLUDE_PATTERN_START):
        end
    ].strip()


def copy_to_clipboard(text):
    payload = base64.b64encode(
        text.encode("utf-8")
    ).decode("utf-8")

    sys.stdout.write(
        f"\033]52;c;{payload}\007"
    )

    print(
        "⚡ Compiled prompt successfully copied via OSC 52!",
        file=sys.stderr
    )


if __name__ == "__main__":
    main()
