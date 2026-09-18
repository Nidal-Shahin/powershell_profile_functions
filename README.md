# Directory Tree Generator for VS Code

A standalone, non-intrusive Node.js script and VS Code task setup that generates a clean ASCII folder tree of your active project workspace directly into a `dir_tree.txt` file.

---

## Features

- **No PowerShell profile required:** Runs independently of OS-level script execution policies.
- **Global VS Code Integration:** Triggers across any workspace via the Command Palette (`Ctrl+Shift+P`) or a custom keyboard shortcut.
- **Smart Filtering:** Excludes common build and dependency folders by default (`node_modules`, `.git`, `dist`, `out`, `.next`, `.firebase`, `.vs`, etc.).

---

## 1. Node.js Script (`global_dir_tree.js`)

Save this file in your user home directory (e.g., `C:\Users\<YourUsername>\global_dir_tree.js`):

```javascript
const fs = require('fs');
const path = require('path');

const defaultExcludes = [
  'node_modules',
  '.firebase',
  'out',
  '.next',
  '.git',
  'dist',
  'build',
  '.vs',
  'global_dir_tree.js',
  'dir_tree.txt'
];

function generateTree(dirPath, prefix = '', excludes = defaultExcludes) {
  let items;
  try {
    items = fs.readdirSync(dirPath, { withFileTypes: true });
  } catch (err) {
    return [];
  }

  // Filter out excluded items and sort directories first
  const filteredItems = items
    .filter(item => !excludes.includes(item.name))
    .sort((a, b) => {
      if (a.isDirectory() === b.isDirectory()) return a.name.localeCompare(b.name);
      return a.isDirectory() ? -1 : 1;
    });

  let lines = [];
  filteredItems.forEach((item, index) => {
    const isLast = index === filteredItems.length - 1;
    const branch = isLast ? '\\-- ' : '+-- ';
    const childPrefix = isLast ? '    ' : '|   ';
    const label = item.isDirectory() ? `${item.name}/` : item.name;

    lines.push(`${prefix}${branch}${label}`);

    if (item.isDirectory()) {
      const subPath = path.join(dirPath, item.name);
      lines = lines.concat(generateTree(subPath, `${prefix}${childPrefix}`, excludes));
    }
  });

  return lines;
}

function run() {
  const currentDir = process.cwd();
  const rootName = path.basename(currentDir);
  const treeLines = [`${rootName}/`, ...generateTree(currentDir)];

  const outputPath = path.join(currentDir, 'dir_tree.txt');
  fs.writeFileSync(outputPath, treeLines.join('\n'), 'utf8');

  console.log(`Created 'dir_tree.txt' (excluded: ${defaultExcludes.join(', ')})`);
}

run();

```

---

## 2. VS Code User Task Configuration (`tasks.json`)

To make the script accessible globally across all projects, add it to your global VS Code user tasks:

1. Press **`Ctrl + Shift + P`** in VS Code.
2. Select **`Tasks: Open User Tasks`**.
3. Paste the following configuration:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Generate Directory Tree",
      "type": "shell",
      "command": "node \"${userHome}/global_dir_tree.js\"",
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "problemMatcher": [],
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": true
      }
    }
  ]
}

```

---

## 3. Keyboard Shortcut Configuration (`keybindings.json`)

To bind the task to a keyboard shortcut (e.g., `Ctrl + d Ctrl + T`):

1. Press **`Ctrl + Shift + P`**.
2. Select **`Preferences: Open Keyboard Shortcuts (JSON)`**.
3. Add the following keybinding rule:

```json
[
  {
    "key": "ctrl+d ctrl+t",
    "command": "workbench.action.tasks.runTask",
    "args": "Generate Directory Tree"
  }
]

```

---

## Usage

1. Open any project workspace in VS Code.
2. Press **`Ctrl + d Ctrl + T`** (or press `Ctrl + Shift + P` > **`Tasks: Run Task`** > **`Generate Directory Tree`**).
3. The script will execute and output a fresh `dir_tree.txt` file in your root workspace directory.
