# BACnetクライアント機能

## 構造図（Mermaid）

### クラス構成図

```mermaid
graph TD
  Driver[ドライバ層 8WAY]
  IFEth[HKC_IFSysEthernet 既存基底]
  IFSer[HKC_IFSysSerial 既存基底]
  IFBacnet[HKC_IFSysBacnet BACnet変換層]
  IFBacnetSer[HKC_IFSysBacnetSerial BACnet変換層 Serial]
  Svc[HKC_BacnetService BACnetサービス]
  Wrapper[HKC_BacnetWrapperAPI stackラッパ]
  StackLib[(bacnet-stack ライブラリ)]
  Cache[接続先テーブル/DeviceIdキャッシュ]

  Driver -->|generalProc8Way / send8Way / recv8Way| IFBacnet
  Driver -->|generalProc8Way / send8Way / recv8Way| IFBacnetSer
  IFBacnet -->|inherits| IFEth
  IFBacnetSer -->|inherits| IFSer
  IFBacnet -->|Read / Write要求| Svc
  IFBacnetSer -->|Read / Write要求| Svc
  IFBacnet -->|getSockAddrfromCacheTable| Cache
  IFBacnetSer -->|getSockAddrfromCacheTable| Cache
  Svc -->|libAccess| Wrapper
  Wrapper -->|関数呼び出し| StackLib

  style IFBacnet fill:#cfe2ff,stroke:#1a4d8c,stroke-width:2px
  style IFBacnetSer fill:#cfe2ff,stroke:#1a4d8c,stroke-width:2px
  style Svc fill:#d9ead3,stroke:#2f6b2f,stroke-width:2px
  style Wrapper fill:#ffe6b3,stroke:#b06a00,stroke-width:2px
  style StackLib fill:#dddddd,stroke:#666,stroke-dasharray:4 3
```

### クラス構成（実装ソース抽出）

| 分類 | クラス/構造体 | 役割 | ファイル |
|---|---|---|---|
| 主体（ドライバ-BACnet変換） | HKC_IFSysBacnetEthernet | HKC_IFSysEthernetを継承。8WAY要求を解析し、ReadProperty/WriteProperty要求をHKC_BacnetServiceへ中継。接続先テーブルからDeviceIdを取得し、応答データを8WAY受信バッファへ格納。 | V10/src/com/interface/library/BACnet/HKC_IFSysBacnetEthernet.h<br>V10/src/com/interface/library/BACnet/HKC_IFSysBacnetEthernet.cpp |
| 主体（ドライバ-BACnet変換: Serial拡張予定） | HKC_IFSysBacnetSerial | HKC_IFSysBacnetEthernetと同一の役割・処理構成を想定。相違点は継承元のみで、HKC_IFSysEthernetの代わりにHKC_IFSysSerialを継承する想定。ReadProperty/WriteProperty要求の中継先やサービス層以降の構成は共通。 | V10/src/com/interface/library/BACnet/HKC_IFSysBacnetSerial.h<br>V10/src/com/interface/library/BACnet/HKC_IFSysBacnetSerial.cpp |
| 制御（BACnetサービス） | HKC_BacnetService | reqReadProperty/reqWritePropertyを受け、BACnet処理を実行して戻り値データを保持。クライアントIFからgetReturnDataで参照される。内部でHKC_BacnetWrapperAPIを保持。 | V10/src/sys/service/HKC_BacnetService.h<br>V10/src/sys/service/HKC_BacnetService.cpp |
| 補助（ライブラリラッパ） | HKC_BacnetWrapperAPI | bacnet-stack関連ライブラリのロード/アンロード、関数ポインタ解決、各API呼び出しをラップ。address_add/address_set_device_TTL等の呼び出し窓口。 | V10/src/sys/service/BACnet/HKC_BacnetWrapperAPI.h<br>V10/src/sys/service/BACnet/HKC_BacnetWrapperAPI.cpp |
| 補助（I/F要求応答構造体） | Bacnet_Send_Common<br>Bacnet_Recv_Common<br>Bacnet_Send_ReadProperty<br>Bacnet_Send_WriteProperty | HKC_IFSysBacnetEthernet/HKC_IFSysBacnetSerialの送受信バッファで使用する要求/応答データ形式。 | Grobal/include/DrvLibraryStruct.h |
| 外部基底クラス | HKC_IFSysEthernet | Ethernet 8WAY通信の共通処理を提供。HKC_IFSysBacnetEthernetはこれを継承し、BACnet独自のsend/recv処理を実装。 | V9/src/com/interface/HKC_IFSysEthernet.h<br>V9/src/com/interface/HKC_IFSysEthernet.cpp |
| 外部基底クラス | HKC_IFSysSerial | Serial 8WAY通信の共通処理を提供する想定基底クラス。HKC_IFSysBacnetSerialではこの基底に差し替える。 | V9/src/com/interface/HKC_IFSysSerial.h |


