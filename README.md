
# kssbox-plugin

## Description

- 从 ide 到开发的全链路工具，包含以下功能：
idea -> product-strategy(产品策略规划) -> prd-writer(PRD撰写) -> harness-sync(harness-sync)

## Introduction

``` bash
# test
claude --plugin-dir ./kssbox-plugin

# install 
claude plugin marketplace add ./

claude plugin install kssbox@kssbox-dev

# link
kssbox list                    # 查看所有 skills 状态
kssbox add all                 # 安装全部
kssbox add harness-sync        # 安装单个
kssbox remove all              # 卸载全部
kssbox remove harness-sync     # 卸载单个
```

## TODO

[ ] 自动循环迭代 skill

## doc

<https://code.claude.com/docs/zh-CN/plugins>
