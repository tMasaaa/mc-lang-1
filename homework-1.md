# Homework-1
- vagrantで`ubuntu18.04`を入れた
```
$ wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
$ sudo apt-add-repository "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-8 main"
$ sudo apt install clang-8 lldb-8 lld-8
```
- treeを入れた
- Makefileを修正した
```
CXX = clang++-8
CXXFLAGS = `llvm-config-8 --cxxflags --ldflags --system-libs --libs all`
```
- これでmakeが通る。
- 追加で、参加者のgistより(UEFIの方)
```
この後にgccのシンボリックリンクを作らないとgccのリンク先がgcc-7になっていたので変更しました。

sudo ln -s /usr/bin/gcc-5 /usr/bin/gcc
```
これ参考にすると、llvm-8とかをllvmにシンボリックリンク貼ればよいのかなあ(既存のllvmと競合しそうなのでやめておいた)

# hw 1-3
lexer.h
```
if(std::isdigit(lastChar)) {
    std::string str = "";
    str += lastChar;
    while(std::isdigit(lastChar = getNextChar(iFile))) {
        str += lastChar;
    }
    // std::cout<< str<<std::endl;
    setnumVal(std::stod(str));
    return tok_number;
}
```
- strtodと書かれていたが、string to intならstodがよい。と思ってreference見たら呼び出してるのは結局strtodなんですね。どちらでもよさそう。(https://cpprefjp.github.io/reference/string/stod.html)
- 言われたとおり実装すればよい。setNumValとか、tok_numberが-5を返すとか、つまづくことは多かった(けど1ファイル内にかかれていて助かった)
- includeはmc.cppで行っているので、普通にstdが使える。分割ファイルの仕組みを知らなかったが、includeは`.cpp`で行うものらしい。

# hw 1-4
lexer.h
```
if (lastChar == '#') { // `#` is 35
    lastChar = getNextChar(iFile);
    while(lastChar != 0 && lastChar != '\n') {
        // getNextChar is return 0 when no char
        // `\n`(LF) is 10
        // EOF is -1 ?  
        lastChar = getNextChar(iFile);
    }
    return gettok();
}
```
- intでやっているところでつまづいた
- あとEOFだとアウトだった。getNextCharが0を返すので、0が来ると読み込んでないということになるので、0がきたらEOFと判断してよい。

# hw 1-5
- `(`を呼び出すのはparserではなくtokenizerの方でやっている
```
getNextToken();
auto Result = ParseExpression();
if(CurTok != ')')
    return LogError("no parenthesis end found");
getNextToken();
return Result;
```
- まじで言われた通りやるとできた。内容も理解。
- CurTokがグローバルで宣言されているのがポイント。

# hw 1-6
- LHSと現在のBinOpが渡される -> そこからRHSを生成して、LHS、BinOp、RHSのセットを返す。
- Exprに進む->getNextToken
```
int tokprec = GetTokPrecedence();
if(tokprec < CallerPrec) // e.g. 20 < 40
    return LHS;
int BinOp = CurTok;
getNextToken();
auto RHS = ParsePrimary();
int NextPrec = GetTokPrecedence();
if(tokprec < NextPrec) {
    RHS = ParseBinOpRHS(tokprec + 1, std::move(RHS));
    if(!RHS)
        return nullptr;
}
LHS = llvm::make_unique<BinaryAST>(BinOp, std::move(LHS), std::move(RHS));
```
- ` RHS = ParseBinOpRHS(tokprec + 1, std::move(RHS));`はRHSをLHSとして再帰呼び出ししている。moveとかよく分からなかった。(moveにするとメモリ消費が抑えられるとか？うーん)

# hw 1-7
- https://anoopsarkar.github.io/compilers-class/llvm-practice.html ここに演算子ごとのIRが載っている
- `const Twine & 	Name = "",`だから名前を`subtmp`とかにしているっぽい
- Twine - A lightweight data structure for efficiently representing the concatenation of temporary values as strings. More...
- https://llvm.org/doxygen/classllvm_1_1Twine.html
```
case '-':
  return Builder.CreateSub(L, R, "subtmp");
case '*':
  return Builder.CreateMul(L, R, "multmp");
```
- これは割と指示通りにするだけなのでつまづかなかった。