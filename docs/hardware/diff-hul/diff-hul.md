# Differences in HUL

## Major functional differences

- データリンク数がHULは1つ、AMANEQは2つ
- データリンクのメディア規格がHULはLANケーブルのみ、AMANQは光ファイバーを選択可能
- HULのNIM I/Oは合計8つ、AMANQは4つ
- HULの電源電圧は5V、AMANEQは+20-35V
- AMANEQのみジッタクリーナーを搭載している
- AMANEQのみDDR3-SDRAMを搭載している
- AMANEQのみ時刻同期ポートを搭載している
- AMANEQはNIM出力用のICを1段省略しており、NIM信号へ変換する回路に個体差が生じやすい。同じ信号を違うAMANQEで出力すると、NIM信号幅が違う事がある。
- HULのDIPスイッチはONで1になるようにFPGA内でロジック反転しているが、AMANEQでは反転せずシルク表記に従うので、ONで0である

## Differences for firmware developers

- HULはメザニンとの接続を全て1.8V駆動のHPバンクで行っているが、AMANEQでは2.5V駆動のHRバンクと1.8V駆動のHPバンクが混ざっている
- HULのフラッシュメモリはSPIx1で接続されているが、AMANEQではSPIx4を使っている