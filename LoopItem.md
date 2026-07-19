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

示例1
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

示例2
- 背包 DlgBagMainSystem.cs     DlgBagMain.cs    SItem_BagSystem.cs
```
[ComponentOf(typeof(EUI.UIBaseWindow))]
public class DlgBagMain : Entity, IAwake, EUI.IUILogic
{
    public DlgBagMainViewComponent View { get => this.GetParentComponent<DlgBagMainViewComponent>(); }

    public Dictionary<int, Scroll_SItem_Bag> bagItemDic = new Dictionary<int, Scroll_SItem_Bag>();
    public UserItemComponent userItemComponent;
    public Dictionary<int, int> itemDtoDic = new Dictionary<int, int>();//背包数据
    public List<int> itemIdList = new List<int>(); // 按索引取 itemId 用
}
```
```
using System.Collections.Generic;
using ET.Client.EUI;
using UnityEngine;

namespace ET.Client
{
	[FriendOf(typeof(DlgBagMain))]
	[FriendOf(typeof(Scroll_SItem_Bag))]
	public static class DlgBagMainSystem
	{
		public static void RegisterUIEvent(this DlgBagMain self)
		{
			self.View.E_CloseBtnButton.onClick.AddListener(() => self.OnBtnCloseClick());
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.AddItemRefreshListener((Transform trans, int idx) =>
			{
				self.OnLoopScrollList_BagItemRefresh(trans, idx);
			});
		}

		public static void OnLoopScrollList_BagItemRefresh(this DlgBagMain self, Transform trans, int idx)
		{
			var item = self.bagItemDic[idx].BindTrans(trans);
			int itemId = self.itemIdList[idx];
			int itemCount = self.itemDtoDic[itemId];
			item.SetInfo(idx, itemId, itemCount);
		}
		public static void OnBtnCloseClick(this DlgBagMain self)
		{
			self.ClientScene().CurrentScene().GetComponent<EUI.UIComponent>().CloseWindow(WindowID.WindowID_BagMain);
		}

		public static void ShowWindow(this DlgBagMain self, Entity contextData = null)
		{
			self.userItemComponent = self.ClientScene().GetComponent<UserItemComponent>();
			self.SetBagItems().Coroutine();
		}

		public static async ETTask SetBagItems(this DlgBagMain self)
		{
			self.itemDtoDic = await self.userItemComponent.GetUserItems();

			// 把 itemId 存到 List 里，方便按 idx 访问
			self.itemIdList = new List<int>(self.itemDtoDic.Keys);

			self.AddUIScrollItems(ref self.bagItemDic, self.itemDtoDic.Count);
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.SetVisible(true, self.itemDtoDic.Count);
		}

		public static void HideWindow(this DlgBagMain self, Entity contextData = null)
		{
			self.RemoveUIScrollItems(ref self.bagItemDic);
		}

		#region 背包列表常用操作

		/// <summary> 重建列表并全量刷新（数量变化时调用） </summary>
		public static void RefreshBagList(this DlgBagMain self)
		{
			self.AddUIScrollItems(ref self.bagItemDic, self.itemDtoDic.Count);
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.SetVisible(true, self.itemDtoDic.Count);
		}

		/// <summary> 仅刷新当前可见 cell（数据变了但数量没变时用，不重建不跳位置） </summary>
		public static void RefreshVisibleCells(this DlgBagMain self)
		{
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.RefreshCells();
		}

		/// <summary> 滚动到指定 itemId 的位置 </summary>
		public static void ScrollToItem(this DlgBagMain self, int itemId)
		{
			int idx = self.itemIdList.IndexOf(itemId);
			if (idx < 0) return;
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.SrollToCell(idx, 0);
		}

		/// <summary> 滚动到指定索引 </summary>
		public static void ScrollToIndex(this DlgBagMain self, int idx)
		{
			if (idx < 0 || idx >= self.itemDtoDic.Count) return;
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.SrollToCell(idx, 0);
		}

		/// <summary> 滚动到最后一项 </summary>
		public static void ScrollToLast(this DlgBagMain self)
		{
			int lastIdx = self.itemDtoDic.Count - 1;
			if (lastIdx < 0) return;
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.SrollToCell(lastIdx, 0);
		}

		/// <summary> 添加物品并刷新列表，自动滚到新物品位置 </summary>
		public static void AddItem(this DlgBagMain self, int itemId, int count)
		{
			self.itemDtoDic[itemId] = count;
			if (!self.itemIdList.Contains(itemId))
				self.itemIdList.Add(itemId);

			self.RefreshBagList();
			self.ScrollToItem(itemId);
		}

		/// <summary> 移除物品并刷新列表 </summary>
		public static void RemoveItem(this DlgBagMain self, int itemId)
		{
			self.itemDtoDic.Remove(itemId);
			self.itemIdList.Remove(itemId);

			self.RefreshBagList();
		}

		/// <summary> 更新物品数量并刷新（不重建 Entity，轻量刷新可见 cell） </summary>
		public static void UpdateItemCount(this DlgBagMain self, int itemId, int count)
		{
			if (!self.itemDtoDic.ContainsKey(itemId)) return;
			self.itemDtoDic[itemId] = count;
			self.View.ELoopScrollList_BagLoopVerticalScrollRect.RefreshCells();
		}

		/// <summary>
		/// 服务端推送道具数据变更入口。根据数量变化自动判断是局部刷新还是全量刷新：
		/// - 已有物品数量变更（>0）→ 局部刷新可见 cell，不跳位置
		/// - 已有物品数量归零 → 移除并全量刷新
		/// - 新物品 → 添加到列表并全量刷新
		/// </summary>
		public static void OnServerItemChanged(this DlgBagMain self, int itemId, int newCount)
		{
			bool exists = self.itemDtoDic.ContainsKey(itemId);
			if (exists)
			{
				// 已有物品
				if (newCount > 0)
				{
					// 数量变更：只更新数据 + 局部刷新可见 cell，不重建列表
					self.itemDtoDic[itemId] = newCount;
					self.RefreshVisibleCells();
				}
				else
				{
					// 数量归零：移除
					self.RemoveItem(itemId);
				}
			}
			else
			{
				// 新物品
				if (newCount > 0)
				{
					self.AddItem(itemId, newCount);
				}
				// newCount <= 0 的新物品直接忽略
			}
		}

		/// <summary>
		/// 服务端批量推送道具变更。
		/// 先统计是否有数量增减，有则全量刷新，无则只局部刷新。
		/// </summary>
		public static void OnServerItemBatchChanged(this DlgBagMain self, Dictionary<int, int> changedItems)
		{
			bool countChanged = false;

			foreach (var kv in changedItems)
			{
				bool exists = self.itemDtoDic.ContainsKey(kv.Key);
				// 新增或移除 → 数量变了
				if (!exists || kv.Value <= 0)
				{
					countChanged = true;
					// 先更新数据，后续一起处理
					if (kv.Value > 0)
						self.itemDtoDic[kv.Key] = kv.Value;
					else
						self.itemDtoDic.Remove(kv.Key);
				}
				else
				{
					self.itemDtoDic[kv.Key] = kv.Value;
				}
			}

			if (countChanged)
			{
				// 有增减 → 重建 itemIdList + 全量刷新
				self.itemIdList = new List<int>(self.itemDtoDic.Keys);
				self.RefreshBagList();
			}
			else
			{
				// 纯数量变化 → 局部刷新可见 cell
				self.RefreshVisibleCells();
			}
		}
		#endregion
	}
}
```
```
using UnityEngine;
using UnityEngine.UI;
namespace ET.Client
{
    [FriendOf(typeof(Scroll_SItem_Bag))]
    public static class SItem_BagSystem
    {
        public static void SetInfo(this Scroll_SItem_Bag self, int idx, int itemId, int count)
        {
            self.DataId = itemId;
            self.Index = idx;

            if (ItemConfigCategory.Instance.TryGet(itemId, out ItemConfig config))
            {
                self.E_IconImage.SetImgSpriteAsync(config.ItemTexture).Coroutine();
                self.E_text_has_numText.text = count.ToString();
                self.E_QualityImage.sprite = AssistantUtils.GetQualitySprite(config.ItemQuality);
            }

            var button = self.uiTransform.GetComponent<Button>();
            button.onClick.RemoveAllListeners();  // 先清掉旧的
            button.onClick.AddListener(() =>
            {
                Debug.Log($"点击了AA--{self.DataId} {self.Index}");
            });
        }
    }
}
```