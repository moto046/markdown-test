
## 修正履歴

| 版数 | 改訂日 | 改訂者 | 改訂内容 |
| --- | --- | --- | --- |
| 1.0 | 2026-06-10 |  | 初版作成 |
| 1.1 | 2026-06-23 |  | 「4. クライアント機能としての制限・注意点」章を追加 |
|     |            |  | 「5. 各Object Typeで指定するProperty一覧」の各表にクライアント対応の列を追加 |

# BACnet エディタ設定ガイド

本資料は、エディタ設定で必要な情報だけを抜粋した一覧です。

## 概要

### 接続方式

BACnet で対応する接続方式は以下の 2 つです。

- **BACnet/BIP（Ethernet）**
- **BACnet/MS/TP（Serial）**

### サーバ/クライアント機能

各接続方式ともにサーバ機能とクライアント機能の両方に対応しています。

### 通信設定の共通/個別について

サーバ/クライアントそれぞれに個別の通信設定を行うことはできず、共通の通信設定が必要です。
例えば BACnet/BIP であれば、サーバが通信ポート 40000 を使用し、クライアントが別ポートを使用する、といったことはできません。サーバ/クライアントともに同一のポート No を使用する必要があります。
ただし、サーバ/クライアントで個別に設定する通信設定も一部存在します。

共通の通信設定の詳細は以下を参照してください。

