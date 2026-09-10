---
layout:     post
title:      "Cocos 转 Spine"
subtitle:   "Cocos 转 Spine"
date:       2026-08-31 01:05:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
visible:    true
tags:
    - Game Development
---
前提：近期将Lost Journey从Cocos2d-x迁移到Unity，最初的动画是由Cocos制作的，因为只有Spine在一直维护，所以计划将Cocos动画转化为spine动画，在Unity中使用。Dragonbone可以直接读取数据。但导出Spine后会有问题，要么节点位置不对，要么有奇怪的bug。

结论：最后总结出动画的转化步骤如下

    1. 使用Dragonbone导入cocos的动画数据，并使用Dragonbone导出Dragonbone 5.5格式的动画，以hero为例，包括hero_ske.json, hero_tex.json, hero_tex.png
    2. 使用DragonBoneToSpineData，将hero_ske.json, hero_tex.json, hero_tex.png整个文件夹拖入DragonBoneToSpineData转化为spine数据的armatureName.json(spine3.3支持的格式)， hero_ske.atlas.txt，hero_tex.png
    3. 使用spine 4.3的纹理解包器，导入hero_ske.atlas.txt可以解包出所有的图片资源
    4. 使用spine3.3导入armatureName.json，设置图片的路径为前面解压的图片资源的文件夹。然后保存项目，然后用spine4.3重新打开项目，然后导出armatureName.json(即spine4.3版本的数据)
    4. armatureName.json会存在问题：关键帧的alpha错误,duration错误,还有bone缩放错误。需要重建 slot 颜色轨道和 bone 缩放轨道，还有等等各种各样的问题。以下提供的Unity代码用于根据cocos动画重建armatureName.json后导出armatureName_output.json
    5. 使用spine4.3导入armatureName_output.json，然后将图片文件夹设置为解包后的Image文件夹，将骨架的名字修改为旧的名字，然后导出即可。

附带4的Unity代码:

