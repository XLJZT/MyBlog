---
title: 3DURP游戏制作笔记
date: 2022-5-10
description:
top_image: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/002.png
cover: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/002.png
categories: 
- Unity
tags: 
- Games
- 游戏制作
- M_studio
---

# 3DURP游戏制作笔记

> 感谢麦扣老师！！！
>
> [b站地址](https://space.bilibili.com/370283072)
>
> 这些是我跟着学习敲得代码，有些可能不一样，完全复制肯定会有问题，主要用来给学习的同学纠错



### 游戏的创建与地图的铺设

使用到的插件有**Pro Grids**,**ProBuilder**,**PloyBrush**

1. 使用到的地图模板为正常模板，然后在Project文件夹中新建URP。

2. 右上角Edit > Project Setting
   1. 将Graphics的Scirptable Render Pipeline Settings修改为创建的**UniversalRenderPiepelineAsset**
   2. Quality的Rendering修改为创建的**UniversalRenderPiepelineAsset**

3. 使用素材时，升级素材的URP。Edit > Render Pipeline > Universal Render Pipeline > Upgrade Project Materials To UniversalRP Materials

4. 光照的设置 Windows > Rendering >Lighting。在Scene窗口生成一个New Lighting Settings。Lighting Mode一般为Baked Indirect（简介烘焙），Lightmapper一般选择为GPU，取消勾选Auto Generate，然后进行Generate Lighting。

   > 如果发生场景颜色发生改变，不是原本的颜色，可以在Lighting > Environment里选择 Environment Lighting > Source 设置为Color。

5. 调整摄像机的技巧：Ctrl + Shift + F，调整物体坐标轴吸附 V键。

6. 导入插件**PloyBrush**并且在导入时将Sample中的URP文件一并导入。 

   > 使用Ploybrush的笔刷刷颜色的时候，发现刷不上颜色的原因是，物体的材质并没有使用Ploybrush的shader创建的材质，用shader创建一个新的材质并修改物体的材质（拖拽也可以），然后进行颜色的涂抹

7. 平台的大小和少数的顶点影响了地图的创作，所以需要**Probuilder**来生成更精致的地图平台

   > 同时按住**Ctrl**和**Alt**键点击Icon的齿轮，会调出单独的设置
   >
   > 使用**Center Pivot**可以调整中心点
   >
   > 关于如何使用preview package：Edit > Project Setting > Package manager ,**Enable Preview Packages**

### 地图的烘Bake

窗口在 Window > AI > Navigation

1. 设置地面为Navigation static，在Navigation > Object里的Navigation Area 设置为Walkable。

   > - 选中物体之后，在Inspector窗口里，第一行有选择是否为Static，下拉可选择类型
   > - 选中物体之后，在Navigation > Object里设置Navigation static为**true**

2. 选中不可移动的物体，比如树木石头等，批量设置为Navigation static，特殊的除外。
3. 为角色添加Navigation agent 组件，设置人物的圆柱体大小以及宽度。
4. 在Navigation > Bake 里同样设置为人物的圆柱体大小以及宽度等其他数据，然后点击bake，在地图上显示为蓝色的为可walk区域。
5. 如果有物体可以移动，并且不想发生穿模效果，可选择添加组件Nav Mesh Obstacle,设置Carve为**True**，就会在地面同时切割出一部分，人物不可walk的区域。

### 鼠标控制移动以及贴图

**所有管理类都不要忘了在游戏里创建一个空对象，添加上脚本**

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;


public class MouseManager : SingleTon<MouseManager>
{

	//事务委托
    public event Action<Vector3> OnMouseClicked;
    public event Action<GameObject> OnClickedEnemy;

    RaycastHit hitInfo;
	//鼠标贴图
    public Texture2D point, doorway, attack, target, arrow;
    
    protected override void Awake()
    {
        base.Awake();
    }

    void Update()
    {
        SetCursorTexture();
        MouseControl();
    }

    void SetCursorTexture()
    {
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if(Physics.Raycast(ray,out hitInfo))
        {
            //切换鼠标贴图
            switch (hitInfo.collider.gameObject.tag)
            {
                case "Ground":
                    Cursor.SetCursor(target, new Vector2(16, 16),CursorMode.Auto);
                    break;
                case "Enemy":
                    Cursor.SetCursor(attack, new Vector2(2, 2), CursorMode.Auto);
                    break;
                case "Attackable":
                    Cursor.SetCursor(attack, new Vector2(2, 2), CursorMode.Auto);
                    break;
                case "Portal":
                    Cursor.SetCursor(doorway, new Vector2(2, 2), CursorMode.Auto);
                    break;
            }
        }
    }

    void MouseControl()
    {
        if (Input.GetMouseButtonDown(0) && hitInfo.collider != null)
        {
            if (hitInfo.collider.gameObject.CompareTag("Ground"))
            {
                OnMouseClicked?.Invoke(hitInfo.point);
            }else if (hitInfo.collider.gameObject.CompareTag("Enemy"))
            {
                OnClickedEnemy?.Invoke(hitInfo.collider.gameObject);
            }
            else if (hitInfo.collider.gameObject.CompareTag("Attackable"))
            {
                OnClickedEnemy?.Invoke(hitInfo.collider.gameObject);
            }
            else if (hitInfo.collider.gameObject.CompareTag("Portal"))
            {
                OnMouseClicked?.Invoke(hitInfo.point);
            }
        }
    }
}

```

**关于鼠标的贴图**

鼠标贴图为**Texture**,一般为无背景的图片，加载到Unity之后，选中贴图并把Texture type修改为**Cursor**。

### Cineramachine

选中Main camera，Gameobject > Align View to Selected,将主相机锁定视角

状态栏Cineramachine选择一个合适的摄像机 ，当前选择的为CinemachineVirtualCanmera,取消主摄像机的Audio Listener，Body选择Framing Transposer,Aim选择Nothing

Follow 选择为人物对象 ，如果人物坐标轴在下方，可以在人物对象里新建一个空物体，然后将坐标轴移动到人物中心位置，将物体拖拽到**Follow**。

> **BODY**
>
> - Do Nothing：不移动虚拟相机
> - Framing Transposer：跟随目标移动，并在屏幕空间保持相机和跟随目标的相对位置。
> - Hard Lock to Target：虚拟相机和跟随目标使用相同位置。
> - Orbital Transposer：相机和跟随目标的相对位置是可变的，还能接收用户的输入。常见于玩家控制的相机。
> - Tracked Dolly：相机沿着预先设置的轨道移动。
> - Transposer：跟随目标移动，并在世界空间保持相机和跟随目标的相对位置固定。
>
> **AIM**
>
> - Do Nothing: 不控制相机的旋转。
> - Composer: 保持目标物体在镜头内
> - Group Composer: 保持多个目标在镜头内
>
> - Hard Look At: 保持目标在镜头的中心
> - POV: 基于玩家的输入旋转相机

**添加环境迷雾**

Window > Rendering > Lighting > Environment > Fog设置为True

> 如果Sence场景没有显示Fog，则需要在窗口中打开Fog开关
>
> Mode：Linear 为最近最远处能见度

**Global** **Volume**

对场景进行后处理，add 其他功能对场景进行渲染

> 如果场景没有显示出效果，在Main Camera > Rendering > Post Peocessing 设置为True

### 遮挡剔除



通过**Shader**创建材质放入到URP的 UniversalRenderPipelineAsset_Renderer ，新建Add Renderer Feature,选择被遮挡的Layer Mask，可以多选，然后设置被遮挡后的材质，Depth设置为True，Depth Test设置为Greater。当被遮挡物体层级Greater时，显示被遮挡后的材质。

同样Add Renderer Feature，选择相同被遮挡的Layer Mask，其他不做修改，当不被遮挡时显示原有材质

**关于遮挡材质的制作**

shader创建：Project > 右键 > Shader > URP >UnLit Shader Graph

> Graph Inspector > Grapj Settings > Universal > Alpha Clip,打开Alpha透明通道

1. 使用Fresnel Effect 制作遮挡的效果，通过新建变量Color修改颜色。通过Multiply混合，传给Base color。
2. Dither制作噪点效果，新建Float修改Dither的X值，out传给Fragment > Alpha通道
3. 创建Flaot 修改Alpha Threshold。
4. 选择Shader创建材质

**关于鼠标射线遮挡**

- 选中遮挡的物体，设置层级为Ignore Raycast
- 关闭遮挡物体的Mesh Collider

### EnemyController

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public enum EnemyStates {GUARD,PARTOL,CHASE,DEAD }
//通过代码添加组件
[RequireComponent(typeof(NavMeshAgent))]
[RequireComponent(typeof(CharacterStats))]

//使用接口
public class EnemyController : MonoBehaviour,IEndGameObserver
{
    NavMeshAgent agent;
    EnemyStates enemyStates;
    Animator anim;
    protected CharacterStats characterStats;
    Collider coll;
    //攻击范围 
    public float sightRadius;
    //巡逻范围
    public float partolRadius;
    public bool isGuard;
    //巡逻停留时间
    public float lookAtTime;
    float remainLookAtTime;
    float speed;
    //cd时间
    float lastAttackTime;
    //下一个前往的巡逻点
    Vector3 wayPoint;
    //起始点
    Vector3 guardPoint;
    Quaternion guardRotation;
    protected GameObject attackTarget;
    bool isWalk, isChase, isFollow, isDie;
    bool playerDie;
    private void Awake()
    {
        agent = GetComponent<NavMeshAgent>();
        anim = GetComponent<Animator>();
        characterStats = GetComponent<CharacterStats>();
        coll = GetComponent<Collider>();
        //保存原有速度；
        speed = agent.speed;
        guardPoint = transform.position;
        remainLookAtTime = lookAtTime;
        lastAttackTime = characterStats.CoolDown;
        guardRotation = transform.rotation;
    }
    void Start()
    {
        if (isGuard)
        {
            enemyStates = EnemyStates.GUARD;
        }
        else
        {
            enemyStates = EnemyStates.PARTOL;
            GetNewPartolPoint();
        }
        //将对象添加进gamemanager里，用于广播
        GameManager.Instance.AddObserver(this);
    }

    // Update is called once per frame
    void Update()
    {
        if (characterStats.CurrentHealth == 0)
        {
            isDie = true;
        }
        if (!playerDie)
        {
            SwitchStates();
            SwitchAnim();
            if (lastAttackTime > 0f)
            {
                lastAttackTime -= Time.deltaTime;
            }
        }
    }
    
    //private void OnEnable()
    //{
    //    Gamemanager.Instance.AddObserver(this);
    //}

    private void OnDisable()
    {
        if (!GameManager.isInitialized) return;
        GameManager.Instance.RemoveObserver(this);
    }

    void SwitchStates()
    {
        if (isDie)
        {
            enemyStates = EnemyStates.DEAD;
        }
        else if (FindPlayer())
        {
            enemyStates = EnemyStates.CHASE;
            
        }
        switch (enemyStates)
        {
            case EnemyStates.GUARD:
                isChase = false;
                isWalk = false;
                if (transform.position!=guardPoint)
                {
                    isWalk = true;
                    agent.isStopped = false;
                    agent.destination = guardPoint;
                    if(Vector3.SqrMagnitude(guardPoint - transform.position) <= agent.stoppingDistance)
                    {
                        isWalk = false;
                        agent.isStopped = true;
                        //旋转会初始的角度
                        Quaternion.Lerp(transform.rotation, guardRotation, 0.1f);
                    }
                }
                break;
            case EnemyStates.PARTOL:
                isChase = false;
                agent.speed = speed * 0.5f;
                if (Vector3.Distance(transform.position, wayPoint) <= 1)
                {
                    isWalk = false;
                    if (remainLookAtTime < 0f)
                    {
                        GetNewPartolPoint();
                        remainLookAtTime = lookAtTime;
                    }
                    else
                        remainLookAtTime -= Time.deltaTime;
                }
                else
                {
                    isWalk = true;
                    agent.destination = wayPoint;
                }
                break;
            case EnemyStates.CHASE:

                agent.speed = speed;
                isWalk = false;
                isChase = true;
                //脱战
                if (!FindPlayer())
                {
                    isFollow = false;
                    agent.destination = transform.position;
                    if (remainLookAtTime > 0)
                    {
                        remainLookAtTime -= Time.deltaTime;
                    }
                    else
                    {
                        if (isGuard)
                        {
                            enemyStates = EnemyStates.GUARD;
                        }
                        else
                        {
                            enemyStates = EnemyStates.PARTOL;
                        }
                    }
                }
                else
                {
                    //追击
                    isFollow = true;
                    agent.isStopped = false;
                    agent.destination = attackTarget.transform.position;
                    //判断是否追击到
                    if (TargetInAttackRange() || TargetInSkillRange())
                    {
                        isFollow = false;
                        agent.isStopped = true;
                        if (lastAttackTime < 0f)
                        {
                            lastAttackTime = characterStats.CoolDown;
                            //暴击判断
                            characterStats.isCritical = Random.value < characterStats.CriticalChance;
                            
                            Attack();
                        }
                    }
                }

                break;
            case EnemyStates.DEAD:
                //enemy死亡时不会发生继续攻击的情况
                coll.enabled = false;
                
                //agent.enabled = false;
                //设置之后不会阻挡Player行动
                agent.radius = 0f;
                Destroy(gameObject, 2f);
                break;
            default:
                break;
        }
    }
    void Attack()
    {
        transform.LookAt(attackTarget.transform);
        anim.SetBool("Critical", characterStats.isCritical);
        if (TargetInAttackRange())
        {
            anim.SetTrigger("Attack");
        }
        if(TargetInSkillRange())
        {
            anim.SetTrigger("Skill");
        }
    }

    bool FindPlayer()
    {
        Collider[] colliderNum = Physics.OverlapSphere(transform.position, sightRadius);
        foreach (var collider in colliderNum)
        {
            
            if (collider.gameObject.CompareTag("Player"))
            {
                attackTarget = collider.gameObject;
                return true;
            }
        }
        return false;
    }

    void SwitchAnim()
    {
        anim.SetBool("Walk", isWalk);
        anim.SetBool("Chase", isChase);
        anim.SetBool("Follow", isFollow);
        anim.SetBool("Die", isDie);
    }

    private void OnDrawGizmosSelected()
    {
        //画出攻击范围圈
        Gizmos.color = Color.green;
        Gizmos.DrawWireSphere(transform.position, sightRadius);
    }
    
    void GetNewPartolPoint()
    {
        float ran_x = Random.Range(-partolRadius, partolRadius);
        float ran_z = Random.Range(-partolRadius, partolRadius);
        //不修改高度，防止坑坑洼洼的地面
        //guardPoint是为了随机在初始位置点的随机移动
        wayPoint = new Vector3(guardPoint.x + ran_x, transform.position.y, guardPoint.z + ran_z);
        NavMeshHit hit;
        //NavMesh.SamplePosition（）方法的最后一位为窗口 Navigation > Area的序列，从1开始
        //方法为 在waypoint的sightRadius范围内寻找1层级（可移动到）的点
        //方法返回的为True和False
        wayPoint = NavMesh.SamplePosition(wayPoint, out hit, sightRadius,1) ? hit.position : transform.position;
        

    }
    bool TargetInAttackRange()
    {
        if (attackTarget != null)
        {
            return Vector3.Distance(attackTarget.transform.position, transform.position) < characterStats.AttackRange;
        }
        else
        {
            return false;
        }
    }
    bool TargetInSkillRange()
    {
        if (attackTarget != null)
        {
            return Vector3.Distance(attackTarget.transform.position, transform.position) < characterStats.SkillRange;
        }
        else
        {
            return false;
        }
    }
    void Hit()
    {
        if (attackTarget != null && transform.IsFacingTarget(attackTarget.transform))
        {
            var targetStats = attackTarget.GetComponent<CharacterStats>();
            characterStats.TakeDamage(characterStats, targetStats);
        }
    }

    public void EndsNotify()
    {
        isChase = false;
        isWalk = false;
        attackTarget = null;
        playerDie = true;
        anim.SetBool("Victory", true);
    }
}

```

### PlayController

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class PlayerController : MonoBehaviour
{
    public float lastAttackTime;
    //攻击目标对象
    GameObject attackTarget;
    CharacterStats characterStats;
    NavMeshAgent agent;
    Animator anim;
    bool isDie;
    float stopDistance;
    private void Awake()
    {
        agent = GetComponent<NavMeshAgent>();
        anim = GetComponent<Animator>();
        characterStats = GetComponent<CharacterStats>();

    }

    private void Start()
    {
        //添加事件
        MouseManager.Instance.OnMouseClicked += MoveToTarget;
        MouseManager.Instance.OnClickedEnemy += EventAttack;
        
		//注册Gamemanager的广播
        GameManager.Instance.RigisterPlayer(characterStats);
        stopDistance = agent.stoppingDistance;
    }
    private void Update()
    {
        isDie = characterStats.CurrentHealth == 0;
        if (isDie)
        {
            GameManager.Instance.NotifyObservers();
        }
        AnimationChange();
        if(lastAttackTime>0f)
            lastAttackTime -= Time.deltaTime;
    }
    void MoveToTarget(Vector3 target)
    {
        if (isDie) return;
        agent.isStopped = false;
        agent.stoppingDistance = stopDistance;
        StopAllCoroutines();
        agent.destination = target;
    }

    void AnimationChange()
    {
        anim.SetFloat("Speed", agent.velocity.sqrMagnitude);
        anim.SetBool("Die", isDie);
    }

    void EventAttack(GameObject target)
    {
        if (isDie) return;

        if (target != null)
        {
            attackTarget = target;
            StartCoroutine("MoveToAttackTarget");
        }
    }
    //动画事件
    void Hit()
    {
        if (attackTarget.CompareTag("Attackable"))
        {
            if (attackTarget.GetComponent<Rock>())
            {
                //在空中也可以击飞
                attackTarget.GetComponent<Rock>().rockStates = Rock.RockStates.HitEnemy;
                //赋予初始速度，防止fixudate时修改rock状态
                attackTarget.GetComponent<Rigidbody>().velocity = Vector3.one;
                attackTarget.GetComponent<Rigidbody>().AddForce(transform.forward * 20f,ForceMode.Impulse);

            }
        }
        else
        {
            var targetStats = attackTarget.GetComponent<CharacterStats>();
            characterStats.TakeDamage(characterStats, targetStats);
        }
    }
    IEnumerator MoveToAttackTarget()
    {
        agent.stoppingDistance = characterStats.AttackRange;
        agent.isStopped = false;
        while (Vector3.Distance(attackTarget.transform.position, transform.position) > characterStats.AttackRange)
        {
            agent.destination = attackTarget.transform.position;
            yield return null;
        }
        agent.isStopped = true;
        if (lastAttackTime < 0)
        {
            transform.LookAt(attackTarget.transform.position);
            //暴击率的计算
            characterStats.isCritical = Random.value < characterStats.CriticalChance;
            anim.SetBool("Critical",characterStats.isCritical);
            anim.SetTrigger("Attack");
            //重置冷却时间
            lastAttackTime = characterStats.CoolDown;
        }
    }
}

```

### CharacterStats

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
public class CharacterStats : MonoBehaviour
{

    public event Action<int, int> UpdateHealthBarOnAttack;

    public Character_SO templateData;
    public Character_SO character;
    public AttackSO attackSO;
    [HideInInspector]
    public bool isCritical;

    private void Awake()
    {
        //用模板为character生成一组数据，防止多个enemy访问同一组数据
        if (templateData != null)
        {
            character = Instantiate(templateData);
        }
    }
    public int MaxHealth
    {
        get { if (character != null) { return character.maxHealth; } else { return 0; } }
        set { character.maxHealth = value; }
    }
    public int CurrentHealth
    {
        get { if (character != null) { return character.currentHealth; } else { return 0; } }
        set { character.currentHealth = value; }
    }
    public int BaseDefence
    {
        get { if (character != null) { return character.baseDefence; } else { return 0; } }
        set { character.baseDefence = value; }
    }
    public int CurrentDefence
    {
        get { if (character != null) { return character.currentDefence; } else { return 0; } }
        set { character.currentDefence = value; }
    }
    public float AttackRange { get {  return attackSO.attackRange; } }
    public float SkillRange { get { return attackSO.skillRange; } }
    public float CoolDown { get { return attackSO.coolDown; } }
    public float MinDamage { get { return attackSO.minDamage; } }
    public float MaxDamage { get { return attackSO.maxDamage; } }

    public float CriticalMultiplier { get { return attackSO.criticalMultiplier; } }
    public float CriticalChance { get { return attackSO.criticalChance; } }
    
    public void TakeDamage(CharacterStats attacker,CharacterStats defender)
    {
        int damage = Mathf.Max(attacker.CurrentDamage() - defender.CurrentDefence, 0);
        defender.CurrentHealth = Mathf.Max(defender.CurrentHealth - damage, 0);
        //defender受伤的动画 暴击才会有
        if (attacker.isCritical)
        {
            defender.GetComponent<Animator>().SetTrigger("Hit");
        }
		//修改防御者血条
        defender.UpdateHealthBarOnAttack?.Invoke(defender.CurrentHealth, defender.MaxHealth);
        if (defender.CurrentHealth <= 0)
        {
            //更新攻击者经验
            attacker.character.UpdateExp(defender.character.killPoint);
        }
    }

    public void TakeDamage(int damage,CharacterStats defender)
    {
        int currentDamage = Mathf.Max(damage - defender.CurrentDefence, 0);
        defender.CurrentHealth = Mathf.Max(defender.CurrentHealth - currentDamage, 0);
        UpdateHealthBarOnAttack?.Invoke(defender.CurrentHealth, defender.MaxHealth);

    }

    int CurrentDamage()
    {
        float damage = UnityEngine.Random.Range(MinDamage, MaxDamage); ;
        if (isCritical)
        {
            damage *= CriticalMultiplier;
            Debug.Log("暴击：" + damage);

        }
        return (int)damage;
    }
}

```

**Character_SO**

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


//filename为创建的文件的文件名，menuName为文件夹目录层级名字
[CreateAssetMenu(fileName = "New Data", menuName = "Character Stats/Data")]

public class Character_SO : ScriptableObject
{
    [Header("Stats Info")]
    public int maxHealth;
    public int currentHealth;
    public int baseDefence;
    public int currentDefence;

    [Header("Kill")]
    public int killPoint;

    [Header("Level")]
    public int currentLevel;
    public int maxLevel;
    public int baseExp;
    public int currentExp;
    public float levelBuff;

    public void UpdateExp(int point)
    {
        currentExp += point;
        if (currentExp >= baseExp)
        {
            currentExp -= baseExp;
            LevelUP();
        }
    }

    private void LevelUP()
    {
        currentLevel = Mathf.Clamp(currentLevel + 1, 0, maxLevel);
        baseExp += (int)(baseExp * LevelMultiplier);
        maxHealth = (int)(maxHealth * LevelMultiplier);
        currentHealth = maxHealth;
    }
    public float LevelMultiplier
    {
        get {return 1+(currentLevel-1)*levelBuff ; }
    }
}

```

**Attack_SO**

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName ="New Attack",menuName ="Attack/Attack Data")]
public class AttackSO : ScriptableObject
{
    [Header("Attack Info")]
    public float attackRange;
    public float skillRange;
    public float coolDown;
    public int minDamage;
    public int maxDamage;
    public float criticalMultiplier;
    public float criticalChance;
    
}

```

### 泛型单例模式Singleton

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SingleTon<T> : MonoBehaviour where T:SingleTon<T>
{
    private static T instance;

    public static T Instance
    {
        get { return instance; }
    }

    protected virtual void Awake()
    {
        if (instance != null)
        {
            Destroy(gameObject);
        }
        else
        {
            instance = (T)this;
        }
    }
    public static bool isInitialized
    {
         get { return instance != null; }
    }
}

```

### 接口Observer Pattern

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public interface IEndGameObserver
{
    //只定义，不实现留给使用的实现这个方法
    void EndsNotify();
}

```

### GameManger

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GameManager : SingleTon<GameManager>
{
    public CharacterStats playerStats;
	
    //接口类型
    List<IEndGameObserver> endGameObeverses = new List<IEndGameObserver>();
    
    protected override void Awake()
    {
        base.Awake();
    }
    public void RigisterPlayer(CharacterStats characterStats)
    {
        playerStats = characterStats;
    }

    public void AddObserver(IEndGameObserver observer)
    {
        endGameObeverses.Add(observer);
    }
    public void RemoveObserver(IEndGameObserver observer)
    {
        endGameObeverses.Remove(observer);
    }

    public void NotifyObservers()
    {
        //遍历添加进来的对象
        //挨个调用EndsNotify（）方法
        foreach (var observer in endGameObeverses)
        {
            observer.EndsNotify();
        }
    }
}

```

### 设置兽人Grunt

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

//继承于EnemyController
public class Grunt : EnemyController
{
    [Header("GRUNT推力")]
    public float kickForce = 15f;
    //动画事件
    void KickOff()
    {
        if (attackTarget != null)
        {
            transform.LookAt(attackTarget.transform);
            Vector3 direction = attackTarget.transform.position - transform.position;
            direction.Normalize();
            //打断player行动，进行击退
            attackTarget.GetComponent<NavMeshAgent>().isStopped = true;
            attackTarget.GetComponent<NavMeshAgent>().velocity = direction * kickForce;
            attackTarget.GetComponent<Animator>().SetTrigger("Dizzy");
        }
    }
}

```

### 拓展方法Extension Method

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//拓展方法 
public static class ExtensionMethod 
{
    private const float dotThreshold = 0.5f;

    //拓展的是this后面的方法,后面的为方法所需要的函数
    //使用方法 :transform.IsFacingTarget(target)
    public static bool IsFacingTarget(this Transform transform,Transform target)
    {
        //向量OA-OB=BA;O为原点；
        Vector3 vectorToTarget = target.position - transform.position;
        vectorToTarget.Normalize();
        //vector3.dot 为看向的方向与vectortoTarget的方向的夹角，正对着为1，背对着为-1，左右为0；
        float dot = Vector3.Dot(transform.forward, vectorToTarget);
        return dot >= dotThreshold;
    }

}
```

### 石头人Golem

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class Golem : EnemyController
{
    [Header("GRUNT推力")]
    public float kickForce = 25f;

    public GameObject rockPerfab;
    public Transform handTransform;
    //animation event
    void KickOff()
    {
        if (attackTarget != null && transform.IsFacingTarget(attackTarget.transform))
        {
            var targetStats = attackTarget.GetComponent<CharacterStats>();
            Vector3 direction = attackTarget.transform.position - transform.position;
            direction.Normalize();
            transform.LookAt(attackTarget.transform);
            attackTarget.GetComponent<NavMeshAgent>().isStopped = true;
            attackTarget.GetComponent<NavMeshAgent>().velocity = direction * kickForce;
            attackTarget.GetComponent<Animator>().SetTrigger("Dizzy");
            characterStats.TakeDamage(characterStats, targetStats);
        }
    }
    //animation event
    void ThrowRock()
    {
        if (attackTarget != null)
        {
            GameObject rock = Instantiate(rockPerfab, handTransform.position, Quaternion.identity);
            rock.GetComponent<Rock>().target = attackTarget;
        }
    }
}

```

### 扔石头

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;
using UnityEngine.Animations;

public class Rock : MonoBehaviour
{
    public enum RockStates { HitPlayer,HitEnemy,HitNothing}
    public RockStates rockStates;

    public GameObject target;
    public GameObject rockBreak;
    public float force;
    public int damage;
    Vector3 direction;
    Rigidbody rb;
    void Start()
    {
        rb = GetComponent<Rigidbody>();
        rb.velocity = Vector3.one;
        rockStates = RockStates.HitPlayer;
        FlyToTarget();
    }

    private void FixedUpdate()
    {
        if (rb.velocity.sqrMagnitude < 0.5f)
        {
            rockStates = RockStates.HitNothing;
        }
    }
   
    void FlyToTarget()
    {
        if (target == null)
            target = FindObjectOfType<PlayerController>().gameObject;
        direction = (target.transform.position - transform.position+Vector3.up ).normalized;
        rb.AddForce(direction * force, ForceMode.Impulse);
    }

    private void OnCollisionEnter(Collision collision)
    {
        switch (rockStates)
        {
            case RockStates.HitPlayer:
                if (collision.collider.CompareTag("Player"))
                {
                    collision.gameObject.GetComponent<NavMeshAgent>().isStopped = true;
                    collision.gameObject.GetComponent<NavMeshAgent>().velocity = direction * force;
                    collision.gameObject.GetComponent<Animator>().SetTrigger("Dizzy");

                    rockStates = RockStates.HitNothing;
                    collision.gameObject.GetComponent<CharacterStats>().TakeDamage(damage, collision.gameObject.GetComponent<CharacterStats>());
                }
                break;
            case RockStates.HitEnemy:
                //如果有这个组件返回true；
                if (collision.gameObject.GetComponent<Golem>())
                {
                    collision.gameObject.GetComponent<CharacterStats>().TakeDamage(damage, collision.gameObject.GetComponent<CharacterStats>());
                    
                    //攻击石头人之后，石头碎裂生成粒子特效
                    Instantiate(rockBreak, transform.position, Quaternion.identity);
                    Destroy(gameObject);
                }
                break;
            case RockStates.HitNothing:
                break;
            default:
                break;
        }


    }
}

```

**粒子特效**

Hierarchy > Effects > Particle System

### 血条显示 HealthBar

**挂载在enemy身上**

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
public class HealthBar : MonoBehaviour
{
    public GameObject healthBarUiPerfab;
	//血条的位置，enemy设置一个空对象，调整位置为头部上方
    public Transform barPoint;
    //是否一直显示 显示的时长
    public bool alwaysVisible;
    public float visibleTime;
    float timeLeft;

    Image healthSlider;
    Transform UIBar;
    Transform cam;
    CharacterStats currentStats;

    private void Awake()
    {
        //获取组件
        currentStats = GetComponent<CharacterStats>();
        //添加事务
        currentStats.UpdateHealthBarOnAttack += UpdateHealthBar;
    }
    private void OnEnable()
    {
        cam = Camera.main.transform;
        foreach (Canvas canvas in FindObjectsOfType<Canvas>())
        {
            if (canvas.renderMode == RenderMode.WorldSpace)
            {
                UIBar = Instantiate(healthBarUiPerfab, canvas.transform).transform;
                healthSlider = UIBar.GetChild(0).GetComponent<Image>();
                UIBar.gameObject.SetActive(alwaysVisible);
            }
        }

    }

    private void UpdateHealthBar(int currentHealth, int Maxhealth)
    {
        if (currentHealth <= 0)
            Destroy(UIBar.gameObject);

        UIBar.gameObject.SetActive(true);
        //攻击时重置显示剩余时间
        timeLeft = visibleTime;
        float sliderPercent = (float)currentHealth / Maxhealth;
        //image Type设置为Filled，fillmethod设置为Horizontal
        healthSlider.fillAmount = sliderPercent;

    }
    private void LateUpdate()
    {
        if (UIBar != null)
        {
            //位置
            UIBar.position = barPoint.position;
            //ui的方向与摄像机的方向相反
            UIBar.forward = -cam.forward;

            if (timeLeft <= 0f&&!alwaysVisible)
            {
                UIBar.gameObject.SetActive(false);
            }
            else
            {
                timeLeft -= Time.deltaTime;

            }
        }


    }
}

```

### PlayerUI

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class PlayHealthBar : MonoBehaviour
{
    Text level;
    Image healthSlider;
    Image expSlider;

    private void Awake()
    {
        level = transform.GetChild(2).GetComponent<Text>();
        healthSlider = transform.GetChild(0).GetChild(0).GetComponent<Image>();
        expSlider = transform.GetChild(1).GetChild(0).GetComponent<Image>();
    }

    private void LateUpdate()
    {
        level.text = "Level  " + GameManager.Instance.playerStats.character.currentLevel.ToString("00");

        UpdateHealth();

        UpdateExp();
    }

    void UpdateHealth()
    {
        float sliderPercent = (float)GameManager.Instance.playerStats.CurrentHealth / GameManager.Instance.playerStats.MaxHealth;
        healthSlider.fillAmount = sliderPercent;
    }
    void UpdateExp()
    {
        float sliderPercent = (float)GameManager.Instance.playerStats.character.currentExp / GameManager.Instance.playerStats.character.baseExp;
        expSlider.fillAmount = sliderPercent;
    }
}

```

>设置ui时，首先需要调整Canvas的Scaler
>
>UI Scale Mode设置为Scale With Screen Size,Match设置为0.5，同时兼顾高度和宽度

>
>
>Canvas有个Canvas Group组件，可以设置取消阻挡射线
>
>也可以通过调整Alpha的值去做转场

### 传送门

使用lit shader

shader制作成材质，创建一个3D > plane，拖拽赋予这个材质，并生成一个传送点位置

枚举类

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TransitionDestination : MonoBehaviour
{
    public enum DestinationTag { Enter,A,B,C}

    public DestinationTag destinationTag;
}

```

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TransiationPoint : MonoBehaviour
{
    public enum TransitionType { SameScence,DifferentScence}

    [Header("Transition Info")]
    public string scenceName;
    public TransitionType transitionType;
    //枚举终点编号
    public TransitionDestination.DestinationTag destinationTag;

    bool canTrans;

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.E) && canTrans)
        {
            ScenceController.Instance.TransitionToDestination(this);
        }
    }
    private void OnTriggerStay(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            canTrans = true;
        }
    }
    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            canTrans = false;
        }
    }
}

