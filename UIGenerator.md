###  UI生成模板
1. 生成UI预制体

```c#
// 定义ui 界面预制体生成路径
private static string prefabsPath = "Assets/ResourceLoad/UI";

//选择创建好的UI后再生成UI预制体文件
    [MenuItem("UIGenarator/CreatPrefabs(选中ui)")]

    public static  void CreatPrefab()
    {
        var selectedGameObject = Selection.activeGameObject;
        if (!selectedGameObject) return;
        var filePath = Path.Combine(prefabsPath, selectedGameObject.name + ".prefab");
        if (File.Exists(filePath))
            File.Delete(filePath);
        GameObject cloneObj = Instantiate(selectedGameObject);
        EditorUtility.DisplayProgressBar("正在创建预制体", filePath, 1);
        var empty = PrefabUtility.CreateEmptyPrefab(filePath);
        PrefabUtility.ReplacePrefab(selectedGameObject, empty);
        GameObject.DestroyImmediate(cloneObj);
        EditorUtility.ClearProgressBar();

        Debug.Log($"prefabs 创建路径  {filePath}");


    }

```
2. 生成UI脚本
```c#
    //定义脚本生成路径
    private static string prefabsPath = "Assets/ResourceLoad/Views";

    //这里包含了UI框架代码
    [MenuItem("UIGenarator/CreatViewScripts")]

    public static void BindViewScripts()
    {
       
        var go = Selection.activeGameObject;
        if (go == null)
        {
            Debug.LogError("请选择对象");
            return;
        }
        if (!Directory.Exists($"{codePath}/{go.name}")) {
            Directory.CreateDirectory($"{codePath}/{go.name}");
        }
       
        using (StreamWriter sw = new StreamWriter($"{codePath}/{go.name}/{go.name}.cs"))
        {
            sw.WriteLine("using EH;");
            sw.WriteLine("using EH.Extend;");
            sw.WriteLine("using EH.UI;");
            sw.WriteLine("using UnityEngine;");
            sw.WriteLine("using UnityEngine.UI;");
            sw.WriteLine("//自动生成于：" + DateTime.Now);
            sw.WriteLine("\n");
            sw.WriteLine("namespace App.Views");
            sw.WriteLine("{");

            sw.WriteLine($"\tpublic class {go.name} : UIBase");
            sw.WriteLine("\t{");
            sw.WriteLine("\t\t//--Replace Field");
            sw.WriteLine("\t\tprotected override void Init()");
            sw.WriteLine("\t\t{");
            sw.WriteLine("\t\t\tbase.Init();");
            sw.WriteLine("\t\t}");

            sw.WriteLine("\t\tprotected override void OnLoaded()");
            sw.WriteLine("\t\t{");
            sw.WriteLine("\t\t\tbase.OnLoaded();");
            sw.WriteLine("\t\t\t//--Replace Component");
            sw.WriteLine("\t\t}");

            sw.WriteLine("\t\tprotected override void OnShow()");
            sw.WriteLine("\t\t{");
            sw.WriteLine("\t\t\tbase.OnShow();");
            sw.WriteLine("\t\t}");

            sw.WriteLine("\t\tprotected override void OnHide()");
            sw.WriteLine("\t\t{");
            sw.WriteLine("\t\t\tbase.OnHide();");
            sw.WriteLine("\t\t}");

            sw.WriteLine("\t}");
            sw.WriteLine("}");


        }
        AssetDatabase.Refresh();
        Debug.Log($"prefabs 创建路径  {$"{codePath}/{go.name}/{go.name}.cs"}");

    }
```

3.在UI脚本中自动获取组件和绑定事件
```c#
    //自动获取UI控件 给UI控件自定义一组映射
    private static Dictionary<string, string> _ComponentDict = new Dictionary<string, string>()
        {
        {"Trans","Transform" },
        {"OldAnim","Animation"},
        {"NewAnim","Animator"},

        {"Rect","RectTransform"},
        {"Canvas","Canvas"},
        {"Group","CanvasGroup"},
        {"VGroup","VerticalLayoutGroup"},
        {"HGroup","HorizontalLayoutGroup"},
        {"GGroup","GridLayoutGroup"},
        {"TGroup","ToggleGroup"},

        {"Btn","Button"},
        {"Img","Image"},
        {"RImg","RawImage"},
        {"Txt","Text"},
        {"Input","InputField"},
        {"Slider","Slider"},
        {"Mask","Mask"},
        {"Mask2D","RectMask2D"},
        {"Tog","Toggle"},
        {"Sbar","Scrollbar"},
        {"SRect","ScrollRect"},
        {"Drop","Dropdown"},
        };
    //UI界面中的控件名称格式为 上方字典中的 Key 加"_" 组件名）：比如 获取一个Button组件，控件的命名为Btn_XXX;
    
    [MenuItem("UIGenarator/BindViewObject")]

    public static void BindViewObject() {

        var go = Selection.activeGameObject;
        if (go == null) {
            Debug.LogError("请选择对象");
            return;
        }
        if (!File.Exists($"{codePath}/{go.name}/{go.name}.cs"))
        {
            Debug.LogError($"{codePath}/{go.name}/{go.name}.cs文件不存在");
            return;
        }
        Transform[] childTransform = go.GetComponentsInChildren<Transform>();
        foreach (var child in childTransform) {
            string comName;
            if (IsValidBind(child, out comName)) {
                Component com = child.GetComponent(comName);
                if (com == null)
                {
                    Debug.LogError($"{child.name}上不存在{comName}的组件");
                    continue;

                }
                string strTxt;
                using (StreamReader rd = new StreamReader($"{codePath}/{go.name}/{go.name}.cs"))
                {
                    strTxt = rd.ReadToEnd();
                    strTxt = strTxt.Replace("//--Replace Field",$"{comName} {child.name};  \n \t\t//--Replace Field");
                    strTxt = strTxt.Replace("//--Replace Component", $"{child.name} = FindChildComp<{comName}>(\"{ child.name}\") ; \n \t\t\t//--Replace Component");
                    //Img_Test = FindChildComp<Image>("Img_Test");
                }
                using (StreamWriter sw = new StreamWriter($"{codePath}/{go.name}/{go.name}.cs"))
                {
                    sw.Write(strTxt);
                    sw.Flush();
                    sw.Close();
                }

            }

        }
        AssetDatabase.Refresh();

    }
    //同理 也可以增加按钮等组件事件的绑定 略
```