```C#
#pragma warning disable ET0004
using System;
using System.Collections.Generic;
using System.IO;
using System.Text;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using UnityEditor;
using UnityEngine;

/// <summary>
/// 依据 Cocos Studio ExportJson 修复转换后的 Spine 4.3 动画数据。
/// </summary>
public static class CocosExportJsonToSpine43RepairEditor
{
    private const string MenuPath = "GameFramework/Project/Spine/5.(final) 修复 Cocos ExportJson 转 Spine4.3 动画数据";
    private const double CocosRuntimeFrameRate = 60d;
    private const double ValueTolerance = 0.000001d;
    private const double MatrixTolerance = 0.0001d;
    private const double EasingApproximationTolerance = 0.0001d;
    private const int MaxEasingSubdivisionDepth = 10;
    private const double RadiansToDegrees = 180d / Math.PI;
    private const double SineControlX1 = 0.36432160614986847d;
    private const double SineControlX2 = 0.6356762953556379d;

    [MenuItem(MenuPath)]
    private static void RepairAnimationData()
    {
        try
        {
            // 选择输入和独立输出路径。
            string sourcePath = EditorUtility.OpenFilePanel("选择 Cocos Studio ExportJson", Application.dataPath, "ExportJson");
            if (string.IsNullOrEmpty(sourcePath))
            {
                return;
            }

            string spinePath = EditorUtility.OpenFilePanel("选择未修复的 Spine 4.3 JSON", Path.GetDirectoryName(sourcePath), "json");
            if (string.IsNullOrEmpty(spinePath))
            {
                return;
            }

            string outputPath = Path.Combine(Path.GetDirectoryName(spinePath), Path.GetFileNameWithoutExtension(spinePath) + "_fixed.json");
            if (File.Exists(outputPath) && !EditorUtility.DisplayDialog("输出文件已存在", $"将覆盖已有文件：\n{outputPath}", "覆盖", "取消"))
            {
                return;
            }

            // 解析源数据并重建 setup pose、动画轨道和绘制顺序。
            JObject sourceRoot = JObject.Parse(File.ReadAllText(sourcePath));
            JObject spineRoot = JObject.Parse(File.ReadAllText(spinePath));
            JObject originalSpineRoot = spineRoot.DeepClone() as JObject;
            SourceData sourceData = ReadSourceData(sourceRoot);
            EnsureSpine43(spineRoot);
            SetupTransformData setupTransformData = BuildSetupTransformData(sourceData);
            RepairResult result = new RepairResult();
            RepairSetupPose(sourceData, setupTransformData, spineRoot, result);
            RepairAnimations(sourceData, setupTransformData, spineRoot, result);

            // 阻止修复误改 skins、附件和允许范围外的数据。
            EnsureOnlyRepairableDataChanged(originalSpineRoot, spineRoot, sourceData);
            ValidateRepairedSetupPose(sourceData, setupTransformData, spineRoot);
            ValidateRepairedTimelines(spineRoot);
            ValidateWithSpineRuntime(spineRoot);

            // 使用 UTF-8 无 BOM 和 LF 写入结果。
            string outputJson = NormalizeLineEndings(spineRoot.ToString(Formatting.Indented)) + "\n";
            File.WriteAllText(outputPath, outputJson, new UTF8Encoding(false));
            AssetDatabase.Refresh();

            Debug.Log(
                $"[CocosExportJsonToSpine43RepairEditor] Spine 4.3 数据修复完成。\n"
                    + $"Setup 骨骼: 修复 {result.RepairedSetupBoneCount}\n"
                    + $"Setup 插槽: 修复 {result.RepairedSetupSlotCount}\n"
                    + $"位移轨道: 重建 {result.RebuiltTranslateTimelineCount}，删除误造 {result.RemovedTranslateTimelineCount}\n"
                    + $"旋转轨道: 重建 {result.RebuiltRotateTimelineCount}，删除误造 {result.RemovedRotateTimelineCount}\n"
                    + $"缩放轨道: 重建 {result.RebuiltScaleTimelineCount}，删除误造 {result.RemovedScaleTimelineCount}\n"
                    + $"剪切轨道: 重建 {result.RebuiltShearTimelineCount}，删除误造 {result.RemovedShearTimelineCount}\n"
                    + $"附件轨道: 重建 {result.RebuiltAttachmentTimelineCount}，删除误造 {result.RemovedAttachmentTimelineCount}\n"
                    + $"颜色轨道: 重建 {result.RebuiltColorTimelineCount}，删除误造 {result.RemovedColorTimelineCount}\n"
                    + $"绘制顺序轨道: 重建 {result.RebuiltDrawOrderTimelineCount}，删除误造 {result.RemovedDrawOrderTimelineCount}\n"
                    + $"输出文件: {outputPath}"
            );
            EditorUtility.RevealInFinder(outputPath);
        }
        catch (Exception exception)
        {
            Debug.LogError($"[CocosExportJsonToSpine43RepairEditor] 修复失败: {exception.Message}\n{exception.StackTrace}");
            EditorUtility.DisplayDialog("Spine 4.3 数据修复失败", exception.Message, "确定");
        }
    }

    /// <summary>
    /// 读取唯一 Cocos armature、同名 animation_data、setup 骨骼和 movement 索引。
    /// </summary>
    private static SourceData ReadSourceData(JObject sourceRoot)
    {
        JArray armatures = sourceRoot["armature_data"] as JArray;
        if (armatures == null || armatures.Count != 1)
        {
            throw new InvalidDataException($"期望 ExportJson 包含 1 个 armature_data，实际为 {armatures?.Count ?? 0} 个。");
        }

        JObject armature = armatures[0] as JObject;
        string armatureName = armature?.Value<string>("name");
        JArray sourceBones = armature?["bone_data"] as JArray;
        if (string.IsNullOrEmpty(armatureName) || sourceBones == null)
        {
            throw new InvalidDataException("ExportJson armature_data[0] 缺少 name 或 bone_data。");
        }

        // 建立 setup 骨骼及显示附件索引。
        Dictionary<string, JObject> sourceBoneMap = BuildNamedObjectMap(sourceBones, "ExportJson setup 骨骼");
        Dictionary<string, List<string>> displayNameMap = new Dictionary<string, List<string>>(StringComparer.Ordinal);
        foreach (KeyValuePair<string, JObject> bonePair in sourceBoneMap)
        {
            JArray displays = bonePair.Value["display_data"] as JArray;
            List<string> displayNames = new List<string>();
            if (displays != null)
            {
                for (int displayIndex = 0; displayIndex < displays.Count; displayIndex++)
                {
                    JObject display = displays[displayIndex] as JObject;
                    string displayName = StripExtension(display?.Value<string>("name"));
                    if (string.IsNullOrEmpty(displayName))
                    {
                        throw new InvalidDataException($"ExportJson 骨骼 {bonePair.Key} 第 {displayIndex} 个 display_data 名称无效。");
                    }

                    displayNames.Add(displayName);
                }
            }

            displayNameMap.Add(bonePair.Key, displayNames);
        }

        // 选择与 armature 同名的 animation_data。
        JArray animationDataList = sourceRoot["animation_data"] as JArray;
        JObject selectedAnimationData = null;
        if (animationDataList != null)
        {
            for (int animationDataIndex = 0; animationDataIndex < animationDataList.Count; animationDataIndex++)
            {
                JObject animationData = animationDataList[animationDataIndex] as JObject;
                if (!string.Equals(animationData?.Value<string>("name"), armatureName, StringComparison.Ordinal))
                {
                    continue;
                }

                if (selectedAnimationData != null)
                {
                    throw new InvalidDataException($"ExportJson 存在重复 animation_data: {armatureName}。");
                }

                selectedAnimationData = animationData;
            }
        }

        JArray movements = selectedAnimationData?["mov_data"] as JArray;
        if (movements == null)
        {
            throw new InvalidDataException($"ExportJson 缺少与 armature {armatureName} 同名的 animation_data.mov_data。");
        }

        Dictionary<string, JObject> movementMap = BuildNamedObjectMap(movements, "ExportJson movement");
        List<string> sourceBoneOrder = CollectNamedOrder(sourceBones, "ExportJson setup 骨骼");
        return new SourceData(sourceBoneMap, sourceBoneOrder, displayNameMap, movementMap);
    }

    /// <summary>
    /// 限定目标为当前工具支持的 Spine 4.3 JSON。
    /// </summary>
    private static void EnsureSpine43(JObject spineRoot)
    {
        string version = spineRoot["skeleton"]?.Value<string>("spine");
        if (string.IsNullOrEmpty(version) || !version.StartsWith("4.3", StringComparison.Ordinal))
        {
            throw new InvalidDataException($"当前工具只支持 Spine 4.3 JSON，输入版本为 {version ?? "缺失"}。");
        }
    }

    /// <summary>
    /// 按 Cocos 双轴角完整重建 Spine setup pose，保留旋转中编码的反射。
    /// </summary>
    /// <summary>
    /// 按 sceneext Bone::applyParentTransform 建立 setup 世界变换和 Spine 局部变换。
    /// </summary>
    private static SetupTransformData BuildSetupTransformData(SourceData sourceData)
    {
        Dictionary<string, CocosWorldTransform> worldTransformMap = new Dictionary<string, CocosWorldTransform>(StringComparer.Ordinal);
        Dictionary<string, SpineLocalTransform> spineLocalTransformMap = new Dictionary<string, SpineLocalTransform>(StringComparer.Ordinal);
        Dictionary<string, double> rotationDirectionMap = new Dictionary<string, double>(StringComparer.Ordinal);
        HashSet<string> visiting = new HashSet<string>(StringComparer.Ordinal);

        // 递归建立 Cocos setup 世界变换，允许源骨骼不是严格父级优先排列。
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            string boneName = sourceData.SourceBoneOrder[boneIndex];
            BuildCocosSetupWorldTransform(boneName, sourceData.SourceBoneMap, worldTransformMap, visiting);
        }

        // 将目标世界矩阵反解为 Spine Normal 继承需要的局部矩阵。
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            string boneName = sourceData.SourceBoneOrder[boneIndex];
            JObject sourceBone = sourceData.SourceBoneMap[boneName];
            string parentName = sourceBone.Value<string>("parent");
            CocosWorldTransform worldTransform = worldTransformMap[boneName];
            Matrix2D localMatrix = string.IsNullOrEmpty(parentName)
                ? worldTransform.Matrix
                : MultiplyInverse(worldTransformMap[parentName].Matrix, worldTransform.Matrix, boneName);
            spineLocalTransformMap.Add(boneName, DecomposeSpineLocalMatrix(localMatrix, boneName));

            double rotationDirection = 1d;
            if (!string.IsNullOrEmpty(parentName) && worldTransformMap[parentName].Matrix.Determinant < 0d)
            {
                rotationDirection = -1d;
            }

            rotationDirectionMap.Add(boneName, rotationDirection);
        }

        return new SetupTransformData(worldTransformMap, spineLocalTransformMap, rotationDirectionMap);
    }

    /// <summary>
    /// 递归计算单根 Cocos setup 骨骼的世界变换。
    /// </summary>
    private static CocosWorldTransform BuildCocosSetupWorldTransform(
        string boneName,
        Dictionary<string, JObject> sourceBoneMap,
        Dictionary<string, CocosWorldTransform> worldTransformMap,
        HashSet<string> visiting
    )
    {
        if (worldTransformMap.TryGetValue(boneName, out CocosWorldTransform existingTransform))
        {
            return existingTransform;
        }

        if (!visiting.Add(boneName))
        {
            throw new InvalidDataException($"ExportJson setup 骨骼层级存在循环: {boneName}。");
        }

        JObject sourceBone = sourceBoneMap[boneName];
        string context = $"ExportJson setup 骨骼 {boneName}";
        double x = ReadRequiredDouble(sourceBone, "x", context);
        double y = ReadRequiredDouble(sourceBone, "y", context);
        double scaleX = ReadRequiredDouble(sourceBone, "cX", context);
        double scaleY = ReadRequiredDouble(sourceBone, "cY", context);
        double skewX = ReadRequiredDouble(sourceBone, "kX", context);
        double skewY = ReadRequiredDouble(sourceBone, "kY", context);
        string parentName = sourceBone.Value<string>("parent");

        // 精确复现 Bone::applyParentTransform 的位置矩阵和分量式继承。
        if (!string.IsNullOrEmpty(parentName))
        {
            if (!sourceBoneMap.ContainsKey(parentName))
            {
                throw new InvalidDataException($"ExportJson setup 骨骼 {boneName} 的父骨骼不存在: {parentName}。");
            }

            CocosWorldTransform parentTransform = BuildCocosSetupWorldTransform(parentName, sourceBoneMap, worldTransformMap, visiting);
            double localX = x;
            double localY = y;
            x = localX * parentTransform.Matrix.A + localY * parentTransform.Matrix.B + parentTransform.X;
            y = localX * parentTransform.Matrix.C + localY * parentTransform.Matrix.D + parentTransform.Y;
            scaleX *= parentTransform.ScaleX;
            scaleY *= parentTransform.ScaleY;
            skewX += parentTransform.SkewX;
            skewY += parentTransform.SkewY;
        }

        CocosWorldTransform result = new CocosWorldTransform(x, y, scaleX, scaleY, skewX, skewY, CreateCocosMatrix(scaleX, scaleY, skewX, skewY));
        worldTransformMap.Add(boneName, result);
        visiting.Remove(boneName);
        return result;
    }

    /// <summary>
    /// 按 TransformHelp::nodeToMatrix 创建 Spine 坐标布局的二维矩阵。
    /// </summary>
    private static Matrix2D CreateCocosMatrix(double scaleX, double scaleY, double skewX, double skewY)
    {
        return new Matrix2D(scaleX * Math.Cos(skewY), scaleY * Math.Sin(skewX), scaleX * Math.Sin(skewY), scaleY * Math.Cos(skewX));
    }

    /// <summary>
    /// 计算 inverse(parent) * world，得到 Spine Normal 继承下的局部矩阵。
    /// </summary>
    private static Matrix2D MultiplyInverse(Matrix2D parent, Matrix2D world, string boneName)
    {
        double determinant = parent.Determinant;
        if (Math.Abs(determinant) <= ValueTolerance)
        {
            throw new InvalidDataException($"ExportJson setup 骨骼 {boneName} 的父矩阵不可逆。");
        }

        return new Matrix2D(
            (parent.D * world.A - parent.B * world.C) / determinant,
            (parent.D * world.B - parent.B * world.D) / determinant,
            (-parent.C * world.A + parent.A * world.C) / determinant,
            (-parent.C * world.B + parent.A * world.D) / determinant
        );
    }

    /// <summary>
    /// 将任意可逆二维矩阵分解为 Spine rotation、signed scale 和 shearY。
    /// </summary>
    private static SpineLocalTransform DecomposeSpineLocalMatrix(Matrix2D matrix, string boneName)
    {
        double scaleX = Math.Sqrt(matrix.A * matrix.A + matrix.C * matrix.C);
        double scaleYLength = Math.Sqrt(matrix.B * matrix.B + matrix.D * matrix.D);
        if (scaleX <= ValueTolerance || scaleYLength <= ValueTolerance)
        {
            throw new InvalidDataException($"ExportJson setup 骨骼 {boneName} 含不可分解的零缩放矩阵。");
        }

        double rotation = Math.Atan2(matrix.C, matrix.A) * RadiansToDegrees;
        double scaleY = matrix.Determinant < 0d ? -scaleYLength : scaleYLength;
        double yAxisRotation = Math.Atan2(matrix.D / scaleY, matrix.B / scaleY) * RadiansToDegrees;
        double shearY = NormalizeDegrees(yAxisRotation - rotation - 90d);
        return new SpineLocalTransform(NormalizeDegrees(rotation), scaleX, scaleY, shearY);
    }

    /// <summary>
    /// 将角度归一到 (-180, 180]。
    /// </summary>
    private static double NormalizeDegrees(double value)
    {
        value %= 360d;
        if (value <= -180d)
        {
            value += 360d;
        }
        else if (value > 180d)
        {
            value -= 360d;
        }

        return value;
    }

    /// <summary>
    /// 写入按 Cocos 世界矩阵反解后的 Spine setup 局部变换。
    /// </summary>
    private static void RepairSetupPose(SourceData sourceData, SetupTransformData setupTransformData, JObject spineRoot, RepairResult result)
    {
        Dictionary<string, JObject> spineBoneMap = BuildNamedObjectMap(spineRoot["bones"] as JArray, "Spine setup 骨骼");

        // 位置保持 Cocos 局部坐标；轴矩阵使用层级反解结果。
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            string boneName = sourceData.SourceBoneOrder[boneIndex];
            JObject sourceBone = sourceData.SourceBoneMap[boneName];
            if (!spineBoneMap.TryGetValue(boneName, out JObject spineBone))
            {
                throw new InvalidDataException($"Spine setup pose 缺少 Cocos 同名骨骼: {boneName}。");
            }

            SpineLocalTransform transform = setupTransformData.SpineLocalTransformMap[boneName];
            WriteOptionalDouble(spineBone, "x", ReadRequiredDouble(sourceBone, "x", $"setup 骨骼 {boneName}"), 0d);
            WriteOptionalDouble(spineBone, "y", ReadRequiredDouble(sourceBone, "y", $"setup 骨骼 {boneName}"), 0d);
            WriteOptionalDouble(spineBone, "rotation", transform.Rotation, 0d);
            WriteOptionalDouble(spineBone, "scaleX", transform.ScaleX, 1d);
            WriteOptionalDouble(spineBone, "scaleY", transform.ScaleY, 1d);
            spineBone.Remove("shearX");
            WriteOptionalDouble(spineBone, "shearY", transform.ShearY, 0d);
            result.RepairedSetupBoneCount++;
        }

        // 按 setup dI 修复 Spine attachment；任意负索引均表示隐藏。
        Dictionary<string, JObject> spineSlotMap = BuildNamedObjectMap(spineRoot["slots"] as JArray, "Spine setup 插槽");
        foreach (KeyValuePair<string, List<string>> displayPair in sourceData.DisplayNameMap)
        {
            if (displayPair.Value.Count == 0)
            {
                continue;
            }

            if (!spineSlotMap.TryGetValue(displayPair.Key, out JObject spineSlot))
            {
                throw new InvalidDataException($"Spine setup pose 缺少 Cocos 同名插槽: {displayPair.Key}。");
            }

            int displayIndex = ReadNormalizedDisplayIndex(
                sourceData.SourceBoneMap[displayPair.Key],
                displayPair.Value.Count,
                $"setup 骨骼 {displayPair.Key}"
            );
            if (displayIndex < 0)
            {
                spineSlot.Remove("attachment");
            }
            else
            {
                spineSlot["attachment"] = displayPair.Value[displayIndex];
            }

            result.RepairedSetupSlotCount++;
        }
    }

    /// <summary>
    /// 逐骨骼比较 Cocos 与 Spine 局部仿射矩阵，验证旋转、反射、缩放和位移均未丢失。
    /// </summary>
    private static void ValidateRepairedSetupPose(SourceData sourceData, SetupTransformData setupTransformData, JObject spineRoot)
    {
        Dictionary<string, JObject> spineBoneMap = BuildNamedObjectMap(spineRoot["bones"] as JArray, "Spine setup 骨骼");
        Dictionary<string, Matrix2D> spineWorldMatrixMap = new Dictionary<string, Matrix2D>(StringComparer.Ordinal);
        Dictionary<string, Vector2D> spineWorldPositionMap = new Dictionary<string, Vector2D>(StringComparer.Ordinal);

        // 按 Spine Normal 继承重新计算世界矩阵和位置。
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            string boneName = sourceData.SourceBoneOrder[boneIndex];
            JObject sourceBone = sourceData.SourceBoneMap[boneName];
            JObject spineBone = spineBoneMap[boneName];
            string parentName = sourceBone.Value<string>("parent");
            double x = spineBone.Value<double?>("x") ?? 0d;
            double y = spineBone.Value<double?>("y") ?? 0d;
            double rotation = spineBone.Value<double?>("rotation") ?? 0d;
            double scaleX = spineBone.Value<double?>("scaleX") ?? 1d;
            double scaleY = spineBone.Value<double?>("scaleY") ?? 1d;
            double shearX = spineBone.Value<double?>("shearX") ?? 0d;
            double shearY = spineBone.Value<double?>("shearY") ?? 0d;
            double xAxis = (rotation + shearX) / RadiansToDegrees;
            double yAxis = (rotation + 90d + shearY) / RadiansToDegrees;
            Matrix2D localMatrix = new Matrix2D(Math.Cos(xAxis) * scaleX, Math.Cos(yAxis) * scaleY, Math.Sin(xAxis) * scaleX, Math.Sin(yAxis) * scaleY);

            Matrix2D worldMatrix = localMatrix;
            Vector2D worldPosition = new Vector2D(x, y);
            if (!string.IsNullOrEmpty(parentName))
            {
                Matrix2D parentMatrix = spineWorldMatrixMap[parentName];
                Vector2D parentPosition = spineWorldPositionMap[parentName];
                worldMatrix = Matrix2D.Multiply(parentMatrix, localMatrix);
                worldPosition = new Vector2D(
                    parentMatrix.A * x + parentMatrix.B * y + parentPosition.X,
                    parentMatrix.C * x + parentMatrix.D * y + parentPosition.Y
                );
            }

            spineWorldMatrixMap.Add(boneName, worldMatrix);
            spineWorldPositionMap.Add(boneName, worldPosition);

            CocosWorldTransform expected = setupTransformData.WorldTransformMap[boneName];
            if (
                Math.Abs(expected.X - worldPosition.X) > MatrixTolerance
                || Math.Abs(expected.Y - worldPosition.Y) > MatrixTolerance
                || Math.Abs(expected.Matrix.A - worldMatrix.A) > MatrixTolerance
                || Math.Abs(expected.Matrix.B - worldMatrix.B) > MatrixTolerance
                || Math.Abs(expected.Matrix.C - worldMatrix.C) > MatrixTolerance
                || Math.Abs(expected.Matrix.D - worldMatrix.D) > MatrixTolerance
            )
            {
                throw new InvalidDataException($"Spine setup 骨骼 {boneName} 未保持 Cocos sceneext 世界变换。");
            }
        }
    }

    /// <summary>
    /// 校验两侧 setup 和动画集合，并按每个 Cocos movement 重建动画数据。
    /// </summary>
    private static void RepairAnimations(SourceData sourceData, SetupTransformData setupTransformData, JObject spineRoot, RepairResult result)
    {
        JObject spineAnimations = spineRoot["animations"] as JObject;
        if (spineAnimations == null)
        {
            throw new InvalidDataException("Spine 文件缺少 animations。");
        }

        // 校验 setup 骨骼、slot 和 attachment 映射。
        Dictionary<string, JObject> spineBoneMap = BuildNamedObjectMap(spineRoot["bones"] as JArray, "Spine setup 骨骼");
        Dictionary<string, JObject> spineSlotMap = BuildNamedObjectMap(spineRoot["slots"] as JArray, "Spine setup 插槽");
        foreach (KeyValuePair<string, JObject> sourceBonePair in sourceData.SourceBoneMap)
        {
            if (!spineBoneMap.ContainsKey(sourceBonePair.Key))
            {
                throw new InvalidDataException($"Spine setup pose 缺少 Cocos 同名骨骼: {sourceBonePair.Key}。");
            }

            List<string> displayNames = sourceData.DisplayNameMap[sourceBonePair.Key];
            if (displayNames.Count == 0)
            {
                continue;
            }

            if (!spineSlotMap.TryGetValue(sourceBonePair.Key, out JObject spineSlot))
            {
                throw new InvalidDataException($"Spine setup pose 缺少 Cocos 同名插槽: {sourceBonePair.Key}。");
            }

            int sourceDisplayIndex = ReadNormalizedDisplayIndex(
                sourceBonePair.Value,
                displayNames.Count,
                $"setup 骨骼 {sourceBonePair.Key}"
            );
            string expectedAttachment = sourceDisplayIndex < 0 ? null : displayNames[sourceDisplayIndex];
            string setupAttachment = spineSlot.Value<string>("attachment");
            if (!string.Equals(setupAttachment, expectedAttachment, StringComparison.Ordinal))
            {
                throw new InvalidDataException(
                    $"Spine setup attachment 与 Cocos dI 不一致: {sourceBonePair.Key}，"
                        + $"{setupAttachment ?? "隐藏"} / {expectedAttachment ?? "隐藏"}。"
                );
            }
        }

        // 要求 movement 与 Spine animation 集合完全一致。
        foreach (JProperty spineAnimationProperty in spineAnimations.Properties())
        {
            if (!sourceData.MovementMap.ContainsKey(spineAnimationProperty.Name))
            {
                throw new InvalidDataException($"ExportJson 缺少 Spine 同名 movement: {spineAnimationProperty.Name}。");
            }
        }

        if (sourceData.MovementMap.Count != spineAnimations.Count)
        {
            throw new InvalidDataException($"ExportJson movement 与 Spine animation 数量不一致: {sourceData.MovementMap.Count} / {spineAnimations.Count}。");
        }

        // 逐 movement 重建骨骼、附件、颜色和绘制顺序轨道。
        List<string> setupSlotOrder = CollectNamedOrder(spineRoot["slots"] as JArray, "Spine setup 插槽");
        foreach (KeyValuePair<string, JObject> movementPair in sourceData.MovementMap)
        {
            JObject spineAnimation = spineAnimations[movementPair.Key] as JObject;
            if (spineAnimation == null)
            {
                throw new InvalidDataException($"Spine 动画 {movementPair.Key} 不是有效对象。");
            }

            MovementData movementData = ReadMovementData(
                movementPair.Key,
                movementPair.Value,
                sourceData.SourceBoneMap,
                sourceData.SourceBoneOrder,
                setupSlotOrder,
                setupTransformData.RotationDirectionMap
            );
            RepairBoneTimelines(sourceData, setupTransformData, movementData, spineAnimation, result);
            RepairSlotTimelines(sourceData, movementData, spineAnimation, result);
            RepairDrawOrderTimeline(sourceData, movementData, setupSlotOrder, spineAnimation, result);
        }
    }

    /// <summary>
    /// 读取 movement 播放速度、时长和骨骼帧，并拒绝无法无损映射的运行时功能。
    /// </summary>
    private static MovementData ReadMovementData(
        string movementName,
        JObject movement,
        Dictionary<string, JObject> sourceBoneMap,
        List<string> sourceBoneOrder,
        List<string> setupSlotOrder,
        Dictionary<string, double> rotationDirectionMap
    )
    {
        int duration = ReadRequiredInt(movement, "dr", $"movement {movementName}");
        double movementScale = ReadRequiredDouble(movement, "sc", $"movement {movementName}");
        if (duration < 0 || !IsFinite(movementScale) || movementScale <= 0d)
        {
            throw new InvalidDataException($"ExportJson movement {movementName} 的 dr 或 sc 无效。");
        }

        EnsureZeroInt(movement, "to", $"movement {movementName}");
        EnsureZeroInt(movement, "drTW", $"movement {movementName}");
        EnsureZeroInt(movement, "twE", $"movement {movementName}");
        double secondsPerFrame = 1d / (CocosRuntimeFrameRate * movementScale);

        JArray movementBones = movement["mov_bone_data"] as JArray;
        if (movementBones == null)
        {
            throw new InvalidDataException($"ExportJson movement {movementName} 缺少 mov_bone_data。");
        }

        Dictionary<string, JObject> movementBoneMap = BuildNamedObjectMap(movementBones, $"ExportJson movement {movementName} 骨骼");
        List<string> movementBoneOrder = CollectNamedOrder(movementBones, $"ExportJson movement {movementName} 骨骼");

        // 校验骨骼引用和动态 z-order 语义。
        ValidateMovementZOrder(movementName, sourceBoneMap, movementBoneMap, setupSlotOrder);

        // 校验全部关键帧字段。
        foreach (KeyValuePair<string, JObject> movementBonePair in movementBoneMap)
        {
            if (!sourceBoneMap.ContainsKey(movementBonePair.Key))
            {
                throw new InvalidDataException($"ExportJson movement {movementName} 引用了 setup 中不存在的骨骼: {movementBonePair.Key}。");
            }

            double delay = movementBonePair.Value.Value<double?>("dl") ?? 0d;
            if (!IsFinite(delay) || Math.Abs(delay) > ValueTolerance)
            {
                throw new InvalidDataException($"ExportJson movement {movementName}.{movementBonePair.Key} 使用未支持的 delay={delay}。");
            }

            JArray frames = movementBonePair.Value["frame_data"] as JArray;
            if (frames == null || frames.Count == 0)
            {
                throw new InvalidDataException($"ExportJson movement {movementName}.{movementBonePair.Key} 缺少 frame_data。");
            }

            int previousFrameIndex = -1;
            for (int frameIndex = 0; frameIndex < frames.Count; frameIndex++)
            {
                JObject frame = RequireFrame(frames, movementName, movementBonePair.Key, frameIndex);
                int sourceFrameIndex = ReadRequiredInt(frame, "fi", $"movement {movementName}.{movementBonePair.Key}.frame_data[{frameIndex}]");
                if (sourceFrameIndex <= previousFrameIndex || sourceFrameIndex > duration)
                {
                    throw new InvalidDataException(
                        $"ExportJson {movementName}.{movementBonePair.Key}.frame_data[{frameIndex}] fi 无效: "
                            + $"{sourceFrameIndex}，前一帧 {previousFrameIndex}，movement dr={duration}。"
                    );
                }

                ValidateFrame(movementName, movementBonePair.Key, frameIndex, frame);
                previousFrameIndex = sourceFrameIndex;
            }
        }

        return new MovementData(
            movementName,
            duration,
            secondsPerFrame,
            sourceBoneMap,
            sourceBoneOrder,
            movementBoneMap,
            movementBoneOrder,
            rotationDirectionMap
        );
    }

    /// <summary>
    /// 按 Cocos setup z 与帧 z 验证 movement 全部关键时刻的稳定绘制顺序。
    /// </summary>
    private static void ValidateMovementZOrder(
        string movementName,
        Dictionary<string, JObject> sourceBoneMap,
        Dictionary<string, JObject> movementBoneMap,
        List<string> setupSlotOrder
    )
    {
        // Spine slot 必须存在同名 Cocos setup 骨骼，才能计算动态绘制顺序。
        for (int slotIndex = 0; slotIndex < setupSlotOrder.Count; slotIndex++)
        {
            string slotName = setupSlotOrder[slotIndex];
            if (!sourceBoneMap.ContainsKey(slotName))
            {
                throw new InvalidDataException($"ExportJson setup 骨骼缺少 Spine slot 同名骨骼: {slotName}。");
            }
        }

        // movement 骨骼必须来自同一 setup；具体 z 变化由 drawOrder 重建阶段处理。
        foreach (string boneName in movementBoneMap.Keys)
        {
            if (!sourceBoneMap.ContainsKey(boneName))
            {
                throw new InvalidDataException($"ExportJson movement {movementName} 引用了 setup 中不存在的骨骼: {boneName}。");
            }
        }
    }

    /// <summary>
    /// 比较 Cocos 绘制顺序；z 相同时保留 Spine setup slot 次序。
    /// </summary>
    /// <summary>
    /// 读取指定时刻 setup z 与最新帧 z 的和。
    /// </summary>
    /// <summary>
    /// 校验单个 Cocos 帧的变换、显示、颜色和补间字段。
    /// </summary>
    private static void ValidateFrame(string movementName, string boneName, int frameIndex, JObject frame)
    {
        string context = $"movement {movementName}.{boneName}.frame_data[{frameIndex}]";
        double x = ReadRequiredDouble(frame, "x", context);
        double y = ReadRequiredDouble(frame, "y", context);
        double scaleX = ReadRequiredDouble(frame, "cX", context);
        double scaleY = ReadRequiredDouble(frame, "cY", context);
        double skewX = ReadRequiredDouble(frame, "kX", context);
        double skewY = ReadRequiredDouble(frame, "kY", context);
        ReadRequiredInt(frame, "dI", context);
        ReadRequiredInt(frame, "z", context);
        int tweenEasing = ReadRequiredInt(frame, "twE", context);
        ReadRequiredBoolean(frame, "tweenFrame", context);

        if (!IsFinite(x) || !IsFinite(y) || !IsFinite(scaleX) || !IsFinite(scaleY) || !IsFinite(skewX) || !IsFinite(skewY))
        {
            throw new InvalidDataException($"ExportJson {context} 含无效数值。");
        }

        EnsureZeroInt(frame, "twR", context);
        ValidateEasingParameters(frame, tweenEasing, context);
        EnsureEmptyProperty(frame, "evt", context);
        EnsureEmptyProperty(frame, "sd", context);
        EnsureEmptyProperty(frame, "sdE", context);
        EnsureEmptyProperty(frame, "mov", context);

        JObject color = frame["color"] as JObject;
        if (frame["color"] != null && color == null)
        {
            throw new InvalidDataException($"ExportJson {context}.color 不是有效对象。");
        }

        if (color != null)
        {
            ReadColorByte(color, "r", context);
            ReadColorByte(color, "g", context);
            ReadColorByte(color, "b", context);
            ReadColorByte(color, "a", context);
        }
    }

    /// <summary>
    /// 校验自定义缓动参数，并拒绝普通缓动携带无效参数。
    /// </summary>
    private static void ValidateEasingParameters(JObject frame, int tweenEasing, string context)
    {
        if (tweenEasing != -1)
        {
            EnsureEmptyProperty(frame, "twEP", context);
            return;
        }

        JArray parameters = RequireCustomEasingParameters(frame, context);
        if (Math.Abs(parameters[1].Value<double>()) > ValueTolerance || Math.Abs(parameters[7].Value<double>() - 1d) > ValueTolerance)
        {
            throw new InvalidDataException($"ExportJson {context}.twEP 必须从进度 0 连续过渡到 1。");
        }
    }

    /// <summary>
    /// 读取 Cocos CUSTOM_EASING 使用的 4 个二维 Bezier 点。
    /// </summary>
    private static JArray RequireCustomEasingParameters(JObject frame, string context)
    {
        JArray parameters = frame["twEP"] as JArray;
        if (parameters == null || parameters.Count != 8)
        {
            throw new InvalidDataException($"ExportJson {context}.twEP 必须包含 8 个数值。");
        }

        for (int parameterIndex = 0; parameterIndex < parameters.Count; parameterIndex++)
        {
            double value = parameters[parameterIndex].Value<double>();
            if (!IsFinite(value))
            {
                throw new InvalidDataException($"ExportJson {context}.twEP[{parameterIndex}] 数值无效。");
            }
        }

        return parameters;
    }

    /// <summary>
    /// 删除无源语义的变换轨道，并按 Cocos 关键帧重建位移、旋转、缩放和剪切。
    /// </summary>
    private static void RepairBoneTimelines(
        SourceData sourceData,
        SetupTransformData setupTransformData,
        MovementData movementData,
        JObject spineAnimation,
        RepairResult result
    )
    {
        JObject spineBones = spineAnimation["bones"] as JObject;
        Dictionary<string, BakedTransformTimelines> bakedTimelineMap = BuildBakedTransformTimelines(sourceData, setupTransformData, movementData);

        // 先删除无源通道，以及将由逐帧反解完全替换的旧轨道。
        if (spineBones != null)
        {
            List<JProperty> emptyBoneProperties = new List<JProperty>();
            foreach (JProperty boneProperty in spineBones.Properties())
            {
                JObject targetTimelines = boneProperty.Value as JObject;
                if (bakedTimelineMap.ContainsKey(boneProperty.Name))
                {
                    RemoveBoneTimeline(targetTimelines, TimelineKind.Translate, result);
                    RemoveBoneTimeline(targetTimelines, TimelineKind.Rotate, result);
                    RemoveBoneTimeline(targetTimelines, TimelineKind.Scale, result);
                    RemoveBoneTimeline(targetTimelines, TimelineKind.Shear, result);
                }
                else
                {
                    movementData.MovementBoneMap.TryGetValue(boneProperty.Name, out JObject movementBone);
                    JArray frames = movementBone?["frame_data"] as JArray;
                    RemoveUnexpectedBoneTimeline(targetTimelines, frames, TimelineKind.Translate, result);
                    RemoveUnexpectedBoneTimeline(targetTimelines, frames, TimelineKind.Rotate, result);
                    RemoveUnexpectedBoneTimeline(targetTimelines, frames, TimelineKind.Scale, result);
                    RemoveUnexpectedBoneTimeline(targetTimelines, frames, TimelineKind.Shear, result);
                }

                if (targetTimelines != null && !targetTimelines.HasValues)
                {
                    emptyBoneProperties.Add(boneProperty);
                }
            }

            for (int propertyIndex = 0; propertyIndex < emptyBoneProperties.Count; propertyIndex++)
            {
                emptyBoneProperties[propertyIndex].Remove();
            }
        }

        // 普通骨骼继续按源关键帧和原始缓动重建轨道。
        foreach (KeyValuePair<string, JObject> movementBonePair in movementData.MovementBoneMap)
        {
            if (bakedTimelineMap.ContainsKey(movementBonePair.Key))
            {
                continue;
            }

            JArray frames = movementBonePair.Value["frame_data"] as JArray;
            bool hasTranslate = HasNonDefaultTimeline(frames, TimelineKind.Translate);
            bool hasRotate = HasNonDefaultTimeline(frames, TimelineKind.Rotate);
            bool hasScale = HasNonDefaultTimeline(frames, TimelineKind.Scale);
            bool hasShear = HasNonDefaultTimeline(frames, TimelineKind.Shear);
            if (!hasTranslate && !hasRotate && !hasScale && !hasShear)
            {
                continue;
            }

            if (spineBones == null)
            {
                spineBones = new JObject();
                spineAnimation["bones"] = spineBones;
            }

            JObject targetTimelines = GetOrCreateObject(spineBones, movementBonePair.Key);
            WriteRebuiltTimeline(targetTimelines, hasTranslate ? BuildTimeline(movementData, movementBonePair.Key, frames, TimelineKind.Translate) : null, TimelineKind.Translate, result);
            WriteRebuiltTimeline(targetTimelines, hasRotate ? BuildTimeline(movementData, movementBonePair.Key, frames, TimelineKind.Rotate) : null, TimelineKind.Rotate, result);
            WriteRebuiltTimeline(targetTimelines, hasScale ? BuildTimeline(movementData, movementBonePair.Key, frames, TimelineKind.Scale) : null, TimelineKind.Scale, result);
            WriteRebuiltTimeline(targetTimelines, hasShear ? BuildTimeline(movementData, movementBonePair.Key, frames, TimelineKind.Shear) : null, TimelineKind.Shear, result);
        }

        // 受动态非等比缩放影响的后代使用逐帧世界矩阵反解结果。
        foreach (KeyValuePair<string, BakedTransformTimelines> bakedPair in bakedTimelineMap)
        {
            if (spineBones == null)
            {
                spineBones = new JObject();
                spineAnimation["bones"] = spineBones;
            }

            JObject targetTimelines = GetOrCreateObject(spineBones, bakedPair.Key);
            WriteRebuiltTimeline(targetTimelines, bakedPair.Value.Translate, TimelineKind.Translate, result);
            WriteRebuiltTimeline(targetTimelines, bakedPair.Value.Rotate, TimelineKind.Rotate, result);
            WriteRebuiltTimeline(targetTimelines, bakedPair.Value.Scale, TimelineKind.Scale, result);
            WriteRebuiltTimeline(targetTimelines, bakedPair.Value.Shear, TimelineKind.Shear, result);
            if (!targetTimelines.HasValues)
            {
                targetTimelines.Parent?.Remove();
            }
        }
    }

    /// <summary>
    /// 写入重建轨道并更新对应统计。
    /// </summary>
    private static void WriteRebuiltTimeline(JObject targetTimelines, JArray timeline, TimelineKind timelineKind, RepairResult result)
    {
        if (timeline == null)
        {
            return;
        }

        targetTimelines[GetTimelineName(timelineKind)] = timeline;
        if (timelineKind == TimelineKind.Translate)
        {
            result.RebuiltTranslateTimelineCount++;
        }
        else if (timelineKind == TimelineKind.Rotate)
        {
            result.RebuiltRotateTimelineCount++;
        }
        else if (timelineKind == TimelineKind.Scale)
        {
            result.RebuiltScaleTimelineCount++;
        }
        else if (timelineKind == TimelineKind.Shear)
        {
            result.RebuiltShearTimelineCount++;
        }
    }

    /// <summary>
    /// 找出动态非等比缩放后代，并按每个 Cocos 帧反解 Spine 局部变换轨道。
    /// </summary>
    private static Dictionary<string, BakedTransformTimelines> BuildBakedTransformTimelines(
        SourceData sourceData,
        SetupTransformData setupTransformData,
        MovementData movementData
    )
    {
        HashSet<string> nonUniformScaleBoneNames = CollectNonUniformScaleBoneNames(movementData);
        HashSet<string> bakedBoneNames = CollectDescendantsOfBones(sourceData, nonUniformScaleBoneNames);
        Dictionary<string, BakedTransformTimelines> result = new Dictionary<string, BakedTransformTimelines>(StringComparer.Ordinal);
        if (bakedBoneNames.Count == 0)
        {
            return result;
        }

        // 按 setup 顺序建立稳定输出和角度连续状态。
        Dictionary<string, double> previousRotationMap = new Dictionary<string, double>(StringComparer.Ordinal);
        Dictionary<string, double> previousShearMap = new Dictionary<string, double>(StringComparer.Ordinal);
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            string boneName = sourceData.SourceBoneOrder[boneIndex];
            if (bakedBoneNames.Contains(boneName))
            {
                result.Add(boneName, new BakedTransformTimelines());
            }
        }

        // 逐 Cocos 动画帧计算完整世界变换，再相对父世界矩阵反解 Spine 局部值。
        for (int sourceFrameIndex = 0; sourceFrameIndex <= movementData.Duration; sourceFrameIndex++)
        {
            Dictionary<string, CocosWorldTransform> worldTransformMap = BuildAnimatedCocosWorldTransforms(sourceData, movementData, sourceFrameIndex);
            foreach (KeyValuePair<string, BakedTransformTimelines> bakedPair in result)
            {
                string boneName = bakedPair.Key;
                JObject sourceBone = sourceData.SourceBoneMap[boneName];
                string parentName = sourceBone.Value<string>("parent");
                CocosWorldTransform worldTransform = worldTransformMap[boneName];
                Matrix2D localMatrix;
                Vector2D localPosition;
                if (string.IsNullOrEmpty(parentName))
                {
                    localMatrix = worldTransform.Matrix;
                    localPosition = new Vector2D(worldTransform.X, worldTransform.Y);
                }
                else
                {
                    CocosWorldTransform parentTransform = worldTransformMap[parentName];
                    localMatrix = MultiplyInverse(parentTransform.Matrix, worldTransform.Matrix, boneName);
                    localPosition = TransformPointByInverseParent(parentTransform, worldTransform, boneName);
                }

                SpineLocalTransform localTransform = DecomposeSpineLocalMatrix(localMatrix, boneName);
                SpineLocalTransform setupTransform = setupTransformData.SpineLocalTransformMap[boneName];
                double translateX = localPosition.X - ReadRequiredDouble(sourceBone, "x", $"setup 骨骼 {boneName}");
                double translateY = localPosition.Y - ReadRequiredDouble(sourceBone, "y", $"setup 骨骼 {boneName}");
                double rotation = NormalizeDegrees(localTransform.Rotation - setupTransform.Rotation);
                double shearY = NormalizeDegrees(localTransform.ShearY - setupTransform.ShearY);
                if (previousRotationMap.TryGetValue(boneName, out double previousRotation))
                {
                    rotation = UnwrapDegrees(rotation, previousRotation);
                }

                if (previousShearMap.TryGetValue(boneName, out double previousShear))
                {
                    shearY = UnwrapDegrees(shearY, previousShear);
                }

                previousRotationMap[boneName] = rotation;
                previousShearMap[boneName] = shearY;
                double scaleX = localTransform.ScaleX / setupTransform.ScaleX;
                double scaleY = localTransform.ScaleY / setupTransform.ScaleY;
                ValidateBakedLocalTransform(
                    movementData.Name,
                    boneName,
                    sourceFrameIndex,
                    localMatrix,
                    setupTransform,
                    rotation,
                    scaleX,
                    scaleY,
                    shearY
                );
                double time = sourceFrameIndex * movementData.SecondsPerFrame;
                bakedPair.Value.AddFrame(time, translateX, translateY, rotation, scaleX, scaleY, shearY);
            }
        }

        return result;
    }

    /// <summary>
    /// 校验烘焙后的 Spine setup 加动画值能重建目标局部轴矩阵。
    /// </summary>
    private static void ValidateBakedLocalTransform(
        string movementName,
        string boneName,
        int sourceFrameIndex,
        Matrix2D expectedMatrix,
        SpineLocalTransform setupTransform,
        double rotation,
        double scaleX,
        double scaleY,
        double shearY
    )
    {
        double xAxis = (setupTransform.Rotation + rotation) / RadiansToDegrees;
        double yAxis = (setupTransform.Rotation + rotation + 90d + setupTransform.ShearY + shearY) / RadiansToDegrees;
        Matrix2D actualMatrix = new Matrix2D(
            Math.Cos(xAxis) * setupTransform.ScaleX * scaleX,
            Math.Cos(yAxis) * setupTransform.ScaleY * scaleY,
            Math.Sin(xAxis) * setupTransform.ScaleX * scaleX,
            Math.Sin(yAxis) * setupTransform.ScaleY * scaleY
        );
        if (
            Math.Abs(expectedMatrix.A - actualMatrix.A) > MatrixTolerance
            || Math.Abs(expectedMatrix.B - actualMatrix.B) > MatrixTolerance
            || Math.Abs(expectedMatrix.C - actualMatrix.C) > MatrixTolerance
            || Math.Abs(expectedMatrix.D - actualMatrix.D) > MatrixTolerance
        )
        {
            throw new InvalidDataException(
                $"ExportJson movement {movementName}.{boneName} 第 {sourceFrameIndex} 帧层级矩阵反解校验失败。"
            );
        }
    }

    /// <summary>
    /// 收集 movement 中相对 setup 比例为非等比缩放的骨骼。
    /// </summary>
    private static HashSet<string> CollectNonUniformScaleBoneNames(MovementData movementData)
    {
        HashSet<string> result = new HashSet<string>(StringComparer.Ordinal);
        foreach (KeyValuePair<string, JObject> movementBonePair in movementData.MovementBoneMap)
        {
            JObject sourceBone = movementData.SourceBoneMap[movementBonePair.Key];
            JArray frames = movementBonePair.Value["frame_data"] as JArray;
            for (int frameIndex = 0; frameIndex < frames.Count; frameIndex++)
            {
                JObject frame = RequireFrame(frames, movementData.Name, movementBonePair.Key, frameIndex);
                double[] scaleRatios = ReadScaleTimelineValues(
                    sourceBone,
                    frame,
                    $"movement {movementData.Name}.{movementBonePair.Key}.frame_data[{frameIndex}]"
                );
                if (Math.Abs(scaleRatios[0] - scaleRatios[1]) > ValueTolerance)
                {
                    result.Add(movementBonePair.Key);
                    break;
                }
            }
        }

        return result;
    }

    /// <summary>
    /// 收集指定骨骼集合的全部后代，不包含非等比缩放骨骼自身。
    /// </summary>
    private static HashSet<string> CollectDescendantsOfBones(SourceData sourceData, HashSet<string> ancestorNames)
    {
        HashSet<string> result = new HashSet<string>(StringComparer.Ordinal);
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            string boneName = sourceData.SourceBoneOrder[boneIndex];
            string parentName = sourceData.SourceBoneMap[boneName].Value<string>("parent");
            while (!string.IsNullOrEmpty(parentName))
            {
                if (ancestorNames.Contains(parentName))
                {
                    result.Add(boneName);
                    break;
                }

                parentName = sourceData.SourceBoneMap[parentName].Value<string>("parent");
            }
        }

        return result;
    }

    /// <summary>
    /// 计算 movement 指定帧的全部 Cocos sceneext 世界变换。
    /// </summary>
    private static Dictionary<string, CocosWorldTransform> BuildAnimatedCocosWorldTransforms(
        SourceData sourceData,
        MovementData movementData,
        int sourceFrameIndex
    )
    {
        Dictionary<string, CocosWorldTransform> result = new Dictionary<string, CocosWorldTransform>(StringComparer.Ordinal);
        HashSet<string> visiting = new HashSet<string>(StringComparer.Ordinal);
        for (int boneIndex = 0; boneIndex < sourceData.SourceBoneOrder.Count; boneIndex++)
        {
            BuildAnimatedCocosWorldTransform(sourceData.SourceBoneOrder[boneIndex], sourceData, movementData, sourceFrameIndex, result, visiting);
        }

        return result;
    }

    /// <summary>
    /// 递归计算 movement 指定帧的单根 Cocos sceneext 世界变换。
    /// </summary>
    private static CocosWorldTransform BuildAnimatedCocosWorldTransform(
        string boneName,
        SourceData sourceData,
        MovementData movementData,
        int sourceFrameIndex,
        Dictionary<string, CocosWorldTransform> worldTransformMap,
        HashSet<string> visiting
    )
    {
        if (worldTransformMap.TryGetValue(boneName, out CocosWorldTransform existingTransform))
        {
            return existingTransform;
        }

        if (!visiting.Add(boneName))
        {
            throw new InvalidDataException($"ExportJson movement {movementData.Name} 骨骼层级存在循环: {boneName}。");
        }

        JObject sourceBone = sourceData.SourceBoneMap[boneName];
        CocosFrameTransform frameTransform = SampleCocosFrameTransform(movementData, boneName, sourceFrameIndex);
        double x = ReadRequiredDouble(sourceBone, "x", $"setup 骨骼 {boneName}") + frameTransform.X;
        double y = ReadRequiredDouble(sourceBone, "y", $"setup 骨骼 {boneName}") + frameTransform.Y;
        double scaleX = ReadRequiredDouble(sourceBone, "cX", $"setup 骨骼 {boneName}") + frameTransform.ScaleX - 1d;
        double scaleY = ReadRequiredDouble(sourceBone, "cY", $"setup 骨骼 {boneName}") + frameTransform.ScaleY - 1d;
        double skewX = ReadRequiredDouble(sourceBone, "kX", $"setup 骨骼 {boneName}") + frameTransform.SkewX;
        double skewY = ReadRequiredDouble(sourceBone, "kY", $"setup 骨骼 {boneName}") + frameTransform.SkewY;
        string parentName = sourceBone.Value<string>("parent");

        // 完整复现 Bone::applyParentTransform 的位置矩阵和分量式轴继承。
        if (!string.IsNullOrEmpty(parentName))
        {
            CocosWorldTransform parentTransform = BuildAnimatedCocosWorldTransform(
                parentName,
                sourceData,
                movementData,
                sourceFrameIndex,
                worldTransformMap,
                visiting
            );
            double localX = x;
            double localY = y;
            x = localX * parentTransform.Matrix.A + localY * parentTransform.Matrix.B + parentTransform.X;
            y = localX * parentTransform.Matrix.C + localY * parentTransform.Matrix.D + parentTransform.Y;
            scaleX *= parentTransform.ScaleX;
            scaleY *= parentTransform.ScaleY;
            skewX += parentTransform.SkewX;
            skewY += parentTransform.SkewY;
        }

        CocosWorldTransform result = new CocosWorldTransform(x, y, scaleX, scaleY, skewX, skewY, CreateCocosMatrix(scaleX, scaleY, skewX, skewY));
        worldTransformMap.Add(boneName, result);
        visiting.Remove(boneName);
        return result;
    }

    /// <summary>
    /// 按 BoneTweenController 插值规则读取 movement 指定帧的局部增量。
    /// </summary>
    private static CocosFrameTransform SampleCocosFrameTransform(MovementData movementData, string boneName, int sourceFrameIndex)
    {
        if (!movementData.MovementBoneMap.TryGetValue(boneName, out JObject movementBone))
        {
            return CocosFrameTransform.Identity;
        }

        JArray frames = movementBone["frame_data"] as JArray;
        JObject firstFrame = RequireFrame(frames, movementData.Name, boneName, 0);
        int firstFrameIndex = ReadRequiredInt(firstFrame, "fi", $"movement {movementData.Name}.{boneName}.frame_data[0]");
        if (sourceFrameIndex <= firstFrameIndex)
        {
            return ReadCocosFrameTransform(firstFrame, $"movement {movementData.Name}.{boneName}.frame_data[0]");
        }

        // 精确关键帧进入下一段；关键帧之后使用当前帧到下一帧的补间。
        for (int frameIndex = 0; frameIndex < frames.Count - 1; frameIndex++)
        {
            JObject fromFrame = RequireFrame(frames, movementData.Name, boneName, frameIndex);
            JObject toFrame = RequireFrame(frames, movementData.Name, boneName, frameIndex + 1);
            int fromFrameIndex = ReadRequiredInt(fromFrame, "fi", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]");
            int toFrameIndex = ReadRequiredInt(toFrame, "fi", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex + 1}]");
            if (sourceFrameIndex >= toFrameIndex)
            {
                continue;
            }

            CocosFrameTransform fromTransform = ReadCocosFrameTransform(fromFrame, $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]");
            CocosFrameTransform toTransform = ReadCocosFrameTransform(toFrame, $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex + 1}]");
            int fromDisplayIndex = ReadRequiredInt(fromFrame, "dI", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]");
            int toDisplayIndex = ReadRequiredInt(toFrame, "dI", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex + 1}]");
            if (fromDisplayIndex < 0 && toDisplayIndex >= 0)
            {
                return toTransform;
            }

            if (toDisplayIndex < 0 && fromDisplayIndex >= 0)
            {
                return fromTransform;
            }

            bool tweenFrame = ReadRequiredBoolean(fromFrame, "tweenFrame", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]");
            if (!tweenFrame)
            {
                return fromTransform;
            }

            double percent = (sourceFrameIndex - fromFrameIndex) / (double)(toFrameIndex - fromFrameIndex);
            int tweenEasing = ReadRequiredInt(fromFrame, "twE", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]");
            double easedPercent = EvaluateCocosEasing(
                tweenEasing,
                percent,
                fromFrame,
                $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]"
            );
            return CocosFrameTransform.Interpolate(fromTransform, toTransform, easedPercent);
        }

        JObject lastFrame = RequireFrame(frames, movementData.Name, boneName, frames.Count - 1);
        return ReadCocosFrameTransform(lastFrame, $"movement {movementData.Name}.{boneName}.frame_data[{frames.Count - 1}]");
    }

    /// <summary>
    /// 读取单个 Cocos movement 帧的变换分量。
    /// </summary>
    private static CocosFrameTransform ReadCocosFrameTransform(JObject frame, string context)
    {
        return new CocosFrameTransform(
            ReadRequiredDouble(frame, "x", context),
            ReadRequiredDouble(frame, "y", context),
            ReadRequiredDouble(frame, "cX", context),
            ReadRequiredDouble(frame, "cY", context),
            ReadRequiredDouble(frame, "kX", context),
            ReadRequiredDouble(frame, "kY", context)
        );
    }

    /// <summary>
    /// 将 Cocos 世界位置反解到父骨骼局部坐标。
    /// </summary>
    private static Vector2D TransformPointByInverseParent(
        CocosWorldTransform parentTransform,
        CocosWorldTransform worldTransform,
        string boneName
    )
    {
        double determinant = parentTransform.Matrix.Determinant;
        if (Math.Abs(determinant) <= ValueTolerance)
        {
            throw new InvalidDataException($"ExportJson movement 骨骼 {boneName} 的父矩阵不可逆。");
        }

        double deltaX = worldTransform.X - parentTransform.X;
        double deltaY = worldTransform.Y - parentTransform.Y;
        return new Vector2D(
            (parentTransform.Matrix.D * deltaX - parentTransform.Matrix.B * deltaY) / determinant,
            (-parentTransform.Matrix.C * deltaX + parentTransform.Matrix.A * deltaY) / determinant
        );
    }

    /// <summary>
    /// 将角度展开到最接近上一帧的等价值。
    /// </summary>
    private static double UnwrapDegrees(double value, double previousValue)
    {
        while (value - previousValue > 180d)
        {
            value -= 360d;
        }

        while (value - previousValue <= -180d)
        {
            value += 360d;
        }

        return value;
    }

    /// <summary>
    /// 按 Cocos 显示索引、缺失骨骼隐藏和颜色插值语义重建 slot 轨道。
    /// </summary>
    private static void RepairSlotTimelines(SourceData sourceData, MovementData movementData, JObject spineAnimation, RepairResult result)
    {
        JObject spineSlots = spineAnimation["slots"] as JObject;

        // 每个有 display_data 的 Cocos 骨骼对应一个同名 Spine slot。
        foreach (KeyValuePair<string, List<string>> displayPair in sourceData.DisplayNameMap)
        {
            if (displayPair.Value.Count == 0)
            {
                continue;
            }

            JObject movementBone = null;
            movementData.MovementBoneMap.TryGetValue(displayPair.Key, out movementBone);
            JArray frames = movementBone?["frame_data"] as JArray;
            JArray attachmentTimeline = BuildAttachmentTimeline(movementData, displayPair.Key, displayPair.Value, frames);
            JArray colorTimeline = HasColorTimeline(frames) ? BuildTimeline(movementData, displayPair.Key, frames, TimelineKind.Rgba) : null;

            JObject targetTimelines = spineSlots?[displayPair.Key] as JObject;
            bool needsTarget = attachmentTimeline != null || colorTimeline != null;
            if (targetTimelines == null && needsTarget)
            {
                if (spineSlots == null)
                {
                    spineSlots = new JObject();
                    spineAnimation["slots"] = spineSlots;
                }

                targetTimelines = new JObject();
                spineSlots[displayPair.Key] = targetTimelines;
            }

            // 重建离散 attachment 轨道。
            if (attachmentTimeline == null)
            {
                if (targetTimelines != null && targetTimelines.Remove("attachment"))
                {
                    result.RemovedAttachmentTimelineCount++;
                }
            }
            else
            {
                targetTimelines["attachment"] = attachmentTimeline;
                result.RebuiltAttachmentTimelineCount++;
            }

            // 重建 Spine 4.3 rgba 轨道。
            if (colorTimeline == null)
            {
                if (targetTimelines != null && targetTimelines.Remove("rgba"))
                {
                    result.RemovedColorTimelineCount++;
                }
            }
            else
            {
                targetTimelines["rgba"] = colorTimeline;
                result.RebuiltColorTimelineCount++;
            }

            if (targetTimelines != null && !targetTimelines.HasValues)
            {
                targetTimelines.Parent?.Remove();
            }
        }
    }

    /// <summary>
    /// 模拟 Cocos local z 与 order-of-arrival，重建 Spine drawOrder 轨道。
    /// </summary>
    private static void RepairDrawOrderTimeline(
        SourceData sourceData,
        MovementData movementData,
        List<string> setupSlotOrder,
        JObject spineAnimation,
        RepairResult result
    )
    {
        Dictionary<string, int> setupZMap = new Dictionary<string, int>(StringComparer.Ordinal);
        Dictionary<string, int> effectiveZMap = new Dictionary<string, int>(StringComparer.Ordinal);
        Dictionary<string, long> arrivalOrderMap = new Dictionary<string, long>(StringComparer.Ordinal);

        // 建立 setup z 和初始到达顺序。
        for (int slotIndex = 0; slotIndex < setupSlotOrder.Count; slotIndex++)
        {
            string slotName = setupSlotOrder[slotIndex];
            if (!sourceData.SourceBoneMap.TryGetValue(slotName, out JObject sourceBone))
            {
                throw new InvalidDataException($"ExportJson setup 骨骼缺少 Spine slot 同名骨骼: {slotName}。");
            }

            int setupZ = sourceBone.Value<int?>("z") ?? 0;
            setupZMap.Add(slotName, setupZ);
            effectiveZMap.Add(slotName, setupZ);
            arrivalOrderMap.Add(slotName, slotIndex);
        }

        // 收集所有可能重置或改变 z 的关键时刻。
        SortedSet<int> sourceFrameIndices = new SortedSet<int>();
        foreach (string boneName in movementData.MovementBoneOrder)
        {
            JArray frames = movementData.MovementBoneMap[boneName]["frame_data"] as JArray;
            for (int frameIndex = 0; frameIndex < frames.Count; frameIndex++)
            {
                JObject frame = RequireFrame(frames, movementData.Name, boneName, frameIndex);
                sourceFrameIndices.Add(ReadRequiredInt(frame, "fi", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]"));
            }
        }

        // Cocos 只在 effective z 真正改变时刷新 order-of-arrival。
        long nextArrivalOrder = setupSlotOrder.Count;
        List<string> previousOrder = new List<string>(setupSlotOrder);
        JArray drawOrderTimeline = new JArray();
        foreach (int sourceFrameIndex in sourceFrameIndices)
        {
            for (int boneOrderIndex = 0; boneOrderIndex < movementData.MovementBoneOrder.Count; boneOrderIndex++)
            {
                string boneName = movementData.MovementBoneOrder[boneOrderIndex];
                if (!setupZMap.ContainsKey(boneName))
                {
                    continue;
                }

                JObject frame = FindFrameAt(movementData.MovementBoneMap[boneName]["frame_data"] as JArray, sourceFrameIndex);
                if (frame == null)
                {
                    continue;
                }

                int effectiveZ = setupZMap[boneName] + (frame.Value<int?>("z") ?? 0);
                if (effectiveZ == effectiveZMap[boneName])
                {
                    continue;
                }

                effectiveZMap[boneName] = effectiveZ;
                arrivalOrderMap[boneName] = nextArrivalOrder++;
            }

            List<string> currentOrder = new List<string>(setupSlotOrder);
            currentOrder.Sort(
                (left, right) =>
                {
                    int zComparison = effectiveZMap[left].CompareTo(effectiveZMap[right]);
                    return zComparison != 0 ? zComparison : arrivalOrderMap[left].CompareTo(arrivalOrderMap[right]);
                }
            );
            if (AreOrdersEqual(previousOrder, currentOrder))
            {
                continue;
            }

            JObject drawOrderFrame = new JObject();
            WriteOptionalDouble(drawOrderFrame, "time", sourceFrameIndex * movementData.SecondsPerFrame, 0d);
            JArray offsets = new JArray();
            for (int setupIndex = 0; setupIndex < setupSlotOrder.Count; setupIndex++)
            {
                string slotName = setupSlotOrder[setupIndex];
                offsets.Add(new JObject { ["slot"] = slotName, ["offset"] = currentOrder.IndexOf(slotName) - setupIndex });
            }

            drawOrderFrame["offsets"] = offsets;
            drawOrderTimeline.Add(drawOrderFrame);
            previousOrder = currentOrder;
        }

        if (drawOrderTimeline.Count == 0)
        {
            if (spineAnimation.Remove("drawOrder"))
            {
                result.RemovedDrawOrderTimelineCount++;
            }

            return;
        }

        spineAnimation["drawOrder"] = drawOrderTimeline;
        result.RebuiltDrawOrderTimelineCount++;
    }

    /// <summary>
    /// 查找指定 Cocos 帧号的关键帧。
    /// </summary>
    private static JObject FindFrameAt(JArray frames, int sourceFrameIndex)
    {
        if (frames == null)
        {
            return null;
        }

        for (int frameIndex = 0; frameIndex < frames.Count; frameIndex++)
        {
            JObject frame = frames[frameIndex] as JObject;
            int framePosition = frame?.Value<int?>("fi") ?? int.MinValue;
            if (framePosition == sourceFrameIndex)
            {
                return frame;
            }

            if (framePosition > sourceFrameIndex)
            {
                break;
            }
        }

        return null;
    }

    /// <summary>
    /// 判断两个 slot 顺序是否完全一致。
    /// </summary>
    private static bool AreOrdersEqual(List<string> left, List<string> right)
    {
        if (left.Count != right.Count)
        {
            return false;
        }

        for (int index = 0; index < left.Count; index++)
        {
            if (!string.Equals(left[index], right[index], StringComparison.Ordinal))
            {
                return false;
            }
        }

        return true;
    }

    /// <summary>
    /// 删除没有对应源通道的目标骨骼轨道。
    /// </summary>
    private static void RemoveUnexpectedBoneTimeline(JObject targetTimelines, JArray sourceFrames, TimelineKind timelineKind, RepairResult result)
    {
        if (targetTimelines == null || HasNonDefaultTimeline(sourceFrames, timelineKind))
        {
            return;
        }

        RemoveBoneTimeline(targetTimelines, timelineKind, result);
    }

    /// <summary>
    /// 删除指定目标骨骼轨道并更新统计。
    /// </summary>
    private static void RemoveBoneTimeline(JObject targetTimelines, TimelineKind timelineKind, RepairResult result)
    {
        if (targetTimelines == null || !targetTimelines.Remove(GetTimelineName(timelineKind)))
        {
            return;
        }

        if (timelineKind == TimelineKind.Translate)
        {
            result.RemovedTranslateTimelineCount++;
        }
        else if (timelineKind == TimelineKind.Rotate)
        {
            result.RemovedRotateTimelineCount++;
        }
        else if (timelineKind == TimelineKind.Scale)
        {
            result.RemovedScaleTimelineCount++;
        }
        else if (timelineKind == TimelineKind.Shear)
        {
            result.RemovedShearTimelineCount++;
        }
    }

    /// <summary>
    /// 将一条 Cocos 连续轨道转为 Spine 4.3 关键帧，并在二次缓动中点拆段。
    /// </summary>
    private static JArray BuildTimeline(MovementData movementData, string targetName, JArray sourceFrames, TimelineKind timelineKind)
    {
        JArray result = new JArray();
        double rotationDirection = movementData.RotationDirectionMap[targetName];
        for (int frameIndex = 0; frameIndex < sourceFrames.Count; frameIndex++)
        {
            JObject sourceFrame = RequireFrame(sourceFrames, movementData.Name, targetName, frameIndex);
            double sourceFrameIndex = ReadRequiredInt(sourceFrame, "fi", $"movement {movementData.Name}.{targetName}.frame_data[{frameIndex}]");
            double time = sourceFrameIndex * movementData.SecondsPerFrame;
            double[] values = ReadSpineTimelineValues(movementData, targetName, sourceFrame, timelineKind, rotationDirection, frameIndex);

            // sceneext 在 movement 开始时立即应用首帧，即使首帧 fi 大于 0。
            if (frameIndex == 0 && sourceFrameIndex > 0d)
            {
                result.Add(CreateTimelineFrame(0d, values, timelineKind));
            }

            JObject spineFrame = CreateTimelineFrame(time, values, timelineKind);
            if (frameIndex >= sourceFrames.Count - 1)
            {
                result.Add(spineFrame);
                continue;
            }

            JObject nextSourceFrame = RequireFrame(sourceFrames, movementData.Name, targetName, frameIndex + 1);
            double nextSourceFrameIndex = ReadRequiredInt(nextSourceFrame, "fi", $"movement {movementData.Name}.{targetName}.frame_data[{frameIndex + 1}]");
            double nextTime = nextSourceFrameIndex * movementData.SecondsPerFrame;
            double[] nextValues = ReadSpineTimelineValues(
                movementData,
                targetName,
                nextSourceFrame,
                timelineKind,
                rotationDirection,
                frameIndex + 1
            );
            bool tweenFrame = ReadRequiredBoolean(sourceFrame, "tweenFrame", $"movement {movementData.Name}.{targetName}.frame_data[{frameIndex}]");
            int tweenEasing = ReadRequiredInt(sourceFrame, "twE", $"movement {movementData.Name}.{targetName}.frame_data[{frameIndex}]");

            // stepped 优先于缓动类型。
            if (!tweenFrame)
            {
                spineFrame["curve"] = "stepped";
                result.Add(spineFrame);
                continue;
            }

            if (tweenEasing == 0)
            {
                result.Add(spineFrame);
                continue;
            }

            if (tweenEasing == 3)
            {
                spineFrame["curve"] = CreateBezierCurve(time, nextTime, values, nextValues, SineControlX1, 0d, SineControlX2, 1d);
                result.Add(spineFrame);
                continue;
            }

            if (tweenEasing == 7)
            {
                spineFrame["curve"] = CreateBezierCurve(time, nextTime, values, nextValues, 1d / 3d, 0d, 2d / 3d, 0d);
                result.Add(spineFrame);
                continue;
            }

            if (tweenEasing == 6)
            {
                double middleTime = (time + nextTime) * 0.5d;
                double[] middleValues = InterpolateValues(values, nextValues, 0.5d);
                spineFrame["curve"] = CreateBezierCurve(time, middleTime, values, middleValues, 1d / 3d, 0d, 2d / 3d, 1d / 3d);
                result.Add(spineFrame);

                JObject middleFrame = CreateTimelineFrame(middleTime, middleValues, timelineKind);
                middleFrame["curve"] = CreateBezierCurve(middleTime, nextTime, middleValues, nextValues, 1d / 3d, 2d / 3d, 2d / 3d, 1d);
                result.Add(middleFrame);
                continue;
            }

            string easingContext = $"movement {movementData.Name}.{targetName}.frame_data[{frameIndex}]";
            if (tweenEasing == 21)
            {
                AppendCircularEaseInOutFrames(result, time, nextTime, values, nextValues, timelineKind);
                continue;
            }

            if (IsBezierApproximationEasing(tweenEasing))
            {
                AppendApproximatedEasingFrames(
                    result,
                    time,
                    nextTime,
                    values,
                    nextValues,
                    timelineKind,
                    tweenEasing,
                    sourceFrame,
                    easingContext
                );
                continue;
            }

            throw new InvalidDataException($"ExportJson {easingContext} 使用未支持的 twE={tweenEasing}。");
        }

        return result;
    }

    /// <summary>
    /// 判断缓动是否使用自适应三次 Bezier 分段逼近。
    /// </summary>
    private static bool IsBezierApproximationEasing(int tweenEasing)
    {
        return tweenEasing == -1
            || tweenEasing == 1
            || tweenEasing == 2
            || tweenEasing == 4
            || tweenEasing == 9
            || tweenEasing == 13
            || tweenEasing == 27;
    }

    /// <summary>
    /// 使用自适应三次 Bezier 分段重建 Cocos 缓动。
    /// </summary>
    private static void AppendApproximatedEasingFrames(
        JArray result,
        double time,
        double nextTime,
        double[] values,
        double[] nextValues,
        TimelineKind timelineKind,
        int tweenEasing,
        JObject sourceFrame,
        string context
    )
    {
        List<EasingBezierSegment> segments = new List<EasingBezierSegment>();

        // 自适应拆分缓动，三次及以下多项式会自然收敛为最少分段。
        BuildEasingBezierSegments(tweenEasing, sourceFrame, context, 0d, 1d, 0, segments);

        // 每段写入起点和绝对控制点；终点由下一段或源关键帧提供。
        for (int segmentIndex = 0; segmentIndex < segments.Count; segmentIndex++)
        {
            EasingBezierSegment segment = segments[segmentIndex];
            double segmentTime = time + (nextTime - time) * segment.Start;
            double[] segmentValues = InterpolateValues(values, nextValues, segment.StartProgress);
            JObject segmentFrame = CreateTimelineFrame(segmentTime, segmentValues, timelineKind);
            segmentFrame["curve"] = CreateEasingBezierCurve(time, nextTime, values, nextValues, segment);
            result.Add(segmentFrame);
        }
    }

    /// <summary>
    /// 使用四段圆弧 Bezier 重建 Circ_EaseInOut，避免中点无限斜率失真。
    /// </summary>
    private static void AppendCircularEaseInOutFrames(
        JArray result,
        double time,
        double nextTime,
        double[] values,
        double[] nextValues,
        TimelineKind timelineKind
    )
    {
        const double arcControl = 0.265216489839544d;
        double quarterAngle = Math.PI * 0.25d;
        List<EasingArcSegment> segments = new List<EasingArcSegment>();

        // 前半段沿圆心 (0, 0.5) 的四分之一圆拆成两段。
        for (int segmentIndex = 0; segmentIndex < 2; segmentIndex++)
        {
            double startAngle = segmentIndex * quarterAngle;
            double endAngle = startAngle + quarterAngle;
            Vector2D startPoint = new Vector2D(0.5d * Math.Sin(startAngle), 0.5d * (1d - Math.Cos(startAngle)));
            Vector2D endPoint = new Vector2D(0.5d * Math.Sin(endAngle), 0.5d * (1d - Math.Cos(endAngle)));
            Vector2D startTangent = new Vector2D(0.5d * Math.Cos(startAngle), 0.5d * Math.Sin(startAngle));
            Vector2D endTangent = new Vector2D(0.5d * Math.Cos(endAngle), 0.5d * Math.Sin(endAngle));
            segments.Add(
                new EasingArcSegment(
                    startPoint,
                    new Vector2D(startPoint.X + arcControl * startTangent.X, startPoint.Y + arcControl * startTangent.Y),
                    new Vector2D(endPoint.X - arcControl * endTangent.X, endPoint.Y - arcControl * endTangent.Y)
                )
            );
        }

        // 后半段沿圆心 (1, 0.5) 的四分之一圆拆成两段。
        for (int segmentIndex = 0; segmentIndex < 2; segmentIndex++)
        {
            double startAngle = -Math.PI * 0.5d + segmentIndex * quarterAngle;
            double endAngle = startAngle + quarterAngle;
            Vector2D startPoint = new Vector2D(1d + 0.5d * Math.Sin(startAngle), 0.5d + 0.5d * Math.Cos(startAngle));
            Vector2D endPoint = new Vector2D(1d + 0.5d * Math.Sin(endAngle), 0.5d + 0.5d * Math.Cos(endAngle));
            Vector2D startTangent = new Vector2D(0.5d * Math.Cos(startAngle), -0.5d * Math.Sin(startAngle));
            Vector2D endTangent = new Vector2D(0.5d * Math.Cos(endAngle), -0.5d * Math.Sin(endAngle));
            segments.Add(
                new EasingArcSegment(
                    startPoint,
                    new Vector2D(startPoint.X + arcControl * startTangent.X, startPoint.Y + arcControl * startTangent.Y),
                    new Vector2D(endPoint.X - arcControl * endTangent.X, endPoint.Y - arcControl * endTangent.Y)
                )
            );
        }

        // 写入每段起点及绝对时间和值控制点。
        for (int segmentIndex = 0; segmentIndex < segments.Count; segmentIndex++)
        {
            EasingArcSegment segment = segments[segmentIndex];
            double segmentTime = time + (nextTime - time) * segment.Start.X;
            JObject segmentFrame = CreateTimelineFrame(
                segmentTime,
                InterpolateValues(values, nextValues, segment.Start.Y),
                timelineKind
            );
            segmentFrame["curve"] = CreateAbsoluteEasingBezierCurve(
                time,
                nextTime,
                values,
                nextValues,
                segment.FirstControl,
                segment.SecondControl
            );
            result.Add(segmentFrame);
        }
    }

    /// <summary>
    /// 递归生成满足误差阈值的 Cocos 缓动 Bezier 分段。
    /// </summary>
    private static void BuildEasingBezierSegments(
        int tweenEasing,
        JObject sourceFrame,
        string context,
        double start,
        double end,
        int depth,
        List<EasingBezierSegment> result
    )
    {
        double length = end - start;
        double startProgress = EvaluateCocosEasing(tweenEasing, start, sourceFrame, context);
        double endProgress = EvaluateCocosEasing(tweenEasing, end, sourceFrame, context);
        double firstSample = EvaluateCocosEasing(tweenEasing, start + length / 3d, sourceFrame, context);
        double secondSample = EvaluateCocosEasing(tweenEasing, start + length * 2d / 3d, sourceFrame, context);
        double firstEquation = 27d * firstSample - 8d * startProgress - endProgress;
        double secondEquation = 27d * secondSample - startProgress - 8d * endProgress;
        double firstControlProgress = (2d * firstEquation - secondEquation) / 18d;
        double secondControlProgress = (2d * secondEquation - firstEquation) / 18d;

        // 检查分段内部误差，超限时从中点继续拆分。
        double maximumError = 0d;
        double[] samples = { 1d / 6d, 0.5d, 5d / 6d };
        for (int sampleIndex = 0; sampleIndex < samples.Length; sampleIndex++)
        {
            double localPercent = samples[sampleIndex];
            double expected = EvaluateCocosEasing(tweenEasing, start + length * localPercent, sourceFrame, context);
            double actual = EvaluateCubicBezier(startProgress, firstControlProgress, secondControlProgress, endProgress, localPercent);
            maximumError = Math.Max(maximumError, Math.Abs(expected - actual));
        }

        if (maximumError > EasingApproximationTolerance && depth < MaxEasingSubdivisionDepth)
        {
            double middle = (start + end) * 0.5d;
            BuildEasingBezierSegments(tweenEasing, sourceFrame, context, start, middle, depth + 1, result);
            BuildEasingBezierSegments(tweenEasing, sourceFrame, context, middle, end, depth + 1, result);
            return;
        }

        result.Add(new EasingBezierSegment(start, end, startProgress, firstControlProgress, secondControlProgress));
    }

    /// <summary>
    /// 计算 Cocos TweenFunction 对应的归一化缓动进度。
    /// </summary>
    private static double EvaluateCocosEasing(int tweenEasing, double time, JObject sourceFrame, string context)
    {
        if (tweenEasing == 0)
        {
            return time;
        }

        if (tweenEasing == -1)
        {
            JArray parameters = RequireCustomEasingParameters(sourceFrame, context);
            double inverse = 1d - time;
            return parameters[1].Value<double>() * inverse * inverse * inverse
                + 3d * parameters[3].Value<double>() * time * inverse * inverse
                + 3d * parameters[5].Value<double>() * time * time * inverse
                + parameters[7].Value<double>() * time * time * time;
        }

        if (tweenEasing == 1)
        {
            return -Math.Cos(time * Math.PI * 0.5d) + 1d;
        }

        if (tweenEasing == 2)
        {
            return Math.Sin(time * Math.PI * 0.5d);
        }

        if (tweenEasing == 3)
        {
            return -0.5d * (Math.Cos(Math.PI * time) - 1d);
        }

        if (tweenEasing == 4)
        {
            return time * time;
        }

        if (tweenEasing == 6)
        {
            double doubled = time * 2d;
            if (doubled < 1d)
            {
                return 0.5d * doubled * doubled;
            }

            doubled -= 1d;
            return -0.5d * (doubled * (doubled - 2d) - 1d);
        }

        if (tweenEasing == 7)
        {
            return time * time * time;
        }

        if (tweenEasing == 9)
        {
            double doubled = time * 2d;
            if (doubled < 1d)
            {
                return 0.5d * doubled * doubled * doubled;
            }

            doubled -= 2d;
            return 0.5d * (doubled * doubled * doubled + 2d);
        }

        if (tweenEasing == 13)
        {
            return time * time * time * time * time;
        }

        if (tweenEasing == 21)
        {
            double doubled = time * 2d;
            if (doubled < 1d)
            {
                return -0.5d * (Math.Sqrt(1d - doubled * doubled) - 1d);
            }

            doubled -= 2d;
            return 0.5d * (Math.Sqrt(1d - doubled * doubled) + 1d);
        }

        if (tweenEasing == 27)
        {
            const double overshoot = 1.70158d * 1.525d;
            double doubled = time * 2d;
            if (doubled < 1d)
            {
                return doubled * doubled * ((overshoot + 1d) * doubled - overshoot) * 0.5d;
            }

            doubled -= 2d;
            return doubled * doubled * ((overshoot + 1d) * doubled + overshoot) * 0.5d + 1d;
        }

        throw new InvalidDataException($"ExportJson {context} 使用未支持的 twE={tweenEasing}。");
    }

    /// <summary>
    /// 计算三次 Bezier 在指定参数处的数值。
    /// </summary>
    private static double EvaluateCubicBezier(double start, double control1, double control2, double end, double time)
    {
        double inverse = 1d - time;
        return inverse * inverse * inverse * start
            + 3d * inverse * inverse * time * control1
            + 3d * inverse * time * time * control2
            + time * time * time * end;
    }

    /// <summary>
    /// 使用归一化绝对时间和进度创建 Spine Bezier 控制点。
    /// </summary>
    private static JArray CreateAbsoluteEasingBezierCurve(
        double time,
        double nextTime,
        double[] values,
        double[] nextValues,
        Vector2D firstControl,
        Vector2D secondControl
    )
    {
        JArray curve = new JArray();
        double duration = nextTime - time;
        for (int valueIndex = 0; valueIndex < values.Length; valueIndex++)
        {
            double valueDelta = nextValues[valueIndex] - values[valueIndex];
            curve.Add(time + duration * firstControl.X);
            curve.Add(values[valueIndex] + valueDelta * firstControl.Y);
            curve.Add(time + duration * secondControl.X);
            curve.Add(values[valueIndex] + valueDelta * secondControl.Y);
        }

        return curve;
    }

    /// <summary>
    /// 创建单个缓动分段的 Spine 绝对时间和值控制点。
    /// </summary>
    private static JArray CreateEasingBezierCurve(
        double time,
        double nextTime,
        double[] values,
        double[] nextValues,
        EasingBezierSegment segment
    )
    {
        JArray curve = new JArray();
        double duration = nextTime - time;
        double segmentLength = segment.End - segment.Start;
        double firstControlTime = time + duration * (segment.Start + segmentLength / 3d);
        double secondControlTime = time + duration * (segment.Start + segmentLength * 2d / 3d);
        for (int valueIndex = 0; valueIndex < values.Length; valueIndex++)
        {
            double valueDelta = nextValues[valueIndex] - values[valueIndex];
            curve.Add(firstControlTime);
            curve.Add(values[valueIndex] + valueDelta * segment.FirstControlProgress);
            curve.Add(secondControlTime);
            curve.Add(values[valueIndex] + valueDelta * segment.SecondControlProgress);
        }

        return curve;
    }

    /// <summary>
    /// 按 Cocos display index 重建附件切换；movement 未包含骨骼时在 0 秒隐藏。
    /// </summary>
    private static JArray BuildAttachmentTimeline(MovementData movementData, string boneName, List<string> displayNames, JArray sourceFrames)
    {
        if (sourceFrames == null)
        {
            return new JArray(new JObject());
        }

        JArray result = new JArray();
        int currentDisplayIndex = ReadNormalizedDisplayIndex(
            movementData.SourceBoneMap[boneName],
            displayNames.Count,
            $"setup 骨骼 {boneName}"
        );
        for (int frameIndex = 0; frameIndex < sourceFrames.Count; frameIndex++)
        {
            JObject sourceFrame = RequireFrame(sourceFrames, movementData.Name, boneName, frameIndex);
            int displayIndex = ReadNormalizedDisplayIndex(
                sourceFrame,
                displayNames.Count,
                $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]"
            );
            if (displayIndex == currentDisplayIndex)
            {
                continue;
            }

            // sceneext 播放 movement 时立即应用首帧 displayIndex，后续帧才使用 fi 时间。
            double time = frameIndex == 0
                ? 0d
                : ReadRequiredInt(sourceFrame, "fi", $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]")
                    * movementData.SecondsPerFrame;
            JObject attachmentFrame = new JObject();
            WriteOptionalDouble(attachmentFrame, "time", time, 0d);
            if (displayIndex >= 0)
            {
                attachmentFrame["name"] = displayNames[displayIndex];
            }

            result.Add(attachmentFrame);
            currentDisplayIndex = displayIndex;
        }

        return result.Count == 0 ? null : result;
    }

    /// <summary>
    /// 读取转换为 Spine 相对 setup 语义后的轨道值。
    /// </summary>
    private static double[] ReadSpineTimelineValues(
        MovementData movementData,
        string boneName,
        JObject sourceFrame,
        TimelineKind timelineKind,
        double rotationDirection,
        int frameIndex
    )
    {
        if (timelineKind != TimelineKind.Scale)
        {
            return ReadTimelineValues(sourceFrame, timelineKind, rotationDirection);
        }

        return ReadScaleTimelineValues(
            movementData.SourceBoneMap[boneName],
            sourceFrame,
            $"movement {movementData.Name}.{boneName}.frame_data[{frameIndex}]"
        );
    }

    /// <summary>
    /// 将 Cocos 的 setup 加帧偏移缩放转换为 Spine 相对 setup 倍率。
    /// </summary>
    private static double[] ReadScaleTimelineValues(JObject sourceBone, JObject sourceFrame, string context)
    {
        // 校验 setup 缩放可作为 Spine 相对倍率分母。
        double setupScaleX = ReadRequiredDouble(sourceBone, "cX", $"{context} 对应 setup 骨骼");
        double setupScaleY = ReadRequiredDouble(sourceBone, "cY", $"{context} 对应 setup 骨骼");
        if (Math.Abs(setupScaleX) <= ValueTolerance || Math.Abs(setupScaleY) <= ValueTolerance)
        {
            throw new InvalidDataException($"ExportJson {context} 对应 setup 骨骼含零缩放，无法转换 Spine 相对缩放。");
        }

        // 按 Cocos 加法语义还原缩放，再转换为 Spine setup 倍率。
        double frameScaleX = ReadRequiredDouble(sourceFrame, "cX", context);
        double frameScaleY = ReadRequiredDouble(sourceFrame, "cY", context);
        double spineScaleX = Math.Abs(setupScaleX - 1d) <= ValueTolerance ? frameScaleX : (setupScaleX + frameScaleX - 1d) / setupScaleX;
        double spineScaleY = Math.Abs(setupScaleY - 1d) <= ValueTolerance ? frameScaleY : (setupScaleY + frameScaleY - 1d) / setupScaleY;
        return new[] { spineScaleX, spineScaleY };
    }

    /// <summary>
    /// 判断 Cocos 帧列表是否包含颜色信息。
    /// </summary>
    private static bool HasColorTimeline(JArray sourceFrames)
    {
        if (sourceFrames == null)
        {
            return false;
        }

        for (int frameIndex = 0; frameIndex < sourceFrames.Count; frameIndex++)
        {
            if (sourceFrames[frameIndex]?["color"] != null)
            {
                return true;
            }
        }

        return false;
    }

    /// <summary>
    /// 判断 Cocos 帧列表的指定变换通道是否偏离默认值。
    /// </summary>
    private static bool HasNonDefaultTimeline(JArray sourceFrames, TimelineKind timelineKind)
    {
        if (sourceFrames == null || timelineKind == TimelineKind.Rgba)
        {
            return false;
        }

        for (int frameIndex = 0; frameIndex < sourceFrames.Count; frameIndex++)
        {
            JObject sourceFrame = sourceFrames[frameIndex] as JObject;
            if (sourceFrame == null)
            {
                return false;
            }

            double[] values = ReadTimelineValues(sourceFrame, timelineKind, 1d);
            double defaultValue = timelineKind == TimelineKind.Scale ? 1d : 0d;
            for (int valueIndex = 0; valueIndex < values.Length; valueIndex++)
            {
                if (Math.Abs(values[valueIndex] - defaultValue) > ValueTolerance)
                {
                    return true;
                }
            }
        }

        return false;
    }

    /// <summary>
    /// 读取 Cocos 帧在指定 Spine 轨道中的值。
    /// </summary>
    private static double[] ReadTimelineValues(JObject sourceFrame, TimelineKind timelineKind, double rotationDirection)
    {
        if (timelineKind == TimelineKind.Translate)
        {
            return new[] { sourceFrame.Value<double?>("x") ?? 0d, sourceFrame.Value<double?>("y") ?? 0d };
        }

        if (timelineKind == TimelineKind.Rotate)
        {
            double skewY = sourceFrame.Value<double?>("kY") ?? 0d;
            return new[] { skewY * RadiansToDegrees * rotationDirection };
        }

        if (timelineKind == TimelineKind.Scale)
        {
            return new[] { sourceFrame.Value<double?>("cX") ?? 1d, sourceFrame.Value<double?>("cY") ?? 1d };
        }

        if (timelineKind == TimelineKind.Shear)
        {
            double skewX = sourceFrame.Value<double?>("kX") ?? 0d;
            double skewY = sourceFrame.Value<double?>("kY") ?? 0d;
            return new[] { 0d, -(skewX + skewY) * RadiansToDegrees * rotationDirection };
        }

        JObject color = sourceFrame["color"] as JObject;
        if (color == null)
        {
            return new[] { 1d, 1d, 1d, 1d };
        }

        return new[]
        {
            ReadColorByte(color, "r", "颜色帧") / 255d,
            ReadColorByte(color, "g", "颜色帧") / 255d,
            ReadColorByte(color, "b", "颜色帧") / 255d,
            ReadColorByte(color, "a", "颜色帧") / 255d,
        };
    }

    /// <summary>
    /// 创建 Spine 4.3 关键帧并省略默认数值字段。
    /// </summary>
    private static JObject CreateTimelineFrame(double time, double[] values, TimelineKind timelineKind)
    {
        JObject frame = new JObject();
        WriteOptionalDouble(frame, "time", time, 0d);
        if (timelineKind == TimelineKind.Translate)
        {
            WriteOptionalDouble(frame, "x", values[0], 0d);
            WriteOptionalDouble(frame, "y", values[1], 0d);
        }
        else if (timelineKind == TimelineKind.Rotate)
        {
            WriteOptionalDouble(frame, "value", values[0], 0d);
        }
        else if (timelineKind == TimelineKind.Scale)
        {
            WriteOptionalDouble(frame, "x", values[0], 1d);
            WriteOptionalDouble(frame, "y", values[1], 1d);
        }
        else if (timelineKind == TimelineKind.Shear)
        {
            WriteOptionalDouble(frame, "x", values[0], 0d);
            WriteOptionalDouble(frame, "y", values[1], 0d);
        }
        else
        {
            frame["color"] = ConvertColorValues(values);
        }

        return frame;
    }

    /// <summary>
    /// 将归一化 Bezier 控制点转换为 Spine 4.3 使用的绝对时间和值控制点。
    /// </summary>
    private static JArray CreateBezierCurve(
        double time1,
        double time2,
        double[] values1,
        double[] values2,
        double controlX1,
        double controlY1,
        double controlX2,
        double controlY2
    )
    {
        JArray curve = new JArray();
        double duration = time2 - time1;
        for (int valueIndex = 0; valueIndex < values1.Length; valueIndex++)
        {
            double valueDelta = values2[valueIndex] - values1[valueIndex];
            curve.Add(time1 + duration * controlX1);
            curve.Add(values1[valueIndex] + valueDelta * controlY1);
            curve.Add(time1 + duration * controlX2);
            curve.Add(values1[valueIndex] + valueDelta * controlY2);
        }

        return curve;
    }

    /// <summary>
    /// 线性计算多通道中间值。
    /// </summary>
    private static double[] InterpolateValues(double[] from, double[] to, double percent)
    {
        double[] result = new double[from.Length];
        for (int valueIndex = 0; valueIndex < from.Length; valueIndex++)
        {
            result[valueIndex] = from[valueIndex] + (to[valueIndex] - from[valueIndex]) * percent;
        }

        return result;
    }

    /// <summary>
    /// 将 RGBA 数值转换为 Spine RRGGBBAA。
    /// </summary>
    private static string ConvertColorValues(double[] values)
    {
        byte red = ConvertColorByte(values[0] * 255d);
        byte green = ConvertColorByte(values[1] * 255d);
        byte blue = ConvertColorByte(values[2] * 255d);
        byte alpha = ConvertColorByte(values[3] * 255d);
        return $"{red:X2}{green:X2}{blue:X2}{alpha:X2}";
    }

    /// <summary>
    /// 将颜色数值限制并转换为字节。
    /// </summary>
    private static byte ConvertColorByte(double value)
    {
        if (!IsFinite(value))
        {
            throw new InvalidDataException($"颜色数值无效: {value}。");
        }

        double clamped = Math.Max(0d, Math.Min(255d, value));
        return (byte)Math.Round(clamped, MidpointRounding.AwayFromZero);
    }

    /// <summary>
    /// 读取 0～255 的必需颜色分量。
    /// </summary>
    private static int ReadColorByte(JObject color, string propertyName, string context)
    {
        int value = ReadRequiredInt(color, propertyName, context + ".color");
        if (value < 0 || value > 255)
        {
            throw new InvalidDataException($"ExportJson {context}.color.{propertyName} 越界: {value}。");
        }

        return value;
    }

    /// <summary>
    /// 校验修复前后只改变允许重建的动画轨道。
    /// </summary>
    private static void EnsureOnlyRepairableDataChanged(JObject before, JObject after, SourceData sourceData)
    {
        if (before == null)
        {
            throw new InvalidDataException("无法复制修复前 Spine JSON。");
        }

        JObject strippedBefore = before.DeepClone() as JObject;
        JObject strippedAfter = after.DeepClone() as JObject;
        StripRepairableSetupPose(strippedBefore, sourceData);
        StripRepairableSetupPose(strippedAfter, sourceData);
        StripRepairableTimelines(strippedBefore);
        StripRepairableTimelines(strippedAfter);
        if (!JToken.DeepEquals(strippedBefore, strippedAfter))
        {
            throw new InvalidDataException("结构保护失败：修复修改了 skins、附件 setup、骨骼层级或允许范围外的动画数据。");
        }
    }

    /// <summary>
    /// 从 JSON 副本移除允许变化的源骨骼 setup 变换字段。
    /// </summary>
    private static void StripRepairableSetupPose(JObject spineRoot, SourceData sourceData)
    {
        Dictionary<string, JObject> spineBoneMap = BuildNamedObjectMap(spineRoot["bones"] as JArray, "Spine setup 骨骼");
        string[] transformNames = { "x", "y", "rotation", "scaleX", "scaleY", "shearX", "shearY" };
        foreach (string boneName in sourceData.SourceBoneMap.Keys)
        {
            if (!spineBoneMap.TryGetValue(boneName, out JObject spineBone))
            {
                continue;
            }

            for (int propertyIndex = 0; propertyIndex < transformNames.Length; propertyIndex++)
            {
                spineBone.Remove(transformNames[propertyIndex]);
            }
        }

        // setup attachment 由 Cocos dI 决定，属于允许修复范围。
        Dictionary<string, JObject> spineSlotMap = BuildNamedObjectMap(spineRoot["slots"] as JArray, "Spine setup 插槽");
        foreach (KeyValuePair<string, List<string>> displayPair in sourceData.DisplayNameMap)
        {
            if (displayPair.Value.Count > 0 && spineSlotMap.TryGetValue(displayPair.Key, out JObject spineSlot))
            {
                spineSlot.Remove("attachment");
            }
        }
    }

    /// <summary>
    /// 从 JSON 副本移除允许变化的六类轨道和 drawOrder，用于结构保护比较。
    /// </summary>
    private static void StripRepairableTimelines(JObject spineRoot)
    {
        JObject animations = spineRoot?["animations"] as JObject;
        if (animations == null)
        {
            return;
        }

        foreach (JProperty animationProperty in animations.Properties())
        {
            JObject animation = animationProperty.Value as JObject;
            JObject bones = animation?["bones"] as JObject;
            RemoveTimelineNamesAndEmptyTargets(bones, new[] { "translate", "rotate", "scale", "shear" });
            if (bones != null && !bones.HasValues)
            {
                animation.Remove("bones");
            }

            JObject slots = animation?["slots"] as JObject;
            RemoveTimelineNamesAndEmptyTargets(slots, new[] { "attachment", "rgba" });
            if (slots != null && !slots.HasValues)
            {
                animation.Remove("slots");
            }

            animation?.Remove("drawOrder");
        }
    }

    /// <summary>
    /// 删除目标对象中的指定轨道，并清理因此变空的目标节点。
    /// </summary>
    private static void RemoveTimelineNamesAndEmptyTargets(JObject container, string[] timelineNames)
    {
        if (container == null)
        {
            return;
        }

        List<JProperty> emptyProperties = new List<JProperty>();
        foreach (JProperty targetProperty in container.Properties())
        {
            JObject timelines = targetProperty.Value as JObject;
            if (timelines == null)
            {
                continue;
            }

            for (int timelineIndex = 0; timelineIndex < timelineNames.Length; timelineIndex++)
            {
                timelines.Remove(timelineNames[timelineIndex]);
            }

            if (!timelines.HasValues)
            {
                emptyProperties.Add(targetProperty);
            }
        }

        for (int propertyIndex = 0; propertyIndex < emptyProperties.Count; propertyIndex++)
        {
            emptyProperties[propertyIndex].Remove();
        }
    }

    /// <summary>
    /// 校验修复轨道帧结构、严格递增时间、字段和 Spine 4.3 曲线格式。
    /// </summary>
    private static void ValidateRepairedTimelines(JObject spineRoot)
    {
        JObject animations = spineRoot["animations"] as JObject;
        if (animations == null)
        {
            throw new InvalidDataException("Spine 文件缺少 animations。");
        }

        // 校验骨骼连续轨道。
        foreach (JProperty animationProperty in animations.Properties())
        {
            JObject bones = animationProperty.Value?["bones"] as JObject;
            if (bones != null)
            {
                foreach (JProperty boneProperty in bones.Properties())
                {
                    JObject timelines = boneProperty.Value as JObject;
                    ValidateTimeline(animationProperty.Name, boneProperty.Name, "translate", timelines?["translate"] as JArray, 2);
                    ValidateTimeline(animationProperty.Name, boneProperty.Name, "rotate", timelines?["rotate"] as JArray, 1);
                    ValidateTimeline(animationProperty.Name, boneProperty.Name, "scale", timelines?["scale"] as JArray, 2);
                    ValidateTimeline(animationProperty.Name, boneProperty.Name, "shear", timelines?["shear"] as JArray, 2);
                }
            }

            // 校验 slot 离散轨道和颜色轨道。
            JObject slots = animationProperty.Value?["slots"] as JObject;
            if (slots == null)
            {
                continue;
            }

            foreach (JProperty slotProperty in slots.Properties())
            {
                JObject timelines = slotProperty.Value as JObject;
                ValidateAttachmentTimeline(animationProperty.Name, slotProperty.Name, timelines?["attachment"] as JArray);
                ValidateTimeline(animationProperty.Name, slotProperty.Name, "rgba", timelines?["rgba"] as JArray, 4);
            }
        }
    }

    /// <summary>
    /// 校验一条连续数值或颜色轨道。
    /// </summary>
    private static void ValidateTimeline(string animationName, string targetName, string timelineName, JArray timeline, int valueCount)
    {
        if (timeline == null)
        {
            return;
        }

        if (timeline.Count == 0)
        {
            throw new InvalidDataException($"Spine 轨道为空: {animationName}.{targetName}.{timelineName}。");
        }

        double previousTime = -1d;
        for (int frameIndex = 0; frameIndex < timeline.Count; frameIndex++)
        {
            JObject frame = timeline[frameIndex] as JObject;
            double time = frame?.Value<double?>("time") ?? 0d;
            if (frame == null || !IsFinite(time) || time <= previousTime)
            {
                throw new InvalidDataException($"Spine 轨道时间无效: {animationName}.{targetName}.{timelineName}[{frameIndex}]。");
            }

            if (timelineName == "rgba")
            {
                string color = frame.Value<string>("color");
                if (string.IsNullOrEmpty(color) || color.Length != 8 || !IsHexColor(color))
                {
                    throw new InvalidDataException($"Spine rgba 颜色无效: {animationName}.{targetName}.rgba[{frameIndex}]。");
                }
            }
            else
            {
                ValidateNumericFrame(animationName, targetName, timelineName, frameIndex, frame);
            }

            ValidateCurve(animationName, targetName, timelineName, frameIndex, frame["curve"], valueCount);
            previousTime = time;
        }
    }

    /// <summary>
    /// 校验骨骼数值关键帧字段。
    /// </summary>
    private static void ValidateNumericFrame(string animationName, string targetName, string timelineName, int frameIndex, JObject frame)
    {
        double firstValue;
        double secondValue;
        if (timelineName == "rotate")
        {
            firstValue = frame.Value<double?>("value") ?? 0d;
            secondValue = 0d;
        }
        else
        {
            double defaultValue = timelineName == "scale" ? 1d : 0d;
            firstValue = frame.Value<double?>("x") ?? defaultValue;
            secondValue = frame.Value<double?>("y") ?? defaultValue;
        }

        if (!IsFinite(firstValue) || !IsFinite(secondValue))
        {
            throw new InvalidDataException($"Spine 轨道数值无效: {animationName}.{targetName}.{timelineName}[{frameIndex}]。");
        }
    }

    /// <summary>
    /// 校验 Spine 4.3 stepped 或绝对控制点 Bezier 曲线。
    /// </summary>
    private static void ValidateCurve(string animationName, string targetName, string timelineName, int frameIndex, JToken curve, int valueCount)
    {
        if (curve == null)
        {
            return;
        }

        if (curve.Type == JTokenType.String && string.Equals(curve.Value<string>(), "stepped", StringComparison.Ordinal))
        {
            return;
        }

        JArray controlPoints = curve as JArray;
        if (controlPoints == null || controlPoints.Count != valueCount * 4)
        {
            throw new InvalidDataException($"Spine 曲线格式无效: {animationName}.{targetName}.{timelineName}[{frameIndex}]。");
        }

        for (int pointIndex = 0; pointIndex < controlPoints.Count; pointIndex++)
        {
            double value = controlPoints[pointIndex].Value<double>();
            if (!IsFinite(value))
            {
                throw new InvalidDataException($"Spine 曲线控制点无效: {animationName}.{targetName}.{timelineName}[{frameIndex}]。");
            }
        }
    }

    /// <summary>
    /// 校验 attachment 关键帧时间和可选名称。
    /// </summary>
    private static void ValidateAttachmentTimeline(string animationName, string slotName, JArray timeline)
    {
        if (timeline == null)
        {
            return;
        }

        double previousTime = -1d;
        for (int frameIndex = 0; frameIndex < timeline.Count; frameIndex++)
        {
            JObject frame = timeline[frameIndex] as JObject;
            double time = frame?.Value<double?>("time") ?? 0d;
            JToken name = frame?["name"];
            if (frame == null || !IsFinite(time) || time <= previousTime || (name != null && name.Type != JTokenType.String && name.Type != JTokenType.Null))
            {
                throw new InvalidDataException($"Spine attachment 轨道无效: {animationName}.{slotName}.attachment[{frameIndex}]。");
            }

            previousTime = time;
        }
    }

    /// <summary>
    /// 使用项目当前 Spine 4.3 runtime 完整解析输出 JSON。
    /// </summary>
    private static void ValidateWithSpineRuntime(JObject spineRoot)
    {
        string json = spineRoot.ToString(Formatting.None);
        Spine.Unity.RegionlessAttachmentLoader attachmentLoader = new Spine.Unity.RegionlessAttachmentLoader();
        Spine.SkeletonJson skeletonJson = new Spine.SkeletonJson(attachmentLoader);
        using (StringReader reader = new StringReader(json))
        {
            Spine.SkeletonData skeletonData = skeletonJson.ReadSkeletonData(reader);
            if (skeletonData == null)
            {
                throw new InvalidDataException("Spine runtime 未能生成 SkeletonData。");
            }
        }
    }

    /// <summary>
    /// 将对象数组转换为名称唯一的索引。
    /// </summary>
    private static Dictionary<string, JObject> BuildNamedObjectMap(JArray sourceItems, string context)
    {
        if (sourceItems == null)
        {
            throw new InvalidDataException($"{context} 数组缺失。");
        }

        Dictionary<string, JObject> result = new Dictionary<string, JObject>(StringComparer.Ordinal);
        for (int itemIndex = 0; itemIndex < sourceItems.Count; itemIndex++)
        {
            JObject sourceItem = sourceItems[itemIndex] as JObject;
            string itemName = sourceItem?.Value<string>("name");
            if (string.IsNullOrEmpty(itemName) || !result.TryAdd(itemName, sourceItem))
            {
                throw new InvalidDataException($"{context} 第 {itemIndex} 项名称无效或重复: {itemName}。");
            }
        }

        return result;
    }

    /// <summary>
    /// 收集名称数组的稳定顺序。
    /// </summary>
    private static List<string> CollectNamedOrder(JArray sourceItems, string context)
    {
        Dictionary<string, JObject> itemMap = BuildNamedObjectMap(sourceItems, context);
        List<string> result = new List<string>(itemMap.Count);
        foreach (KeyValuePair<string, JObject> itemPair in itemMap)
        {
            result.Add(itemPair.Key);
        }

        return result;
    }

    /// <summary>
    /// 读取并校验 Cocos 帧对象。
    /// </summary>
    private static JObject RequireFrame(JArray sourceFrames, string movementName, string boneName, int frameIndex)
    {
        JObject sourceFrame = sourceFrames[frameIndex] as JObject;
        if (sourceFrame == null)
        {
            throw new InvalidDataException($"ExportJson {movementName}.{boneName}.frame_data[{frameIndex}] 不是有效对象。");
        }

        return sourceFrame;
    }

    /// <summary>
    /// 读取必需整数属性。
    /// </summary>
    private static int ReadRequiredInt(JObject source, string propertyName, string context)
    {
        JToken token = source[propertyName];
        if (token == null || token.Type != JTokenType.Integer)
        {
            throw new InvalidDataException($"ExportJson {context} 缺少整数 {propertyName}。");
        }

        return token.Value<int>();
    }

    /// <summary>
    /// 读取必需浮点属性。
    /// </summary>
    private static double ReadRequiredDouble(JObject source, string propertyName, string context)
    {
        JToken token = source[propertyName];
        if (token == null || (token.Type != JTokenType.Integer && token.Type != JTokenType.Float))
        {
            throw new InvalidDataException($"ExportJson {context} 缺少数值 {propertyName}。");
        }

        return token.Value<double>();
    }

    /// <summary>
    /// 读取必需布尔属性。
    /// </summary>
    private static bool ReadRequiredBoolean(JObject source, string propertyName, string context)
    {
        JToken token = source[propertyName];
        if (token == null || token.Type != JTokenType.Boolean)
        {
            throw new InvalidDataException($"ExportJson {context} 缺少布尔值 {propertyName}。");
        }

        return token.Value<bool>();
    }

    /// <summary>
    /// 要求可选整数属性缺失或为零。
    /// </summary>
    private static void EnsureZeroInt(JObject source, string propertyName, string context)
    {
        JToken token = source[propertyName];
        if (token == null)
        {
            return;
        }

        if (token.Type != JTokenType.Integer || token.Value<int>() != 0)
        {
            throw new InvalidDataException($"ExportJson {context} 使用未支持的 {propertyName}={token.ToString(Formatting.None)}。");
        }
    }

    /// <summary>
    /// 要求可选扩展属性缺失、null、空字符串或空数组。
    /// </summary>
    private static void EnsureEmptyProperty(JObject source, string propertyName, string context)
    {
        JToken token = source[propertyName];
        if (token == null || token.Type == JTokenType.Null)
        {
            return;
        }

        if (token.Type == JTokenType.String && string.IsNullOrEmpty(token.Value<string>()))
        {
            return;
        }

        JArray array = token as JArray;
        if (array != null && array.Count == 0)
        {
            return;
        }

        throw new InvalidDataException($"ExportJson {context} 使用未支持的 {propertyName}={token.ToString(Formatting.None)}。");
    }

    /// <summary>
    /// 读取 Cocos displayIndex，并将任意负值归一为隐藏状态 -1。
    /// </summary>
    private static int ReadNormalizedDisplayIndex(JObject source, int displayCount, string context)
    {
        int sourceDisplayIndex = ReadRequiredInt(source, "dI", context);
        if (sourceDisplayIndex >= displayCount)
        {
            throw new InvalidDataException($"ExportJson {context} dI 越界: {sourceDisplayIndex}，display 数量 {displayCount}。");
        }

        return sourceDisplayIndex < 0 ? -1 : sourceDisplayIndex;
    }

    /// <summary>
    /// 获取或创建 JObject 子节点。
    /// </summary>
    private static JObject GetOrCreateObject(JObject parent, string propertyName)
    {
        JObject result = parent[propertyName] as JObject;
        if (result == null)
        {
            result = new JObject();
            parent[propertyName] = result;
        }

        return result;
    }

    /// <summary>
    /// 返回轨道对应的 Spine 字段名。
    /// </summary>
    private static string GetTimelineName(TimelineKind timelineKind)
    {
        if (timelineKind == TimelineKind.Translate)
        {
            return "translate";
        }

        if (timelineKind == TimelineKind.Rotate)
        {
            return "rotate";
        }

        if (timelineKind == TimelineKind.Scale)
        {
            return "scale";
        }

        if (timelineKind == TimelineKind.Shear)
        {
            return "shear";
        }

        return "rgba";
    }

    /// <summary>
    /// 去除 Cocos display 名称末尾扩展名并统一路径分隔符。
    /// </summary>
    private static string StripExtension(string path)
    {
        if (string.IsNullOrEmpty(path))
        {
            return string.Empty;
        }

        string normalized = path.Replace('\\', '/');
        int slashIndex = normalized.LastIndexOf('/');
        int extensionIndex = normalized.LastIndexOf('.');
        if (extensionIndex > slashIndex)
        {
            return normalized.Substring(0, extensionIndex);
        }

        return normalized;
    }

    /// <summary>
    /// 写入非默认浮点字段。
    /// </summary>
    private static void WriteOptionalDouble(JObject target, string propertyName, double value, double defaultValue)
    {
        if (!IsFinite(value))
        {
            throw new InvalidDataException($"待写入数值无效: {propertyName}={value}。");
        }

        if (Math.Abs(value - defaultValue) <= ValueTolerance)
        {
            target.Remove(propertyName);
            return;
        }

        target[propertyName] = value;
    }

    /// <summary>
    /// 将弧度归一到 (-π, π]。
    /// </summary>
    private static double NormalizeRadians(double value)
    {
        double twoPi = Math.PI * 2d;
        value %= twoPi;
        if (value <= -Math.PI)
        {
            value += twoPi;
        }
        else if (value > Math.PI)
        {
            value -= twoPi;
        }

        return value;
    }

    /// <summary>
    /// 判断字符串是否为十六进制颜色。
    /// </summary>
    private static bool IsHexColor(string color)
    {
        for (int index = 0; index < color.Length; index++)
        {
            char value = color[index];
            bool isDigit = value >= '0' && value <= '9';
            bool isUpper = value >= 'A' && value <= 'F';
            bool isLower = value >= 'a' && value <= 'f';
            if (!isDigit && !isUpper && !isLower)
            {
                return false;
            }
        }

        return true;
    }

    /// <summary>
    /// 判断双精度数是否有效。
    /// </summary>
    private static bool IsFinite(double value)
    {
        return !double.IsNaN(value) && !double.IsInfinity(value);
    }

    /// <summary>
    /// 将文本行尾统一为 LF。
    /// </summary>
    private static string NormalizeLineEndings(string value)
    {
        return value.Replace("\r\n", "\n").Replace('\r', '\n');
    }

    private readonly struct EasingArcSegment
    {
        internal readonly Vector2D Start;
        internal readonly Vector2D FirstControl;
        internal readonly Vector2D SecondControl;

        /// <summary>
        /// 创建归一化圆弧 Bezier 分段。
        /// </summary>
        internal EasingArcSegment(Vector2D start, Vector2D firstControl, Vector2D secondControl)
        {
            Start = start;
            FirstControl = firstControl;
            SecondControl = secondControl;
        }
    }

    private readonly struct EasingBezierSegment
    {
        internal readonly double Start;
        internal readonly double End;
        internal readonly double StartProgress;
        internal readonly double FirstControlProgress;
        internal readonly double SecondControlProgress;

        /// <summary>
        /// 创建归一化缓动 Bezier 分段。
        /// </summary>
        internal EasingBezierSegment(
            double start,
            double end,
            double startProgress,
            double firstControlProgress,
            double secondControlProgress
        )
        {
            Start = start;
            End = end;
            StartProgress = startProgress;
            FirstControlProgress = firstControlProgress;
            SecondControlProgress = secondControlProgress;
        }
    }

    private enum TimelineKind
    {
        Translate,
        Rotate,
        Scale,
        Shear,
        Rgba,
    }

    private sealed class SourceData
    {
        /// <summary>
        /// Cocos setup 骨骼索引。
        /// </summary>
        internal readonly Dictionary<string, JObject> SourceBoneMap;

        /// <summary>
        /// ExportJson 中 bone_data 的原始顺序。
        /// </summary>
        internal readonly List<string> SourceBoneOrder;

        /// <summary>
        /// 每根骨骼按 display index 排列的附件名称。
        /// </summary>
        internal readonly Dictionary<string, List<string>> DisplayNameMap;

        /// <summary>
        /// Cocos movement 索引。
        /// </summary>
        internal readonly Dictionary<string, JObject> MovementMap;

        /// <summary>
        /// 创建完成校验的 Cocos 源数据索引。
        /// </summary>
        internal SourceData(
            Dictionary<string, JObject> sourceBoneMap,
            List<string> sourceBoneOrder,
            Dictionary<string, List<string>> displayNameMap,
            Dictionary<string, JObject> movementMap
        )
        {
            SourceBoneMap = sourceBoneMap;
            SourceBoneOrder = sourceBoneOrder;
            DisplayNameMap = displayNameMap;
            MovementMap = movementMap;
        }
    }

    private sealed class SetupTransformData
    {
        /// <summary>
        /// 按 sceneext 规则计算的 Cocos setup 世界变换。
        /// </summary>
        internal readonly Dictionary<string, CocosWorldTransform> WorldTransformMap;

        /// <summary>
        /// 反解后的 Spine setup 局部变换。
        /// </summary>
        internal readonly Dictionary<string, SpineLocalTransform> SpineLocalTransformMap;

        /// <summary>
        /// 每根骨骼动画旋转增量的方向；父世界为反射时取 -1。
        /// </summary>
        internal readonly Dictionary<string, double> RotationDirectionMap;

        /// <summary>
        /// 创建层级变换数据。
        /// </summary>
        internal SetupTransformData(
            Dictionary<string, CocosWorldTransform> worldTransformMap,
            Dictionary<string, SpineLocalTransform> spineLocalTransformMap,
            Dictionary<string, double> rotationDirectionMap
        )
        {
            WorldTransformMap = worldTransformMap;
            SpineLocalTransformMap = spineLocalTransformMap;
            RotationDirectionMap = rotationDirectionMap;
        }
    }

    private sealed class CocosWorldTransform
    {
        internal readonly double X;
        internal readonly double Y;
        internal readonly double ScaleX;
        internal readonly double ScaleY;
        internal readonly double SkewX;
        internal readonly double SkewY;
        internal readonly Matrix2D Matrix;

        /// <summary>
        /// 创建 Cocos 世界变换。
        /// </summary>
        internal CocosWorldTransform(double x, double y, double scaleX, double scaleY, double skewX, double skewY, Matrix2D matrix)
        {
            X = x;
            Y = y;
            ScaleX = scaleX;
            ScaleY = scaleY;
            SkewX = skewX;
            SkewY = skewY;
            Matrix = matrix;
        }
    }

    private readonly struct SpineLocalTransform
    {
        internal readonly double Rotation;
        internal readonly double ScaleX;
        internal readonly double ScaleY;
        internal readonly double ShearY;

        /// <summary>
        /// 创建 Spine 局部变换。
        /// </summary>
        internal SpineLocalTransform(double rotation, double scaleX, double scaleY, double shearY)
        {
            Rotation = rotation;
            ScaleX = scaleX;
            ScaleY = scaleY;
            ShearY = shearY;
        }
    }

    private readonly struct Matrix2D
    {
        internal readonly double A;
        internal readonly double B;
        internal readonly double C;
        internal readonly double D;

        internal double Determinant => A * D - B * C;

        /// <summary>
        /// 创建二维轴矩阵。
        /// </summary>
        internal Matrix2D(double a, double b, double c, double d)
        {
            A = a;
            B = b;
            C = c;
            D = d;
        }

        /// <summary>
        /// 矩阵相乘。
        /// </summary>
        internal static Matrix2D Multiply(Matrix2D left, Matrix2D right)
        {
            return new Matrix2D(
                left.A * right.A + left.B * right.C,
                left.A * right.B + left.B * right.D,
                left.C * right.A + left.D * right.C,
                left.C * right.B + left.D * right.D
            );
        }
    }

    private readonly struct Vector2D
    {
        internal readonly double X;
        internal readonly double Y;

        /// <summary>
        /// 创建二维位置。
        /// </summary>
        internal Vector2D(double x, double y)
        {
            X = x;
            Y = y;
        }
    }

    private sealed class BakedTransformTimelines
    {
        private readonly JArray translate = new JArray();
        private readonly JArray rotate = new JArray();
        private readonly JArray scale = new JArray();
        private readonly JArray shear = new JArray();
        private bool hasTranslate;
        private bool hasRotate;
        private bool hasScale;
        private bool hasShear;

        /// <summary>
        /// 非默认位移轨道；全部帧为默认值时返回 null。
        /// </summary>
        internal JArray Translate => hasTranslate ? translate : null;

        /// <summary>
        /// 非默认旋转轨道；全部帧为默认值时返回 null。
        /// </summary>
        internal JArray Rotate => hasRotate ? rotate : null;

        /// <summary>
        /// 非默认缩放轨道；全部帧为默认值时返回 null。
        /// </summary>
        internal JArray Scale => hasScale ? scale : null;

        /// <summary>
        /// 非默认剪切轨道；全部帧为默认值时返回 null。
        /// </summary>
        internal JArray Shear => hasShear ? shear : null;

        /// <summary>
        /// 追加一帧逐层级反解后的 Spine 局部变换。
        /// </summary>
        internal void AddFrame(
            double time,
            double translateX,
            double translateY,
            double rotation,
            double scaleX,
            double scaleY,
            double shearY
        )
        {
            translate.Add(CreateTimelineFrame(time, new[] { translateX, translateY }, TimelineKind.Translate));
            rotate.Add(CreateTimelineFrame(time, new[] { rotation }, TimelineKind.Rotate));
            scale.Add(CreateTimelineFrame(time, new[] { scaleX, scaleY }, TimelineKind.Scale));
            shear.Add(CreateTimelineFrame(time, new[] { 0d, shearY }, TimelineKind.Shear));
            hasTranslate |= Math.Abs(translateX) > ValueTolerance || Math.Abs(translateY) > ValueTolerance;
            hasRotate |= Math.Abs(rotation) > ValueTolerance;
            hasScale |= Math.Abs(scaleX - 1d) > ValueTolerance || Math.Abs(scaleY - 1d) > ValueTolerance;
            hasShear |= Math.Abs(shearY) > ValueTolerance;
        }
    }

    private readonly struct CocosFrameTransform
    {
        internal static readonly CocosFrameTransform Identity = new CocosFrameTransform(0d, 0d, 1d, 1d, 0d, 0d);
        internal readonly double X;
        internal readonly double Y;
        internal readonly double ScaleX;
        internal readonly double ScaleY;
        internal readonly double SkewX;
        internal readonly double SkewY;

        /// <summary>
        /// 创建 Cocos movement 局部增量。
        /// </summary>
        internal CocosFrameTransform(double x, double y, double scaleX, double scaleY, double skewX, double skewY)
        {
            X = x;
            Y = y;
            ScaleX = scaleX;
            ScaleY = scaleY;
            SkewX = skewX;
            SkewY = skewY;
        }

        /// <summary>
        /// 按 Cocos BoneTweenController 分量线性插值两个 movement 帧。
        /// </summary>
        internal static CocosFrameTransform Interpolate(CocosFrameTransform from, CocosFrameTransform to, double percent)
        {
            return new CocosFrameTransform(
                from.X + (to.X - from.X) * percent,
                from.Y + (to.Y - from.Y) * percent,
                from.ScaleX + (to.ScaleX - from.ScaleX) * percent,
                from.ScaleY + (to.ScaleY - from.ScaleY) * percent,
                from.SkewX + (to.SkewX - from.SkewX) * percent,
                from.SkewY + (to.SkewY - from.SkewY) * percent
            );
        }
    }

    private sealed class MovementData
    {
        /// <summary>
        /// movement 名称。
        /// </summary>
        internal readonly string Name;

        /// <summary>
        /// movement 总帧数。
        /// </summary>
        internal readonly int Duration;

        /// <summary>
        /// Cocos 运行时每动画帧对应的秒数。
        /// </summary>
        internal readonly double SecondsPerFrame;

        /// <summary>
        /// Cocos setup 骨骼索引，用于转换 movement 相对缩放。
        /// </summary>
        internal readonly Dictionary<string, JObject> SourceBoneMap;

        /// <summary>
        /// ExportJson setup 骨骼原始顺序。
        /// </summary>
        internal readonly List<string> SourceBoneOrder;

        /// <summary>
        /// movement 骨骼轨道索引。
        /// </summary>
        internal readonly Dictionary<string, JObject> MovementBoneMap;

        /// <summary>
        /// ExportJson 中 mov_bone_data 的原始顺序。
        /// </summary>
        internal readonly List<string> MovementBoneOrder;

        /// <summary>
        /// 每根骨骼动画旋转增量的方向。
        /// </summary>
        internal readonly Dictionary<string, double> RotationDirectionMap;

        /// <summary>
        /// 创建完成校验的 movement 数据。
        /// </summary>
        internal MovementData(
            string name,
            int duration,
            double secondsPerFrame,
            Dictionary<string, JObject> sourceBoneMap,
            List<string> sourceBoneOrder,
            Dictionary<string, JObject> movementBoneMap,
            List<string> movementBoneOrder,
            Dictionary<string, double> rotationDirectionMap
        )
        {
            Name = name;
            Duration = duration;
            SecondsPerFrame = secondsPerFrame;
            SourceBoneMap = sourceBoneMap;
            SourceBoneOrder = sourceBoneOrder;
            MovementBoneMap = movementBoneMap;
            MovementBoneOrder = movementBoneOrder;
            RotationDirectionMap = rotationDirectionMap;
        }
    }

    private sealed class RepairResult
    {
        /// <summary>
        /// 修复的 setup 骨骼数。
        /// </summary>
        internal int RepairedSetupBoneCount;

        /// <summary>
        /// 修复的 setup 插槽数。
        /// </summary>
        internal int RepairedSetupSlotCount;

        /// <summary>
        /// 重建的位移轨道数。
        /// </summary>
        internal int RebuiltTranslateTimelineCount;

        /// <summary>
        /// 删除的误造位移轨道数。
        /// </summary>
        internal int RemovedTranslateTimelineCount;

        /// <summary>
        /// 重建的旋转轨道数。
        /// </summary>
        internal int RebuiltRotateTimelineCount;

        /// <summary>
        /// 删除的误造旋转轨道数。
        /// </summary>
        internal int RemovedRotateTimelineCount;

        /// <summary>
        /// 重建的缩放轨道数。
        /// </summary>
        internal int RebuiltScaleTimelineCount;

        /// <summary>
        /// 删除的误造缩放轨道数。
        /// </summary>
        internal int RemovedScaleTimelineCount;

        /// <summary>
        /// 重建的剪切轨道数。
        /// </summary>
        internal int RebuiltShearTimelineCount;

        /// <summary>
        /// 删除的误造剪切轨道数。
        /// </summary>
        internal int RemovedShearTimelineCount;

        /// <summary>
        /// 重建的附件轨道数。
        /// </summary>
        internal int RebuiltAttachmentTimelineCount;

        /// <summary>
        /// 删除的误造附件轨道数。
        /// </summary>
        internal int RemovedAttachmentTimelineCount;

        /// <summary>
        /// 重建的颜色轨道数。
        /// </summary>
        internal int RebuiltColorTimelineCount;

        /// <summary>
        /// 删除的误造颜色轨道数。
        /// </summary>
        internal int RemovedColorTimelineCount;

        /// <summary>
        /// 重建的绘制顺序轨道数。
        /// </summary>
        internal int RebuiltDrawOrderTimelineCount;

        /// <summary>
        /// 删除的误造绘制顺序轨道数。
        /// </summary>
        internal int RemovedDrawOrderTimelineCount;
    }
}
#pragma warning restore ET0004

```
