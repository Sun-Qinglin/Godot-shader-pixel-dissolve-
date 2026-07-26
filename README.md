<h1 align="center"><b>Godot-shader(pixel-dissolve)</b></h1>

<img src="https://user-images.githubusercontent.com/73097560/115834477-dbab4500-a447-11eb-908a-139a6edaec5c.gif">

<h1 align=="center">
  A Godot shader can make 2D-object pixel dissolve
</h1>

<h1 align=="center">
  <img src="gif/main_scene.tscn - 模块化godot (Copy2) - Godot Engine 2026-07-26 11-54-29.gif" width="600" alt="pixel dissove">
</h1>

```gdscript
shader_type canvas_item;

// =============================================
//  像素化溶解 Shader（从右往左）
//  适用于 ColorRect / Control 节点
//  Godot 4.x
// =============================================

// ---------- 核心控制 ----------

// 溶解进度：0.0 = 完全显示，1.0 = 完全溶解
uniform float dissolve_progress : hint_range(0.0, 1.0) = 0.0;

// ---------- 像素化参数 ----------

// 像素块宽度（值越大，块越小；值越小，块越大）
uniform float block_width : hint_range(2.0, 128.0) = 24.0;

// 像素块高度
uniform float block_height : hint_range(2.0, 128.0) = 24.0;

// ---------- 溶解边缘参数 ----------

// 边缘随机性强度（0 = 整齐切割，1 = 高度不规则）
uniform float randomness : hint_range(0.0, 1.0) = 0.35;

// 是否启用边缘发光
uniform bool enable_edge_glow = true;

// 边缘发光颜色（RGBA）
uniform vec4 edge_color : source_color = vec4(1.0, 0.55, 0.1, 1.0);

// 边缘发光宽度（以像素块为单位）
uniform float edge_width : hint_range(0.5, 5.0) = 2.0;

// 边缘发光强度
uniform float edge_intensity : hint_range(0.0, 2.0) = 1.0;

void fragment() {
	vec2 uv = UV;
	vec2 block_size = vec2(block_width, block_height);

	// ---- 像素化：计算当前像素所属的块 ----
	vec2 block_id = floor(uv * block_size);

	// ---- 为每个块生成稳定的伪随机值 ----
	float rand = fract(sin(dot(block_id, vec2(12.9898, 78.233))) * 43758.5453);

	// ---- 块中心的 x 坐标，归一化到 [0, 1] ----
	float block_center_x = (block_id.x + 0.5) / block_size.x;

	// ======== 从右往左溶解核心逻辑 ========
	//
	//  dissolve_progress = 0.0  ->  threshold = 1.0  ->  全部可见
	//  dissolve_progress = 0.5  ->  threshold = 0.5  ->  左半可见
	//  dissolve_progress = 1.0  ->  threshold = 0.0  ->  全部溶解
	//
	float threshold = 1.0 - dissolve_progress;

	// 加入随机偏移，但限制在有效范围内
	// 确保 dissolve_progress = 0 时全部可见，= 1 时全部消失
	float random_offset = (rand * 2.0 - 1.0) * randomness;
	float noisy_threshold = threshold + random_offset;

	// 像素化硬边判断
	// 当 threshold >= 1.0 时，所有块都可见（因为 block_center_x <= 1.0）
	// 当 threshold <= 0.0 时，所有块都不可见（因为 block_center_x >= 0.0）
	float visible = step(block_center_x, noisy_threshold);

	// ======== 边缘发光 ========
	vec3 final_color = COLOR.rgb;

	if (enable_edge_glow) {
		// 计算块中心到溶解阈值的距离（转换为块单位）
		float dist_to_edge = abs(block_center_x - noisy_threshold) * block_size.x;

		// 在边缘范围内的块显示发光效果
		float glow = (1.0 - smoothstep(0.0, edge_width, dist_to_edge)) * visible;
		glow *= edge_intensity;

		vec3 edge_rgb = edge_color.rgb;
		final_color = mix(COLOR.rgb, edge_rgb, glow);
	}

	COLOR = vec4(final_color, COLOR.a * visible);
}
```


<p align="center">
  <img src="https://img.shields.io/badge/Godot-4.x-blue.svg">
  <img src="https://img.shields.io/badge/License-MIT-green.svg">
</p>
