# CANパケットのデータ部の構成

- エンディアン表記はビッグエンディアン
- CANパケットのモデルはBMWを指定

## 車両表示速度(mph, miles per hour)

`id=0x1B4, len=5` で固定

- バイト列としては左から 2 バイトのみ利用される
- 速度範囲は0.0~765.0 mphまで表せる


##### mph への変換方法

`mph(1時間に1マイルの速さ) = (((左から2バイト目 - 0xD0) * 0xFF) + 左から1バイト目) / 0x10`

備考: 1 マイル ≒ 1.60934 キロメートル

#### 例
##### 0 mph

```
0x00, 0xD0, 0x00, 0x00, 0x00
(((0xD0 - 0xD0) * 0xFF) + 0x00) / 0x10 = 0(mph)
```

##### 259 mph

```
0x64, 0xE0, 0x00, 0x00, 0x00
(((0xE0 - 0xD0) * 0xFF) + 0x40) / 0x10 = 259(mph)
```

##### 正常なパケットの条件

- 3 バイト目以降のバイト列はすべて`0x00`
- 左から2バイト目の値は`0xD0`以上であること
    - 0xD0未満であると、マイナスの mph になるため、正常なパケットではない

##### 異常パケットの条件

- 正常なパケットの条件を満たさない場合

##### 速度データ部パーサのプロトタイプ宣言

```c
struct struct_error {
    int32_t code; // 正常データの場合は0
    char* message;
}
struct fstar_int32_array {
    int32_t* value; // ex. [0x64, 0xE0]
    struct_error error;
}
// codeの値を確認して、エラーか正常か判定する
fstar_int32_array parseSpeed(uint8_t *data, uint8_t data_length);
```

## ウインカー表示

`id=0x188, len=4` で固定

- 全状態数は３

##### ウインカー表示なし

`0x00, 0x00, 0x00, 0x00`

##### ウインカー左表示

`0x01, 0x00, 0x00, 0x00`

##### ウインカー右表示

`0x02, 0x00, 0x00, 0x00`

##### 正常なパケットの条件

- 左から2バイト目以降のバイト列はすべて`0x00`であること
- 左から1バイト目の値は`0x00 or 0x01 or 0x02`であること

##### 異常パケットの条件

- 正常なパケットの条件を満たさない場合

##### ウインカーデータ部パーサのプロトタイプ宣言

```c
struct struct_error {
    int32_t code; // 正常データの場合は0
    char* message;
}
struct fstar_uint8 {
    uint8_t value; // ex. 0x00
    struct_error error;
}
// codeの値を確認して、エラーか正常か判定する
fstar_uint8 parseIndicator(uint8_t *data, uint8_t data_length);
```

## ドアのロック・アンロック

`id=0x19B, len=6` で固定

- ドア状態を表すビット列は左から 3バイト目の下位4ビットで表す

- 左から順に、右リアドア・左リアドア・右フロントドア・左フロントドアを表すビット

- 全状態数は 2x2x2x2=16

### 例
##### すべてのドアをアンロックした状態

`0x00, 0x00, 0b00000000, 0x00, 0x00, 0x00`

##### すべてのドアをロックした状態

`0x00, 0x00, 0b00001111, 0x00, 0x00, 0x00`

##### すべてのドアをロックした状態から、左フロントドアのみをアンロックした状態

`0x00, 0x00, 0b00001110, 0x00, 0x00, 0x00`

##### すべてのドアをアンロックした状態から、右リアドアのみをロックした状態

`0x00, 0x00, 0b00001000, 0x00, 0x00, 0x00`

##### 正常なパケットの条件

- 左から3バイト目以外のバイト列はすべて`0x00`であること
- 左から3バイト目の値は`0x00`以上`0x0F`以下の値であること

##### 異常パケットの条件

- 正常なパケットの条件を満たさない場合

##### ドアのロック・アンロックデータ部パーサのプロトタイプ宣言

```c
struct struct_error {
    int32_t code; // 正常データの場合は0
    char* message;
}
struct fstar_uint8 {
    uint8_t value; // ex. 0b00001000
    struct_error error;
}
// codeの値を確認して、エラーか正常か判定する
fstar_uint8 parseDoor(uint8_t *data, uint8_t data_length);
```