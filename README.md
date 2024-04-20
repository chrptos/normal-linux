# normal-linux
ふつうのLinux入門学習用

# 3章
- ファイルシステム(p43)
    - regular file, directory, symbolic link, device file, named pipe, UNIX domain socket
    - device file: character device file, block device file
- ストリーム(p48)
    - システムコールでカーネルによってストリームが作られて、また、システムコールを利用してストリームを操作することで入出力を制御します。
    - ストリームには様々な種類があり、ファイルに繋がったものもあれば、デバイスに繋がったものもあります。プロセスはストリーム経由でファイルやデバイスにアクセスします。
    - また、パイプもストリームの一種です。パイプはファイルやデバイスではなく、プロセス通しをつなげます。
    - 他にも、ネットワークに対応しています。例）端末A-プロセスA-カーネルA-ストリーム（ネットワーク）-カーネルB-プロセスB-端末B

# 4章
- ユーザーベース、シェル
- クレデンシャル(p60)

# 5章
- read: ストリームからバイト列を読み込む
- write: ストリームにバイト列を書き込む
- open: ストリームを作る
- close: ストリームを削除する

- 様々な理由からプロセスからストリームを見せるわけには行かないので、ファイルディスクリプタを通してアクセスします。(p76)
    - カーネル（ストリームa=0） --- (ファイルディスクリプタ0)プロセスのように対応付けします。
    - どのプロセスにも3つのストリームが用意されている
        - 標準入力: 0番
        - 標準出力: 1番
        - 標準エラー: 2番

- コラム：本書におけるストリームとは
    - 本書のストリームとは、ファイルディスクリプタで表現され、read()またはwrite()で操作できるもののことです。例えば、ファイルをopen()するとread()またはwrite()を実行できるものが作られますから、そこにはストリームがあります。

# 6章
- バッファリング： システムコールで1バイトずつ読み込むのは低速なので、バッファリングしてそこから読み込む（p１０２）
    - FILE: 型、ファイルディスクリプタとstdioバッファの内容
    - fgetcやfputcのバイト単位の入出力は何に使用するのですか？
        - 文字単位での処理やバイナリ単位での操作（画像とか）
        - バイナリやテキストを判別する必要がなくファイルの内容をそのままコピーする場合など
        - シリアル通信
        - fgetcとgetcharの違い
            - 汎用性の違いがあります。fgetcはファイルストリームを利用して汎用的な読み込みを可能にします。getcharは標準入力(キーボード)のみに限定されています。
        - ungetc: 読み込みを再開できる
- 入力
    - fgets(): 行単位での読み込みを行います。size-1バイトまでしか読み込みません。
        - size-1バイトまでしか読み込まないのは、読み込んだ文字を完全な文字列として読み込むためです。終端文字(`\0`)を追加します。
        - ただ、ちゃんと1行丸ごと読み込んだのか？バッファがいっぱいで読み込めないのか判別できない問題点があります。
    - gets(): 使わないこと。バッファ指定ができないのでバッファオーバーフローを起こす。
    - scanf: gets()と同様にバッファを指定できないので、オーバーフローを起こす危険性があるので使わないでください。
    - puts()とfputs()の違い？
        - puts()は標準出力に固定されており、末尾に'\n'を追加します。
    - getc(): ストリームから1文字読み込む
- 出力
    - printf(): 標準出力に出したい時に利用されますが、フォーマットが必要ない時にprintf()を使う必然性はないはずです。
    - fputs(): 指定したファイルストリームに文字列を書き込みます。標準出力を明示的に指定しなければいけません。
    - puts(): 文字列を標準出力します。末尾に`\n`が追加されてしまいます。
    - putchar(): 単一文字を標準出力します。
    - 注意点: printf()を利用してもいいが、入力に`%`が含まれる場合は致命的なエラーにつながります。`printf(%s, s)`のように`%s`を利用して出力してください。
