# ShallowWater
以下のソースは、Houdiniのvexで二次元の浅水方程式を計算するものです。
このソースは地形が平たんであることを前提にしていますが、これを地形の高さを考慮したものに書き換えてください。

// シミュレーションパラメータ
float g = 9.81;  // 重力加速度
float dt = 0.01; // 時間ステップ
float dx = 0.1;  // x方向の空間ステップ
float dy = 0.1;  // y方向の空間ステップ
float nu = 0.001; // 数値的粘性（拡散）係数

int rows = chi("rows"); // グリッドの行数
int cols = chi("cols"); // グリッドの列数

// 現在のポイントの水の高さと流速
float h = @h;
float u = @u;
float v = @v;

// 境界条件の確認
int i = @ptnum % cols;
int j = @ptnum / cols;
if (i == 0 || j == 0 || i == cols - 1 || j == rows - 1) {
    // 境界での処理
    h = 0.0;
    u = 0.0;
    v = 0.0;
}
else {
    // 隣接ポイントの水の高さと流速
    float h_right = point(0, "h", @ptnum + 1);
    float u_right = point(0, "u", @ptnum + 1);
    float h_left = point(0, "h", @ptnum - 1);
    float u_left = point(0, "u", @ptnum - 1);

    float h_up = point(0, "h", @ptnum + cols);
    float v_up = point(0, "v", @ptnum + cols);
    float h_down = point(0, "h", @ptnum - cols);
    float v_down = point(0, "v", @ptnum - cols);

    // 二次元浅水方程式の計算
    float h_new = h - dt * ( (u_right * h_right - u_left * h_left) / (2 * dx) +
                            (v_up * h_up - v_down * h_down) / (2 * dy));

    float u_new = u - dt * (u * (u_right - u_left) / (2 * dx) + g * (h_right - h_left) / (2 * dx));
    float v_new = v - dt * (v * (v_up - v_down) / (2 * dy) + g * (h_up - h_down) / (2 * dy));

    // 数値的拡散の追加
    h_new += nu * ( (h_right - 2 * h + h_left) / (dx * dx) + (h_up - 2 * h + h_down) / (dy * dy));
    u_new += nu * ( (u_right - 2 * u + u_left) / (dx * dx) + (u - 2 * u + u) / (dy * dy));
    v_new += nu * ( (v - 2 * v + v) / (dx * dx) + (v_up - 2 * v + v_down) / (dy * dy));

    // 負の高さを防ぐ
    if(h_new < 0) h_new = 0;

    // 更新
    h = h_new;
    u = u_new;
    v = v_new;
}

// ポイント属性の更新
setpointattrib(0, "h", @ptnum, h, "set");
setpointattrib(0, "u", @ptnum, u, "set");
setpointattrib(0, "v", @ptnum, v, "set");

// 水の高さでY座標を更新
@P.y = @h;

ChatGPT
地形の高さを考慮に入れた浅水方程式への書き換えには、地形の高さに基づく水の流れの変化を計算に組み込む必要があります。地形の高さデータは通常、別の属性（例えば @height など）として格納されます。このデータを使用して、水の流れに影響を与える地形の起伏を計算に組み込みます。

以下は、地形の高さを考慮に入れた二次元浅水方程式の更新されたVEXコードです。

c
Copy code
// シミュレーションパラメータ
float g = 9.81;  // 重力加速度
float dt = 0.01; // 時間ステップ
float dx = 0.1;  // x方向の空間ステップ
float dy = 0.1;  // y方向の空間ステップ
float nu = 0.001; // 数値的粘性（拡散）係数

int rows = chi("rows"); // グリッドの行数
int cols = chi("cols"); // グリッドの列数

// 現在のポイントの水の高さ、流速、地形の高さ
float h = @h;
float u = @u;
float v = @v;
float terrain_height = @height; // 地形の高さ

