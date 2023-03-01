
### 1. 常量生成模板
```c#
public static void ConstBuild()
    {
        var sb = new StringBuilder();
        sb.AppendLine("public class Const");
        sb.AppendLine("{");
        for(int i = 0; i < 32; i++)
        {
            var name = LayerMask.LayerToName(i);
            name = name.Replace(" ", "_");
            if (!string.IsNullOrEmpty(name))
                sb.AppendLine($"\tpublic const int LAYER_{name.ToUpper()} = {i};\n");
        }
        //先写入内置Tag
        sb.AppendLine($"\tpublic const string {"Tag_Untagged".ToUpper()}= \"Untagged\";\n");
        sb.AppendLine($"\tpublic const string {"Tag_Respawn".ToUpper()}= \"Respawn\";\n");
        sb.AppendLine($"\tpublic const string {"Tag_Finish".ToUpper()}= \"Finish\";\n");
        sb.AppendLine($"\tpublic const string {"Tag_EditorOnly".ToUpper()}= \"EditorOnly\";\n");
        sb.AppendLine($"\tpublic const string {"Tag_MainCamera".ToUpper()}= \"MainCamera\";\n");
        sb.AppendLine($"\tpublic const string {"Tag_Player".ToUpper()}= \"Player\";\n");
        sb.AppendLine($"\tpublic const string {"Tag_GameController".ToUpper()}= \"GameController\";\n");

        //再写自定义Tag
        var asset = AssetDatabase.LoadAllAssetsAtPath("ProjectSettings/TagManager.asset");
        if(asset != null && (asset.Length > 0))
        {
            for(int i = 0; i <asset.Length; i++)
            {
                var so = new SerializedObject(asset[i]);
                var tags = so.FindProperty("tags");
                for(int j = 0; j < tags.arraySize; ++j)
                {
                    var item = tags.GetArrayElementAtIndex(j).stringValue;
                    sb.AppendLine($"\tpublic const string TAG_{item.ToUpper()} =\"{item}\";\n");
                }
            }
        }
        sb.AppendLine("}");
        File.WriteAllText("Assets/GeneratedConst.cs", sb.ToString());
        AssetDatabase.Refresh();
        
    }
```
