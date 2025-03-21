# Keychron V1 Max カスタムキーマップ

## 機能概要

### JIS/US 配列切り替え機能

-   Fn + Esc キーで切り替え可能
-   EEPROM に設定を保存し、再起動後も維持
-   LED によるモード表示

## 実装詳細

### 1. モード切り替え

```c
// モード制御
bool is_jis_mode = true;  // デフォルトはJISモード
#define EEPROM_JIS_MODE_ADDR 10
```

### 2. LED 表示

既存の RGB ライトモードを使用：

-   JIS モード: デフォルトの RGB ライトモード
-   US モード: `RGB_MODE_RAINBOW`（カスタマイズ可能）

### 3. 設定変更方法

1. `RGB_TOG`で提供される標準の RGB モードから選択可能
2. モードは EEPROM に保存され、再起動後も維持

### US モードの LED 設定カスタマイズ方法

1. `keymap.c`内の RGB ライト設定を変更：

```c
// 例：USモードでのLED設定
#define US_MODE_HUE 170  // 青系
#define US_MODE_SAT 255  // 最大彩度
#define US_MODE_VAL 255  // 最大輝度
#define US_MODE_MODE RGBLIGHT_MODE_RAINBOW  // レインボーモード
```

2. アニメーションモードの選択肢：

-   RGBLIGHT_MODE_STATIC_LIGHT
-   RGBLIGHT_MODE_BREATHING
-   RGBLIGHT_MODE_RAINBOW
-   RGBLIGHT_MODE_SNAKE