- 固定長の入出力
    - read()とfread()の違いは、システムコールであるかどうか？バッファを利用するかという点です。
    - FILE型とファイルディスクリプタとfreadの関係性
        - 関係性
            - FILE 型とファイルディスクリプタの橋渡し：
                - FILE 型のストリームは内部的にはファイルディスクリプタを使用してファイルシステムとのやり取りを行います。FILE 型はファイルディスクリプタをより使いやすく抽象化したものです。
                - fdopen 関数は、既存のファイルディスクリプタを FILE 型のストリームに変換します。これにより、低レベルのファイルディスクリプタインターフェースと高レベルの FILE ストリーム間のギャップを埋めます。
            - fread と FILE 型：
                - fread は FILE 型のストリームを介してデータを読み込みます。このプロセスは自動的にバッファリングが行われ、複数の小さなデータ読み込みが最適化されます。
- open(), fdopen(), fopen()の関係性
    - open()はシステムコールでユーザーグループなどのレベルでパーミッションを設定して、fdを作成できる。fopen()は書き込み、読み取りなどのプログラムレベルでのパーミッションを制御できます。
    - open()だけだとfdは作成できてもストリームが作成されないので、フォーマットやバッファなどがないので扱いにくい。なのでfdopen()でfdからストリームを作成するのが一般的です。
    - fopen()はパーミッション指定はできないが、新規fdを作成しつつ、ストリームも作成してくれるので簡単で扱いやすいです。
- なぜストリームが必要なのか？
    - ストリームは、データの連続的な処理に関して多くの利点を提供します。プログラミングにおいて、ストリームはデータの読み込みや書き込みを抽象化し、一連のデータとして扱うことを可能にします。以下では、ストリームの主な利点を詳しく説明します。
        1. 抽象化と単純化
        ストリームはファイル、ネットワーク接続、メモリ内のデータなど、さまざまなデータソースに対して統一されたインターフェースを提供します。この抽象化により、データの具体的な格納方法や転送方法について詳しく知る必要がなく、データの読み書きに集中できます。
        2. 効率性とバッファリング
        多くのストリーム実装では、内部バッファリングが行われます。このバッファリングは、データの小さな塊が頻繁に読み書きされることを避け、大きなブロックでのデータ操作を可能にします。これにより、システムコールの数が減少し、全体的なパフォーマンスが向上します。
        3. 非同期処理のサポート
        ストリームは非同期操作に対応していることが多く、データの読み込みや書き込みをバックグラウンドで行うことができます。これにより、プログラムがデータを待つことなく他のタスクを進められるため、アプリケーションの応答性が向上します。
        4. エラー処理
        ストリームはエラーを検出して処理するメカニズムを備えていることが多いです。たとえば、読み書き操作中にエラーが発生した場合、ストリームは適切なエラー状態を保持し、プログラムがこれを検出して対応できるようにします。
        5. リソースの管理
        ファイルストリームを利用する場合、オープンしたファイルのリソースはストリームオブジェクトに結びつけられ、ストリームがクローズされると同時にリソースも適切に解放されます。これにより、リソースリークのリスクが減少します。
        6. データ変換と処理の柔軟性
        ストリームは、データをそのまま扱うだけでなく、読み込み時や書き込み時にフィルタリングや変換を行うことができます。例えば、暗号化、圧縮、文字コード変換などの処理をストリームを通じて透過的に行うことができます。
        7. 大規模データの取り扱い
        ストリームは大規模なデータや無限に近いデータストリームを扱う際にも有効です。データを小分けにして処理するため、全てのデータを一度にメモリに格納する必要がありません。
    - ストリームはこれらの利点により、様々なアプリケーションでデータを効率的かつ効果的に扱うための重要なツールとなっています。ファイル入出力、ネットワーク通信、データ処理タスクなど、多岐にわたる分野でその利点が生かされています。

# 7章
- linuxにおける行とは？
    - `\n`で終わる文字列のことを指します。
- fgets()を使用していないのは何故？
    1. getc()ならバッファが必要ないので扱いが楽
    2. fgets()を使うと、相当工夫しない限り行の長さが制限される
    3. getc()でも困らない
- getopt()
    - パラメータをとるオプションに関しては、`:`をパラメータの後ろに記述する

# 8章
- 正規表現
- 文字コード

