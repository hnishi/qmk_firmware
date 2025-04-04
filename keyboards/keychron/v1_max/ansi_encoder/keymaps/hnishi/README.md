# Keychron V1 Max カスタムキーマップ

## 機能概要

### JIS/US 配列切り替え機能

-   Fn + Esc キーで切り替え可能
-   EEPROM に設定を保存し、再起動後も維持
-   LED パターンによるモード表示
    -   JIS モード: レインドロップパターン (RGB_MATRIX_RAINDROPS)
    -   US モード: サイクルパターン (RGB_MATRIX_CYCLE_ALL)

## 実装詳細

### 1. モード制御

```c
// デフォルトはUSモード
static bool is_jis_mode = false;

// EEPROMのアドレス定義
#define JIS_MODE_MAGIC 0x1234 // 設定が保存されているかの判定用
#define EEPROM_JIS_MODE_ADDR 10
#define EEPROM_JIS_MAGIC_ADDR 8
```

### 2. 初期化処理

起動時に EEPROM から設定を読み込みます：

```c
void keyboard_post_init_user(void) {
    // EEPROM読み込み
    uint16_t magic = eeprom_read_word((uint16_t *)EEPROM_JIS_MAGIC_ADDR);
    if (magic == JIS_MODE_MAGIC) {
        is_jis_mode = eeprom_read_byte((uint8_t *)EEPROM_JIS_MODE_ADDR);
    } else {
        // 初期値を保存
        eeprom_write_word((uint16_t *)EEPROM_JIS_MAGIC_ADDR, JIS_MODE_MAGIC);
        eeprom_write_byte((uint8_t *)EEPROM_JIS_MODE_ADDR, is_jis_mode);
    }
}
```

### 3. モード切り替え処理

Fn + Esc キーが押されたときの処理：

```c
static void toggle_jis_mode(void) {
    is_jis_mode = !is_jis_mode;
    eeprom_write_byte((uint8_t *)EEPROM_JIS_MODE_ADDR, is_jis_mode);
    // LEDパターンで状態を表示
    if (is_jis_mode) {
        rgb_matrix_mode_noeeprom(RGB_MATRIX_RAINDROPS);
    } else {
        rgb_matrix_mode_noeeprom(RGB_MATRIX_CYCLE_ALL);
    }
}
```

### 4. キー入力処理

JIS モード時のみ、特定のキー入力を変換します：

```c
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
    if (!process_record_keychron_common(keycode, record)) {
        return false;
    }

    // Fnキーの状態を監視
    static bool fn_pressed = false;
    static uint8_t current_layer = 0;
    current_layer = get_highest_layer(layer_state);
    fn_pressed = (current_layer == MAC_FN || current_layer == WIN_FN);

    // Fn + Escで切り替え
    if (keycode == KC_ESC && fn_pressed && record->event.pressed) {
        toggle_jis_mode();
        return false;
    }

    // JISモードの時のみ変換を実行
    if (is_jis_mode && !twpair_on_jis(keycode, record)) {
        return false;
    }

    return true;
}
```
