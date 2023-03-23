---
title: "How to run lazygit in VSCode"
date: 2023-03-23T13:40:59+01:00
draft: false
tags: ["VSCode", "Neovim", "Lazygit", "Editor", "Productivity"]
---

For a long time I have been using Neovim as my main editor, but sometimes I do find myself in situations
where VSCode is more convenient, for example when working with notebooks.
In Neovim I have been using Lazygit combined with Toggleterm in order to quickly access a popup lazygit
window with a quick shortcut. For VSCode there is no plugin to do this AFIK (please let me know if I am wrong).
However I have been able to find a nice workaround using VSCode Tasks.

## Setting up the task

First of all we must configure our tasks. In VSCode, open the command pallette, and select "Open user tasks".
This will open a json file defining your tasks. My tasks file is then set up as follows:

```json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "label": "maximize_terminal",
      "command": "${command:workbench.action.toggleMaximizedPanel}"
    },
    {
      "label": "lazygit",
      "type": "shell",
      "command": "lazygit",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": true,
        "panel": "dedicated",
        "clear": false,
        "close": true
      },
      "dependsOn": "maximize_terminal"
    },
    {
      "label": "close_lazygit",
      "command": "${command:workbench.action.toggleMaximizedPanel}",
      "dependsOn": "lazygit"
    }
  ]
}
```

You can see there are three tasks defined:

- maximize_terminal – maximizes the terminal view so we can focus on lazygit
- lazygit – actually runs lazygit
- close_lazygit – minimize our terminal after we have run the command

Somewhat counterintuitively it is the "close_lazygit" task that we must run, as we rely on the dependsOn property
to make the chainning of commands work. Now you can bind any shortcut to run this. For me I have defined it to
be the same as my neovim shortcut, and since I use the neovim plugin for VSCode, I do it in my neovim configure

```lua
local keymap = vim.api.nvim_set_keymap
local function remap(mode, from, to)
  keymap(mode, from, to, opts)
end

remap("n", "<leader>gg", "<Cmd>call VSCodeNotify('workbench.action.tasks.runTask', 'close_lazygit')<CR>")
```

I hope you found this small guide helpful!
