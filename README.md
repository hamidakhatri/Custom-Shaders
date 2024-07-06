# Custom-Shaders

## Overview

This repository contains three custom shaders created for a Unity project. The shaders include a Surface Shader, an Unlit Shader, and an Extraterrestrial Glow Shader. Each shader is designed to demonstrate different rendering techniques and includes properties that can be adjusted in the Unity Editor.

## Shaders

### 1. Surface Shader

**Description**: A physically-based rendering (PBR) surface shader that supports main color, texture, normal map, metallic, smoothness, and emission properties.

**Explanation**:
- **Properties**: Defines various properties like Main Color, Main Texture, Normal Map, Metallic, Smoothness, Emission Color, and Emission Strength.
- **SubShader**: Contains the main rendering logic.
- **surf function**: Define how the texture, normal map, metallic, smoothness, and emission are combined to give the final appearance of the surface.

**Shader Code**:
```csharp
Shader "Custom/HKSurfaceShader"
{
    Properties
    {
        _Color ("Main Color", Color) = (1,1,1,1)
        _MainTex ("Main Texture", 2D) = "white" {}
        _NormalMap ("Normal Map", 2D) = "bump" {}
        _Metallic ("Metallic", Range(0,1)) = 0.5
        _Smoothness ("Smoothness", Range(0,1)) = 0.5
        _EmissionColor ("Emission Color", Color) = (0,0,0,1)
        _EmissionStrength ("Emission Strength", Range(0,10)) = 1.0
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 200

        CGPROGRAM
        #pragma surface surf Standard fullforwardshadows

        sampler2D _MainTex;
        sampler2D _NormalMap;
        fixed4 _Color;
        half _Metallic;
        half _Smoothness;
        fixed4 _EmissionColor;
        half _EmissionStrength;

        struct Input
        {
            float2 uv_MainTex;
            float2 uv_NormalMap;
        };

        void surf (Input IN, inout SurfaceOutputStandard o)
        {
            // Albedo
            fixed4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;

            // Normal map
            o.Normal = UnpackNormal(tex2D(_NormalMap, IN.uv_NormalMap));

            // Metallic and Smoothness
            o.Metallic = _Metallic;
            o.Smoothness = _Smoothness;

            // Emission
            o.Emission = _EmissionColor.rgb * _EmissionStrength;
        }
        ENDCG
    }
    FallBack "Diffuse"
}
```

### 2. Unlit Shader

**Description**: An unlit shader that supports main color, texture, glow effect, and animated UV distortion.

**Explanation**:
- **Properties**: Defines the Main Color, Main Texture, Glow Color, Glow Strength, Distortion Amount, and Time Speed.
- **SubShader**: Contains the rendering logic for the unlit shader.
- **vert function**: Handles vertex transformation and passes UV coordinates.
- **frag function**: Handles the fragment color calculation, applying distortion and glow effects.

**Shader Code**:
```csharp
Shader "Custom/HKUnlitShader"
{
    Properties
    {
        _Color ("Main Color", Color) = (1,1,1,1)
        _MainTex ("Main Texture", 2D) = "white" {}
        _GlowColor ("Glow Color", Color) = (0,1,0,1)
        _GlowStrength ("Glow Strength", Range(0,1)) = 0.5
        _DistortAmount ("Distortion Amount", Range(0,1)) = 0.1
        _TimeSpeed ("Time Speed", Range(0.1, 10)) = 1
    }
    SubShader
    {
        Tags { "RenderType" = "Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            fixed4 _Color;
            fixed4 _GlowColor;
            float _GlowStrength;
            float _DistortAmount;
            float _TimeSpeed;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = v.uv;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                // Distortion effect
                float time = _Time.y * _TimeSpeed;
                float2 distortedUV = i.uv + float2(sin(i.uv.y * 10 + time) * _DistortAmount, cos(i.uv.x * 10 + time) * _DistortAmount);

                // Texture sampling
                fixed4 texColor = tex2D(_MainTex, distortedUV) * _Color;

                // Glow effect
                fixed4 glow = _GlowColor * _GlowStrength * (1.0 - texColor.a);

                // Final color
                fixed4 col = texColor + glow;
                return col;
            }
            ENDCG
        }
    }
    FallBack "Unlit/Texture"
}
```

### 3. Extraterrestrial Glow

**Description**: An exciting shader that creates an extraterrestrial glow effect with animated UV coordinates for both the main and glow textures.

**Explanation**:
- **Properties**: Defines Main Texture, Glow Texture, Main Color, Glow Color, Glow Intensity, Distortion Amount, Main Texture Time Speed, and Glow Texture Time Speed.
- **SubShader**: Contains the rendering logic for the shader.
- **vert function**: Transforms vertex positions and calculates UV coordinates.
- **frag function**: Distorts UV coordinates for both main and glow textures, samples the textures, and combines the colors with specified tints and intensities.

**Shader Code**:
```csharp
Shader "Custom/HKExtraterrestrialGlow"
{
    Properties
    {
        _MainTex ("Main Texture", 2D) = "white" {}
        _GlowTex ("Glow Texture", 2D) = "white" {}
        _Color ("Main Color", Color) = (0.5,0.5,1,1)
        _GlowColor ("Glow Color", Color) = (0,1,1,1)
        _GlowIntensity ("Glow Intensity", Range(0,10)) = 2.0
        _DistortAmount ("Distortion Amount", Range(0,1)) = 0.1
        _MainTexTimeSpeed ("Main Texture Time Speed", Range(0.1, 5)) = 1.0
        _GlowTexTimeSpeed ("Glow Texture Time Speed", Range(0.1, 5)) = 1.0
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 200

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float4 vertex : SV_POSITION;
                float2 uv : TEXCOORD0;
                float2 uvGlow : TEXCOORD1;
            };

            sampler2D _MainTex;
            sampler2D _GlowTex;
            fixed4 _Color;
            fixed4 _GlowColor;
            float _GlowIntensity;
            float _DistortAmount;
            float _MainTexTimeSpeed;
            float _GlowTexTimeSpeed;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = v.uv;
                o.uvGlow = v.uv;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                float time = _Time.y;
                
                // Distortion effect for the main texture
                float2 distortedUV = i.uv + float2(sin(i.uv.y * 10 + time * _MainTexTimeSpeed) * _DistortAmount, cos(i.uv.x * 10 + time * _MainTexTimeSpeed) * _DistortAmount);
                
                // Animated UV for the glow texture
                float2 animatedGlowUV = i.uvGlow + float2(time * _GlowTexTimeSpeed, time * _GlowTexTimeSpeed);
                
                // Main texture color
                fixed4 mainColor = tex2D(_MainTex, distortedUV) * _Color;

                // Glow texture and effect
                fixed4 glowColor = tex2D(_GlowTex, animatedGlowUV) * _GlowColor;
                glowColor *= _GlowIntensity;

                // Final color
                fixed4 finalColor = mainColor + glowColor;
                finalColor.a = mainColor.a; // Preserve alpha from main texture

                return finalColor;
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```
