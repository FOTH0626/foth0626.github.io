---
title: URP Shader 模板
description: a template for urp shaders
slug: urp-shader-template
image: 
draft: false
categories:
  - Graphics
date: 2025-06-03 15:49:54+08:00
tags:
  - Unity
weight: 0
---
> 首先零帧起手
![](lowb_unity.jpg)   

我就纳闷了，unity6了，urp大版本号都17了，byd 傻逼unity官方，你们引擎团队养的7k号人，都他妈吃屎的吧，到现在连个URP模板都没有，每次创shader默认都是cg语言😅😅，我真他妈服了。  

---
然后是模板，源自养乐多大佬，我之后又根据自己的需求魔改过，后续也可能会继续魔改。  
Github仓库地址：[FothUrpShaderTemplate](https://github.com/FOTH0626/FothUrpShaderTemplate)

```hlsl
Shader "Foth/Template"
{
    Properties
    {
        [Header(Main Maps)]
        _Color ("Color", Color)=(1,1,1,1)
        [NoScaleOffset] _MainTex ("Texture",2D)="white"{}
        [NoScaleOffset] _OtherDataTex ("Other Data Tex",2D) = "white"{}
        [NoScaleOffset] _OtherDataTex2 ("Other Data Tex 2",2D) = "white"{}
        [NoScaleOffset] _OtherDataTex3 ("Other Data Tex 3",2D)= "white"{}

        _AlphaClip("Alpha Clipping",Range(0,1))=0.333


        [Header(Option)]
        [Enum(UnityEngine.Rendering.CullMode)] _Cull ("Cull (Default back)",Float)=2
        [Enum(Off,0,On,1)] _ZWrite ("ZWrite (Default On)",Float)=1
        [Enum(UnityEngine.Rendering.BlendMode)]_SrcBlendMode ("Src blend mode (Default One)",Float)=1
        [Enum(UnityEngine.Rendering.BlendMode)]_DstBlendMode ("Dst blend mode (Default Zero)",Float)=0
        [Enum(UnityEngine.Rendering.BlendOp)]_BlendOp ("Blend operation (Default Add)",Float)=0
        _StencilRef ("Stencil reference",Int)=0
        [Enum(UnityEngine.Rendering.CompareFunction)]_StencilComp ("Stencil compare function",Int)=0
        [Enum(UnityEngine.Rendering.StencilOp)]_StencilPassOp ("Stencil pass operation",Int)=0
        [Enum(UnityEngine.Rendering.StencilOp)]_StencilFailOp ("Stencil fail operation",Int)=0
        [Enum(UnityEngine.Rendering.StencilOp)]_StencilZFailOp ("Stencil Z fail operation",Int)=0
    
        //[Header(SRP Default)]
        //[Toggle(_SRP_DEFAULT_PASS)]_SRPDefaultPass ("SRP Default Pass",Int)=0
        //[Enum(UnityEngine.Rendering.BlendMode)]_SRPSrcBlendMode ("SRP src blend mode (Default One)",Float)=1
        //[Enum(UnityEngine.Rendering.BlendMode)]_SRPDstBlendMode ("SRP dst blend mode (Default Zero)",Float)=0
        //[Enum(UnityEngine.Rendering.BlendOp)]_SRPBlendOp ("SRP blend operation (Default Add)",Float) =0
        
        //_SRPStencilRef ("SRP stencil reference",Int)=0
        //[Enum(UnityEngine.Rendering.CompareFunction)]_SRPStencilComp ("SRP stencil compare function",Int)=0
        //[Enum(UnityEngine.Rendering.StencilOp)]_SRPStencilPassOp ("SRP stencil pass operation",Int)=0
        //[Enum(UnityEngine.Rendering.StencilOp)]_SRPStencilFailOp ("SRP stencil fail operation",Int)=0
        //[Enum(UnityEngine.Rendering.StencilOp)]_SRPStencilZFailOp ("SRP stencil Z fail operation",Int)=0
    }
    

    SubShader
    {
        Tags
        {
            "RenderPipeline" = "UniversalPipeline"
            //
            //"RenderType" = "Background"
            "RenderType" = "Opaque"
            // "RenderType"="Transparent"
            // "RenderType"="TransparentCutout"
            // "RenderType"="Overlay"
            //
            //"Queue" = "Background"
            "Queue"="Geometry"
            //"Queue" = "AlphaTest"
            //"Queue" = "Transparent"
            //"Queue" = "TransparentCutout"
            //"Queue" = "Overlay"
            //"IgnoreProjector" = "True"            
        }
        //LOD 100
        
        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _Color;

        Texture2D _MainTex;
        Texture2D _OtherDataTex;
        Texture2D _OtherDataTex2;
        Texture2D _OtherDataTex3;

        float _AlphaClip;

        CBUFFER_END
        

        struct UniversalAttributes
        {
            float4 positionOS   : POSITION;
            float4 tangentOS    : TANGENT;
            float3 normalOS     : NORMAL;
            float2 texcoord     : TEXCOORD0;
        };

        struct UniversalVaryings
        {
            float2 uv                       :TEXCOORD0;
            float4 positionWSAndFogFactor   :TEXCOORD1;
            float3 normalWS                 :TEXCOORD2;
            float4 tangentWS                :TEXCOORD3;
            float3 viewDirectionWS          :TEXCOORD4;
            float4 positionCS               :SV_POSITION;
        };

        UniversalVaryings MainVS(UniversalAttributes input)
        {
            VertexPositionInputs positionInputs =  GetVertexPositionInputs(input.positionOS.xyz);
            VertexNormalInputs normalInputs = GetVertexNormalInputs(input.normalOS,input.tangentOS);

            UniversalVaryings output;
            output.positionCS =  positionInputs.positionCS;
            output.positionWSAndFogFactor = float4(positionInputs.positionWS,ComputeFogFactor(positionInputs.positionCS.z));
            output.normalWS =  normalInputs.normalWS;
            output.tangentWS.xyz =  normalInputs.tangentWS;
            output.tangentWS.w =  input.tangentOS.w * GetOddNegativeScale();
            output.viewDirectionWS = unity_OrthoParams.w == 0 ? GetCameraPositionWS() -positionInputs.positionWS : GetWorldToViewMatrix()[2].xyz;
            output.uv = input.texcoord;
            return output;
        }

        float4 MainFS (UniversalVaryings input) : SV_Target
        {
            float3 normalWS = normalize(input.normalWS);
            float3 positionWS = input.positionWSAndFogFactor.xyz;
            float3 viewDirectionWS = normalize(input.viewDirectionWS);
            float3 tangentWS = normalize(input.tangentWS.xyz);
            float3 bitangentWS = cross(normalWS,tangentWS) * input.tangentWS.w;
            float2 uv = input.uv;

            return float4(0,0,1,1);
        } 
        
        ENDHLSL

        //Shadow Map Pass
        Pass
        {
            Name "ShadowCaster"
            Tags
            {
            "LightMode"="ShadowCaster"
            }
            ZWrite [_ZWrite]
            ZTest LEqual
            ColorMask 0
            Cull [_Cull]
            
            HLSLPROGRAM
            
            #pragma multi_compile_instancing
            #pragma multi_compile _ DOTS_INSTANCING_ON
            #pragma multi_compile_vertex _ _CASTING_PUNCTUAL_LIGHT_SHADOW
            #pragma vertex vert
            #pragma fragment frag
            
            float3 _LightDirection;
            float3 _LightPosition;

            struct Attributes
            {
                float4 positionOS   : POSITION;
                float3 normalOS     : NORMAL;
                float2 texcoord     : TEXCOORD0;
            };
            struct Varyings
            {
            float2 uv               : TEXCOORD0;
            float4 positionCS       : SV_POSITION;
            };
            
            float4 GetShadowPositionHClip(Attributes input)
            {
                float3 positionWS =  TransformObjectToWorld(input.positionOS.xyz);
                float3 normalWS = TransformObjectToWorldNormal(input.normalOS);
            #if CASTING_PUNCTUAL_LIGHT_SHADOW
                float3 lightDirectionWS = normalize(_LightPosition - positionWS);
            #else
                float3 lightDirectionWS = _LightDirection;
            #endif
                
                float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS,normalWS,lightDirectionWS));
            #if UNITY_REVERSED_Z
                positionCS.z =  min(positionCS.z,UNITY_NEAR_CLIP_VALUE);
            #else
                positionCS.z =  max(positionCS.z,UNITY_NEAR_CLIP_VALUE);
            #endif
    
                return positionCS;
            }


            Varyings vert(Attributes input)
            {
                Varyings output;
                output.uv = input.texcoord;
                output.positionCS = GetShadowPositionHClip(input);
                return output;
            }
            float4 frag(Varyings input):SV_TARGET
            {
                clip(1.0 -_AlphaClip);
                return 0;
            }
            
            ENDHLSL
        }        


        //Zbuffer(Depth)  Pass
        Pass
        {
            Name"DepthOnly"
            Tags
            {
                "LightMode" = "DepthOnly"
            }
            
            ZWrite [_ZWrite]
            ColorMask 0
            Cull [_Cull]
            
            HLSLPROGRAM

            #pragma multi_compile_instancing
            #pragma multi_compile _ DOTS_INSTANCING_ON

            #pragma vertex vert
            #pragma fragment frag

            struct Attributes
            {
                float4 positionOS : POSITION;
            };

            struct Varyings
            {
                float4 positionCS : SV_POSITION;
            };

            Varyings vert(Attributes input)
            {
                Varyings output = (Varyings)0;
                output.positionCS = TransformObjectToHClip(input.positionOS.xyz);

                return output;
            }

            half4 frag(Varyings input): SV_TARGET
            {
                clip(1.0 - _AlphaClip);
                return 0;
            }
            ENDHLSL
        }


        //Main Pass
        Pass
        {
            Name"UniversalForward"
            
            Tags
            {
                "LightMode" = "UniversalForward"
            }
            
            Cull [_Cull]
            Blend[_SrcBlendMode] [_DstBlendMode]
            BlendOp [_BlendOp]
            ZWrite [_ZWrite]
            Stencil
            {
                Ref [_StencilRef]
                Comp [_StencilComp]
                Pass [_StencilPassOp]
                Fail [_StencilFailOp]
                ZFail [_StencilZFailOp]
            }

            HLSLPROGRAM

            #pragma  vertex MainVS
            #pragma  fragment MainFS
            
            ENDHLSL
        }
    }
}
```