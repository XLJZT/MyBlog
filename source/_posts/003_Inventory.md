---
title: 关于Inventory的制作
description: 
date: 2022-5-31
top_image: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220603143654206.png
cover: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220603143654206.png
categories: 
- Unity
tag: 
- Games
- 游戏制作
- M_studio

---

# Introduce

首先还是要感谢麦扣老师，[视频链接](https://www.bilibili.com/video/BV1YJ41197SU?p=2)。

制作之后的(简陋)界面。

![image-20220603143654206](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220603143654206.png)

# UI界面

**UI层级**

- Canvas  `BagManager.cs`
  - Bag  `MoveBag.cs`
    - Title  标题
    - Exit   右上角 ”×“
    - Grid  物品格
    - Info   物品介绍

> Grid 通过使用GridLayoutGroup组件进行排列组合

**预制体Slot的制作**

Slot在这里作为每个格子显示的整体，移动拖拽点击显示等功能都是通过这个Slot。

- Slot  `Slot.cs`
  - Item  Button/`ItemOnDrag.cs`
    - ItemImage  显示的图像
    - Number  显示数量

`Slot.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class Slot : MonoBehaviour
{
    public int slotId;
    public Item item;

    public Image image;

    public Text num;

    public GameObject itemInSlot;

    public string slotInfo;
    public void ItemOnClick()
    {
        BagManager.instance.UpdateInfo(slotInfo);
    }

    public void SetItem(Item item)
    {
        if (item == null)
        {
            itemInSlot.SetActive(false);
            return;
        }
        //赋值
        image.sprite = item.sprite;
        num.text = item.holdNum.ToString();
        slotInfo = item.info;
    }
}

```



# ScriptableObject存储

存储物品的属性:`Item.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//创建新的菜单
[CreateAssetMenu(fileName = "New Item",menuName = "Item/item")]

public class Item : ScriptableObject
{
    public string _name;//物品名字
    public Sprite sprite;//物品图片
    public int holdNum;//持有数量
    
    //使得文本框变大
    [TextArea]
    public string  info;//物品介绍信息
}

```

背包的存储:`Bag.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName ="New Bag",menuName ="Item/bag")]
public class Bag : ScriptableObject
{
    //创建列表，存储物品
    public List<Item> list = new List<Item>();
}

```

物品添加拾取功能`shiqu.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Shiqu : MonoBehaviour
{
    public Item item;

    public Bag bag;

    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.gameObject.CompareTag("Player"))
        {
            //Debug.Log("item" + item.holdNum);
            if (bag.list.Contains(item))
            {
                item.holdNum += 1;
                //Debug.Log("bag:" + bag.list[0].holdNum);
            }
            else
            {
                //bag.list.Add(item);

                //如果有空位则添加
                for (int i = 0; i < bag.list.Count; i++)
                {
                    if (bag.list[i] == null)
                    {
                        bag.list[i] = item;
                        break;
                    }
                }

            }

            Destroy(gameObject);
        }
        BagManager.instance.Refresh();
    }
}

```

# 背包显示

主要思想是当玩家拾取物品、打开背包之后，都会进行一次刷新。

`BagManager.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class BagManager : MonoBehaviour
{
    public static BagManager instance;

    public Bag bag;
    public GameObject slotGrid;
    //public Slot slot;
    public GameObject emptySlot;

    public Text info;

    //显示当前物品的列表，从bag里获取数据
    public List<GameObject> slots = new List<GameObject>();

    private void Awake()
    {
        if (instance != null)
            Destroy(this);
        instance = this;
    }

    private void OnEnable()
    {
        Refresh();
        info.text = "";
    }
    public void UpdateInfo(string itemInfo)
    {
        info.text = itemInfo;
    }

    /*生成格子，有多少物品生成多少格子
    public void CreateSolt(Item item)
    {
        Slot newslot = Instantiate(slot, instance.slotGrid.transform.position, Quaternion.identity);

        newslot.gameObject.transform.SetParent(instance.slotGrid.transform,false);

        newslot.image.sprite = item.sprite;
        newslot.item = item;
        newslot.num.text = item.holdNum.ToString();
    }
    */
    public void Refresh()
    {
        //先清空原物品对象和存储物品数据的列表
        for (int i = 0; i < slotGrid.transform.childCount; i++)
        {
            if (slotGrid.transform.childCount == 0)
                break;
            //删除子集的空物体
            Destroy(slotGrid.transform.GetChild(i).gameObject);
            //清空列表
            slots.Clear();
        }
        //重新生成
        for (int i = 0; i < bag.list.Count; i++)
        {
            //CreateSolt(bag.list[i]);

            //添加空白格并设置父级
            slots.Add(Instantiate(emptySlot));
            //设置false会使得加载的时候图片不被缩放
            slots[i].transform.SetParent(slotGrid.transform,false);
            //设置图片 数量
            slots[i].GetComponent<Slot>().SetItem(bag.list[i]);
            //设置物品的编号
            slots[i].GetComponent<Slot>().slotId = i;
        }
    }
}

```

# 物品拖拽与数据转换

当物品进行拖拽时，先将其父级更改为父级的父级，防止出现遮挡的问题，然后根据射线遮挡判断鼠标移动的位置。

> Slot的Item添加两个组件：
>
> Canvas Group：BlockRaycasts开启，当鼠标移动的时候知道移动的位置
>
> LayoutElement：IgnoreLayout开启，更改父级元素时，不会跳到进入到排序行列

`ItemOnDrag.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
//引用
using UnityEngine.EventSystems;

//使用三个接口
public class ItemOnDrag : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    //移动的物体的初始父级
    public Transform originParent;

    public Bag bag;
    int currentItemId;
    public void OnBeginDrag(PointerEventData eventData)
    {
        originParent = transform.parent;
        currentItemId = originParent.GetComponent<Slot>().slotId;
        //更换父级，使其置于上层不被遮挡
        transform.SetParent(transform.parent.parent);
        transform.position = eventData.position;
        //修改鼠标移动的物体的遮挡属性
        GetComponent<CanvasGroup>().blocksRaycasts = false;
    }

    public void OnDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
        Debug.Log(eventData.pointerCurrentRaycast.gameObject.name);
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        //判断是否属于设置的界面外
        if (eventData.pointerCurrentRaycast.gameObject != null)
        {
            //鼠标射线碰撞到的物体,可能碰到图片或文字
            if (eventData.pointerCurrentRaycast.gameObject.name == "ItemImage" || eventData.pointerCurrentRaycast.gameObject.name == "Number")
            {
                //更换鼠标移动物体的位置和父级
                transform.SetParent(eventData.pointerCurrentRaycast.gameObject.transform.parent.parent);
                transform.position = eventData.pointerCurrentRaycast.gameObject.transform.parent.parent.position;
                //更换两个物体信息
                var temp = bag.list[currentItemId];
                bag.list[currentItemId] = bag.list[eventData.pointerCurrentRaycast.gameObject.GetComponentInParent<Slot>().slotId];
                bag.list[eventData.pointerCurrentRaycast.gameObject.GetComponentInParent<Slot>().slotId] = temp;
                //更换当前鼠标位置物体的位置和父级为原物体的位置与父级
                eventData.pointerCurrentRaycast.gameObject.transform.parent.position = originParent.position;
                eventData.pointerCurrentRaycast.gameObject.transform.parent.SetParent(originParent);

                GetComponent<CanvasGroup>().blocksRaycasts = true;
                return;
            }
            //如果是空值，则交换空值的位置
            //更换当前鼠标位置物体的位置和父级为原物体的位置与父级
            //eventData.pointerCurrentRaycast.gameObject.transform.GetChild(0).SetParent(originParent);
            //eventData.pointerCurrentRaycast.gameObject.transform.GetChild(0).position = originParent.position;
            if (eventData.pointerCurrentRaycast.gameObject.name == "Slot(Clone)")
            {
                transform.SetParent(eventData.pointerCurrentRaycast.gameObject.transform);
                transform.position = eventData.pointerCurrentRaycast.gameObject.transform.position;

                bag.list[eventData.pointerCurrentRaycast.gameObject.GetComponentInParent<Slot>().slotId] = bag.list[currentItemId];
                //判断是不是同一个位置
                //同一位置则不能设置为null
                if (eventData.pointerCurrentRaycast.gameObject.GetComponentInParent<Slot>().slotId != currentItemId)
                    bag.list[currentItemId] = null;

                GetComponent<CanvasGroup>().blocksRaycasts = true;
                return;
            }
        }
        //如果是则返回原位置
        transform.SetParent(originParent);
        transform.position = originParent.position;
        GetComponent<CanvasGroup>().blocksRaycasts = true;


    }
}

```

# 窗口移动

`MoveBag.cs`

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;

public class MoveBag : MonoBehaviour,IDragHandler
{
    RectTransform rectTransform;

    private void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
    }
    public void OnDrag(PointerEventData eventData)
    {
        //移动anchor为鼠标移动的距离
        //调试的时候可能会不跟手，build之后不会出现这种情况
        rectTransform.anchoredPosition += eventData.delta;
    }

    // Start is called before the first frame update

}

```