# 9章
- ファイルシステム
    - /bin: ブートするときの基本コマンド、/usr/binや/binはディストリビューションが管理するので、自分でインストールするコマンドは/usr/local/binに配置するべきです。
    - /sbin: 管理者用のコマンド置き場です。/usr/sbinには平常時用のシステム管理コマンドやサーバープログラムを置きます。
    - /lib: ライブラリを配置します。C言語のライブラリの他、PerlやRuby、Pythonなどの言語のライブラリを置くこともあります。
    - /etc: 各マシンの設定ファイルを置きます。
    - /dev: デバイスファイルを置きます。
    - /proc: プロセスファイルシステムがマウントされます。プロセスファイルシステムはプロセスをファイルシステムで表現します。プロセスIDが1のプロセス情報を見たければ、/proc/1の中を見れば良いです。
    - /usr: 複数のマシンで共有可能なファイルを置きます。ネットワークファイルシステム（NFSやSAMBA）などを用いて別のマシンのファイルシステムをネットワーク越しにマウントします。
        - /usr/local: 各システムの管理者用のディレクトリ
    - /var: 頻繁に書き換えられるファイルを置きます。共有ファイルはここに置くべきではありません。ログやメールボックスが置かれます。
        - /var/log: ログファイル
        - /var/run: 起動中のプロセスIDが保存されます。
    - /sys: デバイスやデバイスドライバの情報
    - /boot: linuxのカーネルプログラム
    - /tmp: /var/tmpはリブートしても消されない、/tmpは消されます。
    - ディレクトリを分ける基準について（p193）
# 10章
    - ディレクトリの内容を読む
        - ディレクトリの正体は構造体（ディレクトリエントリ）
        - ディレクトリトラバース：　ディレクトリを再帰的に走査する。DFSやBFSがある。
    - ハードリンク
        - Linuxでは2つ以上の名前をファイルに付けることができる。→link
            - `echo 'this is file.' > a`
            - `ln a b`でaとbから実態ファイルへアクセス可能になる。
    - シンボリックリンク
        - 名前から名前にリンクする
        - 対応する実態が存在しなくても良い、アクセスするときに実態との対応付けを行う。
            - ショートカットもリンク先がありませんとなるので、それと同じこと
        - ファイルシステムを跨いで別名を付けられる
        - ディレクトリにも別名が付けられる
# 11章
    - 原則として、CPUとメモリが1つずつしかなければ、プロセスは1つしか動かないということ。
        - なので仮想CPUとメモリでプロセス専用のCPUとメモリを作り出している
    - CPUを増やすには？
        - 非常に短い単位で実行プロセスを切り替えれば、あたかもプロセス専用のCPUがあるかのように見えます。
    - メモリを増やすには？
        - プロセスから見えるアドレスをカーネルとCPUによって処理して、実際のアドレスに翻訳するような仕組みを作ります。
        - 論理アドレスと呼ばれます。実際のアドレスは物理アドレスと呼ばれます。
        - プロセスごとにアドレスを用意することで、プロセス同士は隔離されます。
    - 仮想CPUの実現
        - タイムスライス、スケジューラ、ディスパッチャによって実現されています。優先度を設定して担当プロセスを切り替えていきます。
    - 仮想メモリの実現
        - プロセスのアドレス空間をページ単位（4KB or 8KB)に分割して論理アドレスと物理アドレスをページ単位で紐付けします。
        - この時、論理アドレスは必ずしも物理アドレスに紐づいている必要はなく、必要になれば紐付けすれば良いです。
    - 64bitアーキテクチャ
        - 理論上使用可能な論理アドレス空間が、2^64=16エクサバイトであるコンピュータアーキテクチャを指します。
    - 仮想メモリ機構の応用
        - ページングはページ単位に、スワッピングはプロセス全体単位で論理アドレスの退避と再開を行う
        - 逆に物理メモリとファイルを対応付けしたものがメモリマップトファイルと呼ばれている
        - 共有メモリは特定範囲の物理メモリを複数プロセスで共有する。GIMPは画像ファイルを共有することで処理を軽量化している。
    - プログラムができるまで
        - プリプロセス: `#include`などを処理して純粋なC言語のソースコードを出力する
        - コンパイル: C言語からアセンブリ言語に変換する
        - アセンブル: アセンブリ言語を機械語を含むオブジェクトファイルに変換する
        - リンク: スタティックリンクはコンパイル時に含まれていて、ダイナミックリンクはプログラム実行時に必要なライブラリを読み出す。