// 境界条件の確認
int i = @ptnum % cols;
int j = @ptnum / cols;
if (i == 0 || j == 0 || i == cols - 1 || j == rows - 1) {
    // 境界での処理
    h = 0.0;
    u = 0.0;
    v = 0.0;
}
else {
    // 隣接ポイントの水の高さ、流速、地形の高さ
    float h_right = point(0, "h", @ptnum + 1);
    float u_right = point(0, "u", @ptnum + 1);
    float terrain_right = point(0, "height", @ptnum + 1);

    float h_left = point(0, "h", @ptnum - 1);
    float u_left = point(0, "u", @ptnum - 1);
    float terrain_left = point(0, "height", @ptnum - 1);

    float h_up = point(0, "h", @ptnum + cols);
    float v_up = point(0, "v", @ptnum + cols);
    float terrain_up = point(0, "height", @ptnum + cols);

    float h_down = point(0, "h", @ptnum - cols);
    float v_down = point(0, "v", @ptnum - cols);
    float terrain_down = point(0, "height", @ptnum - cols);

    // 地形による水の高さの調整
    float adjusted_h = h + terrain_height;
    float adjusted_h_right = h_right + terrain_right;
    float adjusted_h_left = h_left + terrain_left;
    float adjusted_h_up = h_up + terrain_up;
    float adjusted_h_down = h_down + terrain_down;

    // 二次元浅水方程式の計算（地形を考慮）
    float h_new = h - dt * ( (u_right * adjusted_h_right - u_left * adjusted_h_left) / (2 * dx) +
                            (v_up * adjusted_h_up - v_down * adjusted_h_down) / (2 * dy));

    float u_new = u - dt * (u * (u_right - u_left) / (2 * dx) + g * (adjusted_h_right - adjusted_h_left) / (2 * dx));
    float v_new = v - dt * (v * (v_up - v_down) / (2 * dy) + g * (adjusted_h_up - adjusted_h_down) / (2 * dy));

    // 数値的拡散の追加
    h_new += nu * ( (h_right - 2 * h + h_left) / (dx * dx) + (h_up - 2 * h + h_down) / (dy * dy));
    u_new += nu * ( (u_right - 2 * u + u_left) / (dx * dx) + (u - 2 * u + u) / (dy * dy));
    v_new += nu * ( (v - 2 * v + v) / (dx * dx) + (v_up - 2 * v + v_down) / (dy * dy));

    // 負の高さを防ぐ
    if(h_new < terrain_height) h_new = terrain_height;

    // 更新
    h = h_new - terrain_height;
    u = u_new;
    v = v_new;
}

// ポイント属性の更新
setpointattrib(0, "h", @ptnum, h, "set");
setpointattrib(0, "u", @ptnum, u, "set");
setpointattrib(0, "v", @ptnum, v, "set");

// 水の高さと地形の高さでY座標を更新
@P.y = h + terrain_height;
このコードでは、各ポイントの地形の高さを取得し、水の高さの計算にその高さを加えて、地形の起伏を考慮します。また、最終的な水の高さは地形の高さを差し引いて計算されます。これにより、地形の影響を反映したより現実的な水の動きがシミュレートされます。

User
このソースだと、hが0のところにも水があるような挙動になって、すべてh=0にして地形の起伏だけある状態なのに存在しない水が発生するような挙動になっていました。修正をお願いします。
ChatGPT
ご指摘の通り、提供したコードでは、地形の高さがあるにもかかわらず水がない（h = 0）場所においても水が存在しているような挙動を示す可能性があります。この問題を解決するために、地形の高さと実際の水の高さ（h）の関係をより適切に扱う必要があります。

具体的には、地形の高さ以上の水の高さ（h）のみを流動させるように計算を修正します。以下は修正されたVEXコードです。

c
Copy code
// シミュレーションパラメータ
float g = 9.81;  // 重力加速度
float dt = 0.01; // 時間ステップ
float dx = 0.1;  // x方向の空間ステップ
float dy = 0.1;  // y方向の空間ステップ
float nu = 0.001; // 数値的粘性（拡散）係数

int rows = chi("rows"); // グリッドの行数
int cols = chi("cols"); // グリッドの列数

