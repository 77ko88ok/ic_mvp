# bevy_shader_material
my personal training of bevy
copied and modified in Japanese from https://bevy.org/examples/shaders/shader-material/

#### **Step 1: マテリアルとコンポーネントの拡張（データ構造の準備）**
現在の `CustomMaterial` は単純な色しか持てませんが、これを「油絵のパラメータ」を受け取れるように拡張します。

*   **Rust側の変更点**:
    *   `CustomMaterial` に、量子化の度合い（`pixelation_level`）、テクスチャの合成比率（`layer_weight`）、筆致のノイズ強度（`noise_strength`）などのフィールドを追加。
    *   `CanvasProxy` コンポーネントを作成し、エンティティごとにこれらを変えられるようにする。

#### **Step 2: WESLシェーダーの改良（質感の実現）**
添付いただいた `custom_material.wesl` を、上記の「3つの要素」を計算できるように書き換えます。

*   **WESL側の変更点**:
    *   UV座標を量子化する処理を追加（ピクセル感の獲得）。
    *   ノイズを混ぜてUVを歪ませる処理を追加（筆致の獲得）。
    *   複数のテクスチャを合成するロジックを組み込む。

#### **Step 3: 動的な適用（プラグイン的アプローチの基礎）**
`main.rs` でこれらを一括管理する仕組みを整えます。

---

### **最初のコード改善案（Step 1 & 2 の一部）**

まずは、WESLシェーダーの中で**「UVの量子化（ピクセル化）」**と**「簡易的な質感の合成」**ができるように、シェーダーを少しだけ改造するところから始めましょう。

**`custom_material.wesl` の修正イメージ（一部抜粋）:**

```rust
// --- 追加するパラメータ ---
@group(constants::MATERIAL_BIND_GROUP) @binding(0)
var<uniform> material: CustomMaterialData; // ここに pixelation_level 等を含める

// --- フラグメントシェーダーの核となる計算 ---
@fragment
fn fragment(in: VertexOutput) -> @location(0) vec4<f32> {
    let uv: vec2<f32> = in.uv;

    // 1. 量子化（ピクセル化）の計算
    // 例: UVを特定の解像度（例：128px）で丸める
    let resolution: f32 = 128.0; 
    let quantized_uv = floor(uv * resolution) / resolution;

    // 2. ノイズによる歪み（簡易的な筆致のシミュレーション）
    // ※本来はノイズ関数を定義しますが、まずは単純な計算で。
    let distorted_uv = quantized_uv + (sin(uv.y * 50.0) * 0.002); 

    // 3. 質感の合成
    let base_color = textureSample(material_color_texture, material_color_sampler, distorted_uv);
    // ここで、もう一つのテクスチャ（ハイライト用など）を重ねる処理を追加していく
    
    var final_color: vec4<f32> = base_color * material.base_color;
    return final_color;
}
```
