---
title: 탑 뷰 로그라이크 게임
header:
    # 문서 해더 이미지
    # image: /assets/images/portfolio/
    # 페이지에 표시되는 외부 이미지
    teaser: /assets/images/portfolio/
---

로그라이크 시스템 개발하기

[깃허브](https://github.com/mob954325/TopView_Rougelike)

## 기간
개발기간 : 2024.06.03 - 2024.06.29

## 목표
생성된 맵에서 각 방마다 적을 죽여서 능력치를 올린 후 보스를 잡으면 클리어하는 게임 만들기

### 세부 목표
1. 기본 공격 및 강공격이 있는 플레이어 캐릭터 구현
2. 스크립터블 오브젝트를 이용한 데이터 관리
3. 싱글톤 매니저 생성
4. 오브젝트 풀링을 이용한 프리팹 생성
5. 객체 지향적 프로그래밍하기

### 플레이 Gif

![TopView_Rougelike_5s](https://github.com/mob954325/TopView_Rougelike/assets/87255621/b50e8f78-7569-463a-bf52-9e29661f42a7)
  
### 주요 코드

스크립터블 오브젝트 생성 및 사용
```
/// 능력 오브젝트 데이터
[CreateAssetMenu(fileName = "Ability_99_Empty", menuName = "ScriptableObjects/Ability/Ability", order = 1)]
public class AbilityData : ScriptableObject
{
    [Header("능력 정보")]
    // ...

    /// <summary>
    /// 데미지 증가량 ( 활성화 된 투사체 개수가 최대일 때 이만큼 증가 )
    /// </summary>
    public float increaseDamageValue;

    /// <summary>
    /// 투사체 프리팹 오브젝트
    /// </summary>
    public GameObject projectilePrefab;
}

/// 능력 투사체를 가지고 있는 클래스
public class Ability : MonoBehaviour
{
    [SerializeField] AbilityData data;

    /// 능력 데이터 접근 프로퍼티
    public AbilityData Data => data;

    /// 능력 초기화 함수
    /// <param name="data">저장할 능력 데이터</param>
    public void Initialize(AbilityData data)
    {
        this.transform.localPosition = Vector3.zero; // 위지 초기화

        this.data = data;
        this.damage = data.damage;
        this.rotateSpeed = data.speed;
        this.coolTime = data.coolTime;
        projectile = new GameObject[data.maxCount];

        // 최대 개수 만큼 다 만들고 
        for(int i = 0; i < data.maxCount; i++)
        {
            SpawnProjectile(data.code, i);

            if(i > data.minCount - 1)   // min개수만 활성화
            {   
                projectile[i].SetActive(false); // 나머지 비활성화
            }
        }
    }

    // ...
}

```

싱글톤 코드

```
public enum SingletonState : byte
{
    UnInitialized = 0,          // 초기화 안됨
    Initializing,               // 초기화 진행중
    Initialized                 // 초기화 완료
}

/// 싱글톤 구현 클래스 ( DontDestoryObject )
/// <typeparam name="T">싱글톤으로 사용할 클래스</typeparam>
public class Singleton<T> : MonoBehaviour where T : Component
{
    #region Fields

    /// 싱글톤 이름 (로그용)
    protected string singletonName;

    /// 해당 싱글톤 상태
    SingletonState singletonState = SingletonState.UnInitialized;

    /// 해당 싱글톤 객체 ( 인스턴스 )
    private static T instance;

    /// 싱글톤이 처음 실행됬는지 확인하는 변수 ( true : 실행됨 , false : 실행안됨 )
    private bool isInitialized;
    #endregion

    #region Properties
    /// 싱글톤 객체 접근 프로퍼티
    public static T Instance
    {
        get
        {
            // 객체가 없으면 새 싱글톤을 생성
            if (instance == null)
            {
                // 싱글톤 생성
                GameObject obj = new GameObject();      // 새 오즈벡트 생성
                obj.name = "Sington";

                obj.AddComponent<T>();                  // 싱글톤 생성
                DontDestroyOnLoad(obj);                 // 파괴 불가능 오브젝트 설정                
            }
            return instance;
        }
    }

    #endregion

    #region Unity LifeCycle
    private void Awake()
    {
        singletonName = this.gameObject.name;

        //Debug.Log($"{singletonName} : 1. 로딩전 로그");
        if(instance == null)
        {
            // 씬에서 배치된 싱글톤이 없다.
            instance = this as T;
            DontDestroyOnLoad(instance.gameObject);
        }
        else
        {
            // 씬에 싱글톤이 존재한다.
            if (instance != this)   // 해당 싱글톤이 자신이 아니면
            {
                Destroy(this.gameObject);
            }
        }
    }

    private void OnEnable()
    {
        //Debug.Log($"{singletonName} : 2. 씬 로딩 시작");
        SceneManager.sceneLoaded += OnSceneLoaded;
    }

    private void OnDisable()
    {
        SceneManager.sceneLoaded -= OnSceneLoaded;        
        //Debug.Log($"{singletonName} : 6. 씬 로딩 종료");
    }

    /// 씬이 로딩할 때 실행되는 함수 
    /// <param name="scene">씬 정보</param>
    /// <param name="mode">로딩 씬 모드</param>
    private void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        //Debug.Log($"{singletonName} : 3. 씬 로딩 진행 시작");
        if(!isInitialized)
        {
            PreInitialize();
        }

        if(mode != LoadSceneMode.Additive)
        {
            Initializing();
            Initialized();
        }
    }

    #endregion

    #region Protected Method
    /// 싱글톤이 처음 실행될 때 호출되는 함수
    protected virtual void PreInitialize()
    {
        //Debug.Log($"{singletonName} : 4. 최초 초기화");
        isInitialized = true;
    }

    /// Initialized가 호출 되기 전에 실행되는 함수
    protected virtual void Initializing()
    {
        singletonState = SingletonState.Initializing;   // 싱글톤 상태 변환
        //Debug.Log($"{singletonName} : 4-1. 초기화 진행중");


    }

    /// 씬이 로드 됐을 때 호출되는 초기화 함수 ( Additive 아님 )
    protected virtual void Initialized()
    {
        singletonState = SingletonState.Initialized;   // 싱글톤 상태 변환
        //Debug.Log($"{singletonName} : 5. 초기화 완료");
    }
    #endregion
}
```

풀링 시스템
```
/// Object Pooling 클래스
/// <typeparam name="T">재사용할 오브젝트의 최상위 클래스</typeparam>
public class Pool<T> : MonoBehaviour where T : PoolObject
{
    // 1. 오브젝트 생성
    // 2. 생성한 오브젝트 비활성화
    // 3. 오브젝트 사용 함수 작성
    // 3.1 queue 자료구조를 이용해 순차적으로 활성화 ( 활성화 준비 오브젝트들 )
    // 3.2 만약 준비된 오브젝트가 없으면 풀 크기 늘리기 ( 최대한 일어나면 안됨 )
    // 4. 해당 오브젝트가 비활성화 되면 준비큐에 다시 삽입

    /// 풀 오브젝트 프리팹
    public GameObject prefab;

    /// 생성할 오브젝트 개수
    public int totalCount;

    /// 풀에 있는 모든 오브젝트 배열
    T[] pool;

    /// 활성화 준비된 오브젝트 큐
    Queue<T> readyQueue;

    /// 오브젝트 풀을 초기화 하는 함수
    public void Initialize()
    {
        // 큐 초기화
        pool = new T[totalCount];
        readyQueue = new Queue<T>(pool.Length);
        readyQueue.Clear();

        // 오브젝트 생성
        int index = 0;
        while (index < totalCount)
        {
            T comp = Instantiate(prefab, gameObject.transform).GetComponent<T>();
            comp.name = $"{prefab.name}_{index}";

            pool[index] = comp;  

            readyQueue.Enqueue(comp);                               // 큐 삽입
            comp.gameObject.SetActive(false);                       // 각 오브젝트 비활성화
            comp.onDisable = () => { readyQueue.Enqueue(comp); };   // 비활성화 될 때 다시 큐에 삽입

            index++;
        }
    }

    /// 풀에서 오브젝트를 하나 꺼내는 함수
    /// <param name="position">위치값</param>
    /// <param name="rotation">회전값</param>
    public T GetObject(Vector3 position, Quaternion rotation)
    {
        T result = null;

        if(readyQueue.TryPeek(out T comp))
        {
            comp = readyQueue.Peek();    // 큐에서 오브젝트 가져오기
            readyQueue.Dequeue();       // 레디큐에서 제거
            comp.gameObject.SetActive(true);        // 오브젝트 활성화
            comp.transform.position = position;
            comp.transform.rotation = rotation;

            result = comp;
        }
        else
        {
            ExtendPool();   // 확장

            comp = readyQueue.Peek();    // 큐에서 오브젝트 가져오기
            readyQueue.Dequeue();       // 레디큐에서 제거
            comp.gameObject.SetActive(true);        // 오브젝트 활성화

            result = comp;
        }

        return result;
    }

    /// 풀을 확장하는 함수 ( 현재 개수만큼 추가 )
    void ExtendPool()
    {
        int prevCount = totalCount; // 이전 개수 (로그용)
        T[] prevComps = pool;

        // 크기 확장
        totalCount += totalCount;
        pool = new T[totalCount];

        // 이전 배열 추가
        for(int i = 0; i < prevComps.Length; i++)
        {
            pool[i] = prevComps[i]; 
        }

        // 이후 오브젝트 추가
        int index = prevComps.Length;   

        while(index < totalCount)
        {
            if (pool[index] == null)   // 오브젝트가 존재하지않으면 추가
            {
                // 추가 생성
                T comp = Instantiate(prefab, gameObject.transform).GetComponent<T>();
                comp.name = $"{prefab.name}_{index}";
                pool[index] = comp;
                readyQueue.Enqueue(comp);   // 큐 삽입 ( 이미 나머지 오브젝트들을 활성화되어있기 때문에 추가로 생성한 것만 추가 )
                comp.gameObject.SetActive(false);                    // 각 오브젝트 비활성화
                comp.onDisable = () => { readyQueue.Enqueue(comp); };
            }
            index++;
        }

        //Debug.LogWarning($"{this.gameObject.name} 크기 증가 [{prevCount} -> {totalCount}]");
    }

    /// 배열에 있는 모든 오브젝트를 비활성화 하는 함수
    public void DisableAllObjects()
    {
        for(int i = 0; i < pool.Length; i++)
        {
            if (pool[i].gameObject.activeSelf)
            {
                pool[i].gameObject.SetActive(false);
            }
        }
    }
}
```