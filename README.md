# MacBookへのLAPACKのインストールとVsCodeで利用する方法

## 概要
MacOSのPCでC++でLAPACKを使用するための手順を備忘録としてまとめた。

## 参考にしたサイト
[LAPACK/BLAS 設定メモ(Mac)](https://qiita.com/nek0log/items/5733b8b886b9ad93ae11)

[VScodeのinclude Pathの設定をちゃんとする](https://qiita.com/sage-git/items/ffe463c0de05344d721b)

## LAPACKのダウンロード
まずは、以下のサイトからLAPACKの圧縮フォルダをダウンロードする。

[LAPACK -Linear Algebra PACKage](http://www.netlib.org/lapack/)

versionがいくつかあるが、新しい方が色々と改善が施されて性能面で古いversionのものより良い。ただし、ネットに上がってる記事や本で使用されているversionのものがダウンロードしたものより古い場合、関数の使用方法が若干異なる場合があるので注意が必要である。

ここでは、圧縮フォルダを`/Users/UserName/Documents/`に保存した。保存したら、このフォルダを解凍する。

## ターミナル上での操作
1.  フォルダを解凍したら次はターミナルを開き、先ほどフォルダを解凍したディレクトリへ移動する。例えば、筆者の場合は
    ```
    cd Document/lapack-3.10.0
    ```
    と入力した。


2. `make.inc.example`の内容を`make.inc`へコピーする。つまり、
    ```
    cp make.inc.example make.inc
    ```
    を実行する。次に、以下のようにmakeを行い4つの静的ファイルを生成する。
    ```
    make blaslib
    make lapacklib
    make cblaslib
    make lapackelib
    ```
    これにより、`librefblas.a`、 `liblapack.a`、 `libcblas.a`、 `liblapacke.a`が生成される。
3. 4つの静的ファイルを`/usr/local/lib/`へコピーする。
    ```
    sudo cp ./librefblas.a /usr/local/lib/libblas.a
    sudo cp ./liblapack.a /usr/local/lib/liblapack.a
    sudo cp ./libcblas.a /usr/local/lib/libcblas.a
    sudo cp ./liblapacke.a /usr/local/lib/liblapacke.a
    ```

4. includeファイルを`/usr/local/include`へコピーする。
    ```
    sudo cp ./CBLAS/include/cblas*.h /usr/local/include/
    sudo cp ./LAPACKE/include/lapack*.h /usr/local/include/
    ```
    参考にしたサイトでは2行目が`lapacke*.h`をコピーしろ、というふうに
書いてあるが、これでは`lapack.h`がコピーされないので該当箇所を`lapack*.h`へ変更して実行する。

以上で、ターミナル上での操作は終わりである。

## VsCode上での操作
VsCode側で、例えばcblasの関数を使用する以下のコードを実行しようとしたとする。
```
#include <cstdio>
#include <stdlib.h>
#include <cmath>
#include "cblas.h"
#include "lapacke.h"
#include <iostream>

using namespace std;

int main ()
{
   double a[5] = {0.,0.,0.,0.,0.};
   double b[5] = {1.,1.,1.,1.,1.};
   cout << a[1] << b[1] << endl;
   cblas_dcopy(5,a,1,b,1);
   cout << a[1] << b[1] << endl;
   return 0;
}
```
この時、おそらく何もいじらなければエディタ上に次のエラーが出ると思われる。
```
...
#include "cblas.h" ソースファイルを開けません
#include "lapacke.h"　ソースファイルを開けません
...
```
これは、どうやらヘッダーが置いてあるディレクトリへのpathが通っていないために起こるエラーのようである。そこで、`c_cpp_properties.json`というファイルを開く。すると筆者の場合は以下の内容が書かれている。
```
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "macFrameworkPath": [],
            "compilerPath": "/opt/homebrew/bin/gcc-11",
            "cStandard": "gnu17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "macos-gcc-arm64",
            "browse": {
                "limitSymbolsToIncludedHeaders": false
            }
        }
    ],
    "version": 4
}
```
そして、先程のエラーを解決するためには上記の`includePath`に先程のヘッダーファイルへのPathを通す必要がある。筆者の場合は`/usr/local/include/`を加えればよく、最終的にjsonファイルの中身は以下の通りになった。
```
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**"、
                "/usr/local/include/"
            ],
            "defines": [],
            "macFrameworkPath": [],
            "compilerPath": "/opt/homebrew/bin/gcc-11",
            "cStandard": "gnu17",
            "cppStandard": "gnu++17",
            "intelliSenseMode": "macos-gcc-arm64",
            "browse": {
                "limitSymbolsToIncludedHeaders": false
            }
        }
    ],
    "version": 4
}
```
これで、あとはターミナル上で
```
g++ helloworld.cpp -llapacke -lcblas -llapack -lblas
./a.out
```
とすれば良い。