- BACnet/BIP（Ethernet）: [Ethernetでの共通設定](#ethernetでの共通設定)
- BACnet/MS/TP（Serial）: [Serialでの共通設定](#serialでの共通設定)

### クライアント機能の設定

クライアント機能はこれまでと同様にドライバとして PLC1〜8 に設定します。
BACnet/BIP と BACnet/MS/TP のドライバは別々となります。

### サーバ機能の設定

サーバ機能は OPC UA サーバと同様に IIoT 設定にて行うことを想定しています。
BACnet/BIP と BACnet/MS/TP のサーバ設定はそれぞれ別設定となります。
サーバ/クライアント共通の通信設定についても IIoT 設定内で行います。

## 1. 対応するObject Type一覧

| Object Type | 略称 | クライアント機能 | サーバ機能 | 補足 |
|---|---|---|---|---|
| Analog Input | AI | ○ | ○ | |
| Analog Output | AO | ○ | ○ | |
| Binary Input | BI | ○ | ○ | |
| Binary Output | BO | ○ | ○ | |
| Device | - | ○ | ○ | 必須（1インスタンス） |
| Network Port | NP | ○ | ○ | 必須（1インスタンス） |

## 2. 対応するデータ型一覧

| データ型 | バイト数 | 用途例 |
|---|---:|---|
| BOOLEAN | 1 | ON/OFF、有効/無効 |
| Unsigned | 8 | ID、カウンタ、タイムアウト値 |
| REAL | 4 | アナログ値、しきい値、速度 |
| ENUMERATED | 4 | 状態種別、モード種別 |
| BIT STRING | 可変 | 状態フラグ群（複数ビット） |
| CharacterString | 可変 | 名称、説明文 |
| OCTET STRING | 可変 | IPアドレス、MACアドレス等のバイト列 |
| ObjectIdentifier | 4 | Object Type + Instance の識別子 |

補足:

- 可変長データ型は、対象プロパティと設定内容により必要サイズが変わる
- BIT STRING / OCTET STRING は、各Propertyごとに設定可能なバイト数（または範囲）を定義し、Property一覧に明記する方針とする
- CharacterString はエディタでバイト数範囲を 0～255（NULL文字を除く）として設定する
- ただし `Object_Name` は空文字不可のため 1～255 文字で設定する
- Unsigned はモニタッチ実機環境では 8 バイト固定で使用する

## 3. Object Instanceが有効な一覧（設定可能範囲）

### 共通ルール

- Object Identifier の Instance Number は 22bit（0 〜 4,194,303）
- 上記はBACnet識別子としての有効範囲
- AI/AO/BI/BO は Instance 指定可能だが、未指定でも運用上問題ない（必要時のみ設定）

### Object Type別

| Object Type | Instance設定 | 設定可能範囲 | 備考 |
|---|---|---|---|
| AI | 設定可（任意） | 0 〜 4194303 | 未設定でも可。複数作成も可能 |
| AO | 設定可（任意） | 0 〜 4194303 | 未設定でも可。複数作成も可能 |
| BI | 設定可（任意） | 0 〜 4194303 | 未設定でも可。複数作成も可能 |
| BO | 設定可（任意） | 0 〜 4194303 | 未設定でも可。複数作成も可能 |
| Device | 1インスタンス必須 | 1つのみ | システムに必須 |
| NP | 1インスタンス必須 | 1つのみ | システムに必須 |

## Ethernetでの共通設定

BACnetのEthernet運用でサーバ/クライアントで設定が必要な共通パラメータ一覧。
IIoT設定画面内で設定を行う。

**ドライバへのデータ反映:**

以下の共通設定は、PLC1～8で設定したドライバ画面には表示されません。<br>
ただし、設定値は既存のPLC1～8の各画面データへ反映してください。<br>
反映先は、以下の「ドライバの対象設定」を参照してください。

**注意点:**

- 以下のデバイス識別子はBACnetネットワーク上で一意である必要があります。
- BBMDアドレス（FD登録先）、BBMDポートNo（FD登録先）、FD登録TTL（秒）の設定はFD機能を使用する場合のみ設定可能となります。

| エディタでの設定内容 | ドライバの対象設定 | 初期値 | 設定可能範囲 | バイト数 | 対象Property | 対象Object | 対象データ型 | 集合種別 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| デバイス識別子 | なし | 260001 | 0 ~ 4194303 | 4 | Object_Identifier | Device | ObjectIdentifier | 単一値 |
| タイムアウト(*100ms) | タイムアウト時間 | 30 | 0 ~ 999 | 2 | APDU_Timeout | Device | Unsigned | 単一値 |
| リトライ回数 | リトライ回数 | 3 | 0 ~ 255 | 1 | Number_Of_APDU_Retries | Device | Unsigned | 単一値 |
| ポートNo | 自局ポートNo. | 47808 | 0〜65535 | 2 | BACnet_IP_UDP_Port | NP（BACnet/IP） | Unsigned | 単一値 |
|  |  |  |  |  | MAC_Address | NP（BACnet/IP） | OCTET STRING | 単一値 |
| BBMDアドレス（FD登録先）| なし | 0.0.0.0(0) | 0.0.0.0 ~ 255.255.255.255(0 ~ 0xFFFFFFFF) | 4 | FD_BBMD_Address | NP（BACnet/IP） | 構造体 | 単一値 |
| BBMDポートNo（FD登録先）| なし | 47808 | 1024 ~ 65535 | 2 | FD_BBMD_Address | NP（BACnet/IP） | 構造体 | 単一値 |
| FD登録TTL（秒） | なし | 600 | 0〜65535 | 2 | FD_Subscription_Lifetime | NP（BACnet/IP） | Unsigned | 単一値 |

Propertyに関連しない共通設定は以下が該当します。

| エディタでの設定内容 | 初期値 | 設定可能範囲 | バイト数 | 説明 | ドライバの対象設定 |
| --- | --- | --- | --- | --- | --- |
| 接続先ポート | LAN | LAN/LAN2 | 1 | 接続先ポート | 本体側ポート |
| ローカル画面切替時の初期化 | 1 | 0 / 1 | 1 | ローカル画面切替時のProperty値の初期化有無。<br>1の場合はローカル画面移行時にPropertyの値を解放する。<br>0の場合は画面データ変更時にPropertyの値を解放する。 | なし |

## Serialでの共通設定

BACnetのSerial運用でサーバ/クライアントで設定が必要な共通パラメータ一覧。
IIoT設定画面内で設定を行う。

**ドライバへのデータ反映:**

以下の共通設定は、PLC1～8で設定したドライバ画面には表示されません。<br>
ただし、設定値は既存のPLC1～8の各画面データへ反映してください。<br>
反映先は、以下の「ドライバの対象設定」を参照してください。

**注意点:**

- 以下のデバイス識別子はBACnetネットワーク上で一意である必要があります。

| エディタでの設定内容 | ドライバの対象設定 | 初期値 | 設定可能範囲 | バイト数 | 対象Property | 対象Object | 対象データ型 | 集合種別 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| デバイス識別子 | なし | 260001 | 0 ～ 4194303 | 4 | Object_Identifier | Device | ObjectIdentifier | 単一値 |
| タイムアウト(*100ms) | タイムアウト時間 | 30 | 0 ~ 999 | 2 | APDU_Timeout | Device | Unsigned | 単一値 |
| リトライ回数 | リトライ回数 | 3 | 0 ~ 255 | 1 | Number_Of_APDU_Retries | Device | Unsigned | 単一値 |
| ステーションアドレス | 自局番 | 0 | 0〜127 | 1 | MAC_Address | NP（BACnet/MS/TP） | OCTET STRING | 単一値 |
| 通信速度 | ボーレート | 38400 | 9600 / 19200 / 38400 / 57600 / 76800 / 115200 | 4 | Link_Speed | NP（BACnet/MS/TP） | REAL | 単一値 |
| 最大マスターアドレス | なし | 127 | 0〜127 | 1 | Max_Manager | NP（BACnet/MS/TP） | Unsigned | 単一値 |
| 1トークンあたりの最大送信フレーム数 | なし | 1 | 1〜8 | 1 | Max_Info_Frames | NP（BACnet/MS/TP） | Unsigned | 単一値 |

Propertyに関連しない共通設定は以下が該当します。

| エディタでの設定内容 | 初期値 | 設定可能範囲 | バイト数 | 説明 | ドライバの対象設定 |
| --- | --- | --- | --- | --- | --- |
| 接続先ポート | CN1(TSの場合COM1) | CN1/MJ1/MJ2(TSの場合はCOM1/COM3) | 1 | 接続先ポート。RS485で通信可能なポートのみ選択可能 | 本体側ポート |
| ローカル画面切替時の初期化 | 1 | 0 / 1 | 1 | ローカル画面切替時のProperty値の初期化有無。<br>1の場合はローカル画面移行時にPropertyの値を解放する。<br>0の場合は画面データ変更時にPropertyの値を解放する。 | なし |

設定するポート（選択可能なポートを含む）は、PLC1～8やプリンタポートなど、他のシリアル通信で使用中のポートと重複できません。

## 4. クライアント機能としての制限・注意点

本章は、後続の Property 一覧をクライアント機能へ適用して解釈する際の制限事項を整理したものである。

- 本ガイドの Property 一覧にある `初期値設定要否` / `初期値` / `設定可能範囲` / `メモリ設定要否` / `メモリ書込可否` は、サーバ機能向けの定義である。クライアント機能では同じ意味では適用せず、主に `アクセス` / `データ型` / `集合種別` / `バイト数` / `有効bit数` を参照する。
- クライアント機能では、ユーザーが指定した対象項目に対して、読み取りまたは書き込みの通信をそのまま実行する。
- `アクセス` は BACnet 仕様上または一般的な機器実装上の想定アクセス区分を示す参考情報であり、モニタッチが RP / WP を送信する可否そのものを制限するものではない。`R` の Property に対しても、ユーザー指定があれば WP を送信できるが、接続先機器側が拒否する可能性がある。
- `Array（固定長: n要素）` / `Array（可変長）` の Property は、指定した配列 No のみをリードすることはできるが、複数の配列 No をまたいで一括リードすることはできない。複数配列が対象となる場合は、全配列データをリードした後、指定された配列 No に対応するデータを抜き出して取得する。ライト時も同様に、1 つの配列 No への書き込みは対応しているが、複数配列への一括書き込みは対応していない。複数配列が対象となる場合は ReadModifyWrite を実施する。
- `List（固定長: n要素）` / `List（可変長）` の Property は、指定した配列 No のみをリードすることはできず、全配列データのリードのみ対応している。リードを実施する場合は全配列データをリードした後、指定された配列 No に対応するデータを抜き出して取得する。ライト時も同様に全配列データへの書き込みに対応しているため、List 型のライト時は ReadModifyWrite を実施する。なお、リード/ライト時ともに指定された配列 No のデータが存在しない場合（要素の追加・削除に相当する操作を含む）はエラーとなる。
- `List（固定長: n要素）` / `List（可変長）` の Propertyの要素の追加・削除が必要な場合は、通常の 8WAY 通信経由ではなく専用マクロでの対応を別途必要となる。
- 可変長の `CharacterString` / `BIT STRING` / `OCTET STRING` は、Property ごとにデータ長が異なる。クライアント機能で扱う際は、必要なバイト数をユーザーが指定するのではなく、エディタ側で固定で設定を行う必要がある。設定するバイト数は `バイト数` を参照すること。
- `CharacterString` は BACnet 上では UTF-8 を扱えるが、モニタッチの文字列アイテムから UTF-8 を表示または書き込みすることはできない。そのため、接続先 Property が UTF-8 文字列を返す場合は表示・編集方法に注意が必要である。
- `集合種別` が構造体のPropertyについては、ユーザーからの要求がない限りはクライアント機能としての指定は不可とする。

## 5. 各Object Typeで指定するProperty一覧

**固定事項（変更不可）:**

- 各Object Typeで指定可能なPropertyは以下の一覧で固定であり、一覧外のPropertyを追加することはできない。
- 各Propertyのデータ型は変更できない。
- 各Propertyのメモリ割当可否・メモリ書込可否は固定であり、変更できない。

**ユーザー設定可能な事項:**

- メモリ割当が可能なPropertyに対して、メモリ割当をするかどうかはユーザーが選択できる。
- メモリ割当の初期状態はすべて「無」とする（メモリ重複を防ぐため）。

**適用範囲に関する注意:**

- 本セクションの表にある `初期値設定要否` / `初期値` / `設定可能範囲` / `メモリ設定要否` / `メモリ書込可否` は、サーバ機能向けの定義とする。
- クライアント機能として表を参照する場合は、前章「クライアント機能としての制限・注意点」を前提とする。

列の見方:

- `アクセス`: 外部クライアント機能からのアクセス区分（`R` / `RW`）
- `データ型`: BACnetで定義されるデータ型。ObjectIdentifier、BOOLEAN、Unsigned、REAL、ENUMERATED、BIT STRING、CharacterString、OCTET STRING、構造体等
- `集合種別`: Propertyが保持する値の形式。`単一値`、`Array（固定長: n要素）`、`Array（可変長）`、`List（固定長: n要素）`、`List（可変長）` のいずれか
- `クライアント対応`: クライアント機能で指定可能かどうかを示す。対応は `○`、対応していない場合は記載なし
- `初期値設定要否`: エディタで初期値入力が必要な項目は `○`
- `初期値`: 初期値設定可能な項目のみ記載
- `設定可能範囲`: 初期値設定可能な項目で設定できる値の範囲・列挙値
- `メモリ設定要否`: エディタでメモリ割付が必要な項目は `○`
- `メモリ書込可否`: メモリ設定要否が `○` の場合のみ判定（書き込み可能は `○`）
- `バイト数`: Propertyが占有するメモリ領域のバイト数。可変長の場合は「可変」と記載
- `有効bit数`: BIT STRING型Propertyで有効とするbit数。冗長化を避けるためbit割当の詳細は記載しない
- `アクセス` は外部クライアント機能によるBACnetアクセス区分を示し、`メモリ書込可否`（内部メモリ/PLCメモリへの書込可否）とは別概念とする

集合種別の表記ルール:

- `単一値`
- `Array（固定長: n要素）` / `Array（可変長）`
- `List（固定長: n要素）` / `List（可変長）`

注意点：
- Object_Name 初期値の <instance> は、対象Objectの Instance設定で指定した Instance No を初期表示すること。

### AI（Analog Input）

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object_Identifier | 75 | R | ObjectIdentifier | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト識別子 |
| Object_Name | 77 | R | CharacterString | 単一値 | ○ | ○ | "ANALOG INPUT <instance>" | 1〜255文字（一意） | × | - | 0～255 | - | オブジェクト名 |
| Object_Type | 79 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト種別（AI固定） |
| Present_Value | 85 | R | REAL | 単一値 | ○ | ○ | 0.0 | 実装上のREAL範囲 | ○ | ○ | 4 | - | 現在値 |
| Description | 28 | R | CharacterString | 単一値 | ○ | ○ | "" | 0〜255文字 | × | - | 0～255 | - | 説明文 |
| Status_Flags | 111 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 1 | 4 | 状態フラグ |
| Event_State | 36 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | イベント状態 |
| Reliability | 103 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | 信頼性状態 |
| Out_Of_Service | 81 | RW | BOOLEAN | 単一値 | ○ | ○ | false | false / true | ○ | ○ | 1 | - | サービス外フラグ |
| Units | 117 | RW | ENUMERATED | 単一値 | ○ | ○ | 98 | 0〜65535（BACnet Engineering Units列挙。実運用は定義済み列挙値から選択） | ○ | ○ | 4 | - | 工学単位 |
| COV_Increment | 22 | RW | REAL | 単一値 | ○ | ○ | 1.0 | 0より大きいREAL | ○ | ○ | 4 | - | COV通知しきい値 |
| Property_List | 371 | R | ENUMERATED | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持プロパティ一覧 |

### AO（Analog Output）

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object_Identifier | 75 | R | ObjectIdentifier | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト識別子 |
| Object_Name | 77 | R | CharacterString | 単一値 | ○ | ○ | "ANALOG OUTPUT <instance>" | 1〜255文字（一意） | × | - | 0～255 | - | オブジェクト名 |
| Object_Type | 79 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト種別（AO固定） |
| Present_Value | 85 | RW | REAL | 単一値 | ○ | ○ | 0.0 | 実装上のREAL範囲（AOはMin〜Max内で運用） | ○ | ○ | 4 | - | 現在値 |
| Description | 28 | R | CharacterString | 単一値 | ○ | ○ | "" | 0〜255文字 | × | - | 0～255 | - | 説明文 |
| Status_Flags | 111 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 1 | 4 | 状態フラグ |
| Event_State | 36 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | イベント状態 |
| Reliability | 103 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | 信頼性状態 |
| Out_Of_Service | 81 | RW | BOOLEAN | 単一値 | ○ | ○ | false | false / true | ○ | ○ | 1 | - | サービス外フラグ |
| Units | 117 | RW | ENUMERATED | 単一値 | ○ | ○ | 95 | 0〜65535（BACnet Engineering Units列挙。実運用は定義済み列挙値から選択） | ○ | ○ | 4 | - | 工学単位 |
| Min_Pres_Value | 69 | RW | REAL | 単一値 | ○ | ○ | 0 | 実装上のREAL範囲 | ○ | ○ | 4 | - | 許容最小値 |
| Max_Pres_Value | 65 | RW | REAL | 単一値 | ○ | ○ | 100 | 実装上のREAL範囲 | ○ | ○ | 4 | - | 許容最大値 |
| COV_Increment | 22 | RW | REAL | 単一値 | ○ | ○ | 1.0 | 0より大きいREAL | ○ | ○ | 4 | - | COV通知しきい値 |
| Priority_Array | 87 | R | REAL | Array（固定長: 16要素） | ○ | × | - | - | × | - | 64<br>(4x16) | - | 優先度別制御値（16段） |
| Relinquish_Default | 104 | R | REAL | 単一値 | ○ | ○ | 0.0 | 実装上のREAL範囲 | × | - | 4 | - | 優先未指定時の既定値 |
| Current_Command_Priority | 431 | R | Unsigned | 単一値 | ○ | × | - | - | ○ | × | 8 | - | 現在有効な優先度 |
| Property_List | 371 | R | ENUMERATED | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持プロパティ一覧 |

- AOの `Present_Value` を書き込む場合は Priority 指定(1 ~ 16)の設定も必要。初期値は8とする。

### BI（Binary Input）

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object_Identifier | 75 | R | ObjectIdentifier | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト識別子 |
| Object_Name | 77 | R | CharacterString | 単一値 | ○ | ○ | "BINARY INPUT <instance>" | 1〜255文字（一意） | × | - | 0～255 | - | オブジェクト名 |
| Object_Type | 79 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト種別（BI固定） |
| Present_Value | 85 | R | ENUMERATED | 単一値 | ○ | ○ | 0 | INACTIVE(0) / ACTIVE(1) | ○ | ○ | 4 | - | 現在状態（INACTIVE/ACTIVE） |
| Description | 28 | R | CharacterString | 単一値 | ○ | ○ | "" | 0〜255文字 | × | - | 0～255 | - | 説明文 |
| Status_Flags | 111 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 1 | 4 | 状態フラグ |
| Event_State | 36 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | イベント状態 |
| Reliability | 103 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | 信頼性状態 |
| Out_Of_Service | 81 | RW | BOOLEAN | 単一値 | ○ | ○ | false | false / true | ○ | ○ | 1 | - | サービス外フラグ |
| Polarity | 90 | RW | ENUMERATED | 単一値 | ○ | ○ | 0 | NORMAL(0) / REVERSE(1) | ○ | ○ | 4 | - | 極性（通常/反転） |
| Active_Text | 4 | R | CharacterString | 単一値 | ○ | ○ | "Active" | 0〜255文字 | × | - | 0～255 | - | ACTIVE表示文字列 |
| Inactive_Text | 46 | R | CharacterString | 単一値 | ○ | ○ | "Inactive" | 0〜255文字 | × | - | 0～255 | - | INACTIVE表示文字列 |
| Property_List | 371 | R | ENUMERATED | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持プロパティ一覧 |

### BO（Binary Output）

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object_Identifier | 75 | R | ObjectIdentifier | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト識別子 |
| Object_Name | 77 | R | CharacterString | 単一値 | ○ | ○ | "BINARY OUTPUT <instance>" | 1〜255文字（一意） | × | - | 0～255 | - | オブジェクト名 |
| Object_Type | 79 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト種別（BO固定） |
| Present_Value | 85 | RW | ENUMERATED | 単一値 | ○ | ○ | 0 | INACTIVE(0) / ACTIVE(1) | ○ | ○ | 4 | - | 現在状態（INACTIVE/ACTIVE） |
| Description | 28 | R | CharacterString | 単一値 | ○ | ○ | "" | 0〜255文字 | × | - | 0～255 | - | 説明文 |
| Status_Flags | 111 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 1 | 4 | 状態フラグ |
| Event_State | 36 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | イベント状態 |
| Reliability | 103 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | 信頼性状態 |
| Out_Of_Service | 81 | RW | BOOLEAN | 単一値 | ○ | ○ | false | false / true | ○ | ○ | 1 | - | サービス外フラグ |
| Polarity | 90 | RW | ENUMERATED | 単一値 | ○ | ○ | 0 | NORMAL(0) / REVERSE(1) | ○ | ○ | 4 | - | 極性（通常/反転） |
| Active_Text | 4 | R | CharacterString | 単一値 | ○ | ○ | "Active" | 0〜255文字 | × | - | 0～255 | - | ACTIVE表示文字列 |
| Inactive_Text | 46 | R | CharacterString | 単一値 | ○ | ○ | "Inactive" | 0〜255文字 | × | - | 0～255 | - | INACTIVE表示文字列 |
| Priority_Array | 87 | R | ENUMERATED | Array（固定長: 16要素） | ○ | × | - | - | × | - | 64<br>(4x16) | - | 優先度別制御値（16段） |
| Relinquish_Default | 104 | R | ENUMERATED | 単一値 | ○ | ○ | 0 | INACTIVE(0) / ACTIVE(1) | × | - | 4 | - | 優先未指定時の既定値 |
| Current_Command_Priority | 431 | R | Unsigned | 単一値 | ○ | × | - | - | ○ | × | 8 | - | 現在有効な優先度 |
| Property_List | 371 | R | ENUMERATED | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持プロパティ一覧 |

- BOの `Present_Value` を書き込む場合は Priority 指定(1 ~ 16)の設定も必要。初期値は8とする。

### Device

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object_Identifier | 75 | R | ObjectIdentifier | 単一値 | ○ | × | - | - | × | - | 4 | - | デバイス識別子 |
| Object_Name | 77 | R | CharacterString | 単一値 | ○ | ○ | "SimpleServer" | 1〜255文字（一意） | × | - | 0～255 | - | デバイス名 |
| Object_Type | 79 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト種別（Device固定） |
| System_Status | 112 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | デバイス状態（bacnet-stack管理） |
| Vendor_Name | 121 | R | CharacterString | 単一値 | ○ | × | - | - | × | - | 0 ~ 255 | - | ベンダー名 |
| Vendor_Identifier | 120 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | ベンダーID |
| Model_Name | 70 | R | CharacterString | 単一値 | ○ | × | - | - | × | - | 0 ~ 255 | - | モデル名 |
| Firmware_Revision | 44 | R | CharacterString | 単一値 | ○ | × | - | - | × | - | 0 ~ 255 | - | FWリビジョン |
| Application_Software_Version | 12 | R | CharacterString | 単一値 | ○ | × | - | - | × | - | 0 ~ 255 | - | アプリバージョン |
| Protocol_Version | 98 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | BACnetプロトコル版 |
| Protocol_Revision | 139 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | BACnet改訂番号 |
| Protocol_Services_Supported | 97 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 7 | 49 | 対応サービスビット列 |
| Protocol_Object_Types_Supported | 96 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 9 | 65 | 対応Object Typeビット列 |
| Object_List | 76 | R | ObjectIdentifier | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持Object一覧 |
| Max_APDU_Length_Accepted | 62 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | APDU最大長 |
| Segmentation_Supported | 107 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | セグメント対応種別 |
| APDU_Timeout | 11 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | タイムアウト |
| Number_Of_APDU_Retries | 73 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | リトライ回数 |
| Device_Address_Binding | 30 | R | 構造体 | List（可変長） | × | × | - | - | × | - | 可変 | - | アドレスバインディング一覧 |
| Database_Revision | 155 | R | Unsigned | 単一値 | ○ | × | - | - | ○ | × | 8 | - | 構成変更リビジョン |
| Property_List | 371 | R | ENUMERATED | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持プロパティ一覧 |

### NP（Network Port）共通

BACnet/IP、BACnet/MS/TPの接続形式に関係なく共通で必要となるProperty一覧。

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object_Identifier | 75 | R | ObjectIdentifier | 単一値 | ○ | × | - | - | × | - | 4 | - | ポート識別子 |
| Object_Name | 77 | R | CharacterString | 単一値 | ○ | ○ | "Network Port" | 1〜255文字（一意） | × | - | 0～255 | - | ポート名 |
| Object_Type | 79 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | オブジェクト種別（NP固定） |
| Description | 28 | R | CharacterString | 単一値 | ○ | ○ | "" | 0〜255文字 | × | - | 0～255 | - | 説明文 |
| Status_Flags | 111 | R | BIT STRING | 単一値 | ○ | × | - | - | × | - | 1 | 4 | 状態フラグ |
| Reliability | 103 | R | ENUMERATED | 単一値 | ○ | × | - | - | ○ | × | 4 | - | 信頼性状態 |
| Out_Of_Service | 81 | R | BOOLEAN | 単一値 | ○ | × | - | - | × | - | 1 | - | サービス外フラグ |
| Network_Type | 427 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | 通信方式（BIP/MSTP） |
| Protocol_Level | 482 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | プロトコルレベル |
| Changes_Pending | 416 | R | BOOLEAN | 単一値 | ○ | × | - | - | ○ | × | 1 | - | 設定反映待ち有無 |
| Property_List | 371 | R | ENUMERATED | Array（可変長） | ○ | × | - | - | × | - | 可変 | - | 保持プロパティ一覧 |

### NP（BACnet/IP）

BACnet/BIPを使用する場合に必要となるProperty一覧。

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Network_Number | 425 | R | Unsigned | 単一値 | ○ | ○ | 0 | 0〜65534（65535は予約） | × | - | 8 | - | ネットワーク番号 |
| Network_Number_Quality | 426 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | Network_Numberの信頼性 |
| APDU_Length | 399 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | APDU最大長 |
| MAC_Address | 423 | R | OCTET STRING | 単一値 | ○ | × | - | - | × | - | 6 | - | IPv4+UDPポート（6バイト） |
| BACnet_IP_Mode | 408 | RW | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | IP動作モード |
| BACnet_IP_UDP_Port | 412 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | UDPポート番号 |
| FD_BBMD_Address | 418 | RW | 構造体 | 単一値 | × | × | - | - | × | - | 6 | - | 参照BBMDアドレス |
| FD_Subscription_Lifetime | 419 | RW | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | FD登録TTL（秒） |
| IP_Address | 400 | R | OCTET STRING | 単一値 | ○ | × | - | - | × | - | 4 | - | IPv4アドレス |
| IP_Subnet_Mask | 411 | R | OCTET STRING | 単一値 | ○ | × | - | - | × | - | 4 | - | サブネットマスク |
| IP_Default_Gateway | 401 | R | OCTET STRING | 単一値 | ○ | × | - | - | × | - | 4 | - | デフォルトゲートウェイ |
| IP_DNS_Server | 406 | R | OCTET STRING | List（固定長: 3要素） | ○ | × | - | - | × | - | 12<br>(4x3) | - | DNSサーバ一覧 |
| IP_DHCP_Enable | 402 | R | BOOLEAN | 単一値 | ○ | × | - | - | × | - | 1 | - | DHCP有効/無効 |
| IP_DHCP_Lease_Time | 403 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | DHCPリース時間 |
| IP_DHCP_Lease_Time_Remaining | 404 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | DHCPリース残時間 |
| IP_DHCP_Server | 405 | R | OCTET STRING | 単一値 | ○ | × | - | - | × | - | 4 | - | DHCPサーバアドレス |
| Link_Speed | 420 | R | REAL | 単一値 | ○ | × | - | - | × | - | 4 | - | リンク速度 |

### NP（BACnet/MS/TP）

BACnet/MS/TPを使用する場合に必要となるProperty一覧。

| Property | Property Identifier | アクセス | データ型 | 集合種別 | クライアント対応 | 初期値設定要否 | 初期値 | 設定可能範囲 | メモリ設定要否 | メモリ書込可否 | バイト数 | 有効bit数 | 説明 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Network_Number | 425 | R | Unsigned | 単一値 | ○ | ○ | 0 | 0〜65534（65535は予約） | × | - | 8 | - | ネットワーク番号 |
| Network_Number_Quality | 426 | R | ENUMERATED | 単一値 | ○ | × | - | - | × | - | 4 | - | Network_Numberの信頼性 |
| APDU_Length | 399 | R | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | APDU最大長 |
| MAC_Address | 423 | RW | OCTET STRING | 単一値 | ○ | × | - | - | × | - | 1 | - | ステーションアドレス |
| Link_Speed | 420 | RW | REAL | 単一値 | ○ | × | - | - | × | - | 4 | - | 通信速度 |
| Link_Speeds | 421 | R | REAL | Array（固定長: 6要素） | ○ | × | - | - | × | - | 24<br>(4x6) | - | 対応速度一覧 |
| Max_Manager | 64 | RW | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | 最大マスターアドレス |
| Max_Info_Frames | 63 | RW | Unsigned | 単一値 | ○ | × | - | - | × | - | 8 | - | 1トークンあたり最大送信フレーム数 |

## 6. エディタ設定時の注意

- AO/BO の Present_Value を書き込む場合は Priority 設定が必要
- Device と NP は必須Objectのため、未設定にしない
- NP は通信方式（BACnet/IP または BACnet/MS/TP）に応じて指定Propertyが変わる
