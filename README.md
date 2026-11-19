# mirador-physical-ruler

A Mirador 4 plugin that displays a physical dimensions ruler overlay using the IIIF Physical Dimensions (physdim) service.

IIIF マニフェストの canvas に physdim サービスが設定されていれば、ビューア上にスケールルーラーを SVG でオーバーレイ表示する Mirador 4 プラグインです。

**Demo / デモ**: [https://ruler.ldas.jp/](https://ruler.ldas.jp/)

## Installation / インストール

```bash
npm install mirador-physical-ruler
```

## Usage / 使い方

### Basic / 基本

```js
import Mirador from 'mirador';
import physicalRulerPlugin from 'mirador-physical-ruler';

Mirador.viewer(
  { id: 'mirador', windows: [{ manifestId: 'https://example.com/manifest.json' }] },
  physicalRulerPlugin
);
```

### Custom options / カスタム設定

```js
import { createPlugin } from 'mirador-physical-ruler';

const rulerPlugin = createPlugin({
  color: '#ffffff',
  thickness: 34,
});
```

### Options / オプション一覧

| Option | Default | Description |
|---|---|---|
| `color` | `'#ffffff'` | Ruler color / ルーラーの色 |
| `thickness` | `34` | Ruler bar width in px / ルーラーの幅 (px) |
| `fallbackScale` | `null` | Fallback scale (mm/px) when physdim service is absent / physdim がない場合のフォールバック |
| `fallbackUnits` | `'mm'` | Fallback units / フォールバック時の単位 |

The gear button on the top-left corner opens a settings panel where you can change font size, tick size, color, opacity, and display units (mm/cm/in) at runtime.

左上の歯車ボタンから、フォントサイズ・目盛りサイズ・色・透明度・表示単位（mm/cm/in）をランタイムで変更できます。

## Demo with custom manifest / 任意のマニフェストでデモ

You can load any IIIF manifest by appending the `manifest` query parameter:

`?manifest=` パラメータで任意のマニフェストを読み込めます:

```
https://ruler.ldas.jp/?manifest=https://example.com/manifest.json
```

## Adding physdim service to your manifest / physdim サービスの追加方法

Add the following to the `service` property of your canvas:

canvas の `service` プロパティに以下を追加します:

```json
{
  "service": [
    {
      "@context": "http://iiif.io/api/annex/services/physdim/1/context.json",
      "profile": "http://iiif.io/api/annex/services/physdim",
      "physicalScale": 0.0848,
      "physicalUnits": "mm"
    }
  ]
}
```

### Calculating physicalScale / physicalScale の計算方法

```
physicalScale = physical size of the object (mm) / the object's span on the canvas (px)
```

Measure the object on the canvas, not the whole canvas. A scan often includes background, a ruler or a colour bar around the object, and then the canvas size does not represent the object ([Physical Dimensions service](https://iiif.io/api/annex/services/#physical-dimensions)). Dividing by the whole canvas makes every reading too short.

キャンバス全体ではなく、キャンバス上の資料そのものの長さ（px）で割ります。スキャン画像には資料の周りの余白・物差し・カラーチャートが含まれることが多く、キャンバス全体で割ると、ルーラーの読みがすべて短くなります。

Example: the Hyakki Yagyo demo. The paper is 28 cm high and spans about 2,802 px of the 3,082 px canvas.

例（百鬼夜行図のデモ）: 本紙の縦 28cm は、高さ 3,082px のキャンバスのうち約 2,802px にあたります。

| Divide 280 mm by / 割る値 | physicalScale | Canvas length / キャンバスの横の長さ |
|---|---|---|
| Paper span / 本紙の高さ 2,802 px | 0.0999 mm/px | 7.94 m |
| Whole canvas / キャンバス全体 3,082 px | 0.0909 mm/px | 7.22 m (9% short / 9% 短い) |

If scan resolution (dpi) is known and the canvas has the image's pixel size / スキャン解像度がわかっていて、キャンバスと画像のピクセル寸法が同じ場合:

```
physicalScale = 25.4 / dpi
```

`physicalUnits`: `"mm"` or `"in"`

## How it works / 動作の仕組み

1. The plugin wraps `OpenSeadragonViewer` and obtains the OSD instance via `OSDReferences`
2. Listens to `zoom` / `pan` / `animation` events to update viewport information
3. Calculates screen pixels per physical unit from `getBoundsNoRotate()` and `canvasWidth * physicalScale`
4. Renders the ruler as SVG with automatically adjusted tick intervals

## Notes / 注意事項

- **Mirador 4.x only** (for Mirador 3, use [mirador-ruler-plugin](https://github.com/ubleipzig/mirador-ruler-plugin))
- The ruler is not displayed on canvases without a physdim service (unless `fallbackScale` is set)

## Acknowledgements

Inspired by [mirador-ruler-plugin](https://github.com/ubleipzig/mirador-ruler-plugin) (Mirador 3) and [dbmdz/mirador-plugins](https://github.com/dbmdz/mirador-plugins) physicalRuler (Mirador 2).

## License

MIT
