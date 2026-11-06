# CI 结论 · Linux / Node 24

- job: `success`
- commit: `5c7d8f01c26cc0ae4cccf300a8502988b118c943`
- run: 42165956320

## 测试汇总
             [33m[2m✓[22m[39m 4 进程 × 250 个 IMMEDIATE 事务：零 DB_LOCKED、零丢失 [33m 412[2mms[22m[39m
        [2m Test Files [22m [1m[32m8 passed[39m[22m[90m (8)[39m
        [2m      Tests [22m [1m[32m116 passed[39m[22m[90m (116)[39m

## env-paths 实测
        MEASURED linux data=~/.local/share/workreport cfg=~/.config/workreport
        data_dir（实际）: /home/runner/.local/share/workreport
        config_path（实际）: /home/runner/.config/workreport/config.json
        期望 data_dir: /home/runner/.local/share/workreport
        期望 config_dir: /home/runner/.config/workreport
        OK data_dir = /home/runner/.local/share/workreport
        OK config_dir = /home/runner/.config/workreport

## 全局安装冒烟
        OK 安装输出中无 node-gyp / prebuild 痕迹
        OK 全局命令入口: /tmp/wl-gi-ZDJNS3/prefix/bin/workreport
        OK describe：1 行、可 parse、命令数 14
        OK 零运行目录污染
