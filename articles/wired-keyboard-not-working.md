---
title: "キーボードが反応しない時に試したこと"
emoji: "⌨️"
type: "idea"
topics: ["ポエム", "keyboard", "usb", "linux"]
published: true
published_at: 2023-03-13 06:00
---

※ 技術要素は登場しますが、ちょっとした推理物としてお楽しみください。

## 想定読者
- 昼休みを持て余したエンジニア
- キーボードを無駄に買い替えたくないエンジニア

## 導入
ある日、キーボードを60cmぐらいの高さのベッドから床にうっかり落としてしまった。  
キーボードは**その時**から一切反応しなくなった。  
キーボードはそんなことで壊れてしまったのか？それとも別の何かが原因なのだろうか？

## 登場人物
被害を受けたキーボードはKeychron K8という製品だ。
- メカニカルキーボード⌨️
- USB(有線🧬)接続とBluetooth(無線📶)接続ができる
- JIS配列🇯🇵だけでなくUS配列🇺🇸も選択できる
- 青軸🟦が選択できる
- キーボードが光る💡

![ワイヤレスキーボードの絵](/images/articles/wired-keyboard-not-working/wired-keyboard.jpeg)*操作したりUSB接続すると光るキーボード*

US配列×青軸×無線という選択肢を実現できる、極めて素晴らしいキーボードである。  
そして接続していた端末はデスクトップPCで、OSはDebian11(Linux)であった。  
このOSでの利用は保証されているわけではない。しかし**その時**までは適切に動作していたはずだ。

## 本当に⌨️は壊れたのか？
「落下したぐらいでキーボードは壊れない。」  
と私は高をくくっていた。  
(なお筆者は全く同じ高さから新品のHDDを落下させ、破損させたことがある。**HDDは本当に衝撃荷重に弱い。**)  
とにかくまずは色々試した。USBケーブルを**色々と何度も**差し替えたり、このキーボードについてるWin⇄Macモードのトグルや、USB⇄Bluetoothトグルを切り替えて反応も見た。  
**充電ランプはついている。なのにキーボードはつゆも反応してはくれない。**  
次に考えるべきは影響範囲の特定である。  
手元にちょうどMacBook Pro(2019)があったので、Bluetooth接続をさっそく試してみるとキーボードは軽快に動作した。  
つまり**キーボード本体は壊れていない**ことがわかった。  

## コネクタが壊れたのか？
となると疑わしくなるのは、PCとキーボード間を結線するUSBケーブルである。PC側はType-A、キーボード側はType-Cとなっている。  
ケーブルが落下で壊れるとは考えにくいが、コネクタ部分は壊れるかもしれない。  
(なお筆者はType-Cケーブルを結構破壊する。**Type-Cは曲げ応力とねじり応力に弱いのだ。**)  

💻 ← Type C - Type C → ⌨️  
↑のように両側がUSB Type-CのケーブルでMacとキーボードを接続したら動作した。  

つまり**キーボードのType-Cポート**は壊れていない。 

💻 ← Type C -(変換ハブ)- Type A ← Type A-(ケーブル)-Type C → ⌨️  

さらに↑のようにType-CからType-Aへの変換ハブをMacにさして、ケーブルを接続したがやはり動作した。

つまり**ケーブル**も壊れていない。 

残るはデスクトップPC側のType-Aポートだが、別のポートに繋ぎかえてもやはり反応しない。**(USB2.0でも3.0でも関係がない。)**  
マウスが正常に動作しているポートに差し直しても、やはり動作しないのである。

ここまでで導き出された結論は、**ハードウェアは何も壊れていない**ということである。

## さらなる調査
となるとソフトウェア側を調査するしかない。   
エンジニアにとって「キーボードが使えない=何もできない」ではない。  
我々にはsshという**力**があり、MacはLinuxを使役するシンクライアントの役割があったのだ。  
冷静に考えてキーボード落下で**ソフトウェアのコンフィグが変わるわけない**のだが、
> 不可能を排除していったたとき、どんなにありそうもないことであっても、残ったそれが真実に違いない。  -- シャーロック・ホームズ

