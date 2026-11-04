# CI 结论 · Windows / Node 24

- job: `failure`
- commit: `5d5fd29560bfe71befa8cda7d3e498ab7971be97`
- run: 41758492446

## 测试汇总
        [31m     [31m×[31m 4 进程 × 250 个 IMMEDIATE 事务：零 DB_LOCKED、零丢失[39m[32m 116[2mms[22m[39m
        [31m     [31m×[31m 取锁冲突映射为 DB_LOCKED、退出 3（AC-5 后半：busy_timeout 耗尽后不静默重试）[39m[33m 120007[2mms[22m[39m
        [31m⎯⎯⎯⎯⎯⎯⎯[39m[1m[41m Failed Tests 2 [49m[22m[31m⎯⎯⎯⎯⎯⎯⎯[39m
        [41m[1m FAIL [22m[49m test/concurrency.test.ts[2m > [22mAC-3 多进程并发写（第 6 节、D10）[2m > [22m4 进程 × 250 个 IMMEDIATE 事务：零 DB_LOCKED、零丢失
        [41m[1m FAIL [22m[49m test/concurrency.test.ts[2m > [22mAC-3 多进程并发写（第 6 节、D10）[2m > [22m取锁冲突映射为 DB_LOCKED、退出 3（AC-5 后半：busy_timeout 耗尽后不静默重试）
        [2m Test Files [22m [1m[31m1 failed[39m[22m[2m | [22m[1m[32m7 passed[39m[22m[90m (8)[39m
        [2m      Tests [22m [1m[31m2 failed[39m[22m[2m | [22m[1m[32m110 passed[39m[22m[90m (112)[39m
        ::error file=D%3A/a/Wakalog/Wakalog/test/concurrency.test.ts,title=test/concurrency.test.ts > AC-3 多进程并发写（第 6 节、D10） > 4 进程 × 250 个 IMMEDIATE 事务：零 DB_LOCKED、零丢失,line=67,column=30::Error: worke

## env-paths 实测
        MEASURED win32 data=~/AppData/Local/workreport/Data cfg=~/AppData/Roaming/workreport/Config
        data_dir（实际）: C:\Users\runneradmin\AppData\Local\workreport\Data
        config_path（实际）: C:\Users\runneradmin\AppData\Roaming\workreport\Config\config.json
        期望 data_dir: C:\Users\runneradmin\AppData\Local\workreport\Data
        期望 config_dir: C:\Users\runneradmin\AppData\Roaming\workreport\Config
        OK data_dir = C:\Users\runneradmin\AppData\Local\workreport\Data
        OK config_dir = C:\Users\runneradmin\AppData\Roaming\workreport\Config

## 全局安装冒烟
        OK 安装输出中无 node-gyp / prebuild 痕迹
        OK 全局命令入口: C:\Users\RUNNER~1\AppData\Local\Temp\wl-gi-SRqsWF\prefix\workreport.cmd
        OK describe：1 行、可 parse、命令数 14
        OK 零运行目录污染