# 12章
- プロセスに関わるAPI
    - fork: 親から子プロセスを生成する
    - exec: 現在のプロセスを別のプログラムで上書きする
    - wait: 子プロセスの終了を待つ
- プログラムの実行
    1. forkする
    2. 子プロセスで新しいプログラムをexecする
    3. 親プロセスは子プロセスをwaitする
- ゾンビプロセスの基本
    - 子プロセスはexitした後、リソースは利用しないが、プロセステーブルには残ります。プロセステーブルは有限なので回収する必要があります。この状態をゾンビプロセスと呼びます。
    - ゾンビプロセスは、親プロセスでwaitしてあげることで子プロセスからのexitステータスを受け取り、プロセステーブルから削除します。
- ダブルフォークによるデーモン化
    - デーモン化とは？要するにバックグラウンドで動くプログラムのようなものです。
    - 親プロセス→子プロセス→孫プロセスを作成して、子プロセスをexit→親プロセスでwaitしていると、子プロセスから孫プロセスが切り離されてセッションリーダーになることもなく孤立したプロセスができあがります。
    - 孤児プロセスはinitプロセスが新しい親となり、initプロセスは定期的に終了した孤児プロセスを始末します。（字面怖い）
- ファイルディスクリプタ
    - UNIX系のオペレーティングシステムでは、入出力は全てこれらを通して行われる
- パイプ
    - プロセスが生成されると入出力が生成される。
    - forkすると、ファイルディスクリプタもコピーされる。
    - 親の出口を、子の入口につなげたムカデ人間みたいにすることでパイプを実装している
- プロセスのグループ
    - セッション > プロセスグループ > プロセスの単位でまとめられている。
- デーモンプロセス
    - `ps -ef`で全てのプロセスをフルフォーマットで表示できます。
    - ttyが「?」となっているプロセスは端末が割り当てられていない、デーモンプロセスになります。

# 13章
- シグナルに関わるAPI
    - シグナルがプロセスに配送されると、あらかじめ設定されている3つの処理のいずれかが実行されます。
        - 無視する
            - SIGCHLDはデフォルトで無視されるが、プロセスの設定次第では、プロセスで扱うことができる
        - プロセスを終了する
            - SIGINTが代表例です。Ctrl+cのことです。
        - プロセスのコアダンプを作成して異常終了する
            - SIGSEGVなどのセグメンテーションフォールトなどです。
- コアダンプとは？
    - coreはメモリ、dumpはファイルへの書き出しを指します。つまり、プロセスがどのようにメモリを利用したのかをスナップショットしてファイルに書き込む動作を指します。
- 関数ポインタ
    - メモリ空間のテキスト領域に配置された先頭メモリアドレスのこと。

# 14章
- プロセスの環境
    プロセスとそれが関連する「カレントディレクトリ」、「クレデンシャル」、「環境変数」、「使用リソース」の間には密接な関連があります。これらの要素はすべてプロセスの実行コンテキストを形成し、プロセスがシステム上でどのように振る舞うかを定義します。

    ### カレントディレクトリ
    プロセスのカレントディレクトリは、そのプロセスが作業するデフォルトのディレクトリです。ファイルシステムにおけるこのディレクトリは、相対パスによるファイルアクセスの基点となります。つまり、プロセスがファイルを相対パスで指定する場合、そのパスはカレントディレクトリからの相対位置に基づいて解釈されます。プロセスは `chdir()` システムコールを使用してカレントディレクトリを変更することができます。

    ### クレデンシャル
    プロセスのクレデンシャルには、そのプロセスを実行するユーザーID（UID）とグループID（GID）が含まれます。これらのIDは、プロセスがアクセスできるリソースを決定するためにオペレーティングシステムによって使用されます。例えば、ファイルシステムにおいて、プロセスがファイルにアクセスできるかどうかは、そのファイルのパーミッションとプロセスのクレデンシャルに基づいて判断されます。

    ### 環境変数
    環境変数は、プロセスが実行される環境に関する情報を提供します。これにはパス情報、システム設定、ユーザー設定などが含まれ、プロセスが利用するライブラリや実行ファイルの場所を指定するのに使われることが多いです。環境変数は、プロセスが起動する際に親プロセスから継承され、プロセス内で `getenv()` や `setenv()` などのAPIを使用して読み書きされます。

    ### 使用リソース
    プロセスによって使用されるリソースには、CPU時間、メモリ空間、オープンされたファイルディスクリプタ、ネットワーク接続、その他のシステムリソースが含まれます。これらのリソースは、プロセスがどのように実行されるかを決定し、システムの全体的なパフォーマンスに影響を及ぼします。リソースの使用状況は、システムによって監視され、プロセスがリソースの制限を超えた場合には、プロセスを終了させるか、リソースの割り当てを調整することがあります。

    これらの要素はすべて、プロセスがシステム上でどのように機能するかを定義し、プロセスのセキュリティ、アクセス権、性能、および振る舞いを形成します。プロセスのライフサイクル全体を通じて、これらの属性が適切に管理されることが重要です。
