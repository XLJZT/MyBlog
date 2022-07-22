---
title: 对话系统——Dialog System
description: 
date: 2022-6-4
top_image: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/004.png
cover: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/004.png
categories: 
- Unity
tag: 
- Games
- 游戏制作
- M_studio

---

# Introduce

首先还是要感谢麦扣老师，[视频链接](https://www.bilibili.com/video/BV1WJ411Y71J/?spm_id_from=333.788.recommend_more_video.1)。

![image-20220604181812326](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220604181812326.png)

# 文本读取与显示

**这是个txt文件**，记录了对话的内容，根据 A or B 判断说话的人物

```
A
1:zheshidiyijv1?
B
2:zheshidierjv2!!!!!
A
3:disanjvzaizheli.......
B
4:disijvyaojiehsule////
A
5:ending'''''
```



```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class Dialog : MonoBehaviour
{
    
    [Header("组件")]
    public Text textLabel;
    public Image image;

    [Header("文本")]
    public TextAsset textAsset;
    public int index;

    //存储文本
    List<string> textList = new List<string>();
    void Awake()
    {
        GetTextFormFilr(textAsset);
    }
    private void OnEnable()
    {
        //当启用时，显示第一句话
        textLabel.text = textList[index++];
    }
    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            if (index == textList.Count)
            {
                gameObject.SetActive(false);
                index = 0;
            }
            else
                textLabel.text = textList[index++];
        }
    }

    void GetTextFormFilr(TextAsset text)
    {
        textList.Clear();
        index = 0;
        //通过回车符分割
        var list = text.text.Split('\n');
        foreach (var item in list)
        {
            textList.Add(item);
        }
    }
}

```

# 协程逐字显示

使用协程，通过设置变量`timeSpeed`去设置每个字的间隔时间

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class Dialog : MonoBehaviour
{
    
    [Header("组件")]
    public Text textLabel;
    public Image image;

    [Header("文本")]
    public TextAsset textAsset;
    public int index;
    [Header("头像")]
    public Sprite headA, headB;

    public float timeSpeed;
    //判断协程是否结束，防止多个协程同时出现
    //初始时设置为true
    bool textFinshed = true;
    //存储文本
    List<string> textList = new List<string>();
    void Awake()
    {
        GetTextFormFilr(textAsset);
    }
    private void OnEnable()
    {
        //当启用时，显示第一句话
        StartCoroutine(SetText());
    }
    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            if (index == textList.Count)
            {
                gameObject.SetActive(false);
                index = 0;
            }
            else if(textFinshed)
                StartCoroutine(SetText());
        }
    }

    void GetTextFormFilr(TextAsset text)
    {
        textList.Clear();
        index = 0;
        //通过回车符分割
        var list = text.text.Split('\n');
        foreach (var item in list)
        {
            textList.Add(item);
        }
    }

    IEnumerator SetText()
    {
        textFinshed = false;
        //清空textLabel里的文字
        textLabel.text = "";
        //这里不清楚是不是我的文本的问题，获取时总是获取到两个字符，但不知道第二个字符是什么
        switch (textList[index][0])
        {
            case 'A':
                image.sprite = headA;
                index++;
                break;

            case 'B':
                image.sprite = headB;
                index++;
                break;
        }
        //每隔timeSpeed秒进行显示一次
        for (int i = 0; i < textList[index].Length; i++)
        {
            textLabel.text += textList[index][i];

            yield return new WaitForSeconds(timeSpeed);
        }
        //更改index值，并更改状态
        index++;
        textFinshed = true;

    }

}

```

# 中断协程

通过判断输入时的状态判断，选择是否中断协程直接显示

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class Dialog : MonoBehaviour
{
    
    [Header("组件")]
    public Text textLabel;
    public Image image;

    [Header("文本")]
    public TextAsset textAsset;
    public int index;
    [Header("头像")]
    public Sprite headA, headB;

    public float timeSpeed;
    //判断协程是否结束，防止多个协程同时出现
    //初始时设置为true
    bool textFinshed = true;
    //存储文本
    List<string> textList = new List<string>();
    void Awake()
    {
        GetTextFormFilr(textAsset);
    }
    private void OnEnable()
    {
        //当启用时，显示第一句话
        StartCoroutine(SetText());
    }
    // Update is called once per frame
    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            if (index >= textList.Count)
            {
                gameObject.SetActive(false);
                index = 0;
            }
            else if (textFinshed)
            {
                //逐字显示
                StartCoroutine(SetText());
            }
            else if (!textFinshed)
            {
                //结束协程，直接显示
                StopCoroutine(SetText());
                textFinshed = true;
                
                textLabel.text = textList[index];
                index++;
            }


        }
    }

    void GetTextFormFilr(TextAsset text)
    {
        textList.Clear();
        index = 0;
        //通过回车符分割
        var list = text.text.Split('\n');
        foreach (var item in list)
        {
            textList.Add(item);
        }
    }

    IEnumerator SetText()
    {
        textFinshed = false;
        //清空textLabel里的文字
        textLabel.text = "";
        //这里不清楚是不是我的文本的问题，获取时总是获取到两个字符，但不知道第二个字符是什么
        switch (textList[index][0])
        {
            case 'A':
                image.sprite = headA;
                index++;
                break;

            case 'B':
                image.sprite = headB;
                index++;
                break;
        }
        //每隔timeSpeed秒进行显示一次
        for (int i = 0; i < textList[index].Length; i++)
        {
            textLabel.text += textList[index][i];

            yield return new WaitForSeconds(timeSpeed);
        }
        //更改index值，并更改状态
        
        textFinshed = true;
        index++;

    }

}

```

