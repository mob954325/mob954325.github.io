---
title: 3D 액션 어트벤쳐 게임
header:
    # 문서 해더 이미지
    # image: /assets/images/portfolio/
    # 페이지에 표시되는 외부 이미지
    teaser: /assets/images/portfolio/portfolio_banner_24_02.png
---

아텐츠 게임 아카데미에서 진행한 팀 프로젝트

[깃허브](https://github.com/mob954325/3DRPG-TeamProject)

## 기간
개발기간 : 24.03.04 - 24.05.29

## 팀원 소개 및 역할
이성호 - 팀장 및 게임 시스템
오나현 - 플레이어 구현
김하린 - 플레이어의 스킬
박민우 - 라스트 보스
배태정 - 일반 몬스터 및 중형 몬스터
정윤서 - NPC 및 아이템 상호작용, 퀘스트, 상호작용 오브젝트

## 프로젝트 목표
1. 플레이어 캐릭터 요소 ( 움직임, 공격, 스태미너 )
2. 플레이어 스킬 ( 자석, 얼음기둥 생성, 폭탄, 시간 멈추기 )
3. 라스트 보스
4. 일반 몹, 중간몹
5. 인벤토리 시스템
6. 아이템의 장착 및 소비, 아이템 버리기 및 습득
7. 세이브 및 로드
8. 비동기 씬 로드

### 맡은 역할 목표
1. 프로젝트 관리 및 일정 조율
2. 인벤토리 시스템
3. 아이템 추가 및 아이템 드랍
4. 비동기 로딩
5. 세이브 및 로드

### 플레이 영상

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=FVopMLPX0kg" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  
### 주요 코드

인벤토리 (인벤토리 생성자, 아이템 추가, 제거 함수)
```
/// 인벤토리 생성자를 통해 인벤토리 생성
public Inventory(GameObject invenOwner, uint slotSize = 6)
{
    owner = invenOwner;
    maxSlot_size = slotSize;
    slots = new InventorySlot[maxSlot_size];
    tempSlot = new TempSlot(tempIndex);

    for (int i = 0; i < maxSlot_size; i++)
    {
        slots[i] = new InventorySlot((uint)i);
    }
}

/// <summary>
/// 아이템 추가 함수 , 가장 먼저있는 슬롯을 채움
/// </summary>
/// <param name="code">아이템 코드</param>
/// <param name="count">아이템 개수</param>
/// <param name="index">슬롯 인덱스</param>
public void AddSlotItem(uint code, int count = 1, uint index = 0)
{
    if (index >= maxSlot_size) // 슬롯 우뮤 확인
    {
        Debug.Log($"{index}번 슬롯은 존재하지 않습니다.");
        return;
    }

    if (index == 0) // index값이 default값이면 자동 추가
    {
        uint slotIndex = FindSlot(code);

        if(slotIndex >= maxSlot_size)
        {
            Debug.Log("인벤토리가 가득 차있습니다");
            return;
        }

        slots[slotIndex].AssignItem(code, count, out int overCount);

        if (overCount > 0) // 넘친 아이템이 존재한다면
        {
            // 재탐색 후 넣기
            slotIndex = FindSlot(code);
            slots[slotIndex].AssignItem(code, overCount, out _);
        }
    }
    else // 특정 인덱스에 추가
    {
        slots[index].AssignItem(code, count, out int overCount);

        if (overCount > 0) // 넘친 아이템이 존재한다면
        {
            //  재탐색 후 넣기
            uint slotIndex = FindSlot(code);

            if (slotIndex >= maxSlot_size)
            {
                Debug.Log("인벤토리가 가득 차있습니다");
                return;
            }
            else
            {
                slots[slotIndex].AssignItem(code, overCount, out _);
            }
        }
    }
}

/// <summary>
/// 슬롯에 아이템 제거
/// </summary>
/// <param name="count">제거할 슬롯 개수</param>
/// <param name="index">제거할 슬롯 위치 인덱스</param>
public void DiscardSlotItem(int count, uint index = 0)
{
    if(slots[index].SlotItemData == null)
    {
        Debug.Log($"슬롯이 비어있습니다.");
        return;
    }

    slots[index].DiscardItem(count);
}

```

비동기 로딩
```
public class AsyncLoadingScene : MonoBehaviour
{
    #region AsyncLoad Values

    /// 로딩 씬이 끝나고 불려진 다음 씬 이름
    public string nextSceneName = "LoadedSampleScene";

    /// 유니티 비동기 명령 처리 클래스
    AsyncOperation async;

    Slider loadingSlider;
    TextMeshProUGUI loadingDoneText;
    PlayerinputActions inputActions;

    IEnumerator loadingCoroutine;

    float loadRatio;
    public float loadingBarSpeed = 1.0f;

    /// 로딩 완료 여부 ( true : 완료, false : 미완 )
    bool loadingDone = false;
    #endregion

    #region Loading Image
    LoadingImageUI loadingImageUI;

    #endregion

    #region LifeCycle Method
    private void Awake()
    {
        inputActions = new PlayerinputActions();

        loadingImageUI = FindAnyObjectByType<LoadingImageUI>();
        loadingDoneText = FindAnyObjectByType<TextMeshProUGUI>();
        loadingDoneText.gameObject.SetActive(false);
    }

    private void OnEnable()
    {
        inputActions.UI.Enable();
        inputActions.UI.Click.performed += Press;
    }

    private void OnDisable()
    {
        inputActions.UI.Click.performed -= Press;
        inputActions.UI.Disable();
    }

    private void Start()
    {
        nextSceneName = GameManager.Instance.TargetSceneName;

        loadingSlider = FindAnyObjectByType<Slider>();
        loadingCoroutine = AsyncLoadScene();

        StartCoroutine(loadingCoroutine);
        loadingImageUI.ChangeLoadingImages();
    }

    private void Update()
    {
        // 슬라이더의 value가 loadRatio가 될 때까지 계속 증가
        if (loadingSlider.value < loadRatio)
        {
            loadingSlider.value += Time.deltaTime * loadingBarSpeed;
        }
    }

    #endregion

    #region AsyncLoad Method
    /// 클릭시 실행하는 함수
    private void Press(InputAction.CallbackContext context)
    {
        async.allowSceneActivation = loadingDone;
        loadingImageUI.FinishLoadingImage();

        GameManager.Instance.isLoading = false;
    }

    IEnumerator AsyncLoadScene()
    {
        loadRatio = 0.0f;
        loadingSlider.value = loadRatio;

        async = SceneManager.LoadSceneAsync(nextSceneName); // 비동기 로딩 시작
        async.allowSceneActivation = false;                 // 자동 씬 변환 비활성화

        while (loadRatio < 1.0f)
        {
            loadRatio = async.progress + 0.1f; // 진행률 갱신

            yield return null;
        }

        yield return new WaitForSeconds((1 - loadingSlider.value / loadingBarSpeed));

        loadingDoneText.gameObject.SetActive(true);
        loadingDone = true;
    }

    #endregion
}
```

세이브 로드
```

/// 플레이어 데이터를 저장하는 함수
/// <param name="saveIndex">세이브할 슬롯 인덱스</param>
protected virtual void SavePlayerData(int saveIndex)
{
    SaveData data = new SaveData(); // 저장용 클래스 인스턴스 생성
    // 저장용 객체에 데이터 저장
    // Scene 번호 저장
    SceneDatas[saveIndex] = SceneManager.GetActiveScene().buildIndex;
    data.SceneNumber = SceneDatas;

    // Player 정보 저장
    Player player = GameManager.Instance.Player;
    Vector3 curPos = player.gameObject.transform.position;
    Vector3 curRot = player.gameObject.transform.eulerAngles;
    Inventory curInven = player.Inventory;

    //data.playerInfos = new List<PlayerData>[DATA_SIZE]; // 저장할 데이터 초기화
    data.playerInfos = new PlayerData[DATA_SIZE]; // 저장할 데이터 초기화
    PlayerData playerData = new PlayerData(curPos, curRot, curInven); // 저장할 데이터값
    playerDatas[saveIndex] = playerData;    // 플레이어 데이터값 저장

    // 저장용 클래스 인스턴스에 현재 저장된 값 갱신
    for (int i = 0; i < DATA_SIZE; i++)
    {
        data.playerInfos[i] = playerDatas[i];
    }

    //data.playerInfos[saveIndex].Insert(saveIndex, playerDatas[saveIndex]); // SaveData 클래스에 저장

    // save Data file
    string jsonText = JsonUtility.ToJson(data, true); // json 형식 문자열로 변경
    string path = $"{Application.dataPath}/Save/";
    if (!System.IO.Directory.Exists(path))
    {
        // path 폴더가 없다
        System.IO.Directory.CreateDirectory(path); // 폴더 생성
    }

    string fullPath = $"{path}Save.json";               // 전제 경로 만들기
    System.IO.File.WriteAllText(fullPath, jsonText);    // 파일로 저장

    RefreshSaveData();
    Debug.Log("Player Data convert complete");
}

/// 플레이어 데이터 로드
/// <param name="loadIndex">로드할 파일 번호</param>
/// <returns>로드에 성공했으면 true 아니면 false</returns>
protected virtual void LoadPlayerData(int loadIndex)
{
    // 저장한 데이터 불러오기   
    Player player = GameManager.Instance.Player;
    GameManager.Instance.spawnPoint = playerDatas[loadIndex].position; // 플레이어 위치잡기
    player.transform.rotation = Quaternion.Euler(playerDatas[loadIndex].rotation);
    player.Inventory.InventoryClear();

    Inventory inventory = player.Inventory; // 저장할 플레이어 인벤토리 불러오기
    for (int i = 0; i < inventory.SlotSize; i++)
    {
        if (playerDatas[loadIndex].itemDataClass[i].count == 0) // 아이템 개수가 없으면 무시
        {
            continue;
        }
        else // 아이템이 존재하면 아이템 추가
        {
            uint itemCode = (uint)playerDatas[loadIndex].itemDataClass[i].itemCode; // 아이템 코드
            int itemCount = playerDatas[loadIndex].itemDataClass[i].count;          // 아이템 개수

            player.Inventory.AddSlotItem(itemCode, itemCount, (uint)i);
            player.Inventory.SetCoin(playerDatas[loadIndex].gold);
            player.MaxHP = playerDatas[loadIndex].playerHP;

            for(int j = 0; j < System.Enum.GetValues(typeof(QuestType)).Length; j++)
            {
                QuestManager.Instance.checkClearQuests[j] = playerDatas[loadIndex].questClear[j];

            }
        }
    }

    // 씬 불러오기
    string sceneName = System.IO.Path.GetFileNameWithoutExtension(UnityEngine.SceneManagement.SceneUtility.GetScenePathByBuildIndex(SceneDatas[loadIndex])); // 저장한 씬 인덱스로 씬 저장
    GameManager.Instance.ChangeToTargetScene(sceneName, player.gameObject);
}

```