// 現在のポイントの水の高さ、流速、地形の高さ
float h = @h;
float u = @u;
float v = @v;
float terrain_height = @height; // 地形の高さ

// 境界条件の確認
int i = @ptnum % cols;
int j = @ptnum / cols;
if (i == 0 || j == 0 || i == cols - 1 || j == rows - 1) {
    // 境界での処理
    h = 0.0;
    u = 0.0;
    v = 0.0;
}
else {
    // 隣接ポイントの水の高さ、流速、地形の高さ
    // ...（前のコードの隣接ポイントのデータ取得部分は変更なし）

    // 地形による水の高さの調整
    // ...（前のコードの地形による水の高さの調整部分は変更なし）

    // 二次元浅水方程式の計算（地形を考慮）
    // ...（前のコードの二次元浅水方程式の計算部分は変更なし）

    // 数値的拡散の追加
    // ...（前のコードの数値的拡散の追加部分は変更なし）

    // 負の高さを防ぐ
    if(h_new < terrain_height) h_new = terrain_height;

    // 水が存在しない場合（地形の高さと水の高さが等しい場合）の処理
    if(h_new == terrain_height) {
        u_new = 0.0;
        v_new = 0.0;
    }

    // 更新
    h = h_new - terrain_height;
    u = u_new;
    v = v_new;
}

// ポイント属性の更新
// ...（前のコードのポイント属性の更新部分は変更なし）

// 水の高さと地形の高さでY座標を更新
@P.y = h + terrain_height;
この修正により、水が存在しない（h = 0）場所では流速（u, v）が0に設定され、水の動きが発生しないようになります。これにより、地形の起伏があっても水が存在しない場合には、水の動きがシミュレートされないようになります。

User
教えていただいたソースを、下記のように h_new の計算の最初のパラメータを h から adjusted_h  に変えるとそれっぽく動くようになりました。

ただ、かなりそれっぽい動きをするようになったのですが、地形の起伏を大きくすると簡単に発散してしまいます。
大きな波が発生した時に、波がどんどん増幅して壊れるような挙動になっていました。

この安定性を上げるには、どのように修正すればいいでしょうか？


// シミュレーションパラメータ
float g = 9.81;  // 重力加速度
float dt = 0.01; // 時間ステップ
float dx = 0.1;  // x方向の空間ステップ
float dy = 0.1;  // y方向の空間ステップ
float nu = 0.001; // 数値的粘性（拡散）係数

int rows = chi("rows"); // グリッドの行数
int cols = chi("cols"); // グリッドの列数

// 現在のポイントの水の高さ、流速、地形の高さ
float h = @h;
float u = @u;
float v = @v;
float terrain_height = @height; // 地形の高さ