### データフロー図

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
    subgraph IF[HKC_IFSysBacnet / HKC_IFSysBacnetSerial]
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

### 1.9.2 シーケンス図（Read要求）

```mermaid
sequenceDiagram
  participant DM as ドライバマネージャ
  participant BAC as HKC_IFSysBacnet
  participant SVC as HKC_BacnetService
  participant WRP as HKC_BacnetWrapperAPI
  participant STK as bacnet-stack
  participant DEV as BACnet Device

  DM->>BAC: send8Way(Read要求パケットを送信)
  BAC->>BAC: getHandle(接続先テーブルからDeviceIdを取得)
  BAC->>BAC: getSockAddrfromCacheTable(局番→接続先情報を解決)

  alt 接続先情報なし
    BAC-->>DM: recv8Way(エラー応答を返却)
  else 接続先情報あり
    BAC->>BAC: sendWithBacnet(要求コマンドを解析してRead分岐)
    BAC->>BAC: readProperty(要求パラメータを展開してイベント要求を生成)
    BAC->>SVC: reqReadProperty(ReadProperty実行を要求)
    BAC->>BAC: loop.waitEvent(完了通知まで待機)

    SVC->>SVC: execReadProperty(Read処理を実行)
    SVC->>WRP: Fnc_bacnet_read_property_queue(Read要求をキュー投入)
    WRP->>STK: ReadProperty要求を実行
    STK->>DEV: ReadPropertyRequest(オブジェクト/プロパティ読出し)
    DEV-->>STK: ReadPropertyACK(値/ステータス)
    STK-->>WRP: 読出し結果を返却
    WRP-->>SVC: デコード用データを返却
    SVC->>SVC: chkReadWritePropertyReply(応答内容を解析)
    SVC->>SVC: getReturnData(戻り値データを保持)
    SVC-->>BAC: イベント完了通知(結果コードを返却)

    BAC->>BAC: convertDetailErrToEthernetErr(詳細エラーを8WAYエラーへ変換)
    BAC->>BAC: m_aucRxDataへ応答格納(共通応答+読出しデータ)
    BAC-->>DM: recv8Way(応答データを返却)
  end
```

# BACnetサーバ機能

## 構造図（Mermaid）

### クラス構成図

```mermaid
graph TD
    Base[HKC_Service サービス基底]
    Svc[HKC_BacnetService サーバ中核]
    Comm[HKC_BacnetServerCommSetting 通信設定]
    Node[HKC_BacnetVaiableNode ノード管理]
    ConvCls[encode/decode機能]
    Cache[variableNodeList<br/>monitorData / updateData<br/>listMemory]
    Timer[QTimer 周期タスク]
    Wrapper[HKC_BacnetWrapperAPI stackラッパ]
    StackLib[(bacnet-stack ライブラリ)]

    Svc -->|inherits| Base
    Svc -->|owns| Comm
    Svc -->|owns| Node
    Svc -->|uses| ConvCls
    Svc -->|cache| Cache
    Svc -->|controls| Timer
    Svc -->|libAccess| Wrapper
    ConvCls -->|convert/decode支援| Wrapper
    Wrapper -->|関数呼び出し| StackLib

    style Svc fill:#cfe2ff,stroke:#1a4d8c,stroke-width:2px
    style Comm fill:#d9ead3,stroke:#2f6b2f,stroke-width:2px
    style Node fill:#d9ead3,stroke:#2f6b2f,stroke-width:2px
    style ConvCls fill:#d9ead3,stroke:#2f6b2f,stroke-width:2px
    style Wrapper fill:#ffe6b3,stroke:#b06a00,stroke-width:2px
    style StackLib fill:#dddddd,stroke:#666,stroke-dasharray:4 3
```

