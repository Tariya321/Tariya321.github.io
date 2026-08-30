---
title: "使用pymode增强vim编辑python体验"
date: 2025-05-22_20:52
tags:
  - vim
  - 插件
  - python
publish: yes
---

在vimrc中加入如下
```
call plug#begin('~/.vim/plugged')
Plug 'python-mode/python-mode'
call plug#end()
```

然后使用`:PlugInstall`安装插件
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8vim-plug%E6%8F%92%E4%BB%B6%E5%A2%9E%E5%BC%BAvim%E7%BC%96%E8%BE%91%E4%BD%93%E9%AA%8C.png" alt="使用vim-plug插件增强vim编辑体验" width="300" >}}

若打开python显示
```
[Pymode]: error: Pymode requires vim compiled with +python3 (exclusivel
y). Most of features will be disabled.
```

则可以通过python安装支持即可
```shell

# or
pip3 install --user pynvim
```


nvim的`:checkhealth`界面，表明已经支持python编辑
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8vim-plug%E6%8F%92%E4%BB%B6%E5%A2%9E%E5%BC%BAvim%E7%BC%96%E8%BE%91%E4%BD%93%E9%AA%8C-1.png" alt="使用vim-plug插件增强vim编辑体验-1" width="400" >}}

最后再到vimrc里面加上py缩进设置
```
" =============== Python 缩进设置 ===============
autocmd FileType python setlocal expandtab shiftwidth=4 softtabstop=4 tabstop=4

" 保存时自动格式化（可选）

let g:pymode_format = 1
let g:pymode_format_auto = 1
```

pymode还支持查询不规范的写法(可以关闭此提示)
{{< figure src="/attachment/%E4%BD%BF%E7%94%A8vim-plug%E6%8F%92%E4%BB%B6%E5%A2%9E%E5%BC%BAvim%E7%BC%96%E8%BE%91%E4%BD%93%E9%AA%8C-3.png" alt="使用vim-plug插件增强vim编辑体验-3" width="500" >}}

全部配置如下
```c
call plug#begin('~/.vim/plugged')
    Plug 'python-mode/python-mode'
    Plug 'iamcco/markdown-preview.nvim', { 'do': 'cd app & yarn install' } 
    Plug 'vim-python/python-syntax'
    Plug 'neovim/nvim-lspconfig'
    " 自动补全插件
    Plug 'hrsh7th/nvim-cmp'
    Plug 'hrsh7th/cmp-nvim-lsp'
    " Snippets 支持
    Plug 'L3MON4D3/LuaSnip'
    Plug 'saadparwaiz1/cmp_luasnip'
    Plug 'williamboman/mason.nvim'
    Plug 'williamboman/mason-lspconfig.nvim'
call plug#end()

" =============== Python 缩进设置 ===============
autocmd FileType python setlocal expandtab shiftwidth=4 softtabstop=4 tabstop=4

" 保存时自动格式化（可选）
let g:pymode_format = 1
let g:pymode_format_auto = 1

" 启用 LSP
lua << EOF
require("mason").setup()
require("mason-lspconfig").setup {
    ensure_installed = { "pyright" },
}

local lspconfig = require('lspconfig')
lspconfig.pyright.setup{
	# 关闭语法纠正
    on_attach = function(client, bufnr)
		client.server_capabilities.diagnosticProvider = false
    end
}

-- 自动退出前关闭 Location List，防止 :wq 报错
vim.api.nvim_create_autocmd("QuitPre", {
  callback = function()
    for _, win in ipairs(vim.fn.getwininfo()) do
      if win.loclist == 1 then
        vim.cmd("lclose")
      end
    end
  end
})

-- 自动补全配置
local cmp = require'cmp'
cmp.setup {
  sources = {
    { name = 'nvim_lsp' },
    { name = 'luasnip' }
  },
  mapping = cmp.mapping.preset.insert({
    ['<Tab>'] = cmp.mapping.select_next_item(),
    ['<S-Tab>'] = cmp.mapping.select_prev_item(),
    ['<CR>'] = cmp.mapping.confirm({ select = true }),
  }),
  snippet = {
    expand = function(args)
      require('luasnip').lsp_expand(args.body)
    end
  },
}

EOF
```


colorcolumn
当每行超过80字符时，将会亮起红线
巧妙的是，这个设置在24寸屏幕下正好为为半屏的宽度

pep8推荐的超长行处理方式：分行写，并使用括号包裹
```python
# express
total = (
    item.price * item.quantity +
    tax_rate * item.price
)
# func
result = some_function_name(
    arg1, arg2, arg3,
    arg4, arg5, arg6
)
```

如何关闭语法检查？
https://github.com/microsoft/pyright/discussions/3929
```lua
require('lspconfig').pyright.setup{
  on_attach = function(client, bufnr)
    -- 可选：关闭语义高亮
    client.server_capabilities.semanticTokensProvider = nil
    end,
    settings = {
    python = {
      analysis = {
        diagnosticMode = "off",    -- 关闭诊断（红线）
        typeCheckingMode = "off",  -- 关闭类型检查
      }
    }
  }
}
```
并且禁用pymode即可
>写到这里猛然发现，本文是使用pymode增强vim的编辑体验。然而实际过程中，反而是感到拖累了，真正需要的是正确的indent和一些代码补全。仅此而已