という名言もあるしハードウェアの健全性をMacが示した以上、デスクトップを疑うしかできなくなっていた。(最近色々[やらかしてた](https://zenn.dev/link/comments/1e90ead7daecf5)のもあった)  
私はLinuxに詳しいわけではないが、一応Linuxデスクトップを構成できたり、ArchLinuxやCentOS7を使っていたという経験値もある。  
クラウドインスタンスに対して使うことはないであろう`lsusb`コマンドを使うことで、Linuxが認識しているUSBデバイスが列挙できる。  
結果、隣のワイヤレスマウスはしっかり認識しているし、そこら辺にあったUSBメモリもUSBハブも認識する。  
わざわざワイヤレスキーボードだけが、OSから無視されているかのようだ。  
```shell
sudo dmesg
sudo dmesg | grep -i USB
sudo dmesg | grep -i Key
journalctl
```
などで実行色々調べても、何か意味のあるエラーなどが発見できない。  
ひたすらそれらしいワードで検索を進めていき、[usbcore.suspend](https://wiki.archlinux.jp/index.php/%E9%9B%BB%E6%BA%90%E7%AE%A1%E7%90%86)や[blacklist](linux.jp/index.php/カーネルモジュール)の可能性に行きついた。   
`/etc/default/grub`の`GRUB_CMDLINE_LINUX_DEFAULT`に`usbcore.autosuspend=-1`を追加して、`sudo update-grub`するansibleを組んで実行したが、認識しないことに変わりなかった。  
次に`/etc/modprobe.d/`の中に変なblacklist指定がないかも確認したが、nouveau(NVIDIA GPUの旧ドライバ)を除きblacklist指定はなかった。  
xhciなどの単語もちらついたが、USB2.0のポートにわざわざ接続しているので関係はなさそうと考えた。  
そうこうしているうちにうっかりMacの方を`reboot`してしまい、様々な構成上の都合でsshができなくなってしまった。  
解決しなくてもsshできるという余裕が無くなった以上、もう徹底的にやるしかない。

## さらなる実験
結局のところ最も知りたいことは
- 他の有線キーボードであれば問題がないのか？

である。それに加えて
- Bluetooth接続ならば問題がないのか？

も抑えておきたいということで、BluetoothのUSBドングルとエレコムの有線キーボードを購入した。  
有線キーボードの選定基準は「Linuxでも動きそうか」である。Linuxでも動くと謳われているものはなかったが、MacOSやChromeOSでも動くということで期待値で購入した。  
結果としては有線キーボードでもBluetooth接続でも動いた。  
つまり**Linuxは問題なくキーボードを認識できているが、なぜかUSB接続したときだけうまく動かない**らしい。  

## 完全解決編
「なぜUSB接続がうまくいかなくなってしまったのか？」  
この答えを探し求めてさらに検索を推し進めたところ、[reddit](https://www.reddit.com/r/Keychron/comments/ia100g/keychron_k8_not_working_on_wired_mode/)に行き着いた。

曰く
- ケーブルを完全にはめ込んだら直ったよ！
- Type-Cを逆に差し込んだらうまくいったよ！

このとき私は全てを理解した。
落下の衝撃で、ケーブルが微妙に外れたのだということに。  
そして
> とにかくまずは色々試した。USBケーブルを**色々と何度も**差し替えた

の過程の中でType-Cの差し込み方がいつの間にか逆になってしまっていたことに。  

![ワイヤレスキーボードの絵](/images/articles/wired-keyboard-not-working/wrong_usb_connection.jpg)*誤った向き(このままあれこれ操作していた)*  
![ワイヤレスキーボードの絵](/images/articles/wired-keyboard-not-working/correct_usb_connection.jpg)*正しい向き*  
そして[このような記事](https://nlab.itmedia.co.jp/nl/articles/2101/08/news145.html)を即座に思い出した。



## まとめ
- USB Type-Cはリバーシブルだが、裏表がある
    - そのことを身をもって知った
- USB接続されてるように見えても接続されてるとは限らない
    - 給電ランプやキーが光っているとそう見えてしまう
    - `lsusb`は信頼できる
- 冗長性は手札になる
    - キーボードが使えなくてもsshがある
    - 有線が使えなくてもBluetoothがある
    - キーボードがぶっ壊れても信頼できるキーボードがある
    - キーボードがなくてもスクリーンキーボードがある
- トラブルシューティングは推理小説に似ている
    - かといって推理に白熱しすぎてはいけない
    - 答えはすぐそばにあるかもしれない
- キーボードは簡単には壊れない
    - HDDは壊れる
    - Type-Cケーブルは壊れる

後日談  
改めて使ってみると以前使ってたより明らかに接触判定がシビアなので、落下で部分的に壊れたのが正しいのかもしれない。  
まあBluetoothで最初から繋いでおけばよかったということだ。