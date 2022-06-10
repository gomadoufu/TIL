Haskell 環境構築


1. https://www.haskell.org/ghcup/ にアクセスして、ghcupをインストール。
   全てY or P
2. exec  $SHELL -l
3. ghcup tui で確認
4. which stack , which ghc でパスを確認
5. stack new sample && cd sample
6. code .        (https://zenn.dev/ryuu/scraps/4061aaee059d89)
7. Haskell Language Server Extention をインストール
8. Lib.hs を開くと、**How do you want the extension to manage/discover HLS and the relevant toolchain?**と聞かれるので、Yesを選択
9. stack build
10. stack exec myproj-exe

Tips 

もし、stack buildした時の実行ファイルの名前を変えたい時は、package.yamlの executables: 下 myproj-exe を、何かかっこいい名前に変えると変わる

ちょっとhlintやormoluを使いたいが、stack newは大仰すぎる、という時
シンプルに作る
1. mkdir simple
2. cd simple
3. stack init
4. touch hie.yaml
    cradle:
      stack:



 
