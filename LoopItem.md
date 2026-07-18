创建脚本 SItemCodeSpawner.cs 与LoopScrollItemCodeSpawner.cs同一目录下  代码中就是新加个SItem目录

```
using System.Collections.Generic;
using System.Text.RegularExpressions;
using UnityEngine;
using UnityEditor;
using System.IO;
using System.Text;

public partial class UICodeSpawner
{
    static public void SItemCodeSpawner(GameObject gameObject)
    {
        Path2WidgetCachedDict?.Clear();
        Path2WidgetCachedDict = new Dictionary<string, List<Component>>();
        FindAllWidgets(gameObject.transform, "");
        SItemBehaviour(gameObject);
        SItemViewSystem(gameObject);
        AssetDatabase.Refresh();
    }

    static void SItemViewSystem(GameObject gameObject)
    {
        if (null == gameObject)
        {
            return;
        }
        string strDlgName = gameObject.name;
        string strFilePath = Application.dataPath + $"{CodeRootPath}/HotfixView/{UIRootFolder}/UIBehaviour/SItem/";
        if (!System.IO.Directory.Exists(strFilePath))
        {
            System.IO.Directory.CreateDirectory(strFilePath);
        }
        strFilePath = Application.dataPath + $"{CodeRootPath}/HotfixView/{UIRootFolder}/UIBehaviour/SItem/" + strDlgName + "ViewSystem.cs";
        StreamWriter sw = new StreamWriter(strFilePath, false, Encoding.UTF8);
        StringBuilder strBuilder = new StringBuilder();
        strBuilder.AppendLine()
            .AppendLine("using UnityEngine;");
        strBuilder.AppendLine("using UnityEngine.UI;");
        strBuilder.AppendLine("namespace ET.Client");
        strBuilder.AppendLine("{");
        strBuilder.AppendLine("\t[ObjectSystem]");
        strBuilder.AppendFormat("\tpublic class Scroll_{0}DestroySystem : DestroySystem<Scroll_{1}> \r\n", strDlgName, strDlgName);
        strBuilder.AppendLine("\t{");
        strBuilder.AppendFormat("\t\tprotected override void Destroy(Scroll_{0} self )", strDlgName);
        strBuilder.AppendLine("\n\t\t{");
        strBuilder.AppendFormat("\t\t\tself.DestroyWidget();\r\n");
        strBuilder.AppendLine("\t\t}");
        strBuilder.AppendLine("\t}");
        strBuilder.AppendLine("}");
        sw.Write(strBuilder);
        sw.Flush();
        sw.Close();
    }
    static void SItemBehaviour(GameObject gameObject)
    {
        if (null == gameObject)
        {
            return;
        }
        string strDlgName = gameObject.name;
        string strFilePath = Application.dataPath + $"{CodeRootPath}/ModelView/{UIRootFolder}/UIBehaviour/SItem/";
        if (!System.IO.Directory.Exists(strFilePath))
        {
            System.IO.Directory.CreateDirectory(strFilePath);
        }
        strFilePath = Application.dataPath + $"{CodeRootPath}/ModelView/{UIRootFolder}//UIBehaviour/SItem/" + strDlgName + ".cs";

        // 尝试从旧文件中提取自定义内容
        string customContent = "";
        if (File.Exists(strFilePath))
        {
            string oldContent = File.ReadAllText(strFilePath, Encoding.UTF8);
            Match match = Regex.Match(oldContent, @"#region\s+自定义内容[^\n]*\n(?<content>.*?)\n\s*#endregion", RegexOptions.Singleline);
            if (match.Success)
            {
                // trim 掉末尾换行符，避免 Windows \r\n 换行导致 content 末尾残留 \r，二次生成时正则匹配失败
                customContent = match.Groups["content"].Value.TrimEnd('\r', '\n');
            }
        }

        StreamWriter sw = new StreamWriter(strFilePath, false, Encoding.UTF8);
        StringBuilder strBuilder = new StringBuilder();
        strBuilder.AppendLine()
            .AppendLine("using UnityEngine;");
        strBuilder.AppendLine("using UnityEngine.UI;");
        strBuilder.AppendLine("using ET.Client.EUI;");
        strBuilder.AppendLine("namespace ET.Client");
        strBuilder.AppendLine("{");
        strBuilder.AppendLine("\t[EnableMethod]");
        strBuilder.AppendFormat("\tpublic  class Scroll_{0} : Entity,IAwake,IDestroy,IUIScrollItem \r\n", strDlgName)
            .AppendLine("\t{");
        strBuilder.AppendLine("\t\tpublic long DataId {get;set;}");
        strBuilder.AppendLine("\t\tprivate bool isCacheNode = false;");
        strBuilder.AppendLine("\t\tpublic void SetCacheMode(bool isCache)");
        strBuilder.AppendLine("\t\t{");
        strBuilder.AppendLine("\t\t\tthis.isCacheNode = isCache;");
        strBuilder.AppendLine("\t\t}\n");
        strBuilder.AppendFormat("\t\tpublic Scroll_{0} BindTrans(Transform trans)\r\n", strDlgName);
        strBuilder.AppendLine("\t\t{");
        strBuilder.AppendLine("\t\t\tthis.uiTransform = trans;");
        strBuilder.AppendLine("\t\t\treturn this;");
        strBuilder.AppendLine("\t\t}\n");
        CreateWidgetBindCode(ref strBuilder, gameObject.transform);
        CreateDestroyWidgetCode(ref strBuilder, true);
        CreateDeclareCode(ref strBuilder);
        strBuilder.AppendLine("\t\tpublic Transform uiTransform = null;");
        strBuilder.AppendLine("\t\t#region 自定义内容");
        if (!string.IsNullOrEmpty(customContent))
        {
            // 保留原有的自定义内容（使用 AppendLine 保证末尾有换行，避免与 #endregion 粘连）
            strBuilder.AppendLine(customContent);
        }
        strBuilder.AppendLine("\t\t#endregion");
        strBuilder.AppendLine("\t}");
        strBuilder.AppendLine("}");
        sw.Write(strBuilder);
        sw.Flush();
        sw.Close();
    }
}
```
UICodeSpawner.cs脚本  搜索UIItemPrefix 45行左右
```
else if (uiName.StartsWith(UIItemPrefix))//搜索 

else if (uiName.StartsWith(UISItemPrefix))//加入
{
    Debug.LogWarning($"-------- 开始生成滚动列表项: {uiName} 相关代码 -------------");
    SItemCodeSpawner(gameObject);
    Debug.LogWarning($" 开始生成滚动列表项: {uiName} 完毕！！！");
    return;
}
if (transRoot.gameObject.name.StartsWith(UIItemPrefix) //搜索

if (transRoot.gameObject.name.StartsWith(UIItemPrefix) || transRoot.gameObject.name.StartsWith(UISItemPrefix))//加入


private const string UISItemPrefix = "SItem";//LoopScollItem//最后加句
```