```

实现场景转换

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

using UnityEngine.AI;

public class ScenceController : SingleTon<ScenceController>
{
    GameObject player;
    NavMeshAgent playerAgent;
    public void TransitionToDestination(TransiationPoint transiationPoint)
    {
        switch (transiationPoint.transitionType)
        {
            case TransiationPoint.TransitionType.SameScence:
                StartCoroutine(Transition(SceneManager.GetActiveScene().name, transiationPoint.destinationTag));
                break;
            case TransiationPoint.TransitionType.DifferentScence:
                break;
            default:
                break;
        }
    }

    IEnumerator Transition(string scenceName,TransitionDestination.DestinationTag destinationTag)
    {
        player = GameManager.Instance.playerStats.gameObject;

        playerAgent = player.GetComponent<NavMeshAgent>();
        playerAgent.isStopped = true;

       //设置位置与方向transform.SetPositionAndRotation（）
        player.transform.SetPositionAndRotation(GetDestination(destinationTag).transform.position, GetDestination(destinationTag).transform.rotation);
        yield return null;
    }
	
    //遍历获取终点的信息
    TransitionDestination GetDestination(TransitionDestination.DestinationTag destinationTag)
    {
        var entrances = FindObjectsOfType<TransitionDestination>();
        foreach (var item in entrances)
        {
            if (item.destinationTag == destinationTag)
            {
                return item;
            }
        }
        return null;
    }
}

```

### 保存数据

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class SaveManager : SingleTon<SaveManager>
{


    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.K))
        {
            Save(GameManager.Instance.playerStats.character, GameManager.Instance.playerStats.name);
            Debug.Log(GameManager.Instance.playerStats.name);
        }
        if (Input.GetKeyDown(KeyCode.L))
        {
            Load(GameManager.Instance.playerStats.character, GameManager.Instance.playerStats.name);
            
        }
    }

    public void Save(object data,string key)
    {
        //数据转成json
        var jsonData = JsonUtility.ToJson(data, true);
        //json转成二进制
        PlayerPrefs.SetString(key, jsonData);
        PlayerPrefs.Save();
    }
    public void Load(object data, string key)
    {
        if (PlayerPrefs.HasKey(key))
        {
            //PlayerPrefs.GetString(key)将二进制转为json，覆盖到data上
            JsonUtility.FromJsonOverwrite(PlayerPrefs.GetString(key), data);
            Debug.Log("111");

        }
    }
}

```

