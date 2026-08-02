+++ 
draft = true
date = 2026-07-28T19:23:28+02:00
title = "How i configure LSP in Neovim"
description = "How i configure LSP in Neovim"
slug = ""
authors = ["Bjørn Kristian Strand"]
tags = ["neovim"]
categories = ["config"]
externalLink = ""
series = []
+++

One of the most challenging things to configure as a new user to Neovim is the LSP integration, and i've personally spent more hours than i would like to admit on trying to have a setup that is merely _"decent"_. So now that i am happy with my LSP setup, i wanted to go into detail on how it works, such that i can _(maybe)_ save you some time on your own configuration.

The source-code of my Neovim configuration can be found at [bksuup/Neovim-Config](https://github.com/bksuup/Neovim-Config)

## Base setup

For my general neovim config i am using the [Lazy](https://github.com/folke/lazy.nvim) package manager. Since Neovim 0.12.0 you can use neovim's built in package manager [pack](https://neovim.io/doc/user/pack/), but since i started using Lazy on the versions before pack was available, and because i quite like Lazy, i am still going to be using it going forward.

If you are unfamiliar with the [Language Server Protocol (LSP)](https://microsoft.github.io/language-server-protocol/), you have two components, the client, in this case Neovim, and the language server installed on the host system. I am using [Mason.nvim](https://github.com/mason-org/mason.nvim) to install and configure the language servers directly via my neovim configuration

In addition to the "strictly" LSP stuff within my `lsp.lua` file, i have added some additional functionalities that are nice to know about
- [conform.nvim](https://github.com/stevearc/conform.nvim): used for autoformatting, configuration file for conform are located at `Neovim-Config/lua/custom/autoformat.lua`. We are installing it here since conform depends on having the LSP functionality set up in order to work.
- [fidget.nvim](https://github.com/j-hui/fidget.nvim): (description from the github page) "Extensible UI for Neovim notifications and LSP progress messages". This one is optional, i find it useful.

## The configuration

The configuration is mainly split up in two parts inside the `return{}` lua block.

Firstly we are installing [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig), it's dependencies, and the optional plugins for added functionality.

``` lua
"neovim/nvim-lspconfig",
dependencies = {
  {
    "folke/lazydev.nvim",
    ft = "lua", -- only load on lua files
    opts = {
      library = {
        -- See the configuration section for more details
        -- Load luvit types when the `vim.uv` word is found
        { path = "${3rd}/luv/library", words = { "vim%.uv" } },
      },
    },
  },
  "williamboman/mason.nvim",
  "williamboman/mason-lspconfig.nvim",
  "WhoIsSethDaniel/mason-tool-installer.nvim",

  { "j-hui/fidget.nvim", opts = {} },

  -- Autoformatting
  "stevearc/conform.nvim",

  -- Schema Information
  "b0o/SchemaStore.nvim",
},
```

Most of these dependencies has been covered already, but 