- ログイン
    `systemd`、`getty`、および`dup`の関係性を理解するには、Linuxシステムでのログインプロセスとこれらのコンポーネントがどのように連携して機能するかを詳しく見ていく必要があります。各コンポーネントがシステムのログインプロセスにおいて果たす役割を順に説明します。

    ### systemd
    `systemd`はLinuxのinitシステムであり、システム起動時にサービスとプロセスの管理を行います。システムが起動すると、`systemd`はさまざまなターゲット（サービスグループ）に従ってサービスを起動します。これには、ユーザーがログインするための仮想コンソールやグラフィカルユーザーインターフェース（GUI）などが含まれます。`systemd`は`getty`サービスを管理する責任も持ち、特定のTTY（端末）デバイスで`getty`プロセスを起動するように設定されています。

    ### getty
    `getty`は端末ラインを初期化し、ログインプロンプトを表示するプログラムです。`getty`は`systemd`によって指定された仮想コンソールや物理端末上で起動され、ユーザー名とパスワードの入力を待ち受けます。ユーザーがログイン情報を入力すると、`getty`はその情報を使用して`login`プログラムを起動し、さらにユーザー認証を進めます。

    ### dup
    `dup`はファイルディスクリプタを複製するシステムコールです。このコンテキストでは、`getty`がログインセッションを管理する際に重要な役割を果たします。`getty`がユーザーからの入力を受け付けた後、`login`プログラムを実行する前に、標準入力（stdin）、標準出力（stdout）、および標準エラー（stderr）のファイルディスクリプタを適切な端末デバイス（通常は`/dev/tty`）に重ね合わせるために`dup`（または`dup2`）を使用します。これにより、`login`プロセスが端末と対話するための入出力が正しく設定されます。

    ### ログイン処理のフロー
    1. **システム起動**: `systemd`がシステムを起動し、必要なサービスを順に起動します。
    2. **getty起動**: `systemd`が特定のTTYデバイスで`getty`を起動します。
    3. **ユーザー入力待ち**: `getty`がログインプロンプトを表示し、ユーザー名とパスワードの入力を待ちます。
    4. **ユーザー認証**: 入力された認証情報を元に、`getty`は`login`プログラムを起動します。このとき、`dup`を使用して`login`プロセスのstdin, stdout, stderrをTTYに接続します。
    5. **セッション開始**: 認証が成功すると、ユーザーに対してシェルが起動され、ログインセッションが開始されます。

    以上のプロセスにより、Linuxシステムはユーザーが安全にログインできる環境を提供し、各ユーザーのセッションを効果的に管理します。
# 15章
- ストリームであることには変わりないのでread()とwrite()が利用できるはず。であれば、そのストリームを生成するopen()が問題になります。
- ソケット
    - ソケットとは、ストリームを接続できる口部分です。ソケットは様々な種類があり、クライアントやサーバーなど多岐にわたります。
- クライアント側のソケットAPI
    - クライアント ----> サーバー へストリームを接続するには、次の2つのシステムコールが必要です。
    - socket(2): ソケットを作る
    - connect(2): 接続する相手を指定してストリームを繋ぐ


# Ref
- ファイルディスクリプタ: https://furu-se-blog.com/file-descriptor/