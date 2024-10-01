# Software

ここではRBCP経由でAMANEQをスロー制御するためのソフトウェアについて述べます。
SPADI-Aで開発中のNestDAQソフトウェアについては、NestDAQのユーザーガイドを参照してください。

## hul-common-lib

[Github repository](https://github.com/spadi-alliance/hul-common-lib)

hul-common-libはHULのファームウェアをベースにして開発されたも回路用の共通ライブラリです。
AMANEQ、CIRASAME、RAYRAWの制御はこの共通ライブラリで行う事が可能です。
**非常に残念ながらHULは古いソフトから移行が済んでおらず、現在hul-common-libではHULファームウェアの制御は出来ません。**

**Directory structure**

```
.
└── hul-common-lib/
    ├── HulCore
    └── Common
```

HulCoreにはRBCP通信を行うクラス等、共通のライブラリメソッドが格納されています。
コンパイルするとこの部分をまとめたshared libraryが生成されます。
後述するamaneq-softは、このshared libraryを必要とします。

Common内にはいくつか実行体を作るためのソースファイルが格納されています。
生成される実行体で、AMANEQの制御は全て行う事が出来るようになっています。
他のソフトへ制御シーケンスを移植する際には、これらソースファイルの記述を参考にしてください。

### Executable files

hul-common-libをコンパイルすると生成される実行体について説明します。
**全ての実行体は引数無しで実行するとusageが表示されます。**
取るべき引数を確認するために活用してください。

#### write_register

SiTCPのRBCP経由でlocal bus moduleへレジスタを書き込むための実行体です。RBCP通信クラスを包含しています。
`write_register`と`read_register`の2種類でAMANEQのファームウェアは全て制御できるようになっていますが、制御手順が非常に煩雑な機能については別途実行体を用意しています。

IPアドレス192.168.10.16のローカルアドレス0x40100000へ0x1Aという値を2-byte幅で書き込む例です。
```shell
               [IP]          [Addr]      [val] [bytes]
write_register 192.168.10.16 0x40100000  0x1A  2
```
この関数は4-byte書き込みまで対応しています。
RBCPは本来1-byte幅でしか同じアドレスに書けないのですが、プログラムの中で同一ローカルアドレスに対して他バイトの値を書き込んでいるように見せかけています。

レジスタ長が8-bitの倍数でない場合、例えば13-bit幅のレジスタに2-byte書き込みを行った場合、存在しない上位3-bit部分の値は何を入れても無視されます。

#### read_register

SiTCPのRBCP経由でlocal bus moduleからレジスタを読み出すための実行体です。
読み出し結果は標準出力へ表示されます。

IPアドレス192.168.10.16のローカルアドレス0x40100000から2-byte幅で値を読み出す例です。
```shell
              [IP]          [Addr]     [bytes]
read_register 192.168.10.16 0x40100000 2
```

#### check_spi_device

ボード上のSPIフラッシュメモリの型番を調べます。マニュファクチャIDをメモリから読み出して判別しているので、正常に読み出せていてもソフト側がそれを知らないとunknownと表示されてしまいます。
そのような現象に遭遇したらgithub repositoryでissueを挙げていください。

#### erase_eeprom

SiTCPのライセンスやIPアドレスなどの情報が格納されている不揮発性メモリ（EEPROM）のデータを全消去する実行体です。
[SiTCPのTCP接続が再接続の際に非常に待たされる](https://hul-official.gitlab.io/hul-ug/practical/main/)という現象を解決するために必要な実行体です。
通常は使用しません。

#### flash_memory_programmer

MCSファイルをSiTCP経由でフラッシュメモリに書くための実行体です。
通常MCSファイルはJTAG経由でフラッシュメモリへ書き込みますが、ローカルバスモジュールのFMPを経由して書き込むためのプログラムです。
当然ですが、FMPが実装されたファームウェアがFPGA上で動いている事が前提条件です。

フラッシュメモリの消去、書き込み、ベリファイとJTAGで書き込むときと同じ手順を踏みます。
ベリファイの結果最後に`MCS is verified`と表示されることを確認してください。
Mismatchと表示された場合、通信エラー等で書き込んだデータと読み出したデータが一致していません。
Verifiedと表示されるまで繰り返し書き込んでください。
Mismatchと表示されているのに電源を落としたりFPGAをリコンフィグしてしまったりすると、ボードが起動しなくなります。
その場合はJTAGケーブルを使って対処してください。

RBCPを用いて数MBあるファイルを送信するので、何度もRBCP通信が行われます。
RBCPはUDP通信のためパケットロスの可能性があり、ネットワークが不安定だったり、他の通信パケットが沢山流れている環境だと失敗しやすいです。
また、無線経由でも失敗しやすいです。
`flash_memory_programmer`を使う際には、他のプログラムから該当AMANEQへアクセスするのをやめ、クリーンな状況で書き込んでください。

MCSをフラッシュメモリへ書き込んでもFPGAの中身はそのままです。
書いた内容をFPGAへ反映させるためにはボードの電源を一度落とすか、`reconfig_fpga`を実行してFPGAを再コンフィグしてください。

#### gen_user_reset

Local Bus ControllerからBCTリセットを発行します。
FPGA内部の多くのローカルバスモジュールがリセットされるのでレジスタの再設定を行ってください。

#### get_version

ファームウェアのIDとバージョンを取得します。
何かおかしいと思った時にとりあえず実行するプログラムです。

