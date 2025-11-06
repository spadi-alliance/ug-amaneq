# Mikumari Clock Hub

## Overview

Mikumari Clock Hub (MikuClockHub)は上流から時刻同期を受け、更に16台の下流モジュールを同期する事が出来る、クロック分配ネットワーク上のリレーモジュールです。
Root-modeを有しており、上流から自身を切り離して新しいROOTモジュールとして動作することが出来ます。
Mikumari Clock Root firmwareの開発が中断することから、version 2.7以降ではroot-modeで起動した本モジュールをクロック分配ネットワークの頂点に置くことを推奨します。
単にClockHubと表記する事もあります。
ファームウェア内部に64chの1ns精度連続読み出しTDCも有しています。

[Github repository](https://github.com/AMANEQ-official/MikuClockHub)

```
- Unique ID:                  0xF100

- Number of clock port:       16

- Number of inputs:           64
- Timing measurements:        Both edges
- TDC precision:              1ns
- Double hit resolution:      ~8ns
- Max TOT length:             4000ns

- Link protocol:              SiTCP
- Default IP:                 192.168.10.16
- Data link speed:            1Gbps

- Data word width:            64bit
- Acceptable max input rate:  ~28MHz/board
- System clock freq.:         125MHz
```

### History

|Version|Date|Changes|
|:----:|:----|:----|
|v2.9|2025.10.14| - Fixed the bit position of the frameFlag1/2 swapped in the heartbeat delimiter flags. <br> - 125 MHz system clock signal can be output from NIM-OUT ports by selecting it via IOM. |
|v2.8|2025.5.28| - Bugfix version of v2.7. <br> - Fixed the bug that the delay function for the trigger assisted mode does not work. <br> - A clock signal of 7.8125 MHz, divided by 16 from the system clock, can now be output from the NIM output port.|
|v2.7|2025.3.5| - Bugfix version of v2.6. <br> - Enabling the function to generate data words with input throttling type-2 start/end data types. |
|v2.6|2025.1.6| - Updating LACCP (v2.1) supporting the frame flag distribution. <br> - Introducing gated scaler. <br> - Introducing IO manager block arranging input/output paths to the NIM IO. <br> - Deprecating the extra 129th TDC input from NIM. <br> - Deprecating DIP2 function. <br> - Changing what the LED lights indicate.|
|v2.5|2024.6.9|事実上の初期版|

# Functions

![BL-DIAGRAM](block-diagram.png "Simplified block diagram of MikuClockHub."){: #BL-DIAGRAM width="80%"}

[図](#BL-DIAGRAM)はMikumari-ClockHubの簡易ブロックダイアグラムです。
CRVもしくは前面MIKUMARIポートでクロック信号を受信し、CDD-OPTメザニンカードを用いて最大16台のリーフモジュールを時刻同期する事が出来ます。
加えて、64ch文のStr-LRTDCを内蔵しており、main inputからの信号を測定する事が出来ます。
入力信号はStr-LRTDCと同様にスケーラーブロックにも接続されています。
Str-LRTDCの機能とスケーラー機能についてはStr-LRTDCのページを参照してください。

スタンドアロンモードを有効にするとクロック受信ポートが無効になり、システムクロックをローカル発振器から生成するようになります。
検出器サブブロックをメインDAQシステムから切り離して独自に試験を行いたい時に活用してください。
この場合は本回路のハートビートユニットが定義するハートビートカウンターとフレーム番号が新しい基準になり、各リーフモジュール内のLACCP fine offsetは本回路のハートビートパルスに対する時間差を示します。
また、ハートビートフレーム状態を切り替える事も可能になります。

本回路上のシステムクロック信号はCDCE62002によって生成されるので、**CDCE62002を未設定のAMANEQではこのファームウェアは動作しません。**
125 MHzの入力から500 MHzと125 MHzのクロック信号を生成するようにCDCE62002を設定してください。

![PORT-MAP](port-map.png "Port map of Str-LRTDC"){: #PORT-MAP width="80%"}

[図](#PORT-MAP)はTDC入力チャンネル番号とMIKUMARIのポート番号を示しています。
16番が受信用のMIKUMARIポートで、0番から15番までがクロック信号の送信側です。

### LED and DIP switch (2025.01.06)

1-3番が点灯していればモジュールとして正常に動作しています。

|LED #||Comment|
|:----:|:----|:----|
|1| DAQ is running| データ読み出し中である事を示します。 |
|3| MIKUMARI (16) link up| MIKUMARIポートの16番がリンクアップしている状態です。 |
|2| Root mode| ROOT modeでモジュールが起動していると点灯します。 |
|4| PLL locked| 全ての内部クロック信号が正常に出力されている状態です。 |

|DIP #||Comment|
|:----:|:----|:----|
|1| SiTCP IP setting | 0: デフォルトIPを使用します <br> 1: ユーザー設定のIPを使用します (要ライセンス)。|
|2| Not in use | |
|3| Root mode | 0: Relay mode (clock-hub) <br> 1: Root mode (clock-root)|
|4| Not in use | |

## Local bus modules

Str-LRTDCには6個のローカルバスモジュールが存在します。
以下がローカルバスアドレスのマップです。

|Local Module|Address range|
|:----|:----|
|Mikumari Utility        |0x0000'0000 - 0x0FFF'0000|
|Streaming TDC           |0x1000'0000 - 0x1FFF'0000|
|IO Manager              |0x2000'0000 - 0x2FFF'0000|
|Scaler                  |0x8000'0000 - 0x8FFF'0000|
|CDCE62002 Controller    |0xB000'0000 - 0xBFFF'0000|
|Self Diagnosis System   |0xC000'0000 - 0xCFFF'0000|
|Flash Memory Programmer |0xD000'0000 - 0xDFFF'0000|
|Bus Controller          |0xE000'0000 - 0xEFFF'0000|

## Streaming-TDC block

入力チャンネル数が異なること以外はStr-LRTDCと同様のため、Str-LRTDCの説明を参照してください。
レジスタアドレスも全く同じです。

## Scaler

Str-LRTDCと同様のため、Str-LRTDCの説明を参照してください。
レジスタアドレスも全く同じです。

## IO Manager

IO ManagerはAMANEQのNIM IOとFPGA内部の信号等の接続関係を管理するモジュールです。
NIMポートから入力された信号をどの内部信号へ接続するか、また内部信号をどのNIMポートから出力するかをSiTCPを通じて変更します。

|Register name|Address|Read/Write|Bit width|Comment|
|:----|:----|:----:|:----:|:----|
|kFrameFlag1In  | 0x20000000|  W/R|2| Setting the NIM-IN port to the internal frame flag-1 signal. It is valid when the module is the root mode. (default 0x0)|
|kFrameFlag2In  | 0x20100000|  W/R|2| Setting the NIM-IN port to the internal frame flag-2 signal. It is valid when the module is the root mode. (default (0x1))|
|kTriggerIn     | 0x20200000|  W/R|2| Setting the NIM-IN port to the internal trigger in signal. (default (0x3))|
|kScrResetIn    | 0x20300000|  W/R|2| Setting the NIM-IN port to the internal scaler reset signal. This signal will be distributed to other modules through MIKUMARI. (default (0x3))|
| |  |  | | |
|kSelOutSig1    | 0x21000000|  W/R|3| Selecting the internal signal to output from the NIM-OUT port 1. |
|kSelOutSig2    | 0x21100000|  W/R|3| Selecting the internal signal to output from the NIM-OUT port 2. |

アドレス値が`0x20X0'0000`のレジスタはNIM-INポートをどの内部信号へ接続するかを決定します。
各レジスタに対して設定可能な値は以下の通りです。

|Register value|Comment|
|:----:|:----|
|0x0| Connecting the NIM-IN port 1 to the corresponding internal signal.|
|0x1| Connecting the NIM-IN port 2 to the corresponding internal signal.|
|0x2| Not in use |
|0x3| Connecting GND to the corresponding internal signal. |

アドレス値が`0x21X0'0000`のレジスタはどの内部信号をNIM-OUTポートへ接続するかを決定します。
各レジスタに対して設定可能な値は以下の通りです。

|Register value|Comment|
|:----:|:----|
|0x0| Connecting the heartbeat signal.|
|0x1| Connecting the TCP connection establish.|
|0x2| Connecting the trigger signal from LACCP.|
|0x3| Connecting the frame flag-1.|
|0x4| Connecting the frame flag-2.|
|0x5| Connecting the div16 clock (7.8125 MHz).|
|0x6| Connecting the system clock signal (125 MHz). |
|0x7| Connecting the logic of 1|
