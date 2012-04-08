Title: My Document
Author: 脇田建
Affiliation: 東京工業大学
Base Header Level: 2
Copyright: Copyright by Ken Wakita, 2012
CSS: file:///Users/wakita/lib/scripts/mmv/markdown.css
Date: 04/08/12 10:54:28

----

# 考察

巨大の Wikipedia のデータを不用意にアクセスするととんでもなく非効率となる．効率の低減を避けるためには，徹底的な Zero copy の方針をとることが重要と思われる．

## データの入力について

データを圧縮形式から伸長してしまえば，それに対して mmap することで効率的にアクセスすることができるが，ディスクの利用効率が悪．できれば，gunzip したストリームに対して処理を行いたい．しかし，ストリームに対して正規表現検索をすることはできない．おそらく，Python の正規表現機能が文字列を想定した C のライブラリを用いて実装されているからだろう．

一方，Wikipedia で利用している SQL ファイルは約1MBのバッファから出力されたらしい．バッファ長さについては，要確認のこと．こちらでもバッファを用意した I/O をすれば効率的にアクセスできるようになる．念のため伸長可能なバッファを用意する．

Zero copy を実現するために，入力の内容の部分文字列を作ることは避けなくてはならない．re.split, re.groups, s[p1:p2] などは部分文字列を作ってしまうので避けること．

## データの出力について

一般的な出力方法では，出力対象物がメモリ中に確保されてしまうため，Zero copy が破綻する．そこで，出力ファイルに直接，書き込むかわりに，それを mmap して，その領域にコピーすればよい．もしかして，これには slice 書式使えばいい？

# 情報

## mmap に対する memoryview について

mmap と memoryview は Python 2.6 でもサポートされているらしいが，Python 2.7 でも mmap object は buffer protocol を実装していないために，mmap object の memoryview を作成することはできなかった．mmap と memoryview を併用する方法は Python 3.2.2 で動作を確認した．

    #!/usr/bin/env python3
    
    import sys, mmap, os
    
    path = 'enwiki-20120307-page.sql'
    p = open(path, 'r')
    m = mmap.mmap(p.fileno(), os.path.getsize(path), access=mmap.ACCESS_READ)
    mm = memoryview(m)


