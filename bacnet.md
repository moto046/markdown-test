```mermaid
graph TD
    subgraph App[HMIアプリ層]
      UI[画面/タグ参照]
      Tag[タグデータ型]
    end
    subgraph Drv[ドライバ層]
      Pkt[8WAYパケット]
      Obj[Object/Property指定組み立て]
    end
    subgraph IF[HKC_IFSysBacnet]
      Req[8WAY要求解析・Read/Write分岐]
      Hand[接続先情報取得]
      Resp[応答結果格納]
    end
    subgraph Svc[HKC_BacnetService]
      Conv[BACnet型変換]
      APDU[BACnet要求キュー投入]
      Ret[戻り値データ保持]
    end
    subgraph Stack[BACnet-stack層]
      Task[Read/Writeタスク実行]
    end
    subgraph Net[ネットワーク]
      Dev[(BACnet Device)]
    end

    UI --> Tag --> Obj --> Pkt --> Req
    Hand --> Req
    Req --> APDU
    Req --> Conv
    Conv --> APDU --> Task --> Dev
    Dev --> Task --> Ret --> Resp --> Pkt --> UI

    style IF fill:#cfe2ff,stroke:#1a4d8c
    style Svc fill:#d9ead3,stroke:#2f6b2f
    style Stack fill:#ffe6b3,stroke:#b06a00
```