// 境界条件の確認
int i = @ptnum % cols;
int j = @ptnum / cols;
if (i == 0 || j == 0 || i == cols - 1 || j == rows - 1) {
    // 境界での処理
    h = 0.0;
    u = 0.0;
    v = 0.0;
}
else {
    // 隣接ポイントの水の高さ、流速、地形の高さ
    float h_right = point(0, "h", @ptnum + 1);
    float u_right = point(0, "u", @ptnum + 1);
    float terrain_right = point(0, "height", @ptnum + 1);

    float h_left = point(0, "h", @ptnum - 1);
    float u_left = point(0, "u", @ptnum - 1);
    float terrain_left = point(0, "height", @ptnum - 1);

    float h_up = point(0, "h", @ptnum + cols);
    float v_up = point(0, "v", @ptnum + cols);
    float terrain_up = point(0, "height", @ptnum + cols);

    float h_down = point(0, "h", @ptnum - cols);
    float v_down = point(0, "v", @ptnum - cols);
    float terrain_down = point(0, "height", @ptnum - cols);

    // 地形による水の高さの調整
    float adjusted_h = h + terrain_height;
    float adjusted_h_right = h_right + terrain_right;
    float adjusted_h_left = h_left + terrain_left;
    float adjusted_h_up = h_up + terrain_up;
    float adjusted_h_down = h_down + terrain_down;

    // 二次元浅水方程式の計算（地形を考慮）
    float h_new = adjusted_h - dt * ( (u_right * adjusted_h_right - u_left * adjusted_h_left) / (2 * dx) +
                            (v_up * adjusted_h_up - v_down * adjusted_h_down) / (2 * dy));
    
    float u_new = u - dt * (u * (u_right - u_left) / (2 * dx) + g * (adjusted_h_right - adjusted_h_left) / (2 * dx));
    float v_new = v - dt * (v * (v_up - v_down) / (2 * dy) + g * (adjusted_h_up - adjusted_h_down) / (2 * dy));

    // 数値的拡散の追加
    h_new += nu * ( (h_right - 2 * h + h_left) / (dx * dx) + (h_up - 2 * h + h_down) / (dy * dy));
    u_new += nu * ( (u_right - 2 * u + u_left) / (dx * dx) + (u - 2 * u + u) / (dy * dy));
    v_new += nu * ( (v - 2 * v + v) / (dx * dx) + (v_up - 2 * v + v_down) / (dy * dy));

    // 負の高さを防ぐ
    if(h_new < terrain_height) h_new = terrain_height;

    // 水が存在しない場合（地形の高さと水の高さが等しい場合）の処理
    if( abs(h_new - terrain_height)<0.001 ) {
        u_new = 0.0;
        v_new = 0.0;
    }

    // 更新
    h = h_new - terrain_height;
    u = u_new;
    v = v_new;
}

// ポイント属性の更新
setpointattrib(0, "h", @ptnum, h, "set");
setpointattrib(0, "u", @ptnum, u, "set");
setpointattrib(0, "v", @ptnum, v, "set");

// 水の高さと地形の高さでY座標を更新
@P.y = h + terrain_height;

// 水の高さがある場所を青くする
if( h > 0.001 ){
    @Cd.r = 0; @Cd.g = 0.5; @Cd.b = 1; 
}else{
    @Cd.r = 1; @Cd.g = 1; @Cd.b = 1; 
}
ChatGPT
数値シミュレーション、特に浅水方程式のような流体シミュレーションにおいて、安定性を上げるためにはいくつかの方法があります。大きな波や急な地形の変化がある場合、シミュレーションが不安定になりやすいので、以下のような手法を試してみると良いでしょう。

時間ステップの調整: dt（時間ステップ）を小さくすることで、シミュレーションの安定性を高めることができます。これは、流体の動きをより細かい時間単位で計算することになり、急激な変化に対する反応を滑らかにします。

数値的粘性の増加: nu（数値的粘性）を増やすことで、流体の動きにより多くの抵抗を加え、波の増幅を抑えることができます。これは、流体の動きをより現実的にするだけでなく、不安定な振る舞いを減少させます。

空間ステップの調整: dx および dy（空間ステップ）を調整することで、グリッドの細かさを変え、シミュレーションの安定性を向上させることができます。ただし、これは計算コストを増加させるため、適切なバランスを見つける必要があります。

波の高さの制限: 波の高さが一定の閾値を超えた場合に、それ以上の高さの増加を制限することで、シミュレーションの発散を防ぐことができます。

高度な数値解法の導入: より高度な数値解法（たとえばRunge-Kutta法など）を導入することで、シミュレーションの精度を向上させ、安定性を確保することができます。

これらの調整は、シミュレーションの特定のニーズや性質に応じて選択および組み合わせることができます。安定性を確保するためには、これらのパラメータを慎重に調整し、異なる条件下でのシミュレーションの挙動を観察することが重要です。


4 / 4

User
ありがとうございます！時間ステップの調整で安定度が大幅に上がりました。
ただ、今度は地形に起伏を足して水がたまるようにすると、たまった水が増幅しているような挙動になっているように見えました。
このような挙動を修正するにはどうすればいいでしょうか？
以下が最新のソースです

