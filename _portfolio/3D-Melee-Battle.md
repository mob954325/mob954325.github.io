---
title: 3D 근접 전투 게임
header:
    # 문서 해더 이미지
    # image: /assets/images/portfolio/
    # 페이지에 표시되는 외부 이미지
    teaser: /assets/images/portfolio/portfolio_banner_24_02.png
---

3D 근접 전투 게임 개인 프로젝트

[깃허브](https://github.com/mob954325/3DGame_1)

## 기간
개발기간 : 24.02.01 - 24.02.25

## 목표
이 프로젝트는 유니티에서의 3D 오브젝트 이해 및 1 대 1 근접 대전 구현을 목표로 한 프로젝트이다.

### 세부 목표
1. 상태 머신으로 적 상태별 행동 구현
2. 애니메이션 이벤트 함수를 이용한 공격 구현 및 소리 재생

### 플레이 Gif

전투  
![ezgif-7-6de74b5209](https://github.com/mob954325/3DGame_1/assets/87255621/8a8a7088-0238-4688-8feb-0e96470e302d)
  
### 주요 코드

적 상태마다 상속할 추상클래스
```
public abstract class EnemyStateBase : MonoBehaviour
{
    /// 적 메인 스크립트
    public EnemyBase enemy;

    /// 현재 State가 시작 될 때 실행되는 내용
    public abstract EnemyStateBase EnterCurrentState();

    /// 현재 State가 종료 될 때 실행되는 내용
    public abstract EnemyStateBase ExitCurrentState();

    /// 현재 State에서 실행되는 내용 (Update)
    public abstract EnemyStateBase RunCurrentState();
}
```

상태머신 구현  
```
public class EnemyStateMachine : MonoBehaviour
{
    /// State가 실행되었는지 확인하는 변수 (false : 실행 x , true : 실행됨)
    bool isEnter = false;

    /// 상태머신에서 죽었는지 확인하는 변수
    bool isDead = false; 

    /// 현재 상태 스크립트 ( Null이면 행동을 안함 )
    public EnemyStateBase currentState;

    void Start()
    {
        // init
        if(currentState != null) // currentState가 있으면 첫 실행문 실행
        {
            isEnter = true;
            currentState.enemy = GetComponent<EnemyBase>(); // state의 enemy 컴포넌트 받기
        }
    }

    void FixedUpdate()
    {
        RunStateMachine();
        OnDieState(); // 죽으면 사망 상태로 이동
    }


    /// 해당 State에서 실행할 내용 (상태를 변경할려면 이 함수에서 리턴해야함)
    private void RunStateMachine()
    {
        if (isEnter)
        {
            isEnter = false;
            currentState?.EnterCurrentState();
        }

        EnemyStateBase nextState = currentState?.RunCurrentState();

        if(nextState != null)
        {
            ChangeStateMachine(nextState);
        }
    }

    /// CurrentState를 nextState로 바꾸는 함수
    private void ChangeStateMachine(EnemyStateBase nextState)
    {
        if (currentState != nextState) // 매개변수와 현재 상태가 같지 않다 -> state 변환 전
        {
            isEnter = true;
            currentState?.ExitCurrentState();
        }

        currentState = nextState; // 다음 state로 변경
        currentState.enemy = GetComponent<EnemyBase>(); // state의 enemy 컴포넌트 받기
    }

    private void OnDieState()
    {
        if(currentState.enemy.IsDie && !isDead)
        {
            isEnter = true; // 진입여부 활성화
            isDead = true;

            // 상태 변경
            currentState = currentState.enemy.SetEnemyState(EnemyBase.State.Death);
            currentState.enemy = GetComponent<EnemyBase>(); // state의 enemy 컴포넌트 받기
        }
    }
}
```

상태 머신에 사용할 스크립트 중 하나인 공격 상태 코드  
```
public class AttackState : EnemyStateBase
{
    public bool isAttack = true;
    bool isBlock = false;

    public override EnemyStateBase EnterCurrentState()
    {
        isBlock = false;
        isAttack = true;
        enemy.speed = 0f;
        enemy.Anim.SetFloat(enemy.SpeedToHash, enemy.speed);// 애니메이션 파라미터 적용

        StopCoroutine(AttackCombo());
        StartCoroutine(AttackCombo()); // 공격 실행

        return this;
    }

    public override EnemyStateBase ExitCurrentState()
    {

        return this;
    }

    public override EnemyStateBase RunCurrentState()
    {
        // 플레이어가 패링을 했으면 피격 애니메이션 실행
        if (enemy.isAttackBlocked && !isBlock)
        {
            isBlock = true;
            OnPlayerParrying();

            if(enemy.Toughness == 0)
                return enemy.SetEnemyState(EnemyBase.State.Faint);
        }

        // 공격이 끝나면 chasing으로 돌아가기
        if (!isAttack)
            return enemy.SetEnemyState(EnemyBase.State.Chasing);

        return this;
    }

    IEnumerator AttackCombo()
    {
        enemy.Anim.SetTrigger(enemy.AttackToHash);
        int randAnimNum = UnityEngine.Random.Range(0, enemy.attackAnimNum); // 랜덤 공격  애니메이션 가져오기 (0 - attackAnimNum 미만 수)

        enemy.Anim.SetInteger(enemy.randomAttackToHash, randAnimNum);

        float animTime = enemy.GetAnimClipLength($"Attack{randAnimNum}");

        yield return new WaitForSeconds(animTime); // 2f / 24.02.25 - 애니메이션 재생시간에 따른 코루틴 대기시간 정하기
        isAttack = false;
    }

    void OnPlayerParrying()
    {
        enemy.Toughness -= 20;
        enemy.Anim.SetTrigger(enemy.DamagedToHash);
        enemy.changeWeaponCollider();
    }
}
```