### クラス構成（実装ソース抽出）

| 分類 | クラス/構造体 | 役割 | ファイル |
|---|---|---|---|
| 主体（サービス） | HKC_BacnetService | HKC_Serviceを継承するBACnetサービスの中核クラス。サーバ起動/停止、初期設定、Object生成、監視メモリ更新、ReadProperty/WriteProperty要求受付、戻り値保持、サーバキャッシュ更新を担う。内部で通信設定、ノード一覧、監視データ、更新データ、タイマ、ライブラリアクセサを保持する。 | V10/src/sys/service/HKC_BacnetService.h<br>V10/src/sys/service/HKC_BacnetService.cpp |
| 制御（周期） | HKC_SysCycleBacnet | HKC_SystemCycleInterfaceを継承し、システム周期で対象メモリのchkMemStatを実施した後、HKC_BacnetServiceのデータ更新処理を呼び出して監視対象データを更新する。 | V10/src/app/control/syscycle/HKC_SysCycleBacnet.h<br>V10/src/app/control/syscycle/HKC_SysCycleBacnet.cpp |
| 制御（ライブラリ抽象化） | HKC_BacnetWrapperAPI | QObject派生。bacnet-stack系ライブラリの動的ロード、関数ポインタ解決、各種BACnet API呼び出しをラップする。Read/Write要求、Who-Is、address_add、address_set_device_TTL、各種encode/decode補助の窓口となる。 | V10/src/sys/service/BACnet/HKC_BacnetWrapperAPI.h<br>V10/src/sys/service/BACnet/HKC_BacnetWrapperAPI.cpp |
| 補助（通信設定） | HKC_BacnetServerCommSetting | 通信設定を保持するクラス。IPアドレス、デバイス名、セッションタイムアウト、接続形式、自局DeviceIdなどを保持し、内部でキャッシュ登録を行う。 | V10/src/sys/service/HKC_BacnetService.h<br>V10/src/sys/service/HKC_BacnetService.cpp |
| 補助（ノード） | HKC_BacnetVaiableNode | BACnetのObject/Propertyに対応する内部ノード表現。 | V10/src/sys/service/HKC_BacnetService.h<br>V10/src/sys/service/HKC_BacnetService.cpp |
| 補助（変換ユーティリティ） | HKD_BacnetNodeInfoConvert 名前空間 | BACNET_APPLICATION_DATA_VALUEとQVector<quint8>、メモリ要求、ノードキー情報の相互変換を行う関数群を提供する。書込用Variant生成、読出し値変換、ノードキー生成、メモリオーダー補正などを担う。 | V10/src/sys/service/HKC_BacnetService.h<br>V10/src/sys/service/HKC_BacnetService.cpp |

### データフロー図

```mermaid
graph TD
    subgraph App[HMIアプリ層]
      UI[画面設定/Object Property定義]
      Mem[内部/PLCメモリ]
    end
    subgraph ClientFunc[クライアント機能]
      IFReq[HKC_IFSysBacnet Read/Write要求]
      IFResp[HKC_IFSysBacnet 応答受信]
    end
    subgraph Cycle[システムサイクル]
      CycleIF[HKC_SystemCycleInterface]
    end
    subgraph Svc[HKC_BacnetService]
      Init[設定読込・Object Property生成]
      Mon[メモリ監視]
      Upd[更新判定]
      Cache[サーバキャッシュ更新]
      SvcReq[Read/Write要求受付]
      ConvSvc[encode/decode機能]
    end
    subgraph Wrap[HKC_BacnetWrapperAPI]
      Req[Read/Write要求処理]
    end
    subgraph Stack[BACnet-stack層]
      Task[周期タスク/コールバック]
    end
    subgraph Net[ネットワーク]
      Client[(BACnet Client)]
    end

    UI --> Init --> Cache
  CycleIF --> Mon
    Mem --> Mon --> Upd --> ConvSvc --> Req --> Task --> Client
    Client --> Task --> Req --> ConvSvc --> Cache --> Upd --> Mem
    IFReq --> SvcReq --> Req --> Task --> Client
    Client --> Task --> Req --> SvcReq --> IFResp

    style Svc fill:#cfe2ff,stroke:#1a4d8c
    style Wrap fill:#d9ead3,stroke:#2f6b2f
    style Stack fill:#ffe6b3,stroke:#b06a00
```