// シミュレーションパラメータ
float g = 9.81;  // 重力加速度
float dt = 0.005; // 時間ステップ
float dx = 0.1;  // x方向の空間ステップ
float dy = 0.1;  // y方向の空間ステップ
float nu = 0.001; // 数値的粘性（拡散）係数

int rows = chi("rows"); // グリッドの行数
int cols = chi("cols"); // グリッドの列数

// 現在のポイントの水の高さ、流速、地形の高さ
float h = @h;
float u = @u;
float v = @v;
float terrain_height = @height; // 地形の高さ

// 境界条件の確認
int i = @ptnum % cols;
int j = @ptnum / cols;
if (i == 0 || j == 0 || i == cols - 1 || j == rows - 1) {
    // 境界での処理
    h = 0.0;
    u = 0.0;
    v = 0.0;
}
else {
    // 隣接ポイントの水の高さ、流速、地形の高さ
    float h_right = point(0, "h", @ptnum + 1);
    float u_right = point(0, "u", @ptnum + 1);
    float terrain_right = point(0, "height", @ptnum + 1);

    float h_left = point(0, "h", @ptnum - 1);
    float u_left = point(0, "u", @ptnum - 1);
    float terrain_left = point(0, "height", @ptnum - 1);

    float h_up = point(0, "h", @ptnum + cols);
    float v_up = point(0, "v", @ptnum + cols);
    float terrain_up = point(0, "height", @ptnum + cols);

    float h_down = point(0, "h", @ptnum - cols);
    float v_down = point(0, "v", @ptnum - cols);
    float terrain_down = point(0, "height", @ptnum - cols);

    // 地形による水の高さの調整
    float adjusted_h = h + terrain_height;
    float adjusted_h_right = h_right + terrain_right;
    float adjusted_h_left = h_left + terrain_left;
    float adjusted_h_up = h_up + terrain_up;
    float adjusted_h_down = h_down + terrain_down;

    // 二次元浅水方程式の計算（地形を考慮）
    float h_new = adjusted_h - dt * ( (u_right * adjusted_h_right - u_left * adjusted_h_left) / (2 * dx) +
                            (v_up * adjusted_h_up - v_down * adjusted_h_down) / (2 * dy));
    
    float u_new = u - dt * (u * (u_right - u_left) / (2 * dx) + g * (adjusted_h_right - adjusted_h_left) / (2 * dx));
    float v_new = v - dt * (v * (v_up - v_down) / (2 * dy) + g * (adjusted_h_up - adjusted_h_down) / (2 * dy));

    // 数値的拡散の追加
    h_new += nu * ( (h_right - 2 * h + h_left) / (dx * dx) + (h_up - 2 * h + h_down) / (dy * dy));
    u_new += nu * ( (u_right - 2 * u + u_left) / (dx * dx) + (u - 2 * u + u) / (dy * dy));
    v_new += nu * ( (v - 2 * v + v) / (dx * dx) + (v_up - 2 * v + v_down) / (dy * dy));

    // 負の高さを防ぐ
    if(h_new < terrain_height) h_new = terrain_height;

    // 水が存在しない場合（地形の高さと水の高さが等しい場合）の処理
    if( abs(h_new - terrain_height)<0.001 ) {
        u_new = 0.0;
        v_new = 0.0;
    }

    // 更新
    h = h_new - terrain_height;
    u = u_new;
    v = v_new;
}

// ポイント属性の更新
setpointattrib(0, "h", @ptnum, h, "set");
setpointattrib(0, "u", @ptnum, u, "set");
setpointattrib(0, "v", @ptnum, v, "set");

// 水の高さと地形の高さでY座標を更新
@P.y = h + terrain_height;

// 水の高さがある場所を青くする
if( h > 0.01 ){
    @Cd.r = 0; @Cd.g = 0.5; @Cd.b = 1; 
}else{
    @Cd.r = 1; @Cd.g = 1; @Cd.b = 1; 
}
ChatGPT
水がたまるような地形で水の高さが増幅してしまう問題は、シミュレーションの物理モデルにおける一般的な課題です。以下に、この問題を解決するためのいくつかの方法を示します。

