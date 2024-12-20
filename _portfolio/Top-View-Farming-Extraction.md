---
title: 탑 뷰 파밍 익스트렉션
header:
    # 문서 해더 이미지
    # image: /assets/images/portfolio/
    # 페이지에 표시되는 외부 이미지
    teaser: /assets/images/portfolio/portfolio_banner_24_12.png
---

탑 뷰 탈출 파밍 게임

[깃허브](https://github.com/mob954325/TopView_Farming)

## 기간
개발기간 : 24.12.08 - 24.12.19 (12일)

## 목표
간단한 탑 뷰 파밍 탈출 게임 만들기

### 세부 목표
1. 탑 뷰 플레이 만들기
2. 간단한 인벤토리 시스템 구현 ( 아이템 추가, 제거 )
3. Scriptable object로 데이터 추가 ( 적, 아이템 )
4. 2개의 아이템을 조합하여 새로운 아이템 만들기

### 플레이 Gif

이동  
![PlayerMove](https://github.com/user-attachments/assets/e9f88687-788f-42a5-be46-00c3092ceb56)

적 교전  
![FightWithEnemy](https://github.com/user-attachments/assets/c99278b7-0c5d-4c28-8763-d1cfcecfca6b)

인벤토리  
![Farming_System](https://github.com/user-attachments/assets/7a51d857-ba8a-4916-9e66-7ab2d93161d1)

아이템 조합  
![ItemCombination](https://github.com/user-attachments/assets/a982449b-579a-4177-a370-20a4344ae8a3)

아이템 조합 실패  
![NoRecipe](https://github.com/user-attachments/assets/4ea6fd31-e46f-4fea-86ed-aac100743b36)

탈출  
![Extraction](https://github.com/user-attachments/assets/3685f70b-6bbe-4e33-8ce2-6e52706535d2)
  
### 주요 코드

아이템 조합 스크립트  

```
// 레시피 확인을 위해 아이템 코드 2개를 저장할 구조체
public struct Combination
{
    public Combination(ItemCode first, ItemCode second)
    {
        firstCode = first;
        SecondCode = second;
    }

    public ItemCode firstCode;
    public ItemCode SecondCode;
}
```

```
public class CombinationRecipes : MonoBehaviour
{
    // 레시피를 저장할 딕셔너리
    public Dictionary<Combination, ItemCode> recipe;

    private void Awake()
    {
        SetRecipeData();
    }

    // 레시피 데이터 추가 함수수
    private void SetRecipeData()
    {
        recipe = new Dictionary<Combination, ItemCode>();

        // key(두개의 아이템 코드), value(조합 완료된 아이템 코드)
        recipe.Add(new Combination(ItemCode.Damanged_Equipment, ItemCode.Scrap), ItemCode.Button);  // 파손된 장비 + 고철 = 버튼
        recipe.Add(new Combination(ItemCode.Scrap, ItemCode.RustyNail), ItemCode.Spike);            // 고철 + 녹슨 못    = 스파이크
    }

    /// <summary>
    /// 새로운 아이템으로 조합하는 함수 ( 레시피가 없으면 None코드로 반환)
    /// </summary>
    /// <param name="item1">아이템 코드 1</param>
    /// <param name="item2">아이템 코드 1이 아닌 아이템 코드</param>
    /// <returns>있으면 해당 레시피의 value값 없으면 None</returns>
    public ItemCode GetRecipeItem(ItemCode item1, ItemCode item2)
    {
        if (item1 == item2) return ItemCode.None;

        // 아이템 조합 찾기
        ItemCode result = ItemCode.None;
        Combination checkCombination = new Combination(item1, item2);

        foreach(KeyValuePair<Combination,ItemCode> combination in recipe)
        {
            if(CheckRecipe(checkCombination, combination.Key))
            {
                result = combination.Value;
                break;
            }
        }

        return result;
    }

    /// <summary>
    /// 조합법이 존재하는지 확인하는 함수
    /// </summary>
    /// <param name="checkValue">확인할 아이템 조합</param>
    /// <param name="recipe">비교할 아이템 조합</param>
    /// <returns>있으면 true 아니면 false</returns>
    private bool CheckRecipe(Combination checkValue, Combination recipe)
    {
        bool result = false;

        // a b or b a 가 동일하면 true
        if((checkValue.firstCode == recipe.firstCode && checkValue.SecondCode == recipe.SecondCode)     // a b
        || (checkValue.firstCode == recipe.SecondCode && checkValue.SecondCode == recipe.firstCode))    // b a
        {
            result = true;
        }

        return result;
    }
}
```