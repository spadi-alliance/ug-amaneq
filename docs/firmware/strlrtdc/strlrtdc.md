# Streaming low-resolution TDC

## Overview

Streaming low-resolution TDC (Str-LRTDC)は129ch入力の1ns精度連続読み出しTDCです。

- Unique ID: 0x60c4

### History

|Version|Date|Changes|
|:----:|:----|:----|
|v2.5|2024.6.4|事実上の初期版|

## Functions

![BL-DIAGRAM](block-diagram.png "Simplified block diagram of Str-LRTDC."){: #BL-DIAGRAM width="100%"}


[図](#BL-DIAGRAM)はStr-LRTDCの簡易ブロックダイアグラムです。
Main input, メザニンスロット, NIMIN-1を入力として利用し、最大129ch入力を受け付けます。
AMANEQ本体のみで65ch入力が可能であり、129chまで拡張する場合はDCRv2メザニンカードが必要です。
これらの入力信号は連続読み出しTDC (Str-TDC)ブロックとスケーラーブロックに接続されています。
Str-TDCブロックでは1ns精度で信号のリーディングとトレーリングエッジのタイミングを測定し、内部で2つのエッジのペアリングを行いTOTを計算します。
データ送信はSiTCPのTCP機能を使って行います。
Str-TDCはトリガーレスの連続読み出しTDCですが、トリガー信号によって内部ゲートを生成し、選択的にデータを送信するオプション機能が存在します。
TDCブロックの詳細はDAQ機能のセクションで詳しく述べます。

スケーラーブロックは内部に32-bitカウンタを有しており、TDCブロックの動作とは独立に入力信号の数を数えます。
DAQ機能が動いていない時でも、常に入力数を数え続けるフリーランのスケーラーです。
スケーラーデータはRBCPを使ってUDPで読み出します。
RBCPから読み出しリクエストがあるとカウンタ値をラッチして、タイムスタンプを付与してPCへ送信します。
ユーザーはタイムスタンプを見る事で入力レートの計算をすることが出来ます。

MIKUMARIポートはmini-mezzanine CRV (GN-2006-1の場合)かフロントパネルのSFPポート (GN-2006-4)につながっています。
Str-LRTDCはクロック同期経路上のリーフノードであり、これより下流のモジュールを再同期する能力はありません。
MIKUMARIシステムを用いずにスタンドアロンで動作させるときは、システムクロック信号のソースにローカル発振器を選択してください。

![PORT-MAP](port-map.png "Port map of Str-LRTDC"){: #PORT-MAP width="100%"}

[図](#PORT-MAP)はTDC入力チャンネル番号とMIKUMARIのポート番号を示しています。
MIKUMARIのポートはセカンダリが0番にアサインされています。

### LED and DIP switch

MIKUMARIシステムを利用している場合、1-3番がすべて点灯していれば正常です。
スタンドアロンの場合、1番と3番が点灯していれば正常です。

|LED #||Comment|
|:----:|:----|:----|
|1| PLL locked| 全ての内部クロック信号が正常に出力されている状態です。 |
|2| MIKUMARI (0) link up| MIKUMARIポートの0番がリンクアップしている状態です。 |
|3| Ready for DAQ| 時刻同期が完了し、DAQを走らせられる状態である事を示します。 |
|4| DAQ is running| データ読み出し中である事を示します。 |

|DIP #||Comment|
|:----:|:----|:----|
|1| SiTCP IP setting | 0: デフォルトIPを使用します <br> 1: ユーザー設定のIPを使用します (要ライセンス)。|
|2| NIMOUT setting | 0: NIMOUT-1からハートビート信号が出力されます<br>1: NIMOUT-1からLACCPがトリガー信号が出力されます|
|3| Standalone mode | 0: MIKUMARIシステムを使用します<br>1: ローカル発振器を使用しスタンドアロンモードになります|
|4| Not in use | |

<!--
- DIP1: SiTCP IP setting
    - 0: デフォルトIPを使用します
    - 1: ユーザー設定のIPを使用します (要ライセンス)。
- DIP2: NIMOUT setting
    - 0: NIMOUT-1からハートビート信号が出力されます
    - 1: NIMOUT-1からLACCPがトリガー信号が出力されます
- DIP3: Standalone mode
    - 0: MIKUMARIシステムを使用します
    - 1: ローカル発振器を使用しスタンドアロンモードになります
- DIP4: Not in use
-->