流出のモデリング: 地形において水がたまる場所で、水が自然に流出するようなメカニズムを追加することが有効です。これにより、水が無限に増幅されることを防ぎます。

深さ依存の粘性の導入: 水の深さに応じて粘性を増加させることで、深い水域での水の動きを遅くし、増幅を抑制することができます。

非線形抵抗の追加: 流速が一定の閾値を超えた場合、非線形に抵抗を増加させることで、急激な流れの増幅を防ぐことができます。

波の高さの制限: 特定の高さ以上の波が発生しないように、波の高さを制限することも一つの方法です。これにより、水が過剰に増幅されるのを防ぎます。

地形との相互作用の強化: 地形と水の相互作用をより詳細にモデル化することで、地形による自然な流れのパターンを再現し、水の増幅を抑えることができます。

これらの方法を実装する際には、シミュレーションの特性や目的に合わせて、適切なバランスを見つけることが重要です。また、これらの方法を実装するには、コードに追加の計算式や条件分岐を組み込む必要があり、計算コストの増加に注意が必要です。

User
水の高さの増幅を抑えたサンプルソースをお願いします。
ChatGPT
水の高さの増幅を抑えるためには、主に流出モデリングと深さ依存の粘性を導入するアプローチを取ることが効果的です。以下に示すサンプルコードでは、これらの要素を組み込みます。

流出モデリング: 特定の高さを超えた水は、自然に流出するようにします。これにより、水が一定以上たまることを防ぎます。

深さ依存の粘性: 水の深さに応じて粘性を増加させます。深い水域では水の動きが遅くなり、表面張力や内部摩擦が強くなります。

以下は、これらの変更を加えたサンプルコードです。

c
Copy code
// シミュレーションパラメータ
float g = 9.81;    // 重力加速度
float dt = 0.005;  // 時間ステップ
float dx = 0.1;    // x方向の空間ステップ
float dy = 0.1;    // y方向の空間ステップ
float nu_base = 0.001; // 基本的な粘性係数
float overflow_threshold = 1.0; // 水の高さの流出閾値

int rows = chi("rows"); // グリッドの行数
int cols = chi("cols"); // グリッドの列数

// 現在のポイントの水の高さ、流速、地形の高さ
float h = @h;
float u = @u;
float v = @v;
float terrain_height = @height; // 地形の高さ

// ...（境界条件の確認および隣接ポイントのデータ取得は変更なし）

// 地形による水の高さの調整
// ...（地形による水の高さの調整は変更なし）

// 二次元浅水方程式の計算（地形を考慮）
// ...（二次元浅水方程式の計算は変更なし）

// 水の高さに応じた粘性の調整
float nu = nu_base * (1.0 + h); // 水の深さに応じて粘性を増加

// 数値的拡散の追加（粘性を考慮）
h_new += nu * ( (h_right - 2 * h + h_left) / (dx * dx) + (h_up - 2 * h + h_down) / (dy * dy));
u_new += nu * ( (u_right - 2 * u + u_left) / (dx * dx) + (u - 2 * u + u) / (dy * dy));
v_new += nu * ( (v - 2 * v + v) / (dx * dx) + (v_up - 2 * v + v_down) / (dy * dy));

// 負の高さを防ぐ
if(h_new < terrain_height) h_new = terrain_height;

// 流出処理：一定の高さを超えた水は流出する
if(h_new - terrain_height > overflow_threshold) {
    h_new = terrain_height + overflow_threshold;
}

// ...（残りの更新処理およびポイント属性の更新は変更なし）

// 水の高さと地形の高さでY座標を更新
@P.y = h + terrain_height;

// 水の高さがある場所を青くする
// ...（水の高さに応じた色の変更は変更なし）
このコードは、水の高さが一定の閾値（overflow_threshold）を超えた場合、その水が流出するように処理しています。また、水の深さに応じて粘性（nu）が増加するように設定しており、深い水域での動きを遅くしています。これらの変更により、水の高さの増幅を抑えることが期待できます。
