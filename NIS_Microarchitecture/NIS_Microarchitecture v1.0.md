# NIS_Microarchitecture Main v1.0
```mermaid
flowchart TD
    subgraph 中央控制器
        direction UD
        I1[FIFO取指] --> J1[中央路由]
        J1[中央路由] --> J2[解码]
    end
    J2 --> A[执行<br/>并行运算]
    J2 --> B[执行<br/>逻辑运算]
```