prefab是以SItem开头  生成代码
UnityEditor里 搜索ItemPrefabs 加入新的文件夹SItem

示例
名字为 SItem_MailInfo   (水平的)
- 在目标prefab里,右键EUI/LoopHorizontalScrollRect...创建出来,看属性面板填入PrefabName
- 新建的SItem_MailInfo.prefab 这个SItem里要加个组件LayoutElement,使用PreferredWidth选项,右键SpawnEUICode,生成代码
- 页面脚本里
```
view里     
    public Dictionary<int, Scroll_SItem_MailInfo> itemMailDic = new();
System里在RegisterUIEvent方法里加入代码
    self.View.ELoopSL_MailLoopHorizontalScrollRect.AddItemRefreshListener((Transform trans, int idx) =>{    self.OnLoopMailHandler(trans, idx);});
        方法
    public static void OnLoopMailHandler(this DlgOperateDialog self, Transform transform, int idx)
    {
        var item = self.itemMailDic[idx].BindTrans(transform);
        // item.E_titleText.text = "邮件" + idx;
        item.SetMainInfo(idx);
    }
    ShowWindow方法里 数据处理
    public static void ShowWindow(this DlgOperateDialog self, Entity contextData = null)
    {
        self.AddUIScrollItems(ref self.itemMailDic, 50);
        self.View.ELoopSL_MailLoopHorizontalScrollRect.SetVisible(true, 50);
    }
    HideWindow方法里 数据移除
    public static void HideWindow(this DlgOperateDialog self)
    {
        self.RemoveUIScrollItems(ref self.itemMailDic);
    }

 //ItemSysyem  新建个脚本SItem_MailInfoSystem.cs
     [FriendOf(typeof(Scroll_SItem_MailInfo))]
    public static class SItem_MailInfoSystem
    {
        public static void SetMainInfo(this Scroll_SItem_MailInfo self, int mailId)//页面指定的方法
        {
            self.DataId = mailId;
            self.E_titleText.text = $"标题{mailId}";
            var button = self.uiTransform.GetComponent<Button>();
            button.onClick.RemoveAllListeners();  // 若想点击对 先清掉旧的
            button.onClick.AddListener(() =>
            {
                Debug.Log($"点击了AA--{self.DataId} {mailId}");    
            });
        }
    }   

``` 