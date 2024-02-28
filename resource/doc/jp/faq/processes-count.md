# プロセスをいくつ開始すべきですか

## プロセス数の設定方法
プロセス数は```count```プロパティによって決定されます（Windowsシステムではプロセス数の設定はサポートされていません）。次のコードのようになります
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## 4つのプロセスを起動して外部サービスを提供します ##

$http_worker->count = 4;

...
```

## プロセス数の設定には以下の条件を考慮する必要があります
1. CPUコア数
2. メモリサイズ
3. ビジネスがIO集約型なのかCPU集約型なのか

## プロセス数の設定原則
1. 各プロセスが占有するメモリの合計が総メモリよりも小さくなければなりません（通常、各ビジネスプロセスは約40M程度のメモリを占有します）
2. IO集約型の場合、つまりビジネスに**ブロッキング式**IOが関与する場合（一般的にはMysql、Redisなどのストレージアクセスはブロッキング的です）、プロセス数を大きくしても構いません。例えば、CPUコア数の3倍に設定します。ビジネスが非常に多くのブロッキング待機を含む場合は、プロセス数を適宜増やし、例えばCPUコア数の8倍以上に設定します。なお、**ブロッキング式**IOはCPU集約型ではありません。
3. CPU集約型の場合、つまりビジネスに**ブロッキング式**IOコストがない場合、例えば非同期IOでネットワークリソースを読み取る場合、ビジネスコードがプロセスをブロックしない場合、プロセス数をCPUコア数と同じに設定できます。

## プロセス数の設定参考値
ビジネスコードがIO集約型の場合は、IOの集約度に応じてプロセス数を設定します。たとえば、CPUコア数の3〜8倍です。

ビジネスコードがCPU集約型の場合は、プロセス数をCPUコア数に設定できます。

## 注意
Workerman自体のIOはすべて非ブロッキングです。例えば```Connection->send```などはすべて非ブロッキングであり、CPU集約型の操作に属します。自分のビジネスがどの種類のタイプに偏っているのか分からない場合、プロセス数をCPUコア数の3倍程度に設定してください。
また、プロセス数は多いほど良いとは限りません。プロセス数が多すぎると、プロセス切り替えのコストが増加し、パフォーマンスに一定の影響を与えます。