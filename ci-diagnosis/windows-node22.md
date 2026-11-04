# CI 结论 · Windows / Node 22

- job: `success`
- commit: `9e9398c65d3813d13682ec7cd9aafdf9983a355a`
- run: 41758952981

## 测试汇总
             [33m[2m✓[22m[39m 4 进程 × 250 个 IMMEDIATE 事务：零 DB_LOCKED、零丢失 [33m 912[2mms[22m[39m
        [2m Test Files [22m [1m[32m8 passed[39m[22m[90m (8)[39m
        [2m      Tests [22m [1m[32m112 passed[39m[22m[90m (112)[39m

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
        OK 全局命令入口: C:\Users\RUNNER~1\AppData\Local\Temp\wl-gi-yESE9K\prefix\workreport.cmd
        OK describe：1 行、可 parse、命令数 14
        OK 零运行